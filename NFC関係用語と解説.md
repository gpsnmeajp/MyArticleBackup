#はじめに
NFCやFelica関係を調べていると，用語の混乱をよく見かけるので，ざっと解説を書いてみることにしました．
**この解説には誤りを含む可能性があります．ご注意ください**

## 合わせて読むと良い記事
わかりやすい図で理解が深まります．

+ [勝手にNFCに入門してみる](https://qiita.com/kutakutat/items/3333181e91c6c6dbf222)

#NFCとは
NFCの定義自体がいくつもあるが，代表的なものを2つ上げる．
##世間一般的なNFC
なぜか世間一般的に「NFC」と「FeliCa」が別物扱いされる．(NFC/FeliCaなど)
詳細な理由は，後の「セキュアエレメント」に譲る．

なおKDDIのスタンスはこちらの様子．オンライン上の文書で多く見かける．
(JR,ドコモ,Sonyの3社で設立の，鍵情報などを管理するフェリカネットワークス社に入り込めないためと推測)

ダブル対応スマホも発売済み：FeliCaとの違いは「世界標準＋低コスト」　KDDIがNFCへの取り組みを説明 - ITmedia Mobile 
http://www.itmedia.co.jp/mobile/articles/1303/04/news135.html

##NFC Forumの立ち位置
NFC ForumはSony(FeliCa陣営)とNXP(Mifare陣営)が組んでできたもの．
同じ13.56MHz帯を使い，ほぼ同タイミングで生まれたFeliCaとMifareの間の干渉の対策や，データ交換の仕組みの制定などを行っている．

そのため**NFC対応と言った場合，MifareもFeliCaも「通信に関しては」対応している**．
ので，「Suicaは使えないがSuicaの残高は読めるスマホ」というのが現れる．
詳細な理由は，後の「セキュアエレメント」にて説明する．

またNFCIP-1(ISO 18092)としてMifareとFeliCaが，NFCIP-2(ISO 21481)としてType BとRFID(ISO 15693,通称Type-V)がある．

NFC ‐ 通信用語の基礎知識 
http://www.wdic.org/w/WDIC/NFC

NFCの概要 - KDDI用語集
http://www.kddi.com/yogo/%E9%80%9A%E4%BF%A1%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9/NFC.html

NFC Labo
https://sites.google.com/site/nfclabo/home

##ところで
TransferJetやRFIDは，近接通信ではあるがNFC扱いされることが少ない．
これは，周波数が違うためであったり，互換性がなかったりするためである．

もっとも，NFCは大きなくくりではRFIDの子供である，
が，Type-Vもあったりしてよくわからない．

#NFCの3モード
NFCでは以下の3つのモードが定義されている

+ Reader/Writer Mode(必須)
+ Peer-to-Peer Mode(必須)
+ Card Emulation Mode(オプション)

このうち必須のなのは記載のとおり上２つである．
つまり，
「NFC-A/B/Fのすべてに対応していること」
「タグやカードを読み書きできること」
「かざした端末同士で通信できること」
が，NFCマークをつける条件である．

**「カードとして振る舞えること」は条件に入っていない**．

なお，かざした端末同士で通信できる規格でも，NFC-DEPとそれ以外の独自規格が存在する．

ちなみに，iPhoneはPeer-to-Peer Modeをサポートしていないようにみえる．チップはサポートしてるかもしれないが．

#NFCのType-A/B/F...(Technology)
NFC-AとかNFC Type-Aとか呼ばれるいろいろがあるが，これらは何かというと「通信規格の仕様」である．
**「通信規格」であって「セキュリティ」や「中身」は含まない点に注意．**

+ NFC-A (Type-A) : MIFARE NXP (ISO-1443/A)
+ NFC-B (Type-B) : Type-B Motorola (ISO-1443/B)
+ NFC-F (Type-F) : FeliCa Sony (JIS X 6319-4)

これらは，周波数は同じだが，変調方式からして違う．
Type-Aは106kbpsだが，Type-Fは212kbps/424kbps出る．
ので，タグにはType-A，相互通信にはType-Fを優先して使う傾向があるようだ．

NFCを名乗るにはこの「通信規格全てに対応している必要がある」が，**「セキュリティには対応する必要はない」**．

これが後の混乱を招く．

なお通称NFC-V (Type-V)のISO 15693は含まれていないが，Androidだと何故か読める．

主によく使われるのはType-A(Mifare)で，次点でType-FのFeliCaであろう．
Type-Bはおそらくだが，接触型スマートカードに非接触の機能を付ける場合によく使われるフシがある．(マイナンバーカードなど)

※通信速度と変調方法，符号化
リーダー→カード

|方法|Type-A|Type-B|Type-F|
|---|---|---|---|
|変調方法|ASK100%|ASK10%|ASK10%|
|符号化|変型ミラー|NRZ-L|マンチェスタ|
|伝送速度|106kbps(～828kbps)|106～828kbps|212～424kbps|

カード→リーダー

|方法|Type-A|Type-B|Type-F|
|---|---|---|---|
|変調方式|OOK|BPSK|ASK10%|
|符号化|マンチェスタ|NRZ|マンチェスタ|
|伝送速度|106kbps(～828kbps)|106～828kbps|212～424kbps|

※Type-Aは符号化と変調方式をType-B方式に切り替えて速度上げる方法もある模様

NTT技術ジャーナル2008.3 非接触ICカード技術
http://www.ntt.co.jp/journal/0803/files/jn200803081.pdf

SONY 
https://www.sony.co.jp/Products/felica/about/scheme.html

#NFCのType-1/2/3/4...(Platform)
ではType-1/2/3/4の番号の方は何かというと，「NDEFタグの規格」である．

+ Type-1  : Topaz
+ Type-2  : Mifare Ultralight
+ Type-3  : FeliCa
+ Type-4A : Mifare DESfire (ISO-1443A)
+ Type-4B : ISO-1443B

この表から見てわかるように，MIFARE Classicは対象外である．
そのため，MIFARE ClassicをNDEFタグとして使用するにはNXP社の独自フォーマットに対応する必要があり，MIFARE Classicにアクセスできない機種も存在する．

ちなみに，FeliCaは高額で複雑なFeliCa Standardでもタグとして振る舞うことができるが，ほぼNDEFタグ用として設計されたFeliCa Liteを使用するのが経済的である．

さらに捕捉．
Type-3タグとして使うことを前提に考えられているFeliCa Liteだが，初期状態ではType-3タグとして動作しない．(システムコード0x12FCに反応しない)
NDEFフラグを立てる必要があり，これを1次発行と呼んでいる場合がある．
(1次発行で固定化される領域ではあるが，NDEFフラグを立てること自体は別に1次発行ではない．)


#ISO-DEPとNFC-DEPとメモリと...(Protocol)
##ISO-DEP (ISO 14443-4)
ここでの話は主に
Reader/Writer Mode
Card Emulation Mode
の話となる．

よく混乱するのだが，Type-Aには複数の通信プロトコルが混在している．

+ Type-2  : Mifare Ultralight
+ Type-4A : Mifare DESfire(ISO 14443-4(T=CL))

の2つが存在している理由もそれである．

Type-2の方は，ISO/IEC 14443-3A 「初期化と衝突防止」に準じている．
物理層・論理層の対応であり，比較的単純なメモリのような構造をしている．
コマンドも，ブロック読み取り・ブロック更新・認証などの比較的単純なコマンドで構成されている．

Type-4の方は，ISO/IEC 14443-4 「伝送プロトコル」に準じている．上記に加え共通の通信層やファイル構造を載せた構造である．
これがISO-DEPと呼ばれているもので，いわゆるスマートカードに必要な階層構造や認証を扱う．
カード側を賢い通信機器として扱うのだ．

APDUというプロトコルを使って通信をし，これは接触型ICカードにおけるT=1と同等である．
T=CL(Contact Less)と呼ばれる．
コマンドは，ファイルの選択・ファイルの読み取り・ファイルの更新の他，様々なコマンドがある．

この場合においては下位層でType-Aを使っているのかType-Bを使っているのかは考える必要はない．

##FeliCa
ちなみに，FeliCa Standardは，1ブロック16Byteのメモリのような構造をしつつ，エリア(ディレクトリ)とサービス(ファイル)で階層上に構成され，そこにランダム・サイクリック・パースのアクセス権をもたせる仕組みである．
さらにそのためのセキュリティの仕組みも，アクセス速度を重視した高速な物となっている．

FeliCa Liteは，このエリアを廃止し，サービスとアクセス権(2つはセットである)を，読み取りと書き込みに絞り，
単純なメモリのように読み書きできるようにしたものである．
通信プロトコルは同等であるため，FeliCa Standardを導入している環境では容易に導入でき，仕組みがシンプルなため安価である．
Type-3タグのために作られたような形をしている．

認証や暗号化通信の機能がLiteにはないが，簡易的な認証とアクセス権の管理ができる機能がLite Sに搭載されている．
ただしこの機能に関してはStandardとの互換性はない．
(代わりにNFC相当でのアクセスしか提供しない機器でも利用できる)

##NFC-DEP (ISO 18092)
ISO-DEPを使えばType-AやType-Bの機器と通信できるが，Type-Fの機器と通信できない．
ということで，Type-AとType-Fを束ねたNFCであるISO 18092において，別のデータ交換仕様が生まれ，それがNFC-DEPである．

というわけでここでの話はPeer-to-Peer Modeの話だ．

NFC-DEPでは，受動・能動通信の手順を決めている．
DEPは最低限の通信の規定でしか無いため，この上にさらにLLCP(リンク層・セッション層)やSNEP(アプリケーション層)が出てくる．

Android BeamなどはNFC-DEP(SNEP)を使用している．

ATR_REQやATR_RESなどはここで出てくるため，ISO 18092を参照すること．

##FLAP (FeliCa Adhoc Link Protocol)
NFC-DEPなどが出る前に存在したらしい．
ドコモの携帯電話についていたiC通信がこれで，名前の通りFeliCa独自規格である．
なお，仕様は非公開らしい．

hiro99ma blog [nfc]復習しよう 
https://hiro99ma.blogspot.jp/2012/11/nfc.html

非接触ICに最適化された「FeliCa」の正体 － ＠IT 
http://www.atmarkit.co.jp/frfid/special/felica/felica03.html

hiro99ma blog: [nfc]DEPとLLCPとSNEP 
http://hiro99ma.blogspot.com/2012/02/nfcdepllcpsnep.html?spref=tw

#NFCとセキュアエレメント(なぜNFC vs Felicaと言われるのか)
**NFCはType-A/B/Fの全てに対応している**と先に述べた．
では，なぜNFC vs FeliCaのような変な話になるのだろう．
これは，消費者目線で考えればわかる．

+ NFC対応=FeliCa対応=Suicaなんかが使える！
+ アプリは一向に出ないし，なんか使えないらしい
+ FeliCa対応してないじゃん！あれ，おサイフケータイとか書いてある方は対応してる...？
+ NFC≠FeliCaなんじゃね？
+ NFC=Type-A/B≠FeliCaだ！ガラパゴスジャパン！！

と．

どうしてこうなるのだろう．
原因は「セキュアエレメント」にある．

##セキュアエレメント
セキュアエレメントとは，言ってしまえば「Card Emulation Mode」(カードとしての振る舞い)を実現するためのチップの総称である．
名前からセキュリティっぽい感じが漂っているが概ね合っている．

原理は以下だ．

+ アプリケーションで決済関係の処理しようものならハッキングされたり盗聴されたりする
+ そうだ！専用チップ内で通信や決済を完結させればいいんだ！

というわけで，カードとしての振る舞いと，カード読み書きは物理的に別の領域でやることになった．
ガラケーのおサイフケータイなんかはこの機能そのままであるし，Apple Payなどもこれである．

##セキュアエレメント載せたくない問題
で，このセキュアエレメント，

+ Type-A/BとFeliCaで全くの別物である．(そら規格違うし仕組みも違う)
+ なぜなら，ICカードに互換性がないのと同じで，セキュアエレメントにも互換性はない．
+ NFCはセキュリティに関することは対象外なので，NFC搭載を名乗るときセキュアエレメントは考慮されないし，そもそもカードエミュレーション自体がオプション．
+ わりとコストがかかる

その結果，日本の携帯電会社は，日本国内トップシェアのFeliCaのセキュアエレメントを載せる．
→海外で決済に使えない．

海外はもっと酷いことに

+ セキュアエレメントの搭載や管理をするのは携帯電話会社
+ 決済サービスはGoogleとかがやりたい
+ 携帯電話会社は自社で決済サービスをしたい
+ 携帯電話に載せる？SDカードに載せる？SIMに載せる？

→セキュアエレメントを載せなかったり使わせなかったり売らなかったり
(一方その横を素通りするApple Pay)

という．

結果，海外製端末は日本で決済に使えず，日本製端末は海外で決済に使えないという事態に陥る．
(そもそも海外製端末にはセキュアエレメント自体乗ってないことも多いのかも)

なおApple管轄のiPhone 7以降は全部入りという富豪戦略を取ったので全世界で使用できる．

##ようはつまり

+ Type-A/B SE
+ FeliCa SE (おサイフケータイ or NFC/Felicaの表記)
+ SEなし

の端末が全部が**NFC**に一括りにされているため，
消費者としては騙された気分になるのである．

ロンドンで見た世界の最新NFC事情――新技術「HCE」と公共交通機関の進展に注目 (1/2) - ITmedia Mobile
http://www.itmedia.co.jp/mobile/articles/1408/12/news028.html

#NFCとHCE
##ハードがダメならソフトで解決しよう
だがしかしそもそも，セキュアエレメントなど必要なのだろうか？と思っている人もいるだろう．

なぜなら，今ではQRコードで決済が行われる時代である．
このQRコードは(デンソーウェーブ開発のセキュアQRコードなどではなく)誰でも読み取り可能なものだ．

というわけで，「アプリでカードとしての振る舞いできるようにすればいいじゃないか！」として2014年に出てきたのが
Host Card Emulation あるいは Host-based Card Emulationと呼ばれる技術である．
これは，もうまんま言ってしまえば「セキュアエレメントとしての役割をアプリに担わせる」技術である．
Android 4.4やWindows Mobileが対応しており，VISAやMasterCardが喜んで参加している．

これはつまり，ソフトウェア的にカードとして振る舞うので，カードエミュレーションの機能がついたNFCチップ(たいてい対応している)端末ならば，ハードとしてセキュアエレメントが搭載されているかに関係なく決済が行えるものである．

これにより，セキュアエレメントで起こっていた不毛な争いが解決する上に，セキュアエレメント非搭載だけどNFC対応な端末でも決済が行えるようになる．
ただし，これはISO-DEPの仕組みを使っているものである．つまり...

##日本で使えないHCE
Type-A/Bでしか使えないHCEは日本ではほとんど意味がなかった．なぜならFeliCaはISO-DEPをベースにして動いていない．
Type-A/Bで動作するクレジットカード
そもそも，日本ではセキュアエレメントと搭載した端末が殆どで，HCEに相当する技術を導入するメリットもない．
NFC搭載だけどFeliCaが使えなくて困る人など，海外端末を輸入している人くらいである．

...が，外国人観光客やオリンピックを控え，そうも言っていられないようになったのか，2016年末頃にAndroid 7でHCE-Fが使えるようになった．
ただしType-A/Bで使えたHCEから比較すると少々自由度が低いが，国内のセキュリティ事情やその他を考慮した結果だろう．
これにより，FeliCaしか扱えない機器でもHCEの恩恵をうけることができるようになった．

AndroidのHCE-Fについて調べてみたメモとサンプルソース
https://qiita.com/gpsnmeajp/items/e525bb5c13511c18dda5

また，2020年オリンピックに向け，クレジットカードのタッチ決済が順次導入されている．
こちらは国際ブランドのものであり，EMVCoに準拠しているらしい．
ISO7816 T=CLとのことなので，Type-A/BのHCEでアクセスできるだろう．

#おまけ:PC/SCとNFC
PC/SCでのカードの取扱は，ISO-DEPに沿っているType-4系であればAPDUで簡単にできる．
一方，多くのNFCタグやカードはISO-DEPに沿っていないType-1/2/3である．

そのため，PC/SC対応のカートリーダー(のドライバ)でも，独自コマンドを実装してType-1/2/3の通信規格に対応している．
例えばPaSoRi RC-S380では，ReadBinary/UpdateBinaryのコマンドを拡張してMifareやFeliCaを扱えるようにしている．
ACR-1251CLでは，ReadBinary/UpdateBinaryはMifareには対応しているが，FeliCaは生のパケットを自力で書かせる仕様である．

#おまけ:カードリーダーの問題
PaSoRi RC-S380(約3000円)は，NFC開発には正直向いていないと感じる．

利点

+ PC/SCと，felicalibと，公式SDKの3種類ある(あとNFPもある)
+ NFP対応でWindowsのNFC通信機能が使える
+ FeliCa専用ソフトが動く

欠点

+ Mac・Linuxに対応していない(お高いSDKを購入すれば使えないことはないらしい)
+ PC/SCはおせっかい仕様でHCE/HCE-Fと通信を確立できない(DEPモードに入るらしい？)
+ PC/SCはろくに設定できる項目がなく，ドライバのGUIから設定する必要がある(ポーリング間隔とか色々)
+ HCEは公式SDKを，HCE-Fはfelicalibか公式SDKを使わないとまともに通信できない．(PC/SCではポーリングすらできない)
+ 公式SDKは利用規約が厳しくソースの公開すらはばかられる
+ フリーで開発しようとすると，felicalibか，libpafeかnfcpyあたりを使う

※libpafeやnfcpyを使うとlinuxでも動きます．


個人的には[ACR1251CL](https://www.yodobashi.com/product/100000001003065479/)(約3000円)がおすすめである．
が，FeliCaの開発には向いていない

+ NTT Communicationから販売されており日本の法規をクリアしている
+ PC/SCのAPIが公開されており親切なPDFマニュアルがダウンロードできる
+ Mac・Linuxでも使える
+ 余計なことをしないのでHCE(Type-A)とも普通に通信できる(タイムアウトも無限...？)
+ FeliCaは反応が良くないことがある．
+ FeliCaへのパケット生成はほぼ自力である
+ FeliCaのタイムアウトの設定ができないため，HCE-Fは無理

※初回接続時ドライバのインストールに失敗することがあります．
　デバイスマネージャでWindows Smart Card... となっているのを右クリックして，ドライバの更新を行うと，ACR1251として認識します．

#おまけ:タグの入手先
Type-1  : Topaz
不明
[唯一見つけたのはここだが買えなさそう](http://www.chip1stop.com/dispDetail.do?partId=ORGT-0000040)

Type-2  : Mifare Ultralight, NTAG
[amazonでいっぱい売ってる。オレンジタグスをおすすめする](https://www.amazon.co.jp/s/field-keywords=mifare+ultralight)
[Ace shopのお試しセットも便利](http://shop.nfc-acekougyo.co.jp/i-shop/product.asp?cm_id=279475&cm_large_cd=1)

Type-3  : FeliCa-LiteS
[amazonでいっぱい売ってる。オレンジタグスをおすすめする](https://www.amazon.co.jp/s/field-keywords=Felica+Lite)
Felica-Liteと Felica-LiteSは互換性がある。(Sの方がセキュリティが充実した後継で、Liteとしても機能する)
なお、セキュリティ機能(片側認証,両側認証)を試したい場合は、不可逆な変更を含むため、数十枚買うことをおすすめする。

Type-4A : Mifare DESfire (ISO-1443A)
[Ace shopで5枚1350円で手に入る](http://shop.nfc-acekougyo.co.jp/i-shop/product.asp?cm_id=306190)

Type-4B : ISO-1443B
不明。強いて言うならマイナンバーカードとか免許証？

全種類詰め合わせセット(ただしお高い)
[Type1カード／Type2カード／Type3カード／Type4カード／Classic Typeカード／Type3シール／Type2シール／Type2キーホルダーのセット](https://www.amazon.co.jp/dp/B003NOGTLY/)
これに限らず全種類オレンジタグスで入手可能なようだが、要問合せになっている。

[関係ないですが、NFC延長アンテナという商品も](http://shop.nfc-acekougyo.co.jp/i-shop/product.asp?cm_id=323803&cm_large_cd=6&cm_small_cd=2)


#おまけ:海外のNFCハッキング事例
今のところはFeliCaは「破られた」とかの根拠なし記事しか見当たらない．
MifareはClassicに脆弱性があることが知られている．

##Android不正アプリによるRFIDプリペイドカード改ざんが南米で発生 - トレンドマイクロ 
セキュリティが突破されていることで有名なMIFARE Classicを使い続けていたため
対策: より高セキュリティな品種への更新
http://blog.trendmicro.co.jp/archives/10432

##Android NFC のハッキングにより地下鉄の無賃乗車が可能に - SOPHES
Mifare Ultralightを不適切に使うシステムが原因
対策: 適切な操作ロックを行うか，より高セキュリティな品種への更新
https://www.sophos.com/ja-jp/press-office/press-releases/2012/09/jp-android-nfc-hack.aspx

##ちなみに...(セキュリティ運用に関する警告)
NFCのカード固有IDを利用した認証を行っているシステムは，非常に脆弱です．
**セキュリティ保護されていないカード固有IDは，容易にコピーすることができます．**

FeliCaにおいても，開発元であるSonyが「IDmは認証に使わないように」とマニュアルに記載しています．
が，実際問題として「どんなカードでも登録できる」「交通系カードを登録できる」と謳った認証装置・認証機構が日本では多く出回っています．

こういった装置は，市販のカードリーダとPCで容易にコピー・偽装することができるため，「最悪開けられても良い場所」「最悪認証されても良い装置」に使用してください．

なお，

+ 専用カードを用いて認証する装置(大学の学生証を用いたセキュリティなど)
+ カードをかざした後にパスワードを要求する装置(ゲームセンターなど)

であれば問題ありません．


ハッカーは恐ろしいほど簡単にICカードを複製可能、特殊デバイスを使えば遠距離攻撃も可能 - GIGAZINE 
https://gigazine.net/news/20170123-clone-security-badge/
