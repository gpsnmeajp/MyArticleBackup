#概要
端末同士をかざして通信、ってなんか未来感ありますよね。
Android端末自体にAndroid Beamという端末間の通信機能はありますが、色々と検討してみたのでそのメモです。

#注意
+ 実装したわけではありません。単なる検討です
+ 多分に誤りを含んでいる可能性があります。
+ NFCでの(特に容量の大きな)データ交換は、多くの場合非現実的です。
+ BLE等を使ったほうがより良い結果を得られる場合の方が多いでしょう

#手段
+ Android Beam (NFC-DEP, SNEP)
+ R/W + HCE(ISO-DEP)

#Android Beam
NFC規格の本来の目的のP2Pモードで通信する。お互いが対等に振る舞う。
NFC-DEP, LLCP, SNEPの3層を用いて通信する。

LLCPはTCP/UDPのような通信を提供し、SNEPはそれを用いてシンプルなNDEFメッセージの交換を行う。
Android Beamという名前はついているが、単なるURLの送信などの場合はSNEPを用いているため、
実はWindowsなどでも受信できる(PaSoRiで確認)

Android Beamはアプリから利用でき、OS管理下なので比較的容易に使うことができる。
アプリがなくてもデータ共有から送信・受信することができる。
**注意:ファイルの転送においては、BluetoothにハンドオーバーするBTSSPを用いている**

なお、NFC-DEPはAndroid Beamが掴んでいるため、アプリ独自に利用できないようである。
ちなみに通信には、高速なType-Fが優先して使われ、対応していない機器はType-Aなどで接続される。

参考: https://hiro99ma.blogspot.jp/2012/07/android-beam.html
http://lnovel.diary.to/archives/65844081.html
http://www.atmarkit.co.jp/ait/articles/1301/07/news014_2.html


#R/W + HCE
NFC技術の本来の目的のカード間モードで通信する。
片方がカードとして振る舞い、片方がカードリーダーとして振る舞う。

ISO-DEP(or FeliCa)による通信となり、一定の定められた手順以降は自由な手順で通信できる。
当然ながら、きちんと実装すればAndroid以外とも通信できる。

互換性や利便性を求めるならHCE(Android 4.4以降)を。
利便性を捨てて速度を求めるならばHCE-F(Android 7以降のみ)を。

読み取る側も、Reader Mode(Android 4.4以降)を使う必要がある。
(使わないとAndroid Beamで通信を確立しようとしてしまうため)

HCEを使った場合、ロック画面でも通信できるのが利点。

参考: http://tongarism.com/2700
