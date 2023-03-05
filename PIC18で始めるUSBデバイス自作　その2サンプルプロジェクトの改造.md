#はじめに
[前回](https://qiita.com/gpsnmeajp/items/eedb3ed788add25df11c)に引き続き、PIC18FでUSB機器を作ってみます。

#注意
[前回](https://qiita.com/gpsnmeajp/items/eedb3ed788add25df11c)を参照

#今回やること
+ vendor_basicデモの動作を確認します
+ vendor_basicデモから必要なファイルを取り出し改造した自分用プロジェクトを作成します
+ 自分用アプリケーションを作成します

#vendor_basicデモの動作の確認
##デモプロジェクトを開く
前回と同様に、デモプロジェクトのフォルダを開きます。

今回はvendor_basicデモです。
ベンダークラスデバイスとして、バルク転送を行います。

```
C:\microchip\mla\v2017_03_06\apps\usb\device\vendor_basic\firmware
```

そして、以下のプロジェクトを読み込む。

```
low_pin_count_usb_development_kit_pic18f14k50.x
```

##書き込み
[前回](https://qiita.com/gpsnmeajp/items/eedb3ed788add25df11c#%E6%9B%B8%E3%81%8D%E8%BE%BC%E3%81%BF%E3%83%84%E3%83%BC%E3%83%AB%E3%81%AE%E5%88%87%E3%82%8A%E6%9B%BF%E3%81%88)同様に、書き込みツールを切り替え、電源供給設定をして書き込んでください。

##動作確認
PCに接続すると、ドライバは自動認識され、WinUSBデバイスとして動作を始めます。
通信には専用ツールが必要です。

※Chromeの最新版をお使いの場合は、以下のページのWebUSBでも動作確認ができます．
　https://sabowl.sakura.ne.jp/gpsnmeajp/webusb/webusb.htm

MLAには動作確認用のツールも複数同梱されていますが、今回はこのツールを使用します。

```
C:\microchip\mla\v2017_03_06\apps\usb\device\vendor_basic\utilities\plug_and_play_example\bin\plug_and_play_example.exe
```

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/0e27202a-17e6-735c-8be6-107e6463af56.png)

未接続・異なるファームウェアを書き込んだ状態では、以下の画面が出ます。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/d6f1cddb-b7c9-a75e-e3a0-5678e3dc79de.png)

書き込み済みの場合は以下のような画面になります。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/d4e66cb4-6f94-f6ac-a5bf-3b62baeb8556.png)

例によって、公式サンプルボードではないため、ボタン入力は指を近づけたりすることでバタバタ暴れますが、
正常な挙動です。


#改造プロジェクトの作成
公式サンプルで遊ぶだけでは何も楽しくないので、実際に自分用のプロジェクトを作成しましょう。

vendor_basicデモは、以下のような構成になっていますが、さまざまなボードに対応するため冗長な構成になっており、
実際に必要なファイルはこれほど多くはありません。

ので、必要なファイルを抜き出して使いましょう。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/5c4c2ac0-dd2d-31ca-1489-60787a0a07a1.png)

##プロジェクトの新規作成
MPLABXでプロジェクトを新規作成します。

File→New Project
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/f38307b5-b2e2-992a-76d4-843f76254da0.png)

Microchip EmbeddedのStandalone Projectを選択し、Next
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/1bc87a20-d0e2-efe0-9fb3-d5aaad0a1334.png)

FamilyをAdvanced 8-bit MCUs (PIC18)にし、
DeviceをPIC 18F14K50に設定し、Next
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/cc673ea9-f659-36ba-a2c5-b6e0c5059f9b.png)

NoneのままNext
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/af6bd8f4-f162-87ee-5b76-8de94c3673f1.png)

PICkit3を選択しNext
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/939a977b-18b9-aa94-7cea-8a9aa710fd48.png)

XC8を選択してからNext
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/e20def3a-039b-0a62-fe91-0998558f96c3.png)

