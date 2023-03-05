#はじめに
これは自分が調べる過程のメモである。
誤りや追加情報があればコメントにて指摘していただけると助かる。

アプリケーションから、ドライバを書かず、ベンダークラスのUSBデバイスを扱おうとした場合。
主に以下の種類がある

#汎用的USBライブラリ
##libusb 0.1(libusb-win32)
libusb 0.1のWindowsに移植版。
すでにメンテナンスオンリーになっており、積極的な更新はない。

独自のドライバで動く、いわゆるlibusb。
フィルタサービスも使用できる。

Windows 98seからWindows 7まで動作する。
アイソクロナス転送をサポートしている。

Win64ではドライバの署名問題がある...と思ったが、
少なくともWindows 7 64bitで動くようにデジタル署名がついているようである。

ビルドにはWindows Driver Kit (WDK)が必要。
ビルド済みバイナリDLLを使うなら不要。

##libusb 1.0(libusb Windows)
現在メインのlibusb

Windows版においてはバックエンドをWinUSBにしたものらしい。
libusbKが存在する場合はそちらを通してWinUSBを、存在しない場合はWinUSBを直接使用する。
現在メインで更新されている。

ビルドにはWindows Driver Kit (WDK)が必要。
ビルド済みバイナリDLLを使うなら不要。

※static版VC++2015で使用しようとしたところ、互換性問題に引っかかりビルド不可。
 DLL版使えば何の問題もなかった...

##libusbK
現在メインで更新されている。
Windows専用。

libusb-0.1を使って、WinUSBのAPIをエミュレートしている。(いた？)
WinUSB 100%互換のドライバプロジェクト。

その上、WinUSBで対応していないアイソクロナス転送や、その他便利な機能を実装している。

libusb 1.0から見て、以前のlibusbデバイスドライバの役目をするらしい。
これはあくまでバックエンドということらしい。

##WinUSB
Microsoftの作ったlibusb的なもの。
ビルドにはWindows Driver Kit (WDK)が必要。

公式サポートなのでドライバの署名問題はなく、自動認識などで優遇されているようであるが、
アシンクロナスに対応していない、複合デバイスに向いていないなどの不便な点もある。

Windows RT環境ではこれしか使えないという話もある。
また、当然Windows上でしか動作しない。

##usbdk
Windows用のライブラリ。
ドライバやINFの作成と無関係に、USBシステムに直接アクセスするためのライブラリのようである。

libusb 1.0から見て、以前のlibusbフィルタドライバの役目をするらしい。
その性質上、USB機器が全て認識されなくなるなどの場合もある様子。

ビルドにはWindows Driver Kit (WDK)が必要。

#汎用USBドライバ(USBD.DLL)
極力シンプル化したUSB通信を行うためのドライバ。

USBデバイスやパイプをオープンした後は、
Windowsのファイルアクセスと同様に読み書きができる。

電子工作界隈では人気であったが、更新がなされていない。

非公式の64bit版を、自己署名することで一応使用することはできる様子である。
が、かなり非推奨。


##Zadig
ドライバインストーラ

#libusbのサンプル
##普通の
VS2015で動作確認。
プロジェクトのフォルダに、64bit版のlibusb-1.0.libとlibusb-1.0.dllを入れ、
libusb.hも入れて、あとは普通にビルド。

microchip社のPIC-USBサンプル VendorBasicに対応

