# はじめに
Nostrのプロトコル読もうと思った人のための補助になればと思います。
間違い等あったら編集リクエストください。

なお、発展途上のプロトコルのため、未実装やドラフトが目立ちます。
実際にデータを受信してみると謎のデータが付加されていたりします。
このあたりについては、IssueやPRを読むと謎が解決することが多いです。

関連

https://qiita.com/gpsnmeajp/items/77eee9535fb1a092e286

非常に有用な解説記事

https://txt.murakmii.dev/posts/nostr-based-software-practice


いじって見たい人はまずnostr-toolsを触ることをお勧めします。ハマらずに済みます。

https://github.com/nbd-wtf/nostr-tools

## [Nostr](https://github.com/nostr-protocol/nostr)
+ Nostrの特徴と目的
+ TwitterやActivityPub, Scuttlebutt(P2P)との比較と優位性
+ どのように既存の問題を解決するかと動作の概略
+ 発生しうる問題とそれに対する優位性
+ FAQ
+ NIPへのリンク

があります。

最初に読むべき文章であり、だいたい懸念される事柄については考察済みであることがわかります。
一方でかなり楽観的に書かれている節があるので、過信は禁物です。

## [Nostr Implementation Possibilities](https://github.com/nostr-protocol/nips)
実装の可能性の一覧です。事実上のプロトコル仕様書
オープンなプロトコルであり、IssueやPRも読むことをおすすめします。

このページは目次ですが、下部にイベントやメッセージ、タグとそれに対応するNIPの情報がありますので、データの読み解きに非常に便利です。

## [NIP-01: Basic protocol flow description](https://github.com/nostr-protocol/nips/blob/master/01.md)

最も基本となるプロトコルの基礎部分です。
投稿を読み書きするどころか、すべての通信がこの仕様に基づきます。

+ イベントと署名(送信)
+ 要求
+ フィルター(受信・要求する情報種別)
+ 応答

の情報があります。

特にフィルターの内容は重要です。

+ kindsが取得する種別
+ authorsは発信元の公開鍵
+ #eは関連するイベントID
+ #pは関連する公開鍵です。

通常単に情報を取得する場合は、sinceとlimitとkindsを設定すれば取れます。
フォローしている人の情報を取る(TL含む)にはauthers
リプライなどは#p、引用等は#e
と言った感じです。

kind:0がメタデータ(プロフィール)
kind:1がテキスト(投稿)
kind:3がフォロー一覧

kind:3だけ指定して他のフィルタを掛けないと、global TLになります。(非推奨)
複数のフィルタを1接続で並行して動かせるので、それでローカルTL見ながら、DM受け取り、リプライも拾ってみたいな感じで、複数のフィルタを動かしていくことになります。

だいたいこれがわかればクライアント作れると思います。

実装サンプル
https://gist.github.com/gpsnmeajp/09f2ff0bd0c9167fb335aed79311cb26

https://scrapbox.io/nostr/NIP-01

## [NIP-02: Contact List and Petnames](https://github.com/nostr-protocol/nips/blob/master/02.md)

いわゆる自分のフォロー一覧をリレーに送る・取得するときの形式です。
(フィルタのauthorsに自分の公開鍵をセットしてkind:3を取得します。)
これがないとフォローTLを作成できません。

ここから得られた公開鍵をNIP-01のフィルタのauthorsにセットしてkind:1を取得することでフォローしている人のTLを構築することができます。
逆に、全くセットしてないのがいわゆるglobalになります。

他人がフォローしている一覧もこの形で取得できます。
(フィルタのauthorsに相手の公開鍵をセットしてkind:3を取得します。)

## [NIP-03: OpenTimestamps Attestations for Events](https://github.com/nostr-protocol/nips/blob/master/03.md)

タイムスタンプ証明の提案

https://en.wikipedia.org/wiki/OpenTimestamps

## [NIP-04: Encrypted Direct Message](https://github.com/nostr-protocol/nips/blob/master/04.md)

暗号化されたダイレクトメッセージ
受信者の公開鍵と送信者の秘密鍵の組み合わせでできたAES256-CBC共通暗号鍵で暗号化されます。

誰が誰に送ったか、いつどのタイミングで何回送ったかは秘匿されません。
上記のような問題があるので論議中。

https://github.com/nostr-protocol/nips/issues/107

https://scrapbox.io/nostr/NIP-04

## [NIP-05: Mapping Nostr keys to DNS-based internet identifiers](https://github.com/nostr-protocol/nips/blob/master/05.md)

DNSベースの公開鍵マッピング

認証マークをつけるのにNIP-05に基づくファイルをWebサーバーの以下の箇所においたと思います。

```
https://<domain>/.well-known/nostr.json
```

この形式の記述と、逆利用したユーザー検索、及び注意点など。
あくまでこれはマッピングを行うものであって証明ではない点に注意してください。

https://scrapbox.io/nostr/%E3%83%89%E3%83%A1%E3%82%A4%E3%83%B3%E8%AA%8D%E8%A8%BC

