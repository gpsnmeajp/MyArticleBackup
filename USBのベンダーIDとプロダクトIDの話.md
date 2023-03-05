ここには誤った情報が含まれる場合があります。
製品開発などの際には、専門家の意見を参照してください。

# 前提
USB規格には、製品の識別のためにベンダーIDとプロダクトIDがありますが、
テスト用のベンダーIDなどはありません。

同じベンダーIDとプロダクトIDの違う製品が混在すると、それを識別に使っているドライバなどが混乱し、
最悪の場合PCがクラッシュしたりするため、USB-IFがベンダーIDを管理しています。

usb.orgに行くことで、有効/無効なベンダーID一覧を見ることができます。

https://www.usb.org/developers

# 正規の方法
ベンダーIDを入手するには、法人格を持ち(会社を持ち)、USB-IFに申請します。
ベンダーIDのみの入手であれば昔は2万円の支払い、現在は49万円の支払いで、ベンダーIDが入手できます。
1つのベンダーIDの入手で、6万種類の製品の作成ができます。

# [0x04D8]Microchip社のサブライセンスを受ける(推奨)
PICやAVR、USBコントローラを販売しているMicrochip社は、自社のベンダーIDのプロダクトIDを無償でサブライセンスしています。
申請には法人格が必要です。
また、生産数1万ユニットまでの制限があり、それ以上使用する場合は、このサブライセンスによるプロダクトIDを使用できなくなります。

利用の際には、必ずMicrochip社製のマイクロコントローラである必要があり、それ以外で使用すると法的措置を行われる可能性があります。

参照: MicrochipはUSBのVID/PIDに関してサブライセンスを行っていると聞きました。どのように申請すればよいですか？
http://www.microchip.co.jp/support/faq_usb.html

# [0x1FC9]NXP USB VID/PID Programによるサブライセンスを受ける
LPC, Kinetis, I.MXを使用している場合はNXP社の無償サブライセンスを受けることができます。
生産数1万ユニットまでの制限があり、最大3つまでのPIDが無償で取得できます。

参照: NXP USB VID/PID Program
https://community.nxp.com/t5/Kinetis-Microcontrollers/NXP-USB-VID-PID-Program/ta-p/1124867

# [0x16C0]Commercial Licenses for V-USBによるサブライセンスを受ける
AVR用のUSBドライバソフトウェアのV-USBの商用ライセンス版(有料)には、GPLの回避のほか、「無料で」プロダクトIDが付属します。
https://www.obdev.at/products/vusb/license.html

Hobby Licenseでは、非営利目的での5つまでの製品の製造が許されており、約1400円
Entry Level Licenseでは、150製品までの製造が許されており、約2.7万円
Professional Licenseでは、小規模生産用であり、10000製品までの製造が許されており、約8万円(プロダクトIDは2つ付属します)
それを超えた場合は1ユニットあたり0.1ユーロの追加が必要です。

ざっと見た限り、Microchip社のような、V-USBを使用する場合に限るといった、VIDの利用用途制限は無いようです。

そもそも、Hobby Licenseに
>Primarily for people who originally intended to use the free license, but find it too tedious to prepare the publication, or people who need a dedicated USB Product ID.

と書いてあるため、そういうことなのだと思われる。

ちなみに、サブライセンスを受けたVIDではUSBロゴ認証は通らないが、
そもそもV-USBそのものが電気的に仕様違反のため、もともとUSBロゴ認証は通らない、といったようなことが
商用ライセンスの説明に記述されている。

# [0x16C0]V-USBの無償共有USB-IDを使用する
いくつかの技術的な使い方・情報提示を守れば、V-USBのVIDの一部を無償で使えます。

libusbを使ったベンダークラスデバイス
　テキスト名識別用: VID=0x16C0 PID=0x05DC
シリアル番号識別用: VID=0x16C0 PID=0x27D8

汎用HIDクラスデバイス
　テキスト名識別用: VID=0x16C0 PID=0x05DF
シリアル番号識別用: VID=0x16C0 PID=0x27D9

CDCデバイス
　テキスト名識別用: VID=0x16C0 PID=0x05E1
シリアル番号識別用: VID=0x16C0 PID=0x27DD

MIDIデバイス
　テキスト名識別用: VID=0x16C0 PID=0x05E4
シリアル番号識別用: VID=0x16C0 PID=0x27DE

マウス
シリアル番号識別用: VID=0x16C0 PID=0x27DA

キーボード
シリアル番号識別用: VID=0x16C0 PID=0x27DB

ジョイスティック
シリアル番号識別用: VID=0x16C0 PID=0x27DC

