#はじめに
USBデバイスを作ってみたくなりました。

今まではFTDI社のFT232やFT245などを使ってきましたが、より簡単に繋いだり、
マイコンと直結させたりしたいといった欲求を満たすには、USBを勉強する必要があります。

ので、まず、日本では比較的ポピュラーなmicrochip社のPIC 18FシリーズのUSB内蔵品を使って、
USBの世界に触れてみようと考えました。

##注意
+ USBの物理層には触れません
+ USBの接続に係る部分は勉強せず、フレームワークを使用します
+ とりあえずベンダークラスデバイスで、libusb1.0を使ってPCと通信できるのを目標とします。

#今回やること
+ PICの開発環境を導入します
+ USBフレームワークのデモを書き込みます
+ HID mouseデモを実行し、マウスがグルグル回るさまを眺めて終わりです。

#準備するもの
マイコンの開発のため、物理的なデバイスが必要となります。
##マイクロチップ　ＰＩＣｋｉｔ３(￥4,980)
PICの書き込みに使用します。
18F14K50へ書込み可能な他のICSPライターで代用しても構いません。

http://akizukidenshi.com/catalog/g/gM-03608
秋月電子通商で購入できます。
<img src="https://qiita-image-store.s3.amazonaws.com/0/191114/4fa2c248-3f9c-abe5-2a08-0ac5991b88a1.jpeg" width=50%>
引用元: 秋月電子通商

##ＰＩＣ１８Ｆ１４Ｋ５０使用ＵＳＢ対応超小型マイコンボード(￥800)
USB端子・書き込み端子が揃っており、公式サンプルがあります。
ピンヘッダも入っています。USB機器開発を始めるにはうってつけです。

http://akizukidenshi.com/catalog/g/gK-05499/
秋月電子通商で購入できます。

