# はじめに
この記事では、バーチャルモーショントラッカーの開発時のノウハウ、ハマったことについて書いていきます。

Virtual Motion Tracker自体については、以下の記事を参照してください。

https://qiita.com/gpsnmeajp/items/29adc31f30e531fe8023

https://gpsnmeajp.github.io/VirtualMotionTrackerDocument/

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/49ddf69e-a701-2b4a-f37c-aebe25123b48.png)

# 関連記事
https://qiita.com/gpsnmeajp/items/29adc31f30e531fe8023

https://qiita.com/gpsnmeajp/items/050b31d78d1b9757c9db

https://qiita.com/gpsnmeajp/items/e423c699dde7aecb25cc

# OpenVRドライバ開発の何が辛いか
## お作法を文書化してくれ
OpenVRドライバの開発は、当然ですがOpenVRの公式資料を当たりながら作成することになる。
公式資料はほぼこのWikiである。

https://github.com/ValveSoftware/openvr/wiki/Driver-Documentation

ただ、このWikiの内容が絶妙に薄い。最低限のことは載ってるが、調べても細かいことが載っていないことが多発する。
そうなると何を当たるかというと、OpenVRに同梱のサンプル、ヘッダファイル、そしてネット上の試行錯誤の記録である。

## サンプルの読み解きかた
サンプルのコードはopenvr\samples\driver_sample に入っている。
一方、openvr\samples\bin\drivers\sampleに入っている各種ファイルもないと動かないので注意。

### driver_sample.cpp
+ CSampleDeviceDriver(ITrackedDeviceServerDriver, IVRDisplayComponent): 仮想HMD、WakeUp、ごく最低限のデバイスプロパティの設定方法
+ CSampleControllerDriver(ITrackedDeviceServerDriver): 仮想コントローラ、WakeUp、ごく最低限のデバイスプロパティの設定方法
+ CServerDriver_Sample(ITrackedDeviceServerDriver): ドライバの初期化と開放
+ CWatchdogDriver_Sample(IVRWatchdogProvider): ウォッチドッグ

このファイルのコメントはすべて読むこと。重要な情報ばかり書いてある。
ただし、自作のコントローラを作りたい場合は、CSampleDeviceDriverはクラスまるごと不要(ただし姿勢周りの記述はこちらにしかないので注意)、CSampleControllerDriverと、CServerDriver_Sampleがあれば良い。

なお、CWatchdogDriver_Sampleは形だけ必要になる。

各インターフェースクラスの詳細は、ヘッダファイルであるopenvr_driver.h内のコメントが一番詳しい。

#### 概ねの流れ

まず起動時にHmdDriverFactoryが呼び出され、serverとwatchdogを登録

+ watchdogは起動しているだけでいい(通信などのスレッドに使えるらしいがVMTでは使用していない)
+ serverは適時、初期化、フレームごとの呼び出し、開放などが行われる。
+ serverは初期化の際に、各デバイスのインスタンス(CSampleControllerDriverなど)を作って初期化し、TrackedDeviceAddedでOpenVRに登録する(実はこれは後でもいい。必要になったタイミングで追加していってもきちんと反映される。VMTでデバイスがポコポコ増えていくのはこの仕組み)
+ 各デバイスは初期化される時にActivateが呼び出される。ここでシリアル番号等を登録する。

動作中は以下

+ serverのRunFrame()が呼び出される
+ serverは各デバイスのRunFrameを呼び出す。(可変する場合は管理する仕組みが必要になる)
+ 各デバイスのRunFrameで、UpdateXXXXComponent(ボタンや軸)、TrackedDevicePoseUpdated(姿勢)を更新する(と楽。というかこのUpdatedを投げないと反映されないっぽい？)
+ serverはPollNextEventでイベントを吸い上げ、各デバイスのProcessEventを呼び出す
+ 各デバイスはProcessEventで振動などのイベントを処理する(無視しても良い。必要なのは振動くらい)

### driverlog.h, driverlog.cpp
IVRDriverLogを使ってログを出す。
SteamVRの開発者コンソールで出力が見れる。printfに相当するのでかなり重要。

