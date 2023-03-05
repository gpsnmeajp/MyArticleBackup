#参考文献
WindowsデスクトップアプリでBLEのGATTで体温計と血圧計と通信する
https://qiita.com/gebo/items/41da7474936845d77d06

Windows10でBLEデバイスとGATTで通信するメモ
https://qiita.com/Dr10_TakeHiro/items/7446d68cbffeae7c7184

iBeaconをスキャンするWindowsデスクトップアプリ
https://qiita.com/gebo/items/469dd49ddd1e24ce7a42

できる！C#で非同期処理(Taskとasync-await)
https://www.kekyo.net/2016/12/06/6186

C#のコンソールアプリケーションで非同期処理をするときのメモ
https://qiita.com/gpsnmeajp/items/ef21ba4d988a76922bab

#メモ
1.Windows 10対応、Bluetooth 4.0対応のBluetoothアダプタを刺しておく。(Windows標準ドライバで良い)
　なお、Winodws10のスキャン画面でアドバタイズメントの受信チェックができる
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/e3dc62d1-8bc2-b191-d7e8-2cb8f40afab6.png)

2.Visual Stduio 2015 Communityをインストールする。
　この際、ここにチェックを入れる
　![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/eb9e530d-df7e-4dfa-ef7f-dd38c8033eb6.png)

3.C#コンソールアプリケーションプロジェクトを作成する
4.NuGetからUwpDesktopをインストールする
``` Install-Package UwpDesktop```

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/9613ad25-b271-cb01-1b1d-7e3678056419.png)
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/ce459db4-a71b-3c96-cc69-48bedd8d2e08.png)

5.以下のコードを実行する

#受信したアドバタイズメントパケットを片っ端から表示する
こんな感じで検出できる。(なお、Windows標準の検索では出てくるのに、ここでは出てこない機器も居たりする)
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/f379632c-e864-7a9d-b74b-fb11702fb0d4.png)

```C#
using System;
using System.IO;
using System.Threading;
using System.Threading.Tasks;
using Windows.Devices.Bluetooth;
using Windows.Devices.Bluetooth.Background;
using Windows.Devices.Bluetooth.Rfcomm;
using Windows.Devices.Bluetooth.Advertisement;
using Windows.Devices.Bluetooth.GenericAttributeProfile;

namespace ConsoleApplication2
{
    class Program
    {
        static BluetoothLEAdvertisementWatcher watcher;

        static void Main(string[] args)
        {
            Console.WriteLine("Start");
            watcher = new BluetoothLEAdvertisementWatcher();
            watcher.Received += Watcher_Received;
            watcher.ScanningMode = BluetoothLEScanningMode.Passive;
            watcher.Start();
            Thread.Sleep(60000);
            watcher.Stop();
            Console.WriteLine("Stop");
        }

        private static void Watcher_Received(BluetoothLEAdvertisementWatcher sender, BluetoothLEAdvertisementReceivedEventArgs args)
        {
            Console.WriteLine("---Received---");
            var bleServiceUUIDs = args.Advertisement.ServiceUuids;

            Console.WriteLine("Found");
            Console.WriteLine("MAC:" + args.BluetoothAddress.ToString());
            Console.WriteLine("NAME:" + args.Advertisement.LocalName.ToString());
            Console.WriteLine("ServiceUuid");
            foreach (var uuidone in bleServiceUUIDs)
            {
                Console.WriteLine(uuidone.ToString());
            }
            Console.WriteLine("---END---");
            Console.WriteLine("");
        }
    }
}

```

#一定時間スキャンした上で見つかったものを重複なく表示する

```C#
using System;
using System.IO;
using System.Threading;
using System.Threading.Tasks;
using Windows.Devices.Bluetooth;
using Windows.Devices.Bluetooth.Background;
using Windows.Devices.Bluetooth.Rfcomm;
using Windows.Devices.Bluetooth.Advertisement;
using Windows.Devices.Bluetooth.GenericAttributeProfile;
using System.Collections.Generic;

namespace ConsoleApplication2
{
    class Program
    {
        static BluetoothLEAdvertisementWatcher watcher;
        static Dictionary<ulong, string> dict = new Dictionary<ulong, string>();

        static void Main(string[] args)
        {
            Console.WriteLine("Start");
            watcher = new BluetoothLEAdvertisementWatcher();
            watcher.Received += Watcher_Received;
            watcher.ScanningMode = BluetoothLEScanningMode.Passive;
            watcher.Start();
            Thread.Sleep(10000);
            watcher.Stop();
            Console.WriteLine("\nStop");

            Console.WriteLine("Found");
            foreach (var d in dict)
            {
                Console.WriteLine("MAC:" + d.Key.ToString() +" NAME:"+ d.Value);
            }
        }

        private static void Watcher_Received(BluetoothLEAdvertisementWatcher sender, BluetoothLEAdvertisementReceivedEventArgs args)
        {
            Console.Write("!");
            var bleServiceUUIDs = args.Advertisement.ServiceUuids;
            dict[args.BluetoothAddress] = args.Advertisement.LocalName; //こうでないと重複例外が出る
        }
    }
}

```