詳細は以下を参照してください。
https://github.com/obdev/v-usb/blob/master/usbdrv/USB-IDs-for-free.txt

# [0x1D50]Openmoko Incによるサブライセンスを受ける
オープンソース携帯電話の制作のための企業Openmoko Incは、そのプロジェクトの中止に合わせ、
オープンソースコミュニティ製のUSB機器用のプロダクトIDのサブライセンスを行っています。

オープンソースのソフトウェア・ハードウェアであれば、個人でも利用申請できるようです。
PIC16F1454シリーズ用のブートローダーなどにも割当たっています。
(性質上、販売なしのソフトウェアオンリーのプロジェクトだと思われます。)

IRkitやBLEduinoなどがこれを利用しているようです。
申請済みのリストはリンク先で見ることができます。

Open registry for community / homebrew USB Product IDs
http://wiki.openmoko.org/wiki/USB_Product_IDs

# [0x1209]pid.codesによるサブライセンスを受ける
同じく、オープンソースハードウェア・ソフトウェアのためのプロダクトIDのサブライセンスです。
GitHubにてプルリクエストをすることで申請することができます。

ここもサブライセンス禁止条項ができる前にベンダーIDを取得した組織です。

VID=0x1209　VDI=0x0001は非公開での開発・テスト用のIDとして利用できるようです。
販売・製造・再配布されるデバイスへの利用は禁止されています。(混乱防止のため)
ソースコードに同梱しても良いですが、その場合は同様の警告文を同梱する必要があります。

http://pid.codes/

申請済みのプロダクトIDリストはここから見ることができます。
http://pid.codes/1209/

主張
>pid.codes and the people behind it have never signed an agreement with USB-IF not to reassign or redistribute PID codes.The VID we were gifted was procured from USB-IF by a company that has since ceased trading, and they did so before USB-IF’s terms prohibited sublicense or transferring of VIDs or PIDs.
>It is our belief that USB-IF has no legitimate right to prohibit this activity, and that their actions are limited to ‘revoking’ the original VID, a fairly meaningless pronouncement since they can never reassign it to anyone else. Nevertheless, we hope they will not do so, and will instead choose to work with us to make creating and distributing USB devices more accessible for hobbyists, makers, and small businesses.

概要

+ 当社はUSB-IFと契約を結んだことは一度もない。当然譲渡禁止などの契約もない。
+ 現在利用しているVIDは、USB-IFが転売・譲渡を禁止する前に、他者から調達したものである。
+ USB-IFにはこの活動を禁止する正当な権利はない。
+ できることといえばVIDの取り消しであるが、結局その後再割当てができなくなる為、その行為は無意味である。
+ まあ、そうしないでくれると助かるけどね

# [0x16D0]MCS Electronicsから購入する(非推奨)
現在ではプロダクトIDの販売はUSB-IFにより禁止されていますが、2009年までは禁止されていませんでした。
そのため、MCS Electronics(AVRのBASIC開発環境BASCOM-AVRの販売元)は自社のプロダクトIDを販売していました。

約2000円で1つのプロダクトIDを購入することができます。
https://www.mcselec.com/index.php?page=shop.product_details&flypage=shop.flypage&product_id=92&option=com_phpshop&Itemid=1

ただし、商品ページにも記載されているように、USB-IFはMCS ElectronicsのベンダーIDを一方的に取り消しました。
そのため、販売されているベンダーIDとプロダクトIDはもはや有効なものではありません。

ただし、後述する理由を踏まえると有効性はないわけではありません。
(法的な問題を除きます)


主張

>That changed in June 2009 when USB-ORG wrote an email that it was not permitted to sell VID or PID.
MCS pointed out that in The Netherlands you can not enforce or change rules AFTER a product/service is sold. 
So we continued as usual. 
In reaction USB-ORG offered our money back but that would mean existing customers would not have a valid PID anymore. 
So we got some more letters and finally USB-ORG wrote a letter that they revoked our VID.

>Of course our VID can not be re-used by USB-ORG. This means that our VID will still be unique.

概要

+ 本サイトでは2005年以来販売をしている。当時、転売・販売禁止条項はなかった。
+ 2009年にUSBORGが勝手に突然規約を変更し、転売を禁止した。
+ オランダの法では、販売されたものの権利をあとから変更することはできないと主張したところ、USBORGは一方的に返金してきた。
+ さらにVID一方的には取り消された。
+ しかしUSBORGはこのVIDを再割り当てすることはもはやできないのは明白である。
+ IDのユニーク性の意味しか(元々)ないが、それが欲しい人々のために販売は継続している