https://scrapbox.io/nostr/NIP-05%E8%AA%8D%E8%A8%BC%E3%83%90%E3%83%83%E3%82%B8%E3%81%AE%E4%BB%98%E3%81%91%E6%96%B9

## [NIP-06: Basic key derivation from mnemonic seed phrase](https://github.com/nostr-protocol/nips/blob/master/06.md)

Bitcoinウォレットのようにキーワードからバイナリシードを導出して決定論的に秘密鍵を生成する提案

## [NIP-07: window.nostr capability for web browsers](https://github.com/nostr-protocol/nips/blob/master/07.md)

ブラウザやブラウザ拡張によって、署名や公開鍵の取得・暗号化と復号化を行うインターフェースが提供されるときの仕様の定義。
これにより、Webアプリケーションに毎度秘密鍵を渡すことなく安全に利用できる。

すでに以下の実装があることが示されています。

+ nos2x
+ Alby
+ Blockcore
+ nos2x-fox

使うとこんな感じ。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/aa8fd1bd-5649-1a85-aaa2-77766379fcb2.png) ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/712f41db-75b6-a071-0028-85a1bc020f55.png)

https://scrapbox.io/nostr/nos2x%E3%81%AE%E3%82%BB%E3%83%83%E3%83%88%E3%82%A2%E3%83%83%E3%83%97%E3%81%A8%E4%BD%BF%E3%81%84%E6%96%B9

ハードウェア署名器との連携にも使えるかもしれません。

https://scrapbox.io/nostr/nostore(iOS_Safari%E7%94%A8nip-07%E6%8B%A1%E5%BC%B5%E6%A9%9F%E8%83%BD)%E3%81%AE%E3%82%BB%E3%83%83%E3%83%88%E3%82%A2%E3%83%83%E3%83%97

https://scrapbox.io/nostr/NIP-07

実装サンプル
https://gist.github.com/gpsnmeajp/23a2563ec39d4db43f0f922e5f08de87

https://github.com/gpsnmeajp/nip07ex

## [NIP-08: Handling Mentions](https://github.com/nostr-protocol/nips/blob/master/08.md)

メンション(リプライ)の処理方法

メンションは公開鍵をpタグに追加しなければならない。
文中のメンションは#[0]のような形でタグを参照しなければならない

これはユーザのみならず、イベントへ(投稿)の言及も同様。
(いわゆる引用がこれだと思われる)

受け取ったクライアントは、リンクにしたりコンテンツのプレビューを行う。

https://scrapbox.io/nostr/%E5%BC%95%E7%94%A8Repost%E3%81%8C%E3%81%A9%E3%81%AE%E3%82%88%E3%81%86%E3%81%AB%E8%A6%8B%E3%81%88%E3%82%8B%E3%81%AE%E3%81%8B%E5%95%8F%E9%A1%8C

## [NIP-09: Event Deletion](https://github.com/nostr-protocol/nips/blob/master/09.md)

削除を定義
eタグで指定した投稿は削除したことになる。

(なおオプションであり、必ず消えるわけではない)

## [NIP-10: Conventions for clients' use of e and p tags in text events.](https://github.com/nostr-protocol/nips/blob/master/10.md)

テキスト投稿におけるeタグとpタグの使い方

返信時の単一eタグの位置に依る意味付け(非推奨)
返信時の複合eタグによる意味付け(推奨)

返信に関与しているユーザを表すpタグ

https://scrapbox.io/nostr/NIP-10

## [NIP-11: Relay Information Document](https://github.com/nostr-protocol/nips/blob/master/11.md)

リレー自身のメタ情報の定義
Websocket接続時に返される。

ここでリレーの管理者の連絡先等の他、リレーが何の機能に対応しているかのNIPリストが返されます。

## [NIP-12: Generic Tag Queries](https://github.com/nostr-protocol/nips/blob/master/12.md)

汎用タグクエリ
フィルタに1文字の要素がある場合、それを購読できるようにする仕組み(標準のe,pタグ以外も)

例示で以下のものが挙げられている

URL(r) : Webページに対するコメントシステム
ハッシュタグ(t): ハッシュタグ
ジオハッシュ(g): ジオハッシュで物理位置に紐づけ

https://scrapbox.io/nostr/NIP-12

## [NIP-13: Proof of Work](https://github.com/nostr-protocol/nips/blob/master/13.md)

Proof of Workを用いたスパム対策の提案

なお、PoWは現在においてはビットコインで有名ですが、元々1993年にメールのスパム対策として提案されていたので先祖返りに近いです。

https://hazm.at/mox/distributed-system/blockchain/proof-of-work.html

## [NIP-14: Subject tag in text events.](https://github.com/nostr-protocol/nips/blob/master/14.md)

投稿に件名を付けられるようにする提案

## [NIP-15: End of Stored Events Notice](https://github.com/nostr-protocol/nips/blob/master/15.md)

リレーから保存済み情報の終わり(ここから先はストリーミング)である旨を通知する仕組み。
EOSE