任意のプロジェクト名(ここでは「USBVENDER_MYPROJECT」)と
任意のプロジェクト保存先(ここではC:\mplabx_projects)
を入力し、ついでにEncodingはUTF-8にして、Next
(これをしないと日本語のコメントが化けたはず)

![image.png](https://qiita-image-sto.s3.amazonaws.com/0/191114/d884a4c7-1192-1b95-6150-cdd9d895cf89.png)

これでプロジェクトの作成が完了する。
ちなみに、今まで開いたプロジェクトを閉じるには、右クリックしてCloseなので注意。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/ec9af757-1a92-c0ac-770b-eb8467d66920.png)

##プロジェクトの設定
右クリックし、プロジェクトの設定を開く。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/84a526b4-d432-632b-bedc-8652fdbc4fac.png)

電源供給の設定は今まで行ったと同じなので省略。

XC8　global optionsのXC8 compilerを開き、
Include directoriesの「...」をクリック。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/9d53d8e8-e964-43f3-faee-f44f8481e384.png)

「Enter or 'Browse' string here」と書かれたところをダブルクリックし、「.」と入力しEnter。
これをしないと.hファイルが認識されない。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/2e5dd723-8a82-1e30-7da9-c9afcc8ef311.png)

OKをクリックし、設定を閉じる。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/fde43292-1d52-649c-e981-e27ab7e8d28a.png)

##ファイルのコピー
プロジェクトフォルダ(ここではC:\mplabx_projects\USBVENDER_MYPROJECT.X)を開き、USBベンダークラスを利用するのに必要なファイルをコピーする。
###ユーザーファイルのコピー
```
C:\microchip\mla\v2017_03_06\apps\usb\device\vendor_basic\firmware\low_pin_count_usb_development_kit_pic18f14k50.x
```
より

+ fixed_address_memory.h

```
C:\microchip\mla\v2017_03_06\apps\usb\device\vendor_basic\firmware\demo_src
```
より

+ main.c
+ usb_config.h
+ usb_descriptors.c
+ usb_events.c

をプロジェクトのフォルダにコピーします。
主に書き換えるのは以上のファイルになります。

###フレームワークファイルのコピー
```
C:\microchip\mla\v2017_03_06\framework\usb\inc
```
より

+ usb.h
+ usb_ch9.h
+ usb_common.h
+ usb_device.h
+ usb_device_generic.h
+ usb_hal.h
+ usb_hal_pic18.h

```
C:\microchip\mla\v2017_03_06\framework\usb\src
```
より

+ usb_device.c
+ usb_device_generic.c
+ usb_device_local.h

をプロジェクトのフォルダにコピーします。

これらのファイルは基本的に触りません。


##ファイルの書き換え
ファイルを書き換えていきます。

###fixed_address_memory.h
中身を消去し、以下に置き換え。

```cpp:fixed_address_memory.h
#ifndef FIXED_MEMORY_ADDRESS_H
#define FIXED_MEMORY_ADDRESS_H

#define FIXED_ADDRESS_MEMORY

#define IN_DATA_BUFFER_ADDRESS           @0x240
#define OUT_DATA_BUFFER_ADDRESS          @0x280
#define CONTROL_BUFFER_ADDRESS_TAG       @0x2C0

#endif //FIXED_MEMORY_ADDRESS
```

###main.c
中身を消去し、以下に置き換え。
本来app_device_vendor_basic.cである中身をmain関数に書き直しています。
app_led_usb_status.cの機能は削除。

