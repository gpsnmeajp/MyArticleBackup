# はじめに
こんなときはありませんか？

+ メカとかロボットとかなんか作ったはいいけど、画面作るのがめんどくさい
+ タッチとか音声とか通信とかスマホを使ったほうが早いけど、周辺機器を繋ぎたい
+ スマホとUSBで繋ぎたいが、スマホの電力を使うんじゃなくてむしろ充電したい
+ 古いAndroidスマホと自分の電子機器を繋ぎたい

そういうときに使える仕組みとして、

+ 有線(USB)でAndroidデバイスと接続
+ 電源供給は機器側からスマホに行う
+ 非常にシンプルに通信できる

そういう仕組をAndroid OSがサポートしています。
これがAndroid Open Accessoryです。

# Android Open Accessoryとは
Android Open Accessoryってご存知でしょうか？
機能としては上述のものを実現するためのものなのですが、なんと初出は2011年のGoogle IO。
Android OSのサポートバージョンはなんとAndroid 2.3.4からです。

10年前のあまり使われていなさそうなプロトコルが使い物になるのか？と思うかもしれませんが、Android Autoでも使われているらしく廃止される予定がなさそうです。
実際、Android 9で問題なく動作するのを確認しています。

かつては、Android Accessory Development Kitという公式のサンプルデバイスがありましたが、今は見つかりません。

# サンプルについて
サンプルは以下にあります。ライセンスはCC0です。
PCやRaspberry Piを周辺機器扱いして接続できるよう、CとPythonのサンプルを入れています。
また、Android側での動作確認用にサンプルのKotlinアプリの基本コードも入れています。

https://github.com/gpsnmeajp/SegsAoASampleProject

# Android Open Accessoryの概要
内容は基本的に以下に準じています。
https://source.android.com/devices/accessories/aoa?hl=ja
https://developer.android.com/guide/topics/connectivity/usb/accessory?hl=ja

Android Open Accessoryには、主にAndroidデバイス側の視点と、接続する周辺機器の視点があります。

## Androidデバイス側について
https://developer.android.com/guide/topics/connectivity/usb/accessory?hl=ja

Androidデバイス側からは、アプリからUSBアクセサリAPIを使ってアクセスすることができます。JavaあるいはKotlinが使用できます。

簡単なコードを書くと以下の感じです。

```kotlin
//注意: このコードは動作確認していません
        //アクセサリを探す
        val usbManager = getSystemService(Context.USB_SERVICE) as UsbManager
        val accessory = usbManager.accessoryList?.firstOrNull()
        //アクセサリを開いてファイルディスクリプタを得る
        val parcelFileDescriptor = usbManager.openAccessory(accessory)

        //無事開けていればストリームを取得
        parcelFileDescriptor?.fileDescriptor?.also { fd ->
            inputStream = FileInputStream(fd)
            outputStream = FileOutputStream(fd)
        }
        
        //データの読み書き
        outputStream?.write(d)
        inputStream?.read(byteArray)
```

とても簡単ですね！
ここでread/writeは、USBでの単純なbulk in/outになります。

Android Manifestとaccessory_filterを記述すると、特定の属性を持ったデバイスが接続されたときにアプリを自動で立ち上げてくれるようにもなります。

でもこれだけでは多くの場合実用になりません。
ランタイムパーミッションがUSB接続ごとに必要だったり、通信切断時の処理などです。

また、streamをread/writeする際にブロックするという問題もあります。
これはavailableが使用できないためより深刻な問題になります。

結局、サンプルではThreadでRead/Writeし、「周辺機器側では常に(ある程度の間隔で)データを送受信してくれる」という仮定を置くことにしました。

https://github.com/gpsnmeajp/SegsAoASampleProject/blob/main/android/kotlin/MainActivity.kt

詳しい問題は、以下のIssue Trackerを参照ください。

https://issuetracker.google.com/issues/36933798

## 周辺機器側について
https://source.android.com/devices/accessories/aoa?hl=ja

周辺機器側は、AOAプロトコルに基づいて実装する。AOAプロトコルはv1とv2がありますが、
v2はv1を拡張して音声とHIDに対応させたものであり、単純なデータ通信に使うのであればv1で十分です。

ではAOA v1とはどんなプロトコルなのか。基本的には以下2つだけです。

1. Android側がアクセサリーモードじゃないとき(初回接続)、周辺機器の詳細(名称やバージョン、URI)を伝えて、アクセサリーモードに切り替えさせる
(この時の情報がランタイムパーミッションの情報や、アプリの起動に使用されます)

2. Android側がアクセサリーモードのとき(接続中)、Bulk in/outで通信する(アプリのread/writeに直結)

2は、単なるbluk in/outなので、キーポイントは1になります。

Androidデバイスがアクセサリーモードかどうかを判別するには、ベンダーIDとプロダクトIDを確認します。
プロダクトID=0x18D1で、ベンダーID=0x2D00(通常) または 0x2D01(開発者モード)であれば、アクセサリーモードです。

