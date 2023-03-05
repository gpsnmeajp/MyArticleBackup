# はじめに
こんなのを作ってみたい人向けの記事です。
これはVRオーバーレイアプリケーションで、すなわち現在動作しているVR空間に関係なく独立して空間に存在できるアプリです。
(特定のVRゲームに関係なく動くということです。アプリケーションのロード画面でさえ動きます。)

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">VRMMO風メニューツールのVaNiiMenu v0.07a beta公開しました。<br>機能追加としては、アラーム機能と、SF風スキン、8bit風効果音が追加されました。<br>また、たくさんの不具合の修正と、使い勝手の向上のための改良がされています。<br>詳しくは更新履歴を参照してください。<a href="https://t.co/YMe6P7U6Gh">https://t.co/YMe6P7U6Gh</a> <a href="https://t.co/vXjeJLrKu0">pic.twitter.com/vXjeJLrKu0</a></p>&mdash; Segmentation Fault (@Seg_Faul) <a href="https://twitter.com/Seg_Faul/status/1080366378635292674?ref_src=twsrc%5Etfw">2019年1月2日</a></blockquote>

本記事では、VR空間内で画面をタップして操作できる2Dユーザーインターフェースを実装します。
すでにあるライブラリとコードを用いて、コードを書くことなく画面を掴んで移動できるところまで実装していきます。
これにより、C#の知識がなくともルーム空間に配置できるVR写真立てくらいは作れることになります。

# 対応環境
VRオーバーレイはOpenVRの機能を利用しているため以下のような動作になります。

+ HTC VIVE/VIVE ProならほぼすべてのVR空間内で動作
+ Steam VR系HMDならおそらく同じくほぼすべてのVR空間内で動作
+ Oculus RiftおよびWindows MRの場合はSteam VR上で動くVR空間の上でのみ動作(ネイティブ不可)
+ ALVRでは3DoFなら操作不能だが見ることは可能、6DoFなら一応動作


# 更新履歴
+ v0.02d EasyOpenVRUtilをリリースページに切り替え、fpsの注意を追記
+ v0.02c カメラのFarについて注意
+ v0.02b 例の追加、動作環境の追加、タイトルの変更、レイアウトの修正
+ v0.02a 細かい引っ掛かりポイントを修正
+ v0.02  PositionManagerScript.csにて、ビルドすると毎回初期位置が原点になる問題を修正
+ v0.01a 説明文の修正、今後の展望の追記、OpenVRとUnityEditorに関する注意書きの追記 
+ v0.01  初公開

# なんで空間タップなんてこんな回りくどい方法を？
VRオーバーレイアプリケーションというのは、名前の通りVR空間にオーバーレイします。
VR空間で使えるのはVRコントローラですが、VRソフトウェアではすでにコントローラのボタンは使用されていることが多く、キーボードショートカットなどのように安易にボタンを使用することができません。
特にVRゲームにおいては、ボタンは特別な存在であり、ホーム空間に戻ったり、ポーズしたりといった大切な動作に割りあたっている事が多いです。

しかしながら、VR空間においては人の体は常に動くものなので、動きに対しては許容性があります。
そこに着目したのがVaNiiMenuで、人の動きだけでメニューの呼び出し、タップによる操作などを完結させているため、VRゲームに与える影響を最小限に抑えます。
(※任意の動きすら許容されない場面ではそもそもメニューが有ったとしても操作できないため除外できます)

今回はこの、タップによる操作の基礎部分を実装してみましょう。
# 実装する機能
・オーバーレイでuGUIのCanvasを表示します
・タップでボタンに反応するようにします
・カーソルを表示します
# 準備するもの
### Unity 2017.4.15f1 Personal
おなじみ開発環境
バージョンは多少違っても動くと思います。
### SteamVR Unity Plugin v2.0.1
UnityからSteam VRを扱う必須ライブラリです。アセットストアにもあります。
https://github.com/ValveSoftware/steamvr_unity_plugin
https://github.com/ValveSoftware/steamvr_unity_plugin/releases

追記: Unity 2021を使用する場合は、SteamVR Unity Plugin v2.7.3 (sdk 1.14.15)