```cpp:main.c
#include "main.h"

#include "usb.h"
#include "usb_device.h"
#include "usb_device_generic.h"

//----
unsigned char  INPacket[USBGEN_EP_SIZE]  IN_DATA_BUFFER_ADDRESS = {0};
unsigned char OUTPacket[USBGEN_EP_SIZE] OUT_DATA_BUFFER_ADDRESS = {0};

static USB_HANDLE USBGenericOutHandle = 0;
static USB_HANDLE USBGenericInHandle = 0;

//USB Communication has started
void user_EVENT_CONFIGURED()
{
    USBGenericOutHandle = 0; //NULL
    USBGenericInHandle = 0; //NULL

    USBEnableEndpoint(USBGEN_EP_NUM,USB_OUT_ENABLED|USB_IN_ENABLED|USB_HANDSHAKE_ENABLED|USB_DISALLOW_SETUP);
    USBGenericOutHandle = USBGenRead(USBGEN_EP_NUM,(uint8_t*)&OUTPacket,USBGEN_EP_SIZE); //receive start flag set
}

void user_EVENT_RESUME(){}//USB Communication Resumed
void  user_EVENT_SUSPEND(){}//USB Communication Suspended
void user_EVENT_SOF(){}//USB Communication Frame received
void user_EVENT_EP0_REQUEST(){}//USB Control Connection received
void user_init(){}//on Boot

void user_loop()
{
    if(!USBHandleBusy(USBGenericOutHandle))
    {
        switch(OUTPacket[0])
        {
            case 0x80:
                break;
            case 0x81:
                if(!USBHandleBusy(USBGenericInHandle))
                {
                    INPacket[0] = 0x81;
                    if(false){
                        INPacket[1] = 0x01;
                    }else{
                        INPacket[1] = 0x00;
                    }
                    USBGenericInHandle = USBGenWrite(USBGEN_EP_NUM,(uint8_t*)&INPacket,USBGEN_EP_SIZE);
                }
                break;
        }
        USBGenericOutHandle = USBGenRead(USBGEN_EP_NUM,(uint8_t*)&OUTPacket,USBGEN_EP_SIZE);
    }

}

void interrupt Interrupt(void)
{
    #if defined(USB_INTERRUPT)
        USBDeviceTasks();
    #endif
}

void main(void)
{
    user_init();

    USBDeviceInit();
    USBDeviceAttach();
    
    while(1)
    {
        //defined in "usb_config.h"
        #if defined(USB_POLLING)
            USBDeviceTasks();
        #endif

        //not initialized
        if( USBGetDeviceState() < CONFIGURED_STATE )
        {
            continue;
        }

        //device Suspended
        if( USBIsDeviceSuspended()== true )
        {
            continue;
        }
        user_loop();
    }
}

```


###usb_events.c
以下に置き換え。

```cpp:usb_events.c
#include "main.h"

#include "usb.h"
#include "usb_device.h"
#include "usb_device_generic.h"

bool USER_USB_CALLBACK_EVENT_HANDLER(USB_EVENT event, void *pdata, uint16_t size)
{
    switch((int)event)
    {
        case EVENT_TRANSFER:
            break;

        case EVENT_SOF:
            user_EVENT_SOF();
            break;

        case EVENT_SUSPEND:
            user_EVENT_SUSPEND();
            break;

        case EVENT_RESUME:
            user_EVENT_RESUME();
            break;

        case EVENT_CONFIGURED:
            user_EVENT_CONFIGURED();
            break;

        case EVENT_SET_DESCRIPTOR:
            break;

        case EVENT_EP0_REQUEST:
            USBCheckVendorRequest();
            user_EVENT_EP0_REQUEST();
            break;

        case EVENT_BUS_ERROR:
            break;

        case EVENT_TRANSFER_TERMINATED:
            break;

        default:
            break;
    }
    return true;
}

```

###main.h
新規作成