```cpp:main.cpp
#include <stdio.h>
#include "libusb.h"
#pragma comment(lib,"libusb-1.0.lib")

void die(char* s)
{
	printf("Error: ");
	puts(s);
	exit(-1);

	libusb_exit(NULL);
}

int main()
{
	int ret;
	
	//初期化
	if ( libusb_init(NULL) < 0 )
		die("libusb_init");
	printf("Init\n");

	//デバッグレベルの設定(通信失敗時などにメッセージが出るようになる)
	libusb_set_debug(NULL, LIBUSB_LOG_LEVEL_WARNING);
	printf("Set debug\n");

	//デバイスをVIDとPIDを指定してオープン
	libusb_device_handle *devh = libusb_open_device_with_vid_pid(NULL, 0x04D8, 0x0053);
	if ( devh == NULL)
		die("libusb_open_device_with_vid_pid");
	printf("libusb_open_device_with_vid_pid\n");

	//デバイスのインターフェースの使用権を要求
	//これをしないと、以降通信をしようとしても
	//libusb: error [winusbx_submit_bulk_transfer] unable to match endpoint to an open interface - cancelling transfer
	//が出てしまいうまく行かない。
	ret = libusb_claim_interface(devh,0);
	if ( ret == 0 )
	{
		printf("OK\n");
	}else if ( ret == LIBUSB_ERROR_NOT_FOUND )
		die("libusb_claim_interface : LIBUSB_ERROR_NOT_FOUND");
	else if ( ret == LIBUSB_ERROR_BUSY )
		die("libusb_claim_interface : LIBUSB_ERROR_BUSY");
	else if ( ret == LIBUSB_ERROR_OVERFLOW )
		die("llibusb_claim_interface : LIBUSB_ERROR_NO_DEVICE");
	else
		die("libusb_claim_interface : UNKNOWN %d");
	printf("libusb_claim_interface\n");

	//---

	unsigned char data[64]={0x81};
	int data_len=1;

	//---

	//EP=1にバルク送信。タイムアウト1000ms
	ret = libusb_bulk_transfer(devh, LIBUSB_ENDPOINT_OUT | 1, data, sizeof(data), &data_len, 1000);
	if ( ret == 0 )
	{
		printf("Received : %d Bytes\n", data_len);
		for ( int i = 0; i<data_len; i++ )
		{
			printf("%02X ", data[i]);
		}
	} else if ( ret == LIBUSB_ERROR_TIMEOUT )
		die("libusb_bulk_transfer : LIBUSB_ERROR_TIMEOUT");
	else if ( ret == LIBUSB_ERROR_PIPE )
		die("libusb_bulk_transfer : LIBUSB_ERROR_PIPE");
	else if ( ret == LIBUSB_ERROR_OVERFLOW )
		die("libusb_bulk_transfer : LIBUSB_ERROR_OVERFLOW");
	else if ( ret == LIBUSB_ERROR_NO_DEVICE )
		die("libusb_bulk_transfer : LIBUSB_ERROR_NO_DEVICE");
	else
		die("libusb_bulk_transfer : UNKNOWN");

	printf("\nSent\n");

	//---

	//EP=1からバルク受信。タイムアウト1000ms
	ret = libusb_bulk_transfer(devh, LIBUSB_ENDPOINT_IN|1, data, sizeof(data), &data_len, 1000);
	if(ret == 0)
	{
		printf("Received : %d Bytes\n",data_len);
		for(int i=0;i<data_len;i++ )
		{
			printf("%02X ",data[i]);
		}
	}else if ( ret == LIBUSB_ERROR_TIMEOUT )
		die("libusb_bulk_transfer : LIBUSB_ERROR_TIMEOUT");
	else if ( ret == LIBUSB_ERROR_PIPE )
		die("libusb_bulk_transfer : LIBUSB_ERROR_PIPE");
	else if ( ret == LIBUSB_ERROR_OVERFLOW )
		die("libusb_bulk_transfer : LIBUSB_ERROR_OVERFLOW");
	else if ( ret == LIBUSB_ERROR_NO_DEVICE )
		die("libusb_bulk_transfer : LIBUSB_ERROR_NO_DEVICE");
	else
		die("libusb_bulk_transfer : UNKNOWN");


	printf("\nReceived\n");

	//片付け
	libusb_close(devh);
	libusb_exit(NULL);

	printf("Done\n");
	return 0;
}
```

##極力シンプル化したサンプル

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