### EasyOpenVROverlayForUnity
オーバーレイ表示とタップ判定、簡易GUI操作がセットになったライブラリ。私の自作です。
https://sabowl.sakura.ne.jp/gpsnmeajp/unity/EasyOpenVROverlayForUnity/
### EasyOpenVRUtil
OpenVRの姿勢情報などを扱うライブラリ。私の自作です。
https://github.com/gpsnmeajp/EasyOpenVRUtil/releases

# 手順
## 注意
ある程度Unityの操作に慣れている前提で説明します。
わからない場合は、入門サイトを見るか、VRChatのワールド制作などで馴れてから始めてください。

## Unityで3Dプロジェクトを作成してライブラリを導入
Unityで3Dプロジェクトを作成します。

その後、まずSteamVR Unity PluginのUnitypackageを導入します。
完了すると、Steam VR向けの設定画面が出るので「Accept All」をクリックします。
「You made the right choice!」と出ればOK。

次に、EasyOpenVROverlayForUnity.cs、EasyOpenVRUtil.csをAssets **直下に** 入れてください。
(馴れている人はAssets直下でなくても構いませんが、 **PluginsやEditorだけには入れないでください。** 
　特殊フォルダでありコンパイル順に影響してコンパイル失敗します)

これで必要ファイルの導入は完了です。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/806aeb9e-3dcd-c7df-cb66-0d78d25628bc.png)

## Player設定(汎用)
Unityプロジェクトの設定を始めます。

Edit →　Project Settings → Playerを選択。
まずCompany NameとProduct Nameを設定します。

次に、下にあるStandalone設定の中の「Resolution and Presentation」を選択。
・Default Is Full Screenのチェックを外す
・Run In Backgroundのチェックを入れる
・Visible In Backgroundのチェックを入れる
・Force Single Instanceのチェックを入れる
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/3a2d7371-ffc4-b077-3001-a4b8ebfc6141.png)

## Player設定(XR)
UnityプロジェクトのXR設定を行います。
Standalone設定の中の「XR Settings」を選択。

・Virtual Reality Supportedのチェックを入れる
・「Oculus」を選択して、左下のマイナスボタンを押して削除。
・プラスボタンを押して、「None」を選択して追加
・「None」をドラッグして「OpenVR」より上に持ってくる

これで設定は完了です。
※OculusはVRオーバーレイ機能を提供していません。
　また、OpenVRを一番上に持ってくるとVRアプリケーションとして起動してしまうため、
　オーバーレイアプリケーションとして機能させるためにNoneで起動させています。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/c5484c57-49d6-46eb-65c6-8354fc76ed1a.png)

追記: Unity 2021を使用する場合は、SteamVR Unity Plugin v2.7.3 (sdk 1.14.15)を導入後、Unityを再起動し、下記のチェックを外すと同等

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/e81a8361-0668-ba20-f278-52ac8351d8dd.png)

また、EasyOpenVROverlayForUnity.csを開き、以下をコメントアウトするか削除

```cs
OpenVR.System.ResetSeatedZeroPose();
```

## RenderTextureを作成
Assetsを右クリックし、「Create」→「Render Texture」を作成。
ここでは「OverlayRenderTexture」という名前にします。

OverlayRenderTextureをクリックし、サイズを欲しい解像度に設定します。
ここでは800x600にします。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/96418490-0074-8a33-0cdb-c82b96ae1cd0.png)

## Cameraの準備
UIをメイン画面とオーバーレイの両方で表示するために少し工夫します。
HierarchyにGameObjectを作成し(ここでは「CameraMaster」とする)、ここにMain Cameraを子にします。
また、Main Cameraを右クリックからDupulicateして、名前を「Overlay Camera」にして、同じく「CameraMaster」の子にします。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/ffad49db-6394-bf26-e0c0-f30aa4a9643e.png)

次に、カメラの設定をします。
2つのカメラをCtrlを押しながら選択状態にしてください。(2つのカメラが同時に編集できる状態になります。)
以下の設定をします。
・Positionを0,0,0
・Potationを0,0,0
・Scaleを1,1,1
・Clear Flagsを「Solid Color」に

次に、選択を一旦解除して、Overlay Cameraだけを選びます。
以下の設定をします。
・Target Textureに先程作った「OverlayRenderTexture」を登録
・Audio Listenerのチェックを外す
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/d3c8cc47-db40-3d7e-e52e-6748d265bc63.png)

## Canvasの準備
UIを表示するためのCanvasを作成します。
Hierarchyを右クリックして、「UI」→「Canvas」でCanvasを作成します。