そうでない場合は、以下のコントロールリクエストを実施します。

+ つながっているデバイスをとりあえず(ID無視して)開く
+ AOAバージョンをチェックして1以上か確認する(51)
+ 周辺機器情報を送信する(52)
+ アクセサリ切り替えリクエストを送信する(53)

成功すれば、デバイスが再接続され、プロダクトID/ベンダーIDが切り替わります。

```cpp
    printf("+ EnterAccessoryMode()\n");
    bool status = false;
    //通常のデバイスとして開く。
    libusb_device_handle *handle = libusb_open_device_with_vid_pid(ctx, venderid, productid);
    printf("libusb_open_device_with_vid_pid(ctx, %04X, %04X) = %p\n", venderid, productid, handle);
    if (handle == nullptr)
    {
        printf("Fail !\n");
        return status;
    }

    do
    {
        //使用権要求
        int claim = libusb_claim_interface(handle, 0);
        printf("libusb_claim_interface = %d\n", claim);
        if (claim != 0)
        {
            //失敗
            printf("Fail !\n");
            break;
        }

        //アクセサリプロトコル対応チェック
        unsigned char data[2] = {0};
        int result = libusb_control_transfer(handle, 0xC0, 51, 0, 0, data, 2, 1000);
        int version = ((int)data[1] << 8 | (int)data[0]);
        printf("libusb_control_transfer(version check) = %d\n", result);
        printf("Version = %d\n", version);

        if (version > 0)
        {
            //バージョンチェック成功
            printf("Version check ok\n");

            //デバイス情報を送信する
            unsigned char manufacturer_name[] = "THIS IS MANUFACTURER NAME";
            unsigned char model_name[] = "THIS IS MODEL NAME";
            unsigned char description[] = "THIS IS DESCRIPTION";
            unsigned char version[] = "THIS IS VERSION";
            unsigned char URI[] = "http://example.com/";
            unsigned char serial_number[] = "THIS IS SERIAL NUMBER";

            libusb_control_transfer(handle, 0x40, 52, 0, 0, manufacturer_name, strlen((char *)manufacturer_name) + 1, 1000);
            libusb_control_transfer(handle, 0x40, 52, 0, 1, model_name, strlen((char *)model_name) + 1, 1000);
            libusb_control_transfer(handle, 0x40, 52, 0, 2, description, strlen((char *)description) + 1, 1000);
            libusb_control_transfer(handle, 0x40, 52, 0, 3, version, strlen((char *)version) + 1, 1000);
            libusb_control_transfer(handle, 0x40, 52, 0, 4, URI, strlen((char *)URI) + 1, 1000);
            libusb_control_transfer(handle, 0x40, 52, 0, 5, serial_number, strlen((char *)serial_number) + 1, 1000);
            printf("Accessory info send\n");

            //アクセサリ切り替えリクエストを送信する
            libusb_control_transfer(handle, 0x40, 53, 0, 0, nullptr, 0, 1000);
            printf("Accessory on request\n");

            //成功
            status = true;
        }
        //通信切る
        libusb_release_interface(handle, 0);
        printf("libusb_release_interface\n");
    } while (false);

    libusb_close(handle);
    printf("libusb_close\n");
    return status;

```

C++サンプル(libusb)

https://github.com/gpsnmeajp/SegsAoASampleProject/blob/main/pc/cpp/main.cc

Pythonサンプル(pyusb)

https://github.com/gpsnmeajp/SegsAoASampleProject/blob/main/pc/py/program.py

# 結局今でも使う価値あるの？
Bluetoothで通信できてたり、USBホスト+USB-PDとかが満足に使えている場合はそっちのほうが安定しているかもしれません。
ただ、幅広いAndroid端末で使える(=中古で適当に買ってきて部品として組み込んだりできる)とか、電源供給が一応できるなどの利点もあるので、検討の余地はあると思います。

良い電子工作ライフを！

# 参考文献
http://bfin.sakura.ne.jp/?p=501
http://www.picfun.com/android/android01.html
https://developer.android.com/guide/topics/connectivity/usb/accessory?hl=ja
https://zenn.dev/k16/articles/e72ff8bf2a640e
https://libusb.sourceforge.io/api-1.0/group__libusb__asyncio.html#gabb0932601f2c7dad2fee4b27962848ce
https://libusb.sourceforge.io/api-1.0/group__libusb__hotplug.html#ga5ab3955e2110a3099497a66256fb7fab
https://www.slideshare.net/masatakakono1/usb-87470849
https://qiita.com/gpsnmeajp/items/b1282b2d3c14470bbae7
https://developer.android.com/guide/topics/connectivity/usb/accessory?hl=ja
https://poly.hatenablog.com/entry/20110523/p1
http://y-anz-m.blogspot.com/2011/12/androidhello-adk.html
https://qiita.com/kurun_pan/items/f626e763e74e82a44493
https://issuetracker.google.com/issues/36933798