![K-05499.jpg](https://qiita-image-store.s3.amazonaws.com/0/191114/c77bc781-149e-07b2-69d8-aec35cb506f7.jpeg)
引用元: 秋月電子通商

##ハンダこて・ハンダ
適時購入してください

##Windows PC
Windows 10 64bitで動作を確認しています。

#開発環境の導入
##MPLAB X
統合開発環境です。アセンブラが同梱されています。

こちらからダウンロードし、インストールしてください
http://www.microchip.com/mplab/mplab-x-ide
MPLAB® X IDE v4.01で動作を確認しています。

##XC8 Compiler
コンパイラです。USBブートローダーが使えないことに目をつぶれば、
最適化なしのフリー版で十分です。

こちらからダウンロードし、インストールしてください
XC8,XC16,XC32がありますが、XC8だけで十分です。
http://www.microchip.com/mplab/compilers
MPLAB® XC8 Compiler v1.43で動作を確認しています。

##Microchip Libraries for Applications
これがUSBフレームワークとデモになります。

こちらからダウンロードし、インストールしてください
http://www.microchip.com/mplab/microchip-libraries-for-applications
Microchip Libraries for Applications(v2017-03-06)　Windows版で動作を確認しています。

#組み立て
マイコンですので、組み立てが必要です。
このようにピンヘッダをはんだ付けしてください。

今回、回路は組み立てず、USB機能だけ使うので、書き込み用ピンヘッダのみでも構いません。
![A.jpg](https://qiita-image-store.s3.amazonaws.com/0/191114/0e1358ad-537b-cb1e-f404-58ab4b9d7acb.jpeg)

#デモを書き込もう
##MPLAB Xを起動する
スタートメニューから起動してください。

コマンドプロンプトが立ち上がった後、JAVA製のIDEが立ち上がります。
中身はNetBeansらしいです。そのため、NetBeansプラグインが使えるようです。

##デモプロジェクトを開く
File→Open Projectをクリックし、開いたダイアログで以下のパスを開く。

```
C:\microchip\mla\v2017_03_06\apps\usb\device\hid_mouse\firmware
```

そして、以下のプロジェクトを読み込む。

```
low_pin_count_usb_development_kit_pic18f14k50.x
```

##書き込みツールの切り替え
USB Device - HID - Mouse　と書かれたプロジェクトを右クリックしてpropertiesを開く。
開いた画面のメニューの「Conf: [LPCUSBDK_18F14K50]」をクリック。
初期設定では「Hardware Tools」にて「ICD3」が選択されていますが、「PICkit3」をクリックし、OKを押します。
これで書き込みツールが切り替わります。

※OKがクリックできず、Cancelしか選べないときはIDEの調子が悪いことが多いため、
　IDEを起動し直し、プロジェクトを開き直すと行ける場合があります。

![N.png](https://qiita-image-store.s3.amazonaws.com/0/191114/d824aa0d-5cb7-12fb-c98f-c817acc1883c.png)

##書き込み時電源供給の設定
今回は、18F14K50基板を単体で(電源供給すること無く)書き込むため、その設定も行います。
まず、再度プロジェクトのpropertiesを開きます。

先程までICD3が表示されていた左メニューがPICKit3になっていますので、クリック。
![M.png](https://qiita-image-store.s3.amazonaws.com/0/191114/a410585f-9b19-1ff1-a252-20f47f143071.png)

そして、画面上の「Option categories：Memories to Program」をクリックし、「Power」に変更。
「Power target circuit from PICKit3」のチェックボックスをオンにし、OKをクリックします。
これにより、書き込み時にPICKit3から電源供給がされるようになります。
「Voltage Level」は3.0～5.0の間であればOKです。

![O.png](https://qiita-image-store.s3.amazonaws.com/0/191114/6276096f-70aa-0ae1-d613-25880f976a85.png)

##ビルドと書き込み
PICKit3に基板を差し込み、PICKit3自体もPCに接続します。
基板は図の向き、位置に差し込んでください。

![B.jpg](https://qiita-image-store.s3.amazonaws.com/0/191114/04cb3070-616c-a5ba-71df-6c22aa1faed0.jpeg)

そして、Make and Programボタンを押します。
![P.png](https://qiita-image-store.s3.amazonaws.com/0/191114/b4f63c58-5a7a-ccc2-c3bf-b38e77c6ff1f.png)

このような画面が出たら、PICkit3の下にぶら下がっているSNから続く機器を選択してください。

![Q.png](https://qiita-image-store.s3.amazonaws.com/0/191114/54661d44-3914-198a-3a50-539c13143855.png)

本当に電圧掛けていいのか？と聞かれますのでOKをクリック。
![R.png](https://qiita-image-store.s3.amazonaws.com/0/191114/24033c0f-9610-7f0a-0151-668ca1e4a9e8.png)

あとはしばらく待てば、書き込みが完了します。
Output欄にPrograming/Verify completeの表示が出れば完了です。

![S.png](https://qiita-image-store.s3.amazonaws.com/0/191114/d149603f-a521-f423-b5b4-c7d8e9700d83.png)

#デモのテスト
PICKit3から基板を引き抜き、USBケーブルを接続してください。
PICKit3同梱のminiBケーブルでOKです。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/5ffe0d0e-31b7-9731-4840-332352af7d04.png)

しっかり奥まで差し込むと、PICKit3のときより煌々と明るく電源LEDが光り始め、PCにマウスとして認識されるはずです。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/e7ae45bd-680e-24ee-7af0-4cb43083997e.png)

こんな感じでひとりでにマウスがぐるぐる回れば成功です。
ただし、このデモは本来デモ基板用のデータなので、スイッチがつながっている前提らしい挙動をします。
基板の足に適当に触ると、挙動が変わったり止まったりします。

![output.gif](https://qiita-image-store.s3.amazonaws.com/0/191114/f5308f8e-f75e-cdc1-4e04-55c0b25f62c7.gif)

#おわりに
今回は、PIC 18F14K50を使ったUSB機能のデモを動かすまでを紹介しました。
次はベンダークラスでバルク転送するデモを動かした後、
簡略化したプログラムを構築する予定です

[次回はこちら](https://qiita.com/gpsnmeajp/items/b2c8d9521c9eab7208b7)

#疑問
バッファにデータを入れないまま、送信要求を受けるとどうなるのだろう。
NAKで待たされる？あっ、バルク転送タイムアウト？それとも0バイトが帰る？
サンプルの書き方的には0バイトが帰ってるっぽいかな。

#参考文献
###現状のフレームワークの読み解き方、必要なファイルの選別
PICでUSBを動かす（その２） loops/ウェブリブログ 
http://loops.at.webry.info/201404/article_3.html

###18F14K50全般の使い方
はじめてのPIC 18F14K50
http://sky.geocities.jp/home_iwamoto/page/P14K50/P14_00.htm

###日本語データシート
http://ww1.microchip.com/downloads/jp/DeviceDoc/41350D_JP.pdf
