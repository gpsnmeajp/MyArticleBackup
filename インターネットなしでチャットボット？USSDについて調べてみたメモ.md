# はじめに
USSD(Unstructured Supplementary Service Data)は、たまに海外の(特に発展途上国の)携帯電話事情に登場します。
一方で、日本でも実は使われている技術です。

簡単に言うと、インターネット環境のない地域で、携帯電話ネットワークだけで利用者にチャットボットを提供できるような仕組みのようです。

しかし、日本語が使用できないこと、iモードなどですでに通ってきた道のためか、日本では表立って使われることはなく、それゆえに日本語でまとまった資料を見かけないため、まとめてみることにしました。

# 留意事項
私は、携帯電話に関する開発の経験はありません。
そのため、以下の情報は参考文献から集めてきた情報を、想像でまとめているだけです。
正確な情報は参考文献を参照してください。

正確性の保証はなく、誤解している内容も含まれます。
間違いがありましたらコメントにてお知らせいただけると幸いです。

# USSDとはなにか
USSDは、携帯電話ネットワークを使った通信プロトコルの一つです。
SMSの親戚にあたります。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/d726c2be-947e-bfc5-7781-d14c1d9534ef.png)

# なぜ最近流行り始めたのか
アフリカ・アジア地域で、通信事業者がゲートウェイ業者にAPIを開放したため、クラウド上で動くプログラムと、非インターネット接続の携帯電話端末との対話に利用できるようになったため。

これにより、インターネットが利用できないが、携帯電話は普及している地域の携帯電話(低コストな中国製フィーチャーフォンから、スマートフォンまで)に対し、様々なテキストベースのアプリケーションを提供できるようになったようです。
(例えばアフリカでは、インターネットの普及率は18%に対し、携帯電話の普及率は80%を超えるとのこと)

店頭での電子支払いシステムとしても使用されるようです。(QRコードの代わりに、特定の番号を打ち込むことで支払いする)

USSDそのものは非常に古いシステムですが、これらは2013年頃から本格的に普及が始まったようです。

参考: 

https://www.digitalvirgo.com/mobile-payment/ussd/

https://www.ttc.or.jp/maedablog/20141128

https://qz.com/africa/1296120/how-a-20-year-old-mobile-technology-protocol-is-revolutionizing-africa/

https://thinkit.co.jp/article/12148

https://www.amda-minds.org/teabreak20171122/

https://twitter.com/mastercardmea/status/1068973009384304640

# SMSとUSSD
まず、どちらもGSM対応の携帯電話で利用可能なものです。
3Gでも利用可能です。LTEではUSSD over IMS, SMS over IMSで利用可能なようです。(SIPでラップされるようです)

2つの違いについて

### USSD (非構造化補助サービスデータ)
+ モバイル端末を起点に、対話的かつ同期的に通信します。
+ 端末からセッションを張り、USSDアプリケーションを起動してメッセージで対話します。
+ 同期的な通信のため、ストアアンドフォワード(センターに一時的に蓄える)機能は持ちません。(SMSセンターを経由しません)
+ 英数字(GSM 7bit)しか送受信できません。(UCS2:Unicodeに対応する実装もあるようですが)
+ 携帯端末利用者と、事業者側システムの対話に使用します。
+ 同期セッションのためタイムアウトが発生します。(30秒?)

※USSD Pullの場合。USSD Pushという通信事業者側からプッシュで開始するパターンも存在するが、その強力性から限られたシチュエーションでしか使用されていない。Flash SMS(Class 0 SMS)もUSSD Pushと似たような特徴を持つが、USSD Pushは対話可能でFlash SMSは対話不可。

参考: 

https://www.alwaysactivemobile.co.za/ussd-push/

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/46528f87-f939-f2df-43a6-49bd1f7d1cc9.png)

### SMS (ショートメッセージサービス)
+ 端末・ネットワークの両方を起点に非同期的に通信します。
+ セッションはなく、非対話的です。
+ 非同期的な通信であり、ストアアンドフォワード(センターに一時的に蓄える)機能を持ちます。(SMSセンターを経由します)
+ 英数字(GSM 7bit)、Unicode(UCS2)、バイナリを送受信できます。
+ 携帯端末利用者同士のメッセージングに使用しますが、事業者側システムの対話にも使用することがあります。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/d7a94892-ff48-41e3-b727-6aa778b91680.png)

なお、これらは組み合わせて使うことがあります。
(USSDで要求・設定したあと、通知や結果がSMSで飛んでくるなど)

参考: 

https://docs.microsoft.com/ja-jp/windows-hardware/drivers/network/mb-ussd-overview

# USSDコード
USSDは、人間から見ると
> \*#{コマンド}#
> 例: \*#06#  → IMEIの表示
> \*{コマンド}#
> 例: \*000# 
> \*{コマンド}\*{データ}# 
> 例: \*00\*111#

や 


のような形で呼び出します。

