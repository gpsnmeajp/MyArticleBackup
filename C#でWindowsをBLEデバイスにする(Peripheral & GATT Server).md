# はじめに
Bluetooth LEの詳しい話はしません

# 環境構築

### まず大前提として、デバイスマネージャからBluetooth LE対応環境かを確認してください。
(Bluetooth LE Enumeratorがない場合は対応してない環境です。)
<img src=https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/89d01698-57c1-ca91-9d5a-127f4f1f6dc1.png width=30%></img>

対応環境ではない場合は、Bluetooth 4またはBluetooth 5対応と謳っているUSBドングルを購入して、PCに差し込んでください(Amazonで1000円程度です)。
私はこの辺を使っています。

https://www.amazon.co.jp/dp/B098K3H92Z

### Visual Studio 2019 Communityをインストールしてください。
(ProfessionalとかでもOKですが、2017 Expressは使用できません)

https://visualstudio.microsoft.com/ja/downloads/

### Windows SDKをインストールしてください。
Visual Studioをセットアップ中ならこの画面
そうでなければ、コントロールパネル→プログラムと機能→Visual Studio→変更　から、Windows SDKをインストールしてください。
新し目のものにチェックが入っていればだいたい動くはずです。
<img src=https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/0ace9bc3-3501-43a1-df43-5452cf3c0242.png width=50%></img>

### .Net Framework 4.7 コンソールアプリケーションのプロジェクトを作成
<img src=https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/3d709f2c-2094-60ab-265b-b9974f9b6a0c.png width=100%></img>

<img src=https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/3b600214-240a-a7db-5d07-734a8d8e4b26.png width=100%></img>


### Microsoft.Windows.SDK.ContractsをNuGetでインストール
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/93b50c9b-89cb-7d17-9c4a-9af8761556ee.png)

```
install-package Microsoft.Windows.SDK.Contracts
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/7507b638-b09b-ddd7-c47e-0c4de4d9fc00.png)


### ソリューションエクスプローラーから、参照を右クリックして、「packages.config を PackageReference に移行する」を実行
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/4b423d9f-6064-1def-abb5-7013c92c2291.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/4cd5aef5-cad0-fab3-9a6e-4a94860ee39e.png)

これでC#からBluetooth LEが扱えるようになります。

# ソース
ソース中のコメントを参照してください。
LightBlueなどのスマホアプリで見ると、アクセスできると思います。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/f9df4961-4504-d827-7b72-031104620e5c.png)


```cs
using System;
using System.Threading.Tasks;
using System.IO;
using System.Runtime.InteropServices.WindowsRuntime;
using Windows.Devices.Bluetooth;
using Windows.Devices.Bluetooth.GenericAttributeProfile;

//MIT-0

namespace gatt_server
{
    class Program
    {
        static byte cnt = 0;
        static void Main(string[] args)
        {
            //Async-awaitを使えるようにTask内で実行する
            Task.Run(AsyncMain).Wait();
        }