### sample\driver.vrdrivermanifest
ここで、常に有効にするかなどを決める。
"alwaysActivate": trueにし、"name"を適当に変えるくらいで良いはず

### sample\resources\icons
ここに置いたアイコンを、Activateでパス登録しておくと読み込まれる。なお、色味がSteamVRによって勝手に変換されるので注意。無効なときの影とかもなんか勝手に作られます。1種類あれば十分みたい？

### sample\resources\input\mycontroller_profile.json
超重要ファイル。これがないとコントローラどころか姿勢デバイスとして機能しません。
そしてこれに関する情報がネット上にほとんどない。公式ドキュメントを熟読し、SteamVRに同梱されている各種のprofileを頑張って比べて見てください。

VMTで使っているdevice_class、priority、tracker_typesは、すでに存在するProfileを見比べて動作を推測して使っています。
特にこのtracker_types、これがないと体の部位指定が機能しません。

https://github.com/ValveSoftware/openvr/wiki/Input-Profiles

### sample\resources\input\legacy_binding_mycontroller.json
古い形式の入力にしか対応していないソフトでどう振る舞うかを定義するファイルです

### sample\resources\input\vrcompositor_binding_mycontroller.json
アプリケーションに対するBindingです。なんでここに入っているのかわからないんですが... こうやって入れると、アプリケーションから読み込まれるんでしょうか？

### sample\resources\settings\default.vrsettings
重要ファイル。SteamVRの設定ファイルであるsteamvr.vrsettingsに、このファイルの内容を追記したものとして扱われます。
そのため、TrackerRoleやその他設定を予めここに記載しておくと、ユーザーが混乱せずに済みます。

### sample\resources\localization\localization.json
ローカライゼーションファイル。無くても動きます。

## ビルドに注意
VS2019 Community で、ダイナミックリンクライブラリプロジェクトを作成し、ランタイムライブラリをマルチスレッド(/MT)にすれば上手くビルドできます。
確か、この辺で、行を追加すると動いたり動かなくなったりみたいな変な症状に悩まされた記憶があります。ハマり注意。

## 情報が古い
ググっても上記みたいな情報がまとまってるところはあまりありません。
いや、あるんですが、古いんです。

なんでかわからないんですが、SteamVRはコントローラやトラッカーの入力周りに大変更が何度か入ってます。
そのため、古いチュートリアル的なものは、途中までしか動かなかったり、完成しても一部のアプリでしか動作しなかったりします。(VMTも008まではその状態でした)

## 組み込み開発かな？
OpenVR Driver開発というのはつまり、SteamVRに読み込まれるDLLを作成することに等しいです。そのため、ふつう使えるようなデバッグの手段が使えなかったりします。

うっかり例外やポインタでプログラムが落ちようものなら、SteamVRが巻き添えになります。
そしてセーフモードでドライバを読み込んでくれなくなります。

ログも自分で吐き出しておかないと、手がかりは殆どないみたいな状態に陥ります。

あ、多くのユーザーはセーフモードになっても何も気にせず動かして「動かない」というようです。
steamvr.vrsettingsか、OpenVR APIでセーフモードを有効にすることで意図的にその状態を作ることが出来ます。

VMT_Managerにセーフモード検出機能をつけているのはこのためです。
セーフモードも特に何も言わずに仕様が変わっているので注意してください。

## 実はやろうとしていることはトリッキー
まず理解していただきたいのは、HMD + コントローラが同じ空間を扱う、つまり、1つのドライバで動作するのが基本の作りだということ。
つまり、HMDと違う空間で動くコントローラ、トラッカーは、普通ではないことをすることになります。

とはいっても、steamvr.vrsettingsで"activateMultipleDrivers": trueを追加しないといけないのは昔の話。いまは何もしなくても複数のドライバが同時に読み込まれるようです。

あとは、HMDを実装しないドライバを読み込ませれば動きます。

ただし... 空間が合わない。