例えば電話番号の確認や、IMEIの確認、プリペイド残データ容量の確認などでこの番号を目にすることが多いはずです。

https://tabinavi-world.com/asia/thailand/ais-ussd/

特に、海外SIMで目にする機会が多いですが、
日本国内でもドコモの「留守番電話サービスの開始」や「迷惑電話ストップサービス」に使う「サービスコードによる操作」という形でUSSDコードによる操作を見ることができます。

USSDによる操作は、電話番号による操作で音声ガイダンスが帰るのとは違い、応答が文字列で帰り、ユーザーに提示されます。

このコマンド列はUSSDコードと呼ぶことがありますが、実際にはMMIコード(マンマシンインターフェースコード)と呼ばれるものです。

MMIコードは種類が分かれています。
+ USSDコード: 端末が解釈できず、携帯電話ネットワーク上にあるゲートウェイあるいはUSSDアプリケーションが応答を返すもの(プリペイドカード残高の確認など)
+ SSコード: 端末が解釈し、携帯電話ネットワークに要求するもの(着信転送など)
+ 端末固有のMMIコード: 端末が解釈し、端末が特殊な動作をするもの(IMEIの確認、データの全消去など)
+ SIM制御コード: 端末が解釈し、SIMカードを操作するコード(PINの変更など)

参考: 

https://www.techtarget.com/searchnetworking/definition/USSD

https://berlin.ccc.de/~tobias/mmi-ussd-ss-codes-explained.html

# USSDの利点
USSDの利点は、

+ GSM/3G/4Gの携帯電話であればほぼすべての端末で利用可能
+ インターネット接続が必要ない(データ通信やパケット通信契約が不要、別枠での課金)
+ アプリケーションのダウンロードが不要
+ ユーザーとの対話ができる
+ メッセージが残らない(保存されず揮発する)
+ 安価な場合が多い

という点です。

従来、USSDアプリケーションは通信事業者だけが実装できるものでしたが、近年はオープン化され、通信事業者からインターネット経由でクラウドサービスのAPIを呼び出しできるようになったようです。

これにより、利用者側がチープな携帯電話(非インターネット接続)を利用していても(あるいはスマートフォンであっても平等に)、USSDによるインターフェースを提供しているインターネット上のアプリケーションを利用できることになります。
(これにより、専用USSDコード、共有USSDコードなどの概念もあるようです)