VR空間で表示するために以下の設定をします
・Render Modeを「World Space」にします。
・Event Cameraを「Overlay Camera」に設定

Rect Transformが編集できるようになるので
・PosX = 0, PosY =0, PosZ = 0
・Width = 800, Height = 600 (RenderTextureに合わせる)
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/39303a78-5425-f710-fe98-b266b46016e9.png)
また、Hierarchyにて、Canvasを右クリックして、「UI」→「Panel」を生成。
これでCanvasに白色が付きます。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/835d0569-9531-b788-e6c7-87a93512ab43.png)

## Overlayの準備
次にVR空間に投影するための準備をしていきます。
HierarchyにGameObjectを作成し(ここでは「OverlaySystem」とする)、そこにEasyOpenVROverlayForUnityをアタッチしてください。

そして以下のように設定します。
・Render Textureに「OverlayRenderTexture」を設定
・Widthを0.6に設定
・Alphaを1.0に設定
・Overlay Friendly Nameを「MyFirstOverlay」に設定(**注**)
・Overlay Key Nameを「MyFirstOverlay」に設定(**注**)
・Device Trackingのチェックを外す
・Device IndexをNoneに設定
・Laycast Root Objectに先ほど作成した「Canvas」を設定

**注意:Overlay Friendly NameおよびOverlay Key Nameは他のソフトと被ると同時に起動できません。(Key in useエラーが出ます)。
公開するアプリケーションを作成する際は必ずここを変更してください。
また同じツールを同時起動できるようにしたい場合は、ここが自動で変わるようにスクリプトを作成してください**
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/08c8e222-4c81-abe3-85e9-353e1fd05d01.png)

## カメラ位置の調整
「CameraMaster」を選択し、次の設定にします。
・Position X=0,Y=0,Z=-518
・Rotation X=0,Y=0,Z=0
・Scale X=1,Y=1,Z=1

Zの大きさによってCanvasがどれだけぴったりOverlayに表示されるかが変わりますので、CanvasやRender Textureの大きさを変えた場合は調整する必要があるかもしれません。
※特にCanvasを大きくした場合、離した際にCameraのClipping Planesに引っかかって見えなくなることがあるので、その場合はFarを大きな値にしてください
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/97faaec3-46c1-684d-986a-b98729ffc719.png)

## カーソルを作る
コントローラの位置を表示するためのカーソルを作ります。
Hierarchyにて、Canvasを右クリックして、「UI」→「Text」を生成。

以下の設定をします。
・名前を「CursorText」に
・width=100, Height=100
・Textは「+」
・Font Sizeを50に
・Alignmentを水平中央、垂直中央に。
・Raycast Targetをオフに

このオブジェクトを右クリックしてDupulicateして、2つにしておいてください。
右手、左手になります。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/34b176fb-f7d4-b8ac-c53a-7cdbedcc0dcd.png)

## カーソル表示・画面移動サンプルを導入
EasyOpenVRUtilのサンプルとして公開している以下のスクリプトをダウンロードします。
(開いて右上のRawをクリックし、その後、右クリックから名前をつけて保存)
https://github.com/gpsnmeajp/EasyOpenVRUtil/blob/master/sample/PositionManagerScript.cs
**v0.02: ビルドすると毎回起動時に初期位置が原点になる問題があったので修正しました**

PositionManagerScript.csをAssetに入れ、OverlaySystemにアタッチします。

以下の設定をします。
・Easy Open VR Overlayに、「OverlaySystem」を設定
・Left Cursor Text RectTransformに、「CursorText」を設定
・Right Cursor Text RectTransformに、もう一つの「CursorText」を設定
・Canvas Rect Transformに、「Canvas」を設定
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/7995e5ab-b897-517c-4cd8-21f56b7853e6.png)

# 動作確認
この時点で実行すると、VR空間に画面とカーソルが出ていると思います。
(Unityで再生をクリックした瞬間にHMDが向いている方向に出ているので注意してください)
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/aa9f0751-fb1f-ff08-c748-1b9c4ef87664.png)

# 注意: OpenVRとUnityEditorについて
UnityEditorで実行している場合、一度でもOpenVRが初期化された後、SteamVRを閉じてVRシステムを終了すると
OpenVRのハンドルが残ったままになり、以後SteamVRを開き直しても、Unityの再生ボタンを押しても、
UnityEditor自体を再起動するまでの間、オーバーレイが表示されなくなります。