        static async Task AsyncMain() {
            //GattServiceProviderを指定のUUIDで初期化する
            //システムに独自のBLE GATTサービスを追加する。
            //失敗する場合、たいていBluetooth LE対応の環境ではない。(デスクトップPC、古いPC、Bluetooth 3以前のドングルを使用中など)
            //または、システムにより禁止された予約済みUUIDを使用している。

            var gattServiceProviderResult = await GattServiceProvider.CreateAsync(new Guid("00000000-0000-4000-A000-000000000000"));
            if (gattServiceProviderResult.Error != BluetoothError.Success) {
                Console.WriteLine("GATT Serviceの起動に失敗(Bluetooth LE対応デバイスがない?)");
                return;
            }

            var gattServiceProvider = gattServiceProviderResult.ServiceProvider;

            //---

            //ローカルキャラクタリスティック(外部から読み書き可能な値)を定義する
            var cReadWriteParam = new GattLocalCharacteristicParameters
            {
                CharacteristicProperties = GattCharacteristicProperties.Read | GattCharacteristicProperties.Write | GattCharacteristicProperties.Notify, //読み込み & 書き込み & 通知購読可能
                ReadProtectionLevel = GattProtectionLevel.Plain, //誰でも読み込み可能
                WriteProtectionLevel = GattProtectionLevel.Plain, //誰でも書き込み可能
                UserDescription = "cReadWrite" //ユーザーに見える説明(BLEツールを使って読むことができる)
            };

            //定義した情報をもとに、指定のUUIDでサービスに登録する
            var cReadWrite = await gattServiceProvider.Service.CreateCharacteristicAsync(new Guid("00000000-0000-4000-A000-000000000001"), cReadWriteParam);

            //読み込みが発生したときのコールバック定義
            cReadWrite.Characteristic.ReadRequested += async (GattLocalCharacteristic sender, GattReadRequestedEventArgs args) =>
            {
                //接続中のデバイスから読み込まれた
                Console.WriteLine("Read request from " + args.Session.DeviceId.Id);
                var deferral = args.GetDeferral(); //非同期処理完了を知らせるためのDeferral (awaitを使うため)

                var request = await args.GetRequestAsync(); //リクエストを取得

                byte[] buf = new byte[1] { cnt };  //返却値を準備(Streamでもいいが、単純のためにbyte[]を使用)
                request.RespondWithValue(buf.AsBuffer()); //返却

                deferral.Complete(); //非同期完了を通知
            };

            //書き込みが発生したときのコールバック定義
            cReadWrite.Characteristic.WriteRequested += async (GattLocalCharacteristic sender, GattWriteRequestedEventArgs args) =>
            {
                //接続中のデバイスから書き込まれた
                Console.WriteLine("Write request from " + args.Session.DeviceId.Id);
                var deferral = args.GetDeferral(); //非同期処理完了を知らせるためのDeferral (awaitを使うため)

                var request = await args.GetRequestAsync(); //リクエストを取得

                var stream = request.Value.AsStream(); //streamを取得

                //1byteずつ読み込んで表示
                int d = 0;
                while ((d = stream.ReadByte()) != -1)
                {
                    cnt = (byte)d;
                    Console.Write(d.ToString("X"));
                    Console.Write(",");
                }
                Console.WriteLine();

                if (request.Option == GattWriteOption.WriteWithResponse)
                {
                    request.Respond(); //送信側が応答欲しい場合は応答を返す(これをしないと送信側がエラーになる)
                    //System.Exception: 要求された属性要求で思いもよらないエラーが発生したため、要求されたとおりに完了することができませんでした。 (HRESULT からの例外:0x8065000E) の原因になる
                }

                deferral.Complete(); //非同期完了を通知
            };

            //購読者の増減が発生したときのコールバック定義
            cReadWrite.Characteristic.SubscribedClientsChanged += async (GattLocalCharacteristic sender, object args) =>
            {
                //購読者が増えた/減った
                Console.WriteLine("Subscribe Changed(Notify)");
                foreach (var c in sender.SubscribedClients)
                {
                    Console.WriteLine("- Device: " + c.Session.DeviceId.Id);
                }
                Console.WriteLine("- DeviceEnd");
            };

            //サービスをアドバタイジングするパラメータ
            var gattServiceProviderAdvertisingParameters = new GattServiceProviderAdvertisingParameters
            {
                IsConnectable = true, //接続可能
                IsDiscoverable = true //検出可能
            };

            //アドバタイジング開始
            gattServiceProvider.StartAdvertising(gattServiceProviderAdvertisingParameters);

            Console.WriteLine("StartAdvertising...");

            //1秒おきにカウントアップ値をNotifyする
            
            while (true) {

                byte[] bufN = new byte[1] { cnt };
                await cReadWrite.Characteristic.NotifyValueAsync(bufN.AsBuffer());
                Console.WriteLine("Notify " + cnt.ToString("X"));

                await Task.Delay(1000);

                cnt++;
            }

        }

    }
}
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/3260eebe-8325-e244-29d0-481e11c84726.png)


# 参考文献
Bluetooth GATT サーバー
https://docs.microsoft.com/ja-jp/windows/uwp/devices-sensors/gatt-server
