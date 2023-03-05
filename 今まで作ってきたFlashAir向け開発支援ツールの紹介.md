#今まで作ってきたFlashAir向け開発・検証ツールの紹介
Qiitaにいる電子工作ユーザー向けの宣伝を兼ねた何か

#Q. FlashAirってSDカードですよね？
A. 確かに

+ 汎用IOポート付きマイコンとして使え
+ 大容量ストレージ搭載のマイコンとして使え
+ 無線LAN機能を搭載し
+ そこそこ高速なWebサーバー、WebDAVサーバーを内蔵し
+ Webサーバーに搭載された機能から、SDカード領域やIOポートにアクセスし
+ Luaで作成したスクリプトが動作し
+ LuaスクリプトはスタンドアロンでもCGIとしても動作し、
+ SDカードの書き込みに連動した動作もし、
+ ビルトイン関数でFTPアップロードやHTTPSリクエスト、メール送信などの機能を揃え
+ I2CやSPIやUARTの通信機能も内蔵した

**SDカードです**

#FlashTools Lua Editor
FlashAirは、Luaスクリプトをテキストファイルとして、ポンと置くだけで動作します。

**が**、
Q. エラーメッセージはどこに出るん？
A.**でません**

Q. 文法エラーのときは？
A. **黙って落ちます**

というのが流石につらすぎるので、文法エラーや実行時エラーを確認でき、
またそのまま編集&修正&再実行までできるエディタを作りました。

FlashAirのWebサーバー機能を利用し、ブラウザ完結で動作するので、
FlashAir上に書き込んだあとはWi-Fi接続してブラウザを開くだけでOKです。
FlashAirの色々面倒な設定も自動セットアップする機能がついてます。
無限ループのスクリプトも途中で止めたりできるようになっています。(条件がありますが)
スマホ対応なので、スマホでも動きます。

導入マニュアルはこちら
http://seesaawiki.jp/flashair-dev/d/Lua%3a%b3%ab%c8%af%a4%ce%c1%b0%a4%cb...

<img src="http://image02.seesaawiki.jp/f/v/flashair-dev/deb18d1585500b62.gif" width=100%>

最近ファイラーも搭載しました。
<img src="https://qiita-image-store.s3.amazonaws.com/0/191114/52174f09-0c67-f6e7-5977-0cb44d145c4e.png">

#FlashTools GPIO Tester & Checker
FlashAirのIOポートに繋いだ機器の動作確認とかしたいとき、プログラム組んで云々は面倒くさい。
ので、スマホでポチポチHigh出したりLow出したり、入力チェックしたりできるツール作りました。

こちらもブラウザ完結です。
https://sites.google.com/site/gpsnmeajp/tools/flashair-io-tester

<img src="https://qiita-image-store.s3.amazonaws.com/0/191114/26916eb9-89a9-6ccd-e135-9a7bf045cc93.png" width=50%>

#FlashAir Unofficial Configurator
FlashAirの設定ファイルって、公式設定ツール以外にもいろいろな設定があります。
が、それを書くのが面倒なので、全部一気にできるツールを作りました。

が、ちょっと古いツールなので、最近追加された項目には対応していません。ご了承ください。
https://sites.google.com/site/gpsnmeajp/tools/flashair-unofficial-configurator

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/fe35d737-70a7-ec82-2d69-5a6c5f9461b1.png)

#FIH_Setup
[FlashAir IoT Hub](https://flashair-developers.com/ja/)という、IoT機器開発用のサービスが有るのですが、そのためには
ちょっと設定ファイルを書き換える必要があり、面倒そうだったのでそれ用の設定ツールを作成しました。

他にも、IoT Hub用のArduino向けサンプルなどがあります。
http://seesaawiki.jp/flashair-dev/d/IoTHub%b4%d8%cf%a2%a5%c4%a1%bc%a5%eb

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/de542728-b094-3452-af16-28264da86966.png)

#FlashTools Minimal Shell v0.01 beta
FlashAiw W-04のアップデートにより、シリアル通信機能が追加されました。
ので、シェルっぽいものを作ってみよう、と思い立って作ったのがこれです。

FlashAirをスタンドアロンで使うときや、自動アップロードスクリプトの開発中などは、
Wi-Fiが切断されたり使えなかったりするため、ブラウザ上からのデバッグは困難になります。

シリアルポートが使えるようになったのでそこからの通信ができるようになりましたが、
そこを開発するのも面倒かと思います。ので実装しました。

本ツールは、基本的な機能のみの実装になっていますが、Luaスクリプトで簡単に拡張できるのが売りです。
既存のスクリプトの実行もサポートしています。

http://gpsnmeajp.sblo.jp/article/181220245.html
<img src="http://sabowl.sakura.ne.jp/sblo_files/gpsnmeajp/image/FTMS.gif">

#FlashTools Serial Terminal v0.01
ブラウザからシリアルポート叩いてみようぜ！ということで。
一種のデモです。ブラウザからWi-Fiを経由してFlashAirのシリアルポートに繋がった機器と通信ができます。
無駄にArduino IDEのシリアルモニタ風です。
例によって、スマホからでも動きます。

仕様上、高速な通信をさせようとすると取りこぼしたりしますが、こういうこともできますよということで。
実はCGI機能を使用しておらず、起動時実行のLuaスクリプトとは共有メモリ機能を使って通信しています。

http://gpsnmeajp.sblo.jp/article/181245639.html
<img src="http://sabowl.sakura.ne.jp/sblo_files/gpsnmeajp/image/FTST.gif">

#他にも色々あります
スクリプトのサンプルや、開発時のハマりポイントなど

[FlashAir開発者向け非公式wiki](http://seesaawiki.jp/flashair-dev/)
http://seesaawiki.jp/flashair-dev/

をご参照ください。