```cpp:main.h
#ifndef _MAIN_H
#define _MAIN_H

#pragma warning disable 520 //suppress "never called"
#pragma warning disable 362 //suppress "redundant "&" applied to array"

#include <xc.h>
#include <stdbool.h>

#include "fixed_address_memory.h"

void user_EVENT_CONFIGURED();
void user_EVENT_SOF();
void user_EVENT_SUSPEND();
void user_EVENT_RESUME();
void user_EVENT_EP0_REQUEST();

#pragma config CPUDIV = NOCLKDIV
#pragma config USBDIV = OFF
#pragma config FOSC   = HS
#pragma config PLLEN  = ON
#pragma config FCMEN  = OFF
#pragma config IESO   = OFF
#pragma config PWRTEN = OFF
#pragma config BOREN  = OFF
#pragma config BORV   = 30
#pragma config WDTEN  = OFF
#pragma config WDTPS  = 32768
#pragma config MCLRE  = OFF
#pragma config HFOFST = OFF
#pragma config STVREN = ON
#pragma config LVP    = OFF
#pragma config XINST  = OFF
#pragma config BBSIZ  = OFF
#pragma config CP0    = OFF
#pragma config CP1    = OFF
#pragma config CPB    = OFF
#pragma config WRT0   = OFF
#pragma config WRT1   = OFF
#pragma config WRTB   = OFF
#pragma config WRTC   = OFF
#pragma config EBTR0  = OFF
#pragma config EBTR1  = OFF
#pragma config EBTRB  = OFF

#endif
```

##プロジェクトへファイルを追加する
プロジェクトのHeader Filesを右クリックし、「Add Existing Item...」をクリック。
プロジェクトフォルダ内の.hファイルを一括で入れます。
(Shift + クリックの範囲選択が使えます)

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/30690465-09f0-84e1-c9a9-e70d5920f12d.png)

プロジェクトのSource Filesを右クリックし、「Add Existing Item...」をクリック。
プロジェクトフォルダ内の.cファイルを一括で入れます。

このようになればOKです。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/4a626840-d789-2cc2-786b-6e0443b2251e.png)

##ビルドする
F11キーを押してビルドします。
無事通りましたか？

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/a46f9ff5-05a3-556f-55fe-5a1086b67aca.png)


##動作確認
PicKit3を使って18F14K50に書き込み、
plug_and_play_example.exe

```
C:\microchip\mla\v2017_03_06\apps\usb\device\vendor_basic\utilities\plug_and_play_example\bin\plug_and_play_example.exe
```

を起動して、先程のVender Basicデモと同じように動くことを確認してください。
ただし今度は、入出力関係を固定しているため、ボタンは押しっぱなしになります。
空欄になったまま止まった場合は、PICを再接続してみてください。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/0b0c3522-2b14-6697-d24b-e74848ae5723.png)

※Chromeの最新版をお使いの場合は、以下のページのWebUSBでも動作確認ができます．
　https://sabowl.sakura.ne.jp/gpsnmeajp/webusb/webusb.htm

#libusbを使ってアクセスしてみる
Microchip社のデモアプリケーションでアクセスするだけでは面白くありませんので、C言語からアクセスしてみましょう。
VS2015 C++と、libusb-1.0.21で動作を確認しています。

##準備
まず、Visual Studio 2015 Communityが導入されていることとします。
おそらく他のVisual Studioでも動作するとは思いますが、心配な方は導入してください。

また、Win32 コンソールアプリケーションでHello Worldはできるとします。

参考: [Visual Studio 2015のC++でHello World](https://qiita.com/gpsnmeajp/items/1905a74419f8055484d5)

##libusb1.0のダウンロード
libusbの公式サイトへアクセス。
http://libusb.info/

DownloadsのLatest Windows Binariesをクリック。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/9bd85bff-c69e-61df-e4d4-399ca30f0e45.png)

そのまま待つとSourceForgeでダウンロードが始まります。
7z形式でダウンロードされますので、7-zipで解凍してください。

7-zipのダウンロードはここから
https://sevenzip.osdn.jp/

##プロジェクトへの導入
Visual Studioで適当な空のプロジェクトを作成し、その中身のcppファイルが入る場所に

```
libusb-1.0.21\MS32\dll
```
フォルダの中身のlibusb-1.0.dll, libusb-1.0.libm libusb-1.0.pdbをコピー。