つまりどういうことかというと
・1度でもUnityEditorでオーバーレイを立ち上げたなら(SteamVRも自動で起動するはず)、
　UnityEditorを終了するまでSteamVRを閉じないこと
・もしSteamVRを閉じてしまったときは、UnityEditor自体を再起動してプロジェクトを開き直すこと

ちなみに、ビルド後に実行する場合は、VRシステムを終了するとUnityプロセス自体が終了して
OpenVRが開放されるため、この問題は起きにくくなっています。

# 画面移動ボタンを配置
これだけでは面白くないので、画面を移動できるボタンを作ります。

まず、CanvasにButtonを作ります。
Hierarchyにて、Canvasを右クリックして、「UI」→「Button」を生成。

以下の設定にしてください。
・名前を「MoveButton」に
・PosX=0, PosY=0, PosZ=0
・Widhtを160に、Heightを160に
・OnClickの右下の「+」をクリック
・OnClickのObjectにOverlaySytemを設定
・OnClickの「NoFunction」をクリックし、「PositionManagerScript」→「MoveMode」を選択。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/bcb661d0-955b-fa97-d352-36e286bb9c8f.png)

Buttonの子として生成されているTextに以下の設定をします。
・Textを「Move」に変更
・Raycast Targetをオフに設定
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/3292fd96-f0e4-e179-c325-8407494e16c9.png)

最後に、カーソルがボタンに隠れないよう、CursorTextをCanvasの子の一番下に移動します。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/101922a6-4759-4692-97cd-0a754b73366f.png)

# 実行
なにかボタンを押しながら(あるいはトリガーを引きながら)Moveボタンをタップすると、Overlayの位置を移動できると思います。
ボタンを離すとその位置に固定されます。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/d458ee61-0d3c-8aa2-4d7c-8c6d6f86bd6a.png)

# 今後の展望
本記事では、掴んで移動できるVRオーバーレイの実装をご案内しました。
少なくとも私の観測では、VRオーバーレイアプリケーションというのは未だ数少ない分野となっています。
簡単なアプリケーションでも、VR空間で「生活する」という概念を獲得した人々にとってはとても便利なものとなるでしょう。

既存のVRオーバーレイが気に入らないことはよくあると思います。
自分の使い方にあったアプリケーションを作ってみてはいかがでしょうか？

例えば、以下のようなアプリケーションが考えられます。
・VR空間内写真立て
・VR空間内メモ帳
・VR空間内タイマー
・VR空間内キーボード
・VR空間内マウス
・VR空間内デスクトップモニター(hecomiさんのuDesktopDuplicationを使えばかなり簡単にできます！)
　http://tips.hecomi.com/entry/2016/12/04/125641
・uWindowCaptureを使って特定のウィンドウを持ち込んでも面白いです
　http://tips.hecomi.com/entry/2018/08/26/231618
　例: OBS向け放送プレビューVRオーバーレイをつくる
　https://qiita.com/kleus_balut/items/a3f1c4632d3b5b8ab33f

...その他いろいろ。

# おわりに
最後に、本ライブラリとスクリプトについて説明して終わりとさせていただきます。
・EasyOpenVRUtilは、オーバーレイの移動の際のコントローラの位置の取得に使っています。
　かなり素に近い姿勢情報を取り出すライブラリで加速度を取得したりするとジェスチャー操作にも使えます。
・EasyOpenVROverlayForUnityは、とにかく簡単にUnityからOpenVR Overlayを出せるようにするためのライブラリです。
　画面タップ機能はおまけで、位置の調整ができなかったり、**Hierarchyのトップ階層の**RaycastRootObject以下しか操作できなかったり、OnClickしか実装されていなかったり、TextとかのRaycastTargetに反応してしまったりという問題があります。改造するなどして対応してください。
・PositionManagerScript.csは、今回の解説のために自作ツールのスクリプトから抜き出してきたものです。
　特にHMDの位置に合わせた表示位置を生成するsetPosition()関数に関しては、本来より適切なタイミングで呼ぶべき処理なので、適時スクリプトから呼び出してください。
・画面を掴んで動かすときのガクガクが気になる場合は、Unityのフレームレートを90fps以上に設定してください。

それでは、本記事を元にVRオーバーレイアプリケーションが増えることを楽しみにしています。
