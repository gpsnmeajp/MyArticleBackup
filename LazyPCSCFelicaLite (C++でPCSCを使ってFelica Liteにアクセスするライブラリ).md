#LazyPCSCFelicaLite v0.41
FelicaやNFCに興味を持っている，けれどWindowsで動くソフトを作るのはちょっと面倒そう．

というわけで，Visual C++でFelica Liteを読み書きするためのライブラリを作成しました．
面倒くさくても大丈夫．かなりシンプルにアクセスできるようにしました．
.hファイルをincludeするだけで使えます．([VC++のプロジェクトの立ち上げ方がわからない人はこちら](https://qiita.com/gpsnmeajp/items/1905a74419f8055484d5) )

Sony公式のPC/SC機能を使って読み書きするためlibusbの導入などは不要です．

(ただし最新のカードリーダーであるRC-S380が必要です．
　古いカードリーダーで使用したい場合はfelicalibなどを検討してください)

2018/02/03 v0.41,v0.4
2018/02/02 v0.31
2018/02/01 v0.3
2018/01/31 v0.2

zlibライセンスです．
**[ダウンロードはこちら](https://sabowl.sakura.ne.jp/gpsnmeajp/cpp/lazypcscfelicalite/)**

対応カードリーダーはPC/SC対応のPaSoRi RC-S380のみとなっています．
使い方等の詳細は以下をご覧ください．

#このライブラリを使った作例
[EasyPCSCFelicaLiteConsole](https://sabowl.sakura.ne.jp/gpsnmeajp/cpp/easypcscfelicaliteconsole/)
Felica Liteの内容のダンプや発行状態の確認、簡易NDEFタグ化ができるツールです
(データの読み取り，NDEF関係のサンプルです)

[IDm2KeyInput](https://sabowl.sakura.ne.jp/gpsnmeajp/cpp/idm2keyinput/)
バーコードリーダーのようにカードのID情報をキーボード入力するツールです．
(リアルタイム反応のサンプルです)

#解説等について
###PC/SCを使ったFelicaへのアクセスの詳細に関しては以下を参照してください．
[PC/SCでFelica LiteにC言語でアクセスする](https://qiita.com/gpsnmeajp/items/d4810b175189609494ac)

###使用にあたってはFelicaとPC/SCの仕様を確認する必要がある場合があります．
以下のドキュメントを参照してください．
・[SDK for NFC Starter Kit](https://www.sony.co.jp/Products/felica/business/products/ICS-D004.html)のドキュメント
・[FeliCa技術方式の各種コードについて](https://www.sony.co.jp/Products/felica/business/tech-support/index.html)
・[FeliCa Lite-Sスターターマニュアル](https://www.sony.co.jp/Products/felica/business/tech-support/index.html)
・[FeliCa Lite-Sユーザーズマニュアル](https://www.sony.co.jp/Products/felica/business/tech-support/index.html)
・[FeliCa Lite-Sに関するソフトウェア開発テクニカルノート](https://www.sony.co.jp/Products/felica/business/tech-support/index.html)
・[FeliCa Lite-Sセキュリティアプリケーションノート](https://www.sony.co.jp/Products/felica/business/tech-support/index.html)

以下のサイトが大変参考になります．(以下のサイトを多大に参考にさせて頂きこのライブラリを作成できました)
・[EternalWindows セキュリティ / スマートカード](http://eternalwindows.jp/security/scard/scard00.html)
　↑PC/SCの考え方およびエラーコード
・[PC/SC APIを用いてSuicaカードの利用履歴情報の読み取り(tomosoft)](https://tomosoft.jp/design/?p=5543)
　↑Felica使用時の流れ
・[PC/SC Workgroup](http://pcscworkgroup.com/)

・[Cryptography API: Next Generationを使う - Syuhitu the Text editor](http://www.syuhitu.org/other/cng/cng.html)
・[C言語Win32APIだけで乱数 c言語やツールを紹介 - geoblog](http://geoserver.sakura.ne.jp/blog/c%E8%A8%80%E8%AA%9E/c%E8%A8%80%E8%AA%9Ewin32api%E3%81%A0%E3%81%91%E3%81%A7%E4%B9%B1%E6%95%B0/)
・[hiro99ma blog: [felica]FeliCa Lite-Sの相互認証が通った！！](http://hiro99ma.blogspot.com/2014/08/felicafelica-lite-s.html)


#NFCポートの設定とレスポンスについて
デフォルトの設定だと，タッチから反応まで少々時間が(1秒程度)かかります．
公開されている仕様を確認する限り，ポーリング間隔などはアプリケーションから変更できません．

コントロールパネルの「NFCポート／パソリ」にて，
・「ポーリング間隔」を「高速にする」
・「検出対象カード」を「NFC-F」のみにする
と，反応がとても良くなります．
(ただし，検出対象カードを絞ると，元に戻すまでe-taxやマイナンバーポータルで使えなくなります．)


#サンプル
##共通

```cpp:main.h
#include "lazypcscfelicalite.h"

using LazyPCSCFelicaLite::PCSCFelicaLite;
using LazyPCSCFelicaLite::PCSCErrorException;
using LazyPCSCFelicaLite::PCSCIllegalStateException;
using LazyPCSCFelicaLite::PCSCCommandException;
using LazyPCSCFelicaLite::PCSCFatalException;
using LazyPCSCFelicaLite::PCSCCryptographicException;
using LazyPCSCFelicaLite::FelicaErrorException;
using LazyPCSCFelicaLite::FelicaFatalException;
using LazyPCSCFelicaLite::PCSCCardRemovedException;
```

##IDmを表示するサンプル
自動接続を使う場合．

```cpp:main.cpp
int main()
{
	PCSCFelicaLite f = PCSCFelicaLite();

	try
	{
		printf("かざしてください...\n");
		//カードに自動接続
		//カードがかざされるのを無限に待つ
		f.autoConnectToFelica(INFINITE);

		//カードのUID(IDm)を取得
		uint8_t uid[256] = {0};
		int len = f.readUID(uid);

		for ( int i = 0; i<len; i++ )
			printf("%02X ", uid[i]);
		printf("\n");
	} catch ( std::runtime_error e )
	{
		printf("例外: %s\n", e.what());
	}
}
```

##カードをNDEFタグにする(URLを書き込む)
自動接続を使う場合．

```cpp:main.cpp
int main()
{
	PCSCFelicaLite f = PCSCFelicaLite();

	try
	{
		printf("かざしてください...\n");
		//カードに自動接続
		//カードがかざされるのを無限に待つ
		f.autoConnectToFelica(INFINITE);

		//NDEFフラグを有効にする(1次発行は行わない)
		f.enableNDEF(true);

		//カードにURLを書き込む
		f.writeNdefURI(f.NDEF_HTTPS, "qiita.com/gpsnmeajp/items/a378e1829fba1c3a3ed7");
	} catch ( std::runtime_error e )
	{
		printf("例外: %s\n",e.what());
	}
}
```

##カードがかざされているかリアルタイムにチェックする
このような用途には必ずautoConnectToFelicaを使用してください．
isCardsetなどでは意図した挙動にならない場合があります．

```cpp:main.cpp
int main()
{
	PCSCFelicaLite f = PCSCFelicaLite();

	try
	{
		printf("かざしてください...\n");
		while(1)
		{
			if(f.autoConnectToFelica()){
				printf("CARD ON \r");
			}else{
				printf("CARD OFF\r");
			}
		}
	} catch ( std::runtime_error e )
	{
		printf("例外: %s\n", e.what());
	}
}
```

##カードから1ブロック読み込むサンプル
自動接続を使わない場合．

```cpp:main.cpp
int main()
{
	PCSCFelicaLite f = PCSCFelicaLite();

	try
	{
		//サービスを開く
		f.openService();

		//カードがかざされるのを無限に待つ
		printf("かざしてください...\n");
		f.waitForSetCard(INFINITE);

		//カードに接続
		f.connectCard();

		//カードのアドレス0x0000から1ブロック読み込む
		uint8_t buf[16] = {0};
		f.readBinary(0x0000, buf);

		for ( int i = 0; i<16; i++ )
			printf("%02X ", buf[i]);
		printf("\n");

		//カードから切断
		f.disconnectCard();
		//サービスを閉じる
		f.closeService();
	} catch ( std::runtime_error e )
	{
		printf("例外: %s\n",e.what());
	}
}
```

##カードへ1ブロック書き込むサンプル
自動接続を使わない場合．

```cpp:main.cpp
int main()
{
	PCSCFelicaLite f = PCSCFelicaLite();

	try
	{
		//サービスを開く
		f.openService();

		//カードがかざされるのを無限に待つ
		printf("かざしてください...\n");
		f.waitForSetCard(INFINITE);

		//カードに接続
		f.connectCard();

		//カードのアドレス0x0000に1ブロック書き込む
		uint8_t buf[16] = {0xAA,0x55,0xAA,0x55,0xAA,0x55,0xAA,0x55,0xAA,0x55,0xAA,0x55,0xAA,0x55,0xAA,0x55};
		f.updateBinary(0x0000, buf);

		//カードから切断
		f.disconnectCard();
		//サービスを閉じる
		f.closeService();
	} catch ( std::runtime_error e )
	{
		printf("例外: %s\n",e.what());
	}
}
```

##カードが本物かチェックする
1発行されていないカードにカード鍵を書き込んでください．
(f.writeCardKeyのコメントアウトを外して実行するとカード鍵を書き込みます．
　発行は行いません．カード鍵は何度でも書き換えることができます)


```cpp:main.cpp
int main()
{
	PCSCFelicaLite f = PCSCFelicaLite(true);

	//カード鍵
	uint8_t CK1[8] = {0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00};
	uint8_t CK2[8] = {0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00};

	try
	{
		printf("かざしてください...\n");

		//カードに自動接続
		//カードがかざされるのを無限に待つ
		f.autoConnectToFelica(INFINITE);

		//カード鍵を書き込み(初回に行ってください)
		//f.writeCardKey(CK1, CK2);

		if ( f.cardIdCheckMAC_A(CK1, CK2) )
		{
			printf("カード鍵照合OK\n");
		} else
		{
			printf("カード鍵照合NG\n");
		}

	} catch ( std::runtime_error e )
	{
		printf("例外: %s\n", e.what());
	}
}
```

```shell-session:output
[PCSCFelicaLite.PCSCFelicaLite] PCSCFelicaLite(bool debug)
かざしてください...
[PCSCFelicaLite.autoConnectToFelica]カードへの自動接続開始
[PCSCFelicaLite.openService] スマートカードリソースマネージャへの接続に成功
[PCSCFelicaLite.openService] カードリーダー名リスト
[PCSCFelicaLite.openService][0] Sony FeliCa Port/PaSoRi 3.0 0
[PCSCFelicaLite.openService] Sony FeliCa Port/PaSoRi 3.0 0を使用します
[PCSCFelicaLite.isCardSet] カードがセットされています DE0422
[PCSCFelicaLite.isCardSet] カードがセットされています DE0422
[PCSCFelicaLite.connectDirect] 直接接続しました
[PCSCFelicaLite.polling] IDm:XXXXXXXXXXXXXXXX
[PCSCFelicaLite.polling] PMm:XXXXXXXXXXXXXXXX
[PCSCFelicaLite.polling] システムコード:88B4
[PCSCFelicaLite.disconnectCard] 接続を切断しました
[PCSCFelicaLite.readCardTypeCode] カード種別:04
[PCSCFelicaLite.readCardTypeCode] このカードはFelicaです．
[PCSCFelicaLite.readSystemcode] システムコード:88B4
[PCSCFelicaLite.connectCard] カードに接続しました(カード種別:04, システムコード:88B4)
[PCSCFelicaLite.autoConnectToFelica]カードへの接続に成功
[PCSCFelicaLite.makeRandomChallenge] RCを生成しました
[PCSCFelicaLite.makeRandomChallenge] RC1 02 AE C8 FE BE A5 B0 0A
[PCSCFelicaLite.makeRandomChallenge] RC2 63 0D 1D F2 E8 3D B5 BD
[PCSCFelicaLite.updateBinary]  アドレス:0080 書き込み成功
[PCSCFelicaLite.updateBinary] 内容:02,AE,C8,FE,BE,A5,B0,0A,63,0D,1D,F2,E8,3D,B5,BD,
[PCSCFelicaLite.writeRandomChallenge] RCを書き込みました
[PCSCFelicaLite.readBinaryWithMAC_A] アドレス:0082 読み込み完了
[PCSCFelicaLite.readBinaryWithMAC_A] 内容:01,2E,41,29,35,8B,A7,53,00,00,00,00,00,00,00,00,
[PCSCFelicaLite.readBinaryWithMAC_A] MAC:65,A9,1E,14,1C,EB,8E,C2,00,00,00,00,00,00,00,00,
[PCSCFelicaLite.makeSessionKey] セッションキーを生成しました
[PCSCFelicaLite.makeSessionKey] SK1 82 FB 7C 45 BC B6 06 E8
[PCSCFelicaLite.makeSessionKey] SK2 5B A5 DD E9 16 91 CD D6
[PCSCFelicaLite.makeReadMAC_A] MAC_Aを生成しました
[PCSCFelicaLite.makeReadMAC_A] MAC_A 65 A9 1E 14 1C EB 8E C2
[PCSCFelicaLite.compareMAC_A] MAC一致
[PCSCFelicaLite.cardIdCheckMAC_A] カード鍵照合に成功
カード鍵照合OK
[PCSCFelicaLite.~PCSCFelicaLite] デコンストラクタ
[PCSCFelicaLite.disconnectCard] 接続を切断しました
[PCSCFelicaLite.closeService] スマートカードリソースマネージャを解放
```

##相互認証(外部認証)を行う
相互認証の考え方についてはこのサイトが非常にわかりやすいです．
[MARKETEQ セキュアエンブレムにFeliCa Lite-Sを搭載した理由](http://www.marketeq.co.jp/felica_2.html)

1発行されていないカードにカード鍵を書き込み，STATEへのMAC付き書き込みを有効にして実行してください．
(f.writeCardKeyおよびenableSTATEのコメントアウトを外して実行するとカード鍵を書き込みます．
　STATEのMAC付き書き込み設定は一度有効にすると無効に戻すことはできません．
　カード鍵は何度でも書き換えることができますが，WCNTが上限に達すると相互認証ができなくなります．
　1次発行を行うと，WNCTはリセットされまた相互認証を行うことができるようになります)

```cpp:main.cpp
int main()
{
	PCSCFelicaLite f = PCSCFelicaLite(true);

	//カード鍵
	uint8_t CK1[8] = {0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00};
	uint8_t CK2[8] = {0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00};

	try
	{
		printf("かざしてください...\n");

		//カードに自動接続
		//カードがかざされるのを無限に待つ
		f.autoConnectToFelica(INFINITE);

		//カード鍵を書き込み(初回に行ってください)
		//f.writeCardKey(CK1, CK2);

		//相互認証の有効化(初回に行ってください．一度有効にすると以後無効にできません)
		//f.enableSTATE();

		//相互認証を行い，アクセス制限領域を読み書きできるようにする
		f.setSTATE_EXT_AUTH(true,CK1,CK2);

	} catch ( std::runtime_error e )
	{
		printf("例外: %s\n", e.what());
	}
}
```

```shell-session:output
[PCSCFelicaLite.PCSCFelicaLite] PCSCFelicaLite(bool debug)
かざしてください...
[PCSCFelicaLite.autoConnectToFelica]カードへの自動接続開始
[PCSCFelicaLite.openService] スマートカードリソースマネージャへの接続に成功
[PCSCFelicaLite.openService] カードリーダー名リスト
[PCSCFelicaLite.openService][0] Sony FeliCa Port/PaSoRi 3.0 0
[PCSCFelicaLite.openService] Sony FeliCa Port/PaSoRi 3.0 0を使用します
[PCSCFelicaLite.isCardSet] カードがセットされています E40022
[PCSCFelicaLite.isCardSet] カードがセットされています E40022
[PCSCFelicaLite.connectDirect] 直接接続しました
[PCSCFelicaLite.polling] IDm:XXXXXXXXXXXXXXXX
[PCSCFelicaLite.polling] PMm:XXXXXXXXXXXXXXXX
[PCSCFelicaLite.polling] システムコード:88B4
[PCSCFelicaLite.disconnectCard] 接続を切断しました
[PCSCFelicaLite.readCardTypeCode] カード種別:04
[PCSCFelicaLite.readCardTypeCode] このカードはFelicaです．
[PCSCFelicaLite.readSystemcode] システムコード:88B4
[PCSCFelicaLite.connectCard] カードに接続しました(カード種別:04, システムコード:88B4)
[PCSCFelicaLite.autoConnectToFelica]カードへの接続に成功
[PCSCFelicaLite.readBinary] アドレス:0092 読み込み完了
[PCSCFelicaLite.readBinary] 内容:00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,
[PCSCFelicaLite.setSTATE_EXT_AUTH] 外部認証を開始します
[PCSCFelicaLite.updateBinaryWithMAC_A_Auto] 自動MAC_A付き書き込み開始
[PCSCFelicaLite.makeRandomChallenge] RCを生成しました
[PCSCFelicaLite.makeRandomChallenge] RC1 0A 68 AB 92 3C 54 64 C9
[PCSCFelicaLite.makeRandomChallenge] RC2 89 FA D7 C9 31 57 13 D4
[PCSCFelicaLite.updateBinaryWithMAC_A_Auto] RC生成完了
[PCSCFelicaLite.updateBinary]  アドレス:0080 書き込み成功
[PCSCFelicaLite.updateBinary] 内容:0A,68,AB,92,3C,54,64,C9,89,FA,D7,C9,31,57,13,D4,
[PCSCFelicaLite.writeRandomChallenge] RCを書き込みました
[PCSCFelicaLite.updateBinaryWithMAC_A_Auto] RC書き込み完了
[PCSCFelicaLite.makeSessionKey] セッションキーを生成しました
[PCSCFelicaLite.makeSessionKey] SK1 8D F8 9C 02 45 52 32 95
[PCSCFelicaLite.makeSessionKey] SK2 D2 C3 77 8B 81 79 9B 01
[PCSCFelicaLite.updateBinaryWithMAC_A_Auto] セッションキー生成完了
[PCSCFelicaLite.readBinary] アドレス:0090 読み込み完了
[PCSCFelicaLite.readBinary] 内容:7F,FE,FF,00,00,00,00,00,00,00,00,00,00,00,00,00,
[PCSCFelicaLite.updateBinaryWithMAC_A_Auto] WCNT読み取り完了
[PCSCFelicaLite.updateBinaryWithMAC_A_Auto] WCNT: 7F FE FF 00 00 00 00 00
[PCSCFelicaLite.makeWriteMAC_A] MAC_Aを生成しました
[PCSCFelicaLite.makeWriteMAC_A] MAC_A 52 52 EC D2 84 24 EB D7
[PCSCFelicaLite.updateBinaryWithMAC_A_Auto] 書き込みMAC_A生成完了
[PCSCFelicaLite.updateBinaryWithMAC_A]  アドレス:0092 書き込み成功
[PCSCFelicaLite.updateBinaryWithMAC_A] 内容:01,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,
[PCSCFelicaLite.updateBinaryWithMAC_A] MAC:52,52,EC,D2,84,24,EB,D7,7F,FE,FF,00,CC,CC,CC,CC,
[PCSCFelicaLite.updateBinaryWithMAC_A_Auto] MAC_A付き書き込み完了
[PCSCFelicaLite.setSTATE_EXT_AUTH] 外部認証処理に成功しました
[PCSCFelicaLite.~PCSCFelicaLite] デコンストラクタ
[PCSCFelicaLite.disconnectCard] 接続を切断しました
[PCSCFelicaLite.closeService] スマートカードリソースマネージャを解放
```

#PCSCFelicaLiteクラス

##基本:コンストラクタ・デコンストラクタ
###PCSCFelicaLite

```cpp
PCSCFelicaLite()
PCSCFelicaLite(bool debug)
PCSCFelicaLite(bool debug, string name)
```

コンストラクタ．debugをtrueで冗長なデバッグ情報を表示．
かなり詳細に出力するため，困ったときはtrueにすると様々な情報を得られます．
複数インスタンスを使う場合のデバッグ用に名前もつけられるようになりました．

めちゃめちゃ冗長です．
例:カードをNDEFタグにする(URLを書き込む)の場合

```shell-session
[PCSCFelicaLite.PCSCFelicaLite] PCSCFelicaLite(bool debug)
かざしてください...
[PCSCFelicaLite.autoConnectToFelica]カードへの自動接続開始
[PCSCFelicaLite.openService] スマートカードリソースマネージャへの接続に成功
[PCSCFelicaLite.openService] カードリーダー名リスト
[PCSCFelicaLite.openService][0] Sony FeliCa Port/PaSoRi 3.0 0
[PCSCFelicaLite.openService] Sony FeliCa Port/PaSoRi 3.0 0を使用します
[PCSCFelicaLite.isCardSet] カードがセットされています 43640422
[PCSCFelicaLite.isCardSet] カードがセットされています 43640422
[PCSCFelicaLite.connectDirect] 直接接続しました
[PCSCFelicaLite.polling] IDm:XXXXXXXXXXXXXXXX
[PCSCFelicaLite.polling] PMm:XXXXXXXXXXXXXXXX
[PCSCFelicaLite.polling] システムコード:88B4
[PCSCFelicaLite.disconnectCard] 接続を切断しました
[PCSCFelicaLite.readCardTypeCode] カード種別:04
[PCSCFelicaLite.readCardTypeCode] このカードはFelicaです．
[PCSCFelicaLite.readSystemcode] システムコード:88B4
[PCSCFelicaLite.connectCard] カードに接続しました(カード種別:04, システムコード:88B4)
[PCSCFelicaLite.autoConnectToFelica]カードへの接続に成功
[PCSCFelicaLite.isFelicaLite] このカードはFelica Lite系です
[PCSCFelicaLite.isFelicaLite] このカードはFelica Lite系です
[PCSCFelicaLite.readBinary] アドレス:0088 読み込み完了
[PCSCFelicaLite.readBinary] 内容:FF,FF,FF,01,07,00,00,00,00,00,00,00,00,00,00,00,
[PCSCFelicaLite.isFirstIssued] このカードはまだ1次発行されていません
[PCSCFelicaLite.readBinary] アドレス:0088 読み込み完了
[PCSCFelicaLite.readBinary] 内容:FF,FF,FF,01,07,00,00,00,00,00,00,00,00,00,00,00,
[PCSCFelicaLite.enableNDEF] NDEF有効化
[PCSCFelicaLite.updateBinary]  アドレス:0088 書き込み成功
[PCSCFelicaLite.isFelicaLite] このカードはFelica Lite系です
[PCSCFelicaLite.updateBinary]  アドレス:0000 書き込み成功
[PCSCFelicaLite.updateBinary]  アドレス:0001 書き込み成功
[PCSCFelicaLite.updateBinary]  アドレス:0002 書き込み成功
[PCSCFelicaLite.updateBinary]  アドレス:0003 書き込み成功
[PCSCFelicaLite.updateBinary]  アドレス:0004 書き込み成功
[PCSCFelicaLite.updateBinary]  アドレス:0005 書き込み成功
[PCSCFelicaLite.updateBinary]  アドレス:0006 書き込み成功
[PCSCFelicaLite.updateBinary]  アドレス:0007 書き込み成功
[PCSCFelicaLite.updateBinary]  アドレス:0008 書き込み成功
[PCSCFelicaLite.updateBinary]  アドレス:0009 書き込み成功
[PCSCFelicaLite.updateBinary]  アドレス:000A 書き込み成功
[PCSCFelicaLite.updateBinary]  アドレス:000B 書き込み成功
[PCSCFelicaLite.updateBinary]  アドレス:000C 書き込み成功
[PCSCFelicaLite.updateBinary]  アドレス:000D 書き込み成功
[PCSCFelicaLite.writeNdefURI] NDEF書き込み完了 qiita.com/gpsnmeajp/items/a378e1829fba1c3a3ed7
[PCSCFelicaLite.~PCSCFelicaLite] デコンストラクタ
[PCSCFelicaLite.disconnectCard] 接続を切断しました
[PCSCFelicaLite.closeService] スマートカードリソースマネージャを解放
```


###~PCSCFelicaLite()
デコンストラクタ．
disconnectCard()およびcloseService()は自動で行われる．

##基本:自動接続
###bool autoConnectToFelica(uint32_t timeout=0)
・サービスのオープン(openService,closeService)
・カードの検出待ち(waitForSetCard)
・検出カードと安定して通信可能かのチェック(connectDirect,polling,disconnectCard)
・カードへの接続(connectCard)
を自動で行う．

基本的にこのメソッドを使っておけば安定して検出できます．
戻り値は，時間内にカードが検出できた場合true，できなかった場合false

timeoutに0をセットすると現在の状態を確認して即座に戻ります．
timeoutにINFINYTEをセットすると、カードがかざされるのを無限に待ちます．
カードの検出を頻繁に行いたい場合に利用してください．

*※isCardSet()はカードのセットには機敏に反応しますが，カードの取り外しには反応が悪い場合があります．*
*autoConnectToFelicaはそういった場合でもしっかりと現在のカードの状態を判別します．*

##基本:サービスのオープン・クローズ
###void openService()
スマートカードリソースマネージャへの接続と，カードリーダーの取得・検索を行う．
SCardEstablishContextの後，SCardListReadersAを行い，
activeReaderNameとReaderNameListを更新する．
選択されるリーダーは「FeliCa」を含む一番最初にマッチした名称のリーダー．

###void closeService()
スマートカードリソースマネージャからの切断

##基本:カードの状態
###bool isCardSet()
現在のカードの状態を取得する．
かざされている(利用可能である)ならtrue
何らかの理由で利用不能であればfalseを返す．

※ポーリング間隔以内の変化は検出できません．
　また，他のメソッドを短時間の間に使用すると変化の検出を妨害する場合があります．
　短時間の間に，かざされたり取り外されたりする用途には，autoConnectToFelica()を使用してください．

###bool waitForSetCard(uint32_t timeout)
カードがセットされるまで指定時間(ミリ秒)待つ．
timeoutにINFINYTEをセットすると無限に待つ

※短時間の間に，かざされたり取り外されたりする用途には，autoConnectToFelica()を使用してください．

##基本:カードとの接続・切断
###void connectCard(bool exclusive = true)
かざされているカードに接続する．
カードがかざされていない状態で行うと失敗する．
exclusiveをfalseにすると共有モードで開く．

###void connectDirect()
カードの存在にかかわらずカードリーダーに直接接続する．
カードリーダーのシリアルナンバーの取得や，ポーリングなど．

###void disconnectCard()
カード(直接接続時はカードリーダー)から切断する

##基本:カードの読み書き
###void readBinary(uint16_t adr, uint8_t dat[16])
<font color="Red">**カードとの接続が必要**</font>
カードから読み取る．
ReadWithoutEncryptionに相当．
ただし1ブロックのみ．

###void updateBinary(uint16_t adr, uint8_t dat[16]) 
<font color="Red">**カードとの接続が必要**</font>
カードへの書き込み
WriteWithoutEncryptionに相当．
ただし1ブロックのみ．

※今後認証付き書き込み・読み出しのサポートもするかもしれません．

##基本:カード詳細情報の取得
###uint32_t readUID(uint8_t UID[]=NULL)
<font color="Red">**カードとの接続が必要**</font>
カードのIDmおよびUIDを取得．カードの種別により長さが違うため注意．

戻り値:データ長
(引数なし，あるいはNULLの場合はデバッグメッセージに出力する)

###uint32_t readPMm(uint8_t PMm[] = NULL)
<font color="Red">**カードとの接続が必要**</font>
カードのPMmを取得

戻り値:データ長
(引数なし，あるいはNULLの場合はデバッグメッセージに出力する)

###bool isFelica()
connectCard時に取得したカードの種類コードからFelicaかどうかを判別

###bool isFelicaLite()
connectCard時に取得したシステムコードから，Felica Liteかどうかを判別

###uint8_t getCardTypeCode()
connectCard時に取得したカードの種類コードを返す

###uint16_t getSystemcode()
connectCard時に取得したシステムコードを返す

###bool isNdefEnabled()
<font color="Red">**カードとの接続が必要**</font>
NDEFが有効かをチェックする

###bool isFirstIssued()
<font color="Red">**カードとの接続が必要**</font>
1次発行済かをチェックする

###bool isSecondIssued()
<font color="Red">**カードとの接続が必要**</font>
2次発行済かをチェックする

###void readCardTypeString(char *cardTypeString = NULL)
<font color="Red">**カードとの接続が必要**</font>
カード種別文字列を取得

(引数なし，あるいはNULLの場合はデバッグメッセージに出力する)

###uint8_t readCardTypeCode()
<font color="Red">**カードとの接続が必要**</font>
カードの種類コードを取得

###uint16_t readSystemcode(uint16_t scancode = SYSTEMCODE_ANY)
<font color="Red">**カードとの接続が必要**</font>
pollingしてシステムコードを取得する．
scancodeを指定すると，そのコードマスクに一致するカードだけが反応を返す．
(デフォルトはすべてのカードを対象とするSYSTEMCODE_ANY)
SYSTEMCODE_FELICALITE (0x88B4)はすべてのFelica Liteが反応する．
SYSTEMCODE_NFC_TYPE3 (0x12FC)はNDEFが有効なFelica Liteのみが反応する．

※可能な限りgetSystemcode()を使用してください．

##応用:簡易NDEF書き込み
###void enableNDEF(bool en=true)
<font color="Red">**カードとの接続が必要**</font>
NDEFを有効・無効にする．
1次発行はせず，NDEFフラグだけ立てる．
0次発行+NDEFとなる．1次発行前のカードに対してのみ有効．

###void writeNdefURI(uint8_t mode, const char str[]){
<font color="Red">**カードとの接続が必要**</font>
簡易的なNDEF(URI)を書き込む．

mode
0x01->http://www.
0x02->https://www.
0x03->http://
0x04->https://

##応用:内部状態の取得
###void checkReadyContext()
スマートカードリソースマネージャへの接続が有効かをチェックする．

###void checkReadyCard()
カードハンドルが有効かをチェックする．

##応用:デバッグ
###void setDebug(bool mode)
冗長なデバッグ表示を有効・無効にする．

###void setInstanceName(string name)
デバッグ表示に使う名称を設定する

###uint32_t getCurrentState()
最後にisCardSet等で取得したカード状態を取得する．
isCardSetのstate引数に使用することができる．

###const char* FellicaErrorInfo(uint8_t code)
与えられたコードに対するFelicaエラーメッセージを返す．

##上級:接続無しでカード詳細情報の取得
###uint16_t polling(uint8_t UID[] = NULL, uint8_t PMm[] = NULL, uint16_t scancode = SYSTEMCODE_ANY)
<font color="Red">**カードあるいは直接接続が必要**</font>
直接PollingしてIDmとPMmとシステムコードを取得する．
直接接続で利用できるため，カードといちいち接続・切断する必要はない．

scancodeを指定すると，そのコードマスクに一致するカードだけが反応を返す．
(デフォルトはすべてのカードを対象とするSYSTEMCODE_ANY)
SYSTEMCODE_FELICALITE (0x88B4)はすべてのFelica Liteが反応する．
SYSTEMCODE_NFC_TYPE3 (0x12FC)はNDEFが有効なFelica Liteのみが反応する．

(引数なし，あるいはNULLの場合はデバッグメッセージに出力する)

##上級:カードリーダー情報の取得・変更
###void getPaSoRiSerialNumber(char *serialNumberString = NULL)
<font color="Red">**カードあるいは直接接続が必要**</font>
PaSoRiのシリアルナンバー文字列を取得

(引数なし，あるいはNULLの場合はデバッグメッセージに出力する)

###vector<string> getReaderNameList()
カードリーダー名のリスト配列を取得する．
手動でカードリーダーを選択したい場合など，setActiveReaderNameと組み合わせて使う．
取得前の場合は空のvectorが帰る．

###string getActiveReaderName()
現在アクティブになっているカードリーダー名を取得する．
多くの場合は自動選択されたものである．
選択されていない場合は空文字列が帰る．

###void setActiveReaderName(string s)
アクティブなカードリーダー名に指定した文字列を使用する．
getReaderNameListと組み合わせて使うことが多い．

##上級:生ハンドルの取得
###SCARDHANDLE getCardHandle()
SCARDHANDLEを取得する．

###SCARDCONTEXT getCardContext()
SCARDCONTEXTを取得する．

##危険:簡易発行処理
###void FirstIssue()
<font color="Red">**カードとの接続が必要**</font>
簡易的な1次発行(システムブロックのロック)を行う．
<font color="Red">一度発行すると二度と戻せないので注意！</font>

カード鍵およびバージョンは0とし，MAC付き書き換えを有効にする．


#MAC認証処理

##カード鍵書き込み・生成
###void makeRandomChallenge(uint8_t RC1[8], uint8_t RC2[8])
ランダムチャレンジRC1,RC2を暗号論的擬似乱数から生成する．カードに書き込みは行わない．
カード鍵の生成にも利用できるが，二度と同じものは生成しないので注意．

###void writeRandomChallenge(uint8_t RC1[8], uint8_t RC2[8])
指定したランダムチャレンジRC1,RC2をカードに書き込む．
何度でも書き込める．

###void writeCardKey(uint8_t CK1[8], uint8_t CK2[8])
指定したカード鍵CK1,CK2をカードに書き込む．
このカード鍵はいかなる手段でも読み出すことができない．紛失注意．
1次発行を行うまでは何度も書き換えられ，最後に書き込んだものが有効になる．
1次発行後は，特例を除き書き換えもできなくなる．

書き込み試験などでCKを書き換えた後，すぐに試す場合は，RCを書き込んでセッション鍵を再生成させること．
でないとMAC生成に以前のセッション鍵が利用され認証に失敗する．


##相互認証
###void setSTATE_EXT_AUTH(bool auth, uint8_t CK1[8], uint8_t CK2[8])
カード鍵CK1,CK2を使い，ランダムチャレンジ・セッション鍵を自動生成し，相互認証(外部認証)を行う．
STATEのEXT_AUTHへMAC付き書き込みを行う．
trueで認証状態へ移行，falseで非認証状態へ移行する．

STATEへのMAC付き書き込み(相互認証)を有効にしないと認証エラーで利用できない．

###void enableSTATE()
STATEへのMACつき書き込みを有効にし，相互認証を利用できるようにする．
一度書き込むと無効に戻すことはできない．


##MAC付きアクセス
###void readBinaryWithMAC(uint16_t adr, uint8_t dat[16], uint8_t mac[16])
カードからMAC付きで情報を読み取る

###void readBinaryWithMAC_A(uint16_t adr, uint8_t dat[16], uint8_t mac[16])
カードからMAC_A付きで情報を読み取る

###void updateBinaryWithMAC_A_Auto(uint16_t adr, uint8_t dat[16],uint8_t CK1[8], uint8_t CK2[8])
カードへMAC_A付きで情報を書き込む．
セッションキーの生成およびWCNTの取得から自動で行うため，CK1,CK2の指定のみで良い．

###void updateBinaryWithMAC_A(uint16_t adr, uint8_t dat[16], uint8_t mac[16])
カードへMAC_A付きで情報を書き込む．
MACの生成に手間がかかるため，基本的にupdateBinaryWithMAC_A_Autoを利用することを推奨する．


##カード真贋識別
###bool cardIdCheckMAC(uint8_t CK1[8], uint8_t CK2[8])
カード鍵CK1,CK2を使い，MACを読み込んでIDmの検証を行う．
セッションキーの生成・書き込みを自動で行う．

・カード鍵と自分の持っている鍵が一致するか(なりすまし検証)
・通信路でIDmが改ざんされていないか
の検証になる．

Felica LiteおよびFelica Lite-Sの両方で使用できるが，ブロック番号偽装攻撃に対して対処できない．
そのため，製造元はMAC_Aを利用した検証(本ライブラリではcardIdCheckMAC_A)を推奨している．

###bool cardIdCheckMAC_A(uint8_t CK1[8], uint8_t CK2[8])
カード鍵CK1,CK2を使い，MACを読み込んでIDmの検証を行う．
セッションキーの生成・書き込みを自動で行う．

・カード鍵と自分の持っている鍵が正しいか(なりすまし検証)
・通信路でIDmが改ざんされていないか
の検証になる．

Felica Lite-Sでしか使用できない．

##データ真贋性識別
###bool compareMAC(uint8_t BLOCK[16], uint8_t MAC[16], uint8_t RC1[8], uint8_t RC2[8], uint8_t CK1[8], uint8_t CK2[8])
ブロックデータ，RC1，SK1，SK2からFelica LiteのMACを生成し，readBinaryWithMACで取得したMACと比較する．
trueで一致，falseで不一致

・カード鍵と自分の持っている鍵が正しいか(なりすまし検証)
・通信路でデータが改ざんされていないか
の検証になる．

###bool compareMAC_A(uint16_t BlockAdr, uint8_t BLOCK[16], uint8_t MAC[16], uint8_t RC1[8], uint8_t RC2[8], uint8_t CK1[8], uint8_t CK2[8])
ブロック番号，ブロックデータ，RC1，SK1，SK2からFelica Lite-Sの読み出し時MAC_Aを生成し，
readBinaryWithMAC_Aで取得したMACと比較する．
trueで一致，falseで不一致

・カード鍵と自分の持っている鍵が正しいか(なりすまし検証)
・通信路でデータが改ざんされていないか
の検証になる．


##セッションキー・MAC生成
###void makeSessionKey(uint8_t RC1[8], uint8_t RC2[8], uint8_t CK1[8], uint8_t CK2[8], uint8_t SK1[8], uint8_t SK2[8])
RC1,RC2,CK1,CK2からFelica LiteセッションキーSK1，SK2を生成する

###void makeMAC(uint8_t BLOCK[16], uint8_t RC1[8], uint8_t SK1[8], uint8_t SK2[8], uint8_t MAC[8])
ブロックデータ，RC1，SK1，SK2からFelica LiteのMACを生成する．
(Felica Lite-SのMAC_Aではない！Felica Liteユーザーズマニュアルを読むこと！)

###void makeReadMAC_A(uint16_t BlockAdr,uint8_t BLOCK[16], uint8_t RC1[8], uint8_t SK1[8], uint8_t SK2[8], uint8_t MAC[8])
ブロック番号，ブロックデータ，RC1，SK1，SK2からFelica Lite-Sの読み出し時MAC_Aを生成する．
(Felica LiteのMACではない！Felica Lite-Sユーザーズマニュアルを読むこと！)

###void makeWriteMAC_A(uint16_t BlockAdr, uint8_t WCNT[8], uint8_t BLOCK[16], uint8_t RC1[8], uint8_t SK1[8], uint8_t SK2[8], uint8_t MAC[8])
ブロック番号，WCNT，ブロックデータ，RC1，SK1，SK2からFelica Lite-Sの書き込み時MAC_Aを生成する．
(Felica LiteのMACではない！Felica Lite-Sユーザーズマニュアルを読むこと！)

##中レベル処理
###void Key2des3(const uint8_t input[8], const uint8_t XorInput[8], const uint8_t key1[8], const uint8_t key2[8], uint8_t output[8])
2key-3DESを行う．

###void DualKey2des3(const uint8_t input1[8], const uint8_t input2[8], const uint8_t XorInput[8], const uint8_t key1[8], const uint8_t key2[8], uint8_t output1[8], uint8_t output2[8])
2連2key-3DESを行う．Felica Lite用にバイトオーダー入れ替え処理を内包している．


##低レベル処理
###void desEncryptFixedLength(const uint8_t input[8], const uint8_t key[8], const uint8_t IV[8], uint8_t output[8])
8バイト固定長でDES暗号化を行う

###void desDecryptFixedLength(const uint8_t input[8], const uint8_t key[8], const uint8_t IV[8], uint8_t output[8])
8バイト固定長でDES復号化を行う

###uint8_t CryptRand()
暗号論的擬似乱数を生成する

###void swapByteOrder(uint8_t inout[8])
バイトオーダーを入れ替える(ポインタの先を直に入れ替えるので注意！)

###void arryXor(const uint8_t input1[8], const uint8_t input2[8], uint8_t output[8])
8バイトのXOR

###void zeroIV(uint8_t IV[8])
初期化ベクトルを初期化

###void showMAC(const uint8_t input[8])
デバッグ用:8Byteの16進数の表示



##Enum

```cpp
		enum
		{
			ESC_CMD_GET_INFO = 0xC0, //バージョンなど各種情報の取得
			ESC_CMD_SET_OPTION = 0xC1, //情報の設定
			ESC_CMD_TARGET_COMM = 0xC4, //ターゲット通信
			ESC_CMD_SNEP = 0xC6, //SNEP通信
			ESC_CMD_APDU_WRAP = 0xFF, //PC / SC 2.02のAPDU用ラッパ

									  //ESC_CMD_GET_INFO
			DRIVER_VERSION = 0x01, //ドライババージョン(AA.BB.CC.DD)
			FW_VERSION = 0x02,
			VENDOR_ID = 0x04,
			VENDOR_NAME = 0x05,
			PRODUCT_ID = 0x06,
			PRODUCT_NAME = 0x07,
			PRODUCT_SERIAL_NUMBER = 0x08,
			CAPTURED_CARD_ID = 0x10, //0=UNCAPTURED, FF=Unknown
			NFC_DEP_ATR_REG_GENERAL_BYYES = 0x12,

			//----------APDU------------
			APDU_CLA_GENERIC = 0xFF,

			APDU_INS_GET_DATA = 0xCA,
			APDU_INS_READ_BINARY = 0xB0,
			APDU_INS_UPDATE_BINARY = 0xD6,
			APDU_INS_DATA_EXCHANGE = 0xFE,
			APDU_INS_SELECT_FILE = 0xA4,

			APDU_P2_NONE = 0x00,

			//----APDU_INS_GET_DATA----
			APDU_P1_GET_UID = 0x00,
			APDU_P1_GET_PMm = 0x01,
			APDU_P1_GET_CARD_ID = 0xF0,
			APDU_P1_GET_CARD_NAME = 0xF1,
			APDU_P1_GET_CARD_SPEED = 0xF2,
			APDU_P1_GET_CARD_TYPE = 0xF3,
			APDU_P1_GET_CARD_TYPE_NAME = 0xF4,
			APDU_P1_NFC_DEP_TARGET_STATE = 0xF9,

			APDU_LE_MAX_LENGTH = 0x00,

			UID_SIZE_ISO14443B = 4,
			UID_SIZE_PICOPASS = 8,
			UID_SIZE_NFCTYPE1 = 7,
			UID_SIZE_FELICA = 8,

			CARD_SPEED_106KBPS = 0x01,
			CARD_SPEED_212KBPS = 0x02,
			CARD_SPEED_424KBPS = 0x03,
			CARD_SPEED_848KBPS = 0x04,

			CARD_TYPE_UNKNOWN = 0x00,
			CARD_TYPE_ISO14443A = 0x01,
			CARD_TYPE_ISO14443B = 0x02,
			CARD_TYPE_PICOPASSB = 0x03,
			CARD_TYPE_FELICA = 0x04,
			CARD_TYPE_NFC_TYPE_1 = 0x05,
			CARD_TYPE_MIFARE_EC = 0x06,
			CARD_TYPE_ISO14443A_4A = 0x07,
			CARD_TYPE_ISO14443B_4B = 0x08,
			CARD_TYPE_TYPE_A_NFC_DEP = 0x09,
			CARD_TYPE_FELICA_NFC_DEP = 0x0A,

			//APDU_INS_READ_BINARY
			USE_BLOCKLIST = 0x80,
			NO_RFU = 0,

			//----------APDU_INS_DATA_EXCHANGE-----
			APDU_P1_THRU = 0x00,
			APDU_P1_DIRECT = 0x01,
			APDU_P1_NFC_DEP = 0x02,
			APDU_P1_DESELECT = 0xFD,
			APDU_P1_RELEASE = 0xFE,

			APDU_P2_TIMEOUT_AUTO = 0x00,
			APDU_P2_TIMEOUT_50MS = 0x05,
			APDU_P2_TIMEOUT_INFINITY = 0xFF,

			EXCHANGE_POLLING_PACKET_SIZE = 5,
			EXCHANGE_POLLING = 0x00,
			POLLING_REQUEST_SYSTEM_CODE = 0x01,
			POLLING_TIMESLOT_16 = 0x0F,

			//---------NDEF-----------
			NDEF_HTTP_WWW = 0x01,
			NDEF_HTTPS_WWW = 0x02,
			NDEF_HTTP = 0x03,
			NDEF_HTTPS = 0x04,

			//--------Felica-----
			SYSTEMCODE_ANY = 0xFFFF,
			SYSTEMCODE_FELICALITE = 0x88B4,
			SYSTEMCODE_NFC_TYPE3 = 0x12FC,

			ADDRESS_SPAD0 = 0x00,
			ADDRESS_SPAD1 = 0x01,
			ADDRESS_SPAD2 = 0x02,
			ADDRESS_SPAD3 = 0x03,
			ADDRESS_SPAD4 = 0x04,
			ADDRESS_SPAD5 = 0x05,
			ADDRESS_SPAD6 = 0x06,
			ADDRESS_SPAD7 = 0x07,
			ADDRESS_SPAD8 = 0x08,
			ADDRESS_SPAD9 = 0x09,
			ADDRESS_SPAD10 = 0x0A,
			ADDRESS_SPAD11 = 0x0B,
			ADDRESS_SPAD12 = 0x0C,
			ADDRESS_SPAD13 = 0x0D,
			ADDRESS_RC = 0x80,
			ADDRESS_MAC = 0x81,
			ADDRESS_ID = 0x82,
			ADDRESS_D_ID = 0x83,
			ADDRESS_SER_C = 0x84,
			ADDRESS_SYS_C = 0x85,
			ADDRESS_CKV = 0x86,
			ADDRESS_CK = 0x87,
			ADDRESS_MC = 0x88,
			ADDRESS_WCNT = 0x90,
			ADDRESS_MAC_A = 0x91,
			ADDRESS_STATE = 0x92,
			ADDRESS_CRC = 0xA0,
		};
```
##例外クラス
###PCSCErrorException
e.what()で日本語の詳細メッセージ
e.SW1()でSW1
e.SW1()でSW2

PC/SCのエラー応答．通常は発生しない．コマンド誤りか状態が異常
プログラムミスか機器故障を疑うこと

###PCSCCommandException
e.what()で日本語の詳細メッセージ
e.res()でSCardControl，SCardTransmitのリターンコード

PC/SCの関数エラー．まずめったに発生しない．(直接接続中にカード関係操作をした場合を除く)
プログラムミスか機器故障を疑うこと

###PCSCIllegalStateException
e.what()で日本語の詳細メッセージ

本来あるべきではない状態で実行しようとした．
プログラムミス

###PCSCFatalException
e.what()で日本語の詳細メッセージ
e.errorcode()で独自定義エラーコード
e.ret()でリターンコード

カードリーダーがない，カードが異常な応答を返した，PC/SC SERVICEに接続できないなど，
継続不能な，システム的あるいは物理的なランタイム問題

エラーコード

```cpp
		enum
		{
			FAILD_TO_ESTABLISH,
			NO_READERS_AVAILABLE,
			FAILED_TO_DETECT_READER,
			FAILED_TO_GET_CARD_STATUS,
			READER_DISCONNECTED,
			UNKNOWN_STATUS,
			UNKNOWN_ERROR,
			FAILED_TO_DIRECT_CONNECTION,
			CARD_HAS_LOCKED_BY_ANOTHER_PROGRAM,
			READER_NOT_FOUND,
			INVALID_READER_NAME,
		};

```

###PCSCCardRemovedException
e.what()で日本語の詳細メッセージ

カードがない，カードが途中で取り外された
カードがセットされていない状態でconnectCard()を呼び出した場合はかならず出る．

###PCSCCryptographicException
e.what()で日本語の詳細メッセージ

暗号化処理中にエラーが発生した．

###FelicaErrorException
e.what()で日本語の詳細メッセージ
e.high()，e.low()でFelicaエラーコード

Felicaカード側エラー．
書き込み禁止領域に書き込もうとしたり，認証しないで要認証領域に書き込もうとした場合など．
基本的にプログラムミス．SonyのFelica仕様書をよく読むこと．
エラーメッセージに詳細情報が書かれている

###FelicaFatalException
e.what()で日本語の詳細メッセージ
e.errorcode()で独自定義エラーコード

Felicaカード状態異常
1次発行されているカードに対して発行関係の処理をしようとしたなど

エラーコード

```cpp
		enum
		{
			ALREADY_FIRST_ISSUED,
			CARD_IS_NOT_FELICA_LITE,
			INVALID_RESPONSE,
			WCNT_CLIP
		};
```