また、

```
libusb-1.0.21\include\libusb-1.0
```
フォルダの中身のlibusb.hも同じくコピー。

#main.cpp
ソースファイルに、以下のコードを貼り付けてください。
Microchip社VendorBasicデモデバイスを開き、
バルク転送で0x81(ボタン状態の取得)を送信。
0x81(エコーバック)
0x00(ボタン押下)
の返答を受信するプログラムです。


```cpp:main.cpp
#include <stdio.h>
#include "libusb.h"
#pragma comment(lib,"libusb-1.0.lib")

#define VENDERID 0x04D8
#define PRODUCTID 0x0053

#define EP1 0x1

int main()
{
	libusb_device_handle *devh = NULL;
	unsigned char    send_data[64] = {0}; //マイコン側の受信配列と同じか以下の大きさに合わせること
	unsigned char receive_data[64] = {0}; //マイコン側の送信配列と同じか以下の大きさに合わせること
	int data_len;

	int r;

	try
	{
		//初期化
		r = libusb_init(NULL);
		if ( r < 0 )
			throw(libusb_error_name(r));

		//デバッグ設定
		libusb_set_debug(NULL, LIBUSB_LOG_LEVEL_WARNING);

		//デバイスハンドル取得
		devh = libusb_open_device_with_vid_pid(NULL, VENDERID, PRODUCTID);
		if ( devh == NULL )
			throw("Device Not Found\n");

		//使用権要求
		r = libusb_claim_interface(devh, 0);
		if ( r != 0 )
			throw(libusb_error_name(r));

		//送信
		int dummy = 0;
		send_data[0] = 0x81;
		r = libusb_bulk_transfer(devh, LIBUSB_ENDPOINT_OUT | EP1, send_data, sizeof(send_data), &dummy, 1000);
		if ( r != 0 )
			throw(libusb_error_name(r));

		//受信
		r = libusb_bulk_transfer(devh, LIBUSB_ENDPOINT_IN | EP1, receive_data, sizeof(receive_data), &data_len, 1000);
		if ( r != 0 )
			throw(libusb_error_name(r));

		//受信データの表示
		printf("Received : %d Bytes\n", data_len);
		for ( int i = 0; i < data_len; i++ )
			printf("%02X ", receive_data[i]);

		printf("\n");
	} catch ( const char* e )
	{
		//例外処理
		printf(e);
	}

	//開放処理
	if ( devh != NULL )
		libusb_close(devh);
	libusb_exit(NULL);
	return 0;
}

```

#動作確認
ビルドして実行し、以下のような表示になればOKです。
赤枠範囲外(3バイト目以降)は、ゴミのため、PICの電源を入れ直すたびにランダムに変化します。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/d2f6eafe-00a9-b2b7-007e-7616ff5687bf.png)

#おわりに
今回は、PIC 18F14K50を使ってベンダークラスのバルク転送デモを動かし、
そのプロジェクトを簡略化したプロジェクトファイルを作成。
その後、libusbでアクセスするツールを作成しました。

これにてひとまず終了となりますが、今後HIDデバイスなどを取り扱うかもしれません。

#関連
WebUSBからPIC18Fをつついてみる(仮)
https://qiita.com/gpsnmeajp/items/e490a7a5e2901213184b

USBのベンダーIDとプロダクトIDの話
https://qiita.com/gpsnmeajp/items/8eb8ecf0541032f6de0e

#参考文献
###現状のフレームワークの読み解き方、必要なファイルの選別
PICでUSBを動かす（その２） loops/ウェブリブログ 
http://loops.at.webry.info/201404/article_3.html

###18F14K50全般の使い方
はじめてのPIC 18F14K50
http://sky.geocities.jp/home_iwamoto/page/P14K50/P14_00.htm

###日本語データシート
http://ww1.microchip.com/downloads/jp/DeviceDoc/41350D_JP.pdf