# [0xF055]勝手に使用する(非推奨)
USBでベンダーIDとプロダクトIDをユニークなものにする理由は、誤動作の防止の一点です。
これは仕様であり、強制力を持ちません。

製造元などはディスクリプタに記載することができます。
また、ベンダーIDの利用とロゴの利用は別の問題になります。

そのため、既存の製品のIDと被らないのであれば、勝手に使用することを止めることはできません。
単なる番号列に法的な管理が及ぶとも思えません。

そもそもUSBロゴ認証を受けられない(=電気的・プロトコル的に等価と保障されない)のであるから、
USB端子と互換はあり、たまたま動くがUSBではないと主張することも不可能ではない。
(ただし、使用料が無償なだけで、USBに関する特許は存在するため注意)

その発想で行われたのが、VID=0xF055です。
このVIDをフリーオープンソースなハードウェアで勝手に使ってしまおうというものです。
https://f055.io/

すでに利用されているベンダーIDを、USB-IFが割り当てるとそれは単に混乱しか引き起こさないため、
それを嫌って、あえてかぶせることはしないだろうという打算で行われたものです。

MCS Electronicsが販売を続けているのもそれで、一旦発行されたベンダーIDのプロダクトIDをどう使うかは自由であり、
それはUSB-IFの管理の外であるため、一度利用が始まったベンダーIDを取り消しても、
他者に割り当てるとそれは消費者の混乱を引き起こすため、事実上再割当ては不可能という理論です。

事実、MicroPythonなどのプロジェクトや、その他オンライン上のオープンソースなプロジェクトではベンダーIDを0xF055にするのをよく見られます。

ただし、被ったベンダーIDの所有企業との訴訟問題や、USB-IFとの訴訟問題については別になるため、
決して勧められる方法ではありません。

ただ、個人でUSB機器を開発する場合(販売をしない場合)は適当なIDを勝手に利用する他ありません。

小規模な販売をしたい場合は、チップメーカーのサブライセンスを検討することをおすすめします。
オープンなプロジェクトでそのまま利用できるようにしたい場合は、V-USBなどを使用するほうが良いかもしれません。
大規模な販売を検討している場合はベンダーIDの取得を考えてください。

# USBを使用しない
FTDI等のチップを使ったそれなりの通信で我慢する。  
これだけでかなりの面倒ごとから開放されます。

あるいは、~~Wi-Fi~~ 無線LAN、Ethernet、~~Bluetooth~~ などのこういった問題のない規格を使用する。

※無線機器の開発にはまず、技適の取得が問題になってきます。(認可済みモジュールを使用する場合除く)

# 追記:Bluetoothについて
BluetoothはBluetoothで、Bluetooth SIGに(最低)100万円～数千万円ほど払って審査・製品登録しないといけないようです。しないと催促と罰金、および訴訟が行われる模様。
この費用は1製品種別ごとにかかります。

USBよりも厳しく、製品にモジュールを組み込むだけでもこの費用は発生するようです。  
自分で無線回路を作る場合や、認可なしICを使用する場合は飛躍的に費用が増えます。

同人だと無理ですね。無線搭載せずに自分でUSBドングルとか繋いでもらうしかなさそうです。

# 追記: Ethernet, WLANの場合
MACアドレスが必要です。  
UAA(世界で唯一となるMACアドレス)をIEEEから取得する場合、約4000個の製品が作れるもので最低約10万円ほど(805ドル)掛かります。

なお、1個単位でMACアドレスが書き込まれたものが売っていることもあるのでロット数と相談です。

特定のビットを立てることでLAA(ローカル管理アドレス)になるため、試作や研究にはこちらを使用することができます。

なお、大手製造業者が使いまわしているという話もありますが...　　

https://www.geekpage.jp/blog/?id=2020-6-16-1

# 追記:USB-IFの見解
usb-if: No VID for open source
http://www.arachnidlabs.com/blog/2013/10/18/usb-if-no-vid-for-open-source/

要約すると

+ プロダクトIDを他人に再販・譲渡・サブライセンスする目的でのベンダーIDの取得は原則認められていない(Microchip社やFTDI社は例外か？)
+ ホビイストはプロトタイプ用ベンダーIDがあるのでそれを申請すること(但しそれは販売や配布には一切使えない)
+ 割り当て済みおよび割り当てられていないベンダーIDやプロダクトIDの使用は厳重に禁止されている

# ベンダーIDリスト
http://www.linux-usb.org/usb.ids

0x6666が Prototype product Vendor IDとなっているのだが、
幾つかの製品が確認されている模様？

# 参考

https://qiita.com/qa65000/items/24aa199b219e3063dd45