クライアント作成しているとこれの有無は結構重要です。(取得完了がわかるため)

## [NIP-16: Event Treatment](https://github.com/nostr-protocol/nips/blob/master/16.md)

一時的なイベント(入力インジケーターや、ステータスなど保存が不要だったり複数保存する必要がない情報)を扱うための提案

## NIP-18
廃止された仕様(Repost)

https://scrapbox.io/nostr/NIP-18

## [NIP-19: bech32-encoded entities](https://github.com/nostr-protocol/nips/blob/master/19.md)

bech32について。
公開鍵や秘密鍵、投稿IDの不思議な文字列や接頭辞の定義です。

リレー情報やその他メタデータを含んだ長い形式も定義されています。

bech32m**ではない**

参考

https://scrapbox.io/nostr/bech32

https://scrapbox.io/nostr/NIP-19

## [NIP-20: Command Results](https://github.com/nostr-protocol/nips/blob/master/20.md)

リレーからの結果応答とその詳細(エラー等含む)の定義です。
正常、IPアドレスフィルタによる失敗、公開鍵Block、課金必要、レート制限、時刻異常、PoW不足による失敗などの例が挙げられています。

クライアントが識別するための接頭辞と、ユーザーが読むための文が含まれます。

## [NIP-21: nostr: URL scheme](https://github.com/nostr-protocol/nips/blob/master/21.md)

URLスキームを定義しています。

https://scrapbox.io/nostr/NIP-21

## [NIP-22: Event created_at Limits](https://github.com/nostr-protocol/nips/blob/master/22.md)

遠い未来や遠い過去からの投稿が発生しないように、受け入れるタイムスタンプに制限をかけることができる提案です。

## [NIP-25: Reactions](https://github.com/nostr-protocol/nips/blob/master/25.md)

kind:7で、リアクション(いいねや+1)を投稿することができます。
これに関しては、反応対象をeタグ、相手をpタグで指定する必要があります。

+でlike, -でdislikeですが、絵文字を送信することも可能です。
(絵文字の解釈あるいは表示はクライアントに任されます。)

https://scrapbox.io/nostr/NIP-25

## [NIP-26: Delegated Event Signing](https://github.com/nostr-protocol/nips/blob/master/26.md)

委任キーを作成する提案です。
これにより、ルート秘密鍵ではなく、クライアントごとに鍵ペアを作成してそちらに委任して使用するといった方法を提供します。

delegationタグに、委任元の公開鍵と委任期間と権限範囲、そしてこれらに対する委任元の署名がつくことで、委任を証明します。
(多分投稿専用でフォローはできないとか、そういう制限がかけられる。またDMは仕組み上読み書きできないかもしれません)

https://scrapbox.io/nostr/NIP-26

## [NIP-28: Public Chat](https://github.com/nostr-protocol/nips/blob/master/28.md)

公開チャットチャネルを作成し、そこで会話することができます。
チャネルには名前や説明・画像を設定でき、その説明を更新したりできます。

投稿に対する非表示やミュートを実施した場合、その旨が発信されるため、
クライアントによっては複数人が非表示にした投稿は非表示にするといった機能を実装することができます。

https://scrapbox.io/nostr/NIP-28

https://scrapbox.io/nostr/%E3%83%81%E3%83%A3%E3%83%B3%E3%83%8D%E3%83%AB

## [NIP-33: Parameterized Replaceable Events](https://github.com/nostr-protocol/nips/blob/master/33.md)

NIP-16 (一時イベント)の拡張で、kindのみによる置き換えではなく、kindとタグdが一致するものを置き換えるようにする(?)

## [NIP-36: Sensitive Content](https://github.com/nostr-protocol/nips/blob/master/36.md)

センシティブなコンテンツありの投稿に対し、非表示と理由を提供する。

## [NIP-40: Expiration Timestamp](https://github.com/nostr-protocol/nips/blob/master/40.md)

対応するリレーに期限付きメッセージを送るときの形式の提案

## [NIP-42: Authentication of clients to relays](https://github.com/nostr-protocol/nips/blob/master/42.md)

リレーからクライアントに対するチャレンジ&レスポンス認証の要求
ホワイトリストリレーや有料リレー、より保護されたDMを実装するなどの用途に使用できる。

## [NIP-50: Keywords filter](https://github.com/nostr-protocol/nips/blob/master/50.md)

リレーによる検索機能を利用するための提案
スパムフィルタをオフにして検索できることが要求されている

# その他参考情報

https://scrapbox.io/nostr/%E6%8A%80%E8%A1%93%E3%83%BB%E4%BB%95%E6%A7%98%E4%B8%8A%E3%81%AE%E7%94%BB%E5%83%8F%E3%81%AE%E6%89%B1%E3%81%84

https://scrapbox.io/nostr/Nostr%E6%A4%9C%E7%B4%A2%E3%83%9D%E3%83%BC%E3%82%BF%E3%83%AB

https://www.nostr.net/