さらに、USSDは、最初のアクセスこそ無味乾燥なコード(\*AAAA\*BBBB#)ですが、その後は英字の送受信でユーザーとアプリケーションが対話できます。

参考: 

https://qz.com/africa/1296120/how-a-20-year-old-mobile-technology-protocol-is-revolutionizing-africa/

# USSDはどんなところに使われているのか

+ プリペイドの残高確認
+ 電話転送などの設定
+ オンラインバンキング
+ その他アプリケーションの実現(不妊治療、個人間送金、保険、テキストチャット、Facebook、Twitter、その他)

馴染みがないものも多いと思いますが、それは殆どの機能が先進国ではインターネットやリッチな端末アプリケーションで実現されているためです。

USSDは特にアジアやアフリカ地域でよく使われているものです。これは、インターネット不要と端末非依存の性質に依ります。

※逆にテキストベースのUSSDに限界を感じている顧客向けに、WhatAppなどAIチャットボットに移行するソリューションを提供している業者もあります。
　また、USSDはSIM IDによる認証となっている場合が多く、セキュリティに問題があるとして別手段に移行する事業者もいるようです。

# SIM Tool Kit(STK)との連携と、SIM Overlay
GSM/3G/4Gの携帯電話には、SIM Toolkitが内蔵されています。

フィーチャーフォンはもちろん、Androidであればアプリとして、iPhoneは設定→モバイル通信→SIM Applicationにあります。
(一部端末は対応していません。また、日本では通常目にする機会がありません。SIMカード内にアプリケーションが入っていない場合そもそもユーザーから見えないようになっているためです。)

このToolkitは、SIMカード上に書き込まれた複数のアプリを呼び出して処理ができるようにするための、いわばメニューとランチャーとしての機能を持ちます。
(SIMカードは独立したOSとJavaプログラムを持っているスマートカードで、そことAPDUで対話することができます。SIMカード自体に暗号化や署名などの機能も搭載されています)

このSIMカード上に書き込まれたアプリケーションは、様々なことができます。SMSの送受信や、USSDコマンドの送信、電話発信、音を鳴らす、ブラウザの起動、暗号化、署名検証など。

そのため、USSDやSMSを用いて銀行等と通信するアプリをSIMカードに書き込んでおくことで、ユーザーが番号を手打ちすることなく通信を実施することができるようになります。
(STKの通信は、USSDやSMSではなく、PINや鍵によって暗号化されたチャネルによる通信を実施する場合があります。)

ただし、このSIMカード内のアプリは、SIMカードの出荷時に書き込まれているもので、基本的に書き換えができません。(eSIMだとできるんでしょうか？)

そのため、追加のアプリケーションを提供したい各社は、SIM OverlayというSIMカードの上に貼り付けて信号をバイパスしながら追加の機能を提供するものを配布しているようです。

日本では、「貼るSIM」「サブSIM」「変なSIM」として提供されているものと似ています。
これも通信回線の切り替えのためにSIM Toolkitを使用したり、暗号化や署名に「SIMカードに貼り付けたSIMカード」の機能を使用します。

参考: 

https://blog.kaspersky.co.jp/simjacker-sim-espionage/24282/

https://ginnoslab.org/2019/09/27/stattack-vulnerability-in-st-sim-browser-can-let-attackers-globally-take-control-of-hundreds-of-millions-of-the-victim-mobile-phones-worldwide-to-make-a-phone-call-send-sms-to-any-phone-numbers/

https://qiita.com/staybuzz/items/1598bcd10acb592f2a6e

https://xtech.nikkei.com/atcl/nxt/column/18/01113/042800030/

# USSDに触れる
USSDをまともに使用できる(&需要がある)のは、アジアやアフリカ地域のため、そのエリアの携帯電話会社と契約し、USSDゲートウェイを利用することでアプリ開発ができそうです。

また、SORACOMがUSSDの受信・応答に対応している(対話には非対応？)ため、こちらを利用することでも雰囲気は感じられるかもしれません。

https://qiita.com/j3tm0t0/items/4cc271aa65063354ba88

参考: 

https://africastalking.com/ussd

https://blog.soracom.com/ja-jp/2018/07/04/beam-ussd-support/

https://users.soracom.io/ja-jp/docs/air/send-ussd/


# 参考文献
## この記事の内容の大本となるもの
https://www.techtarget.com/searchnetworking/definition/USSD

https://www.ttc.or.jp/maedablog/20141128

https://en.wikipedia.org/wiki/Unstructured_Supplementary_Service_Data

https://docs.microsoft.com/ja-jp/windows-hardware/drivers/network/mb-ussd-overview

https://berlin.ccc.de/~tobias/mmi-ussd-ss-codes-explained.html

https://qz.com/africa/1296120/how-a-20-year-old-mobile-technology-protocol-is-revolutionizing-africa/

https://thinkit.co.jp/article/12148

https://blog.kaspersky.co.jp/simjacker-sim-espionage/24282/

https://ginnoslab.org/2019/09/27/stattack-vulnerability-in-st-sim-browser-can-let-attackers-globally-take-control-of-hundreds-of-millions-of-the-victim-mobile-phones-worldwide-to-make-a-phone-call-send-sms-to-any-phone-numbers/

https://qiita.com/staybuzz/items/1598bcd10acb592f2a6e

https://xtech.nikkei.com/atcl/nxt/column/18/01113/042800030/

https://k-tai.watch.impress.co.jp/docs/news/749640.html

https://xtech.nikkei.com/atcl/nxt/column/18/00138/082900133/

https://blog.soracom.com/ja-jp/2018/07/04/beam-ussd-support/

https://users.soracom.io/ja-jp/docs/air/send-ussd/

https://www.alwaysactivemobile.co.za/ussd-push/

https://www.amda-minds.org/teabreak20171122/

## その他補足情報

https://cheerio-the-bear.hatenablog.com/entry/2019/02/11/152926J

https://blog.kaspersky.co.jp/sim-card-history/9934/

https://africastalking.com/ussd

https://developers.africastalking.com/tutorials/implementing-a-mobile-money-ussd-menu-with-branded-sms

https://developers.africastalking.com/tutorials/implementing-a-mobile-money-ussd-menu-with-branded-sms

https://en.wikipedia.org/wiki/SIM_Application_Toolkit

https://encyclopedia.kaspersky.com/glossary/ussd-unstructured-supplementary-service-data/

https://users.soracom.io/ja-jp/docs/air/send-ussd/

https://jetstream.bz/archives/72206

https://en.m.wikipedia.org/wiki/M-Pesa

https://seamless.se/blogs/the-power-of-ussd/

https://hyperms.com/ussd-stk-solutions/

https://www.cspsprotocol.com/ussd-in-lte/

https://daily.berrymobile.jp/all/2020/12/12/8071/

https://www.twilio.com/docs/glossary/what-is-ucs-2-character-encoding

https://www.twilio.com/docs/glossary/what-is-gsm-7-character-encoding

https://www.twilio.com/blog/communication-humanitarian-operations-ussd-flex

https://en.wikipedia.org/wiki/USSD_Gateway

https://lastinvention.co.za/ussd2chatapp/

https://medium.com/biya-focus/financial-services-ai-chatbots-vs-ussd-b0c2b2c24e20

https://tabinavi-world.com/asia/thailand/ais-ussd/

https://sgrmatha.hatenablog.com/entry/20141128/1417189172