## 空間が合いません！
SteamVRをインストールした時に最初にやるのがプレイスペースの設定です。
あれがなぜ必要かと言うと、ドライバで使う空間と、現実の室内の空間でズレがあるからです。

というより、こう言えばいいでしょうか。
「どこを基準にしたらいいかわからないから、0,0,0とか設定してもよくわからないところに現れてしまう」

これは本当で、ドライバ開発して0,0,0に位置を設定してみるとわかります。
御使いの環境がLightHouse系であれば、2つあるベースステーションのうちの一方の場所に出てくると思います。高さも床ではなくベースステーション基準のはず。
OculusやWindows MRの場合は、プレイスペースのど真ん中かもしれません。こちらだと高さは床面かもしれない。

で、各種ドライバはここにCalibrationの結果を使って行列計算し、変換することで現実の位置とプログラム上の空間を合わせるわけです。

さて、問題です。貴方が今作ったドライバの空間は？どうやってキャリブレーションしますか？
そもそもトラッカーとして使うなら、HMDやコントローラと位置を合わせないといけないですよね？

これが難題。

答えから言うと、以下のとおりです。

+ OpenVRドライバ自体に、座標変換行列を設定しておくといい感じに変換してくれる機能があります
+ OpenVR.ChaperoneSetup.GetWorkingStandingZeroPoseToRawTrackingPose を使うと、OpenVRはシャペロン境界の変換、つまりHMDが使用しているプレイスペースの変換行列を返してくれます。

これが、VMTの「ルーム変換行列(RoomMatrix)の登録」というわけです。
前者はドライバでないと設定できず、後者はアプリケーションでないと取得できないため、VMT_DriverとVMT_Managerで分担しています。

これにより、適当に座標を渡しても、いい感じにHMDと同じ空間で動いてくれるようになります。
なお、Unityとは右手系左手系が違うので、Unityとやり取りする場合は変換をお忘れなく。

## 行列もクォータニオンもわかりません
この辺の計算は、行列計算で行うことが出来ます。VMTも初期はそういう実装でした。
ただし、前述の通り、座標変換自体をOpenVR側でやってくれる機能があるので、そちらを使ったほうがスマートです。

一方で、角速度や加速度の計算、他のデバイスとの位置関係に合わせて連動する機能などを実現するには、行列計算やベクトル、クォータニオンの計算などが必須になります。
数学わからないので、Eigenというライブラリに頼っています。
(これもまた独特ですが... それ自体は調べればなんとかなります。それよりも式の扱いのほうが何倍も大変)

https://eigen.tuxfamily.org/index.php?title=Main_Page

https://qiita.com/yohm/items/a03006790dc1e54a87be

## ユーザーサポートとか無理でしょこれ
上記のあれやこれやを考えるとわかるんですが、環境や実装によって上手く行かなくなりうる要因がたくさんあります。
その上、ドライバの問題は調べるのが大変です。VMTはさらに外部アプリケーションからデータを受け取るのでその要因も、と考えると調べる組み合わせは膨大。

そのため、開発者向けということにして、エンドユーザーのサポートは基本お断りしています。

## 共有メモリかOSCか
ドライバがドライバ単品でなんとかなるのは、Null Driverくらいでしょうか。
結局の所、ハードを使うなりソフトを使うなり何なり、いずれにせよ別プロセスやハードと通信が必要になります。SteamVRの中で動くのに重い処理もあまりできませんし。

その高速さとシンプルさからよく使われるのは共有メモリですが、これって案外扱いは面倒です。
今回は、外部アプリケーションと連携するなど考えた際、多少遅延があっても楽に通信できたほうがいいだろうということで、OSCを使っています。

とはいえ、C++でOSC使う気は最初はなく、VMT ManagerにてOSC→共有メモリの変換をしようとか考えていたこともありましたが... 
OSC Packのおかげで、ドライバ内で通信できるようになったのでとても楽でした。

http://www.rossbencina.com/code/oscpack

## Steam VR Inputの悪夢　その1 ManifestとLegacy Inputと
トラッカーとして一応動くようになって、欲が出てきてボタンとか軸の機能も(どうせ誰も使わないのに)載せてみようと思ったのですが、これがまた大変でした。

+ 初期化時にVRDriverInput()->CreateBooleanComponentなどで登録しておく
+ 適時VRDriverInput()->UpdateBooleanComponentでOpenVR側に更新を投げつける
+ ETrackedControllerRoleを設定しておかないとコントローラとして機能しない(SteamVR側で拾ってはくれるが大体のソフトで動かない)
+ profile.jsonが正しくないといけない
+ legacy_binding.jsonか、アプリごとのbindingのどちらかあるいは両方が正しくないと検出されない

特にこのjsonが問題で、どれかしら欠けていると全く反応しません(JSON書式エラーはもってのほか)
SteamVR自体のコントローラテストでは動くのに、アプリは反応しないなどよくあります。

VRソフト側では許される、古い形式での入力は、Driver側では許されません。
なお、Driver側で新しい形式でも、古いソフトには適切に変換されます。(というかそのためのlegacy_binding.json)

Input Pathの理解は、ドキュメントを見てググって頑張ってください。

## 動くソフト動かないソフト
ここまで順調に行くと、VRChatなどの昔からトラッカーに対応しているソフトは動くようになっていると思います。
一方で、比較的最近トラッカーに対応したソフトは動かないようなことが起きると思います。

これはおそらくですが、上述した「VRソフト側では許される、古い形式での入力」に関わるものと思います。

VRソフト側からトラッカーの姿勢を取得する方法は大きく分けて2つあり、1つはOpenVRの全デバイス情報の取得を使う方法(旧来)、もう一つはSteamVR InputのActionを使う方法(新規)です。

ボタンや軸の場合、旧来の取得方法と、新しい取得方法は共存できず、どちらか一方でのみやる必要があるのですが、デバイス姿勢だけは他の問題もあるのか、旧来の方法と新規の方法両方が使用できます。

そのため、旧来の方法で取得している場合は非常にシンプルにデバイスを指定して取得するため、特に工夫なく動きますが、SteamVR Inputの姿勢情報を使用している場合、SteamVR Inputに基づいた設定(Driver側のProfile, ソフト側のActionとBinding)の両方がきちんと揃っていないと動きません。

これは以下の記事にも書きました。

https://qiita.com/gpsnmeajp/items/e423c699dde7aecb25cc

## Steam VR Inputの悪夢　その2 Binding
Driver側の入出力情報を、ビット列に割り当てて処理していたのが旧来の方法。
一方で、入出力をパス構造にし、それをDriver側とアプリ側でそれぞれ独自に持っておき、そこを橋渡しするBinding設定ファイルを用意することで、

+ アプリ側がリリースした当時に存在しなかったデバイスなどに設定ファイルの変更だけで対応できる。
+ ユーザーがキー入力の対応関係をカスタマイズできる

といった利点があるのがSteamVR Inputです。

が、問題はこのBinding、アプリ開発者で既知のものであれば予め作成して出荷しますが、未知の場合は単に動作しないのです。

その上、SteamVR側で「Bindingがないから動作しないよ！」という風にユーザーに知らせてくれる機能は、HMDやコントローラでのみ対応。
つまり、複数のDriverを組み合わせてトラッカーだけ別Driverみたいな状態の場合、何も言わずにトラッカーだけ動かないみたいなことになります。

トリッキーなことをしている弊害が再来。

その上、SteamVRのVIVEトラッカー管理の仕組みに乗っからないと使い物にならないですが、ここも文書が殆どない。
これが先に述べたtracker_typesの問題です。VIVEトラッカーのProfileやVMTのProfileを見てください。

そして最後に、ユーザーが上記のトラッカー管理と、Bindingの両方を用意しないと動かない問題。
その設定も一癖も二癖もあります。

頑張ってください。

# 最後に
VRオーバーレイアプリケーションを作るためにOpenVRを直叩きしまくっていたので、これらできましたが、何も経験ない状態でDriver作り始めたら無理ゲでは？
