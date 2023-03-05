# はじめに
OpenXR Driverの仕組みがまだ無い現在、自作のVRデバイスを作る場合はOpenVR一択でしょう。([PimaxXR](https://github.com/mbucchia/Pimax-OpenXR)とかを使わない限り)

Virtual Motion Trackerは、バージョン0.14にてSkeletal Inputに対応しました。
これによって、ハンドトラッキングできる自作機器やソフトが実装できるようになりましたが、これを実現するのに苦労した点を記録しておきます。
OpenVR Driver開発者への参考にどうぞ

https://gpsnmeajp.github.io/VirtualMotionTrackerDocument/

前回

https://qiita.com/gpsnmeajp/items/9c41654e6c89c6b9702f

# 初歩
## Skeletal Input以前に

まず、Skeletal Input (Skeleton Inputという表記もありますが)を使うには、前提条件として、Action Inputを理解してください。
ボタンや軸の入力をきちんと実装できるようになってということです。

+ CreateBooleanComponent, UpdateBooleanComponent : ボタンや軸のタッチ(Touch)や押し込み(Click)
+ CreateScalarComponent, UpdateScalarComponent : スライダーやジョイスティックの倒し具合
+ CreateHapticComponent : 振動

これらと、Prop_InputProfilePath_Stringで設定します。

## Skeletal Inputの理解
下記資料を読んで理解します。

https://github.com/ValveSoftware/openvr/wiki/SteamVR-Skeletal-Input

https://github.com/ValveSoftware/openvr/wiki/Hand-Skeleton

https://github.com/ValveSoftware/openvr/wiki/Creating-a-Skeletal-Input-Driver

## Skeletal Inputの実装
Driverにできることは以下の2つです。

+ CreateSkeletonComponent で、骨格情報を定義する。これはInputProfileに記述した情報と合わせる必要があります。各コントローラに1つ
+ UpdateSkeletonComponent で骨格情報(配列)を渡して更新します。

[サンプルコード](https://github.com/ValveSoftware/openvr/wiki/Creating-a-Skeletal-Input-Driver)からわかるように、非常にシンプルです。

骨格情報ってなんやねんというと、[この資料](https://github.com/ValveSoftware/openvr/wiki/Hand-Skeleton)に書いてあります。

まず、以下の手が基準です。

+ vr_glove_left_model.fbx
+ vr_glove_right_model.fbx

(ちなみにUnity上で読み込むと、位置xと、クォータニオンx,wが反転するようです)

ボーンは31本あり、そのうち描画に関係するのは26本で、5本は計算やアニメーションに使用する補助ボーンです。(重要、後に記述する不具合に関係)

つまり、以下のような感じということです。

```cpp
VRInputComponentHandle_t SkeletonComponent{ 0 };
vr::VRBoneTransform_t m_boneTransform[31];

//登録
CreateSkeletonComponent(m_propertyContainer, "/input/skeleton/left", "/skeleton/hand/left", "/pose/raw", EVRSkeletalTrackingLevel::VRSkeletalTracking_Partial, nullptr, 0, &SkeletonComponent);
CreateSkeletonComponent(m_propertyContainer, "/input/skeleton/right", "/skeleton/hand/right", "/pose/raw", EVRSkeletalTrackingLevel::VRSkeletalTracking_Partial, nullptr, 0, &SkeletonComponent);

//更新(2つで1セット)
UpdateSkeletonComponent(SkeletonComponent, EVRSkeletalMotionRange::VRSkeletalMotionRange_WithController, m_boneTransform, skeletonBoneCount);
UpdateSkeletonComponent(SkeletonComponent, EVRSkeletalMotionRange::VRSkeletalMotionRange_WithoutController, m_boneTransform, skeletonBoneCount);
```

EVRSkeletalTrackingLevel::VRSkeletalTracking_Partial　は、そういうもんです(全く追跡できない場合や、完全に追従できる場合を除き、基本的にこれ)
制限範囲としてnullptr, 0　を指定すると、制限範囲がないFist相当になります。

更新は、コントローラ付きとなしで必ず1セットになります。

## Skeletal Inputの検証
SteamVR自体に内蔵されているコントローラのテストでは、ボタンやトリガー等はわかりますが、Skeletal Inputは確認できません。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/ebf63a8d-56e7-77d4-9956-c6a21b385453.png)

Skeletal Inputの検証には対応アプリケーションが必要です。
バーチャルモーションキャプチャーや、Half-Life：Alyxなど。

テストツールを作りましたので、こちらでも検証できます。

https://github.com/gpsnmeajp/SkeletonPoseTester

# 泥臭い話
## 参考にするものがない、ググっても出てこない
UpdateSkeletonComponentとかググっても情報が出てこないんだこれ。
オープンソースで対応してるドライバが全然引っかからないので、自分しか作ってないかと思いました。

もちろんそんなことはないです。
参考にできるオープルソースのコードは以下です。


https://github.com/search?q=UpdateSkeletonComponent&type=code

## PCが死ぬ
SteamVRが機嫌損ねると、PCが綺麗にフリーズします。一切応答しなくなります。
マウスやキーボードも反応しなくなり、音楽も止まります。電源を強制的に切るしかないです。

2つ以上のUnity EditorでSteamVRを起動したり操作したり描画を握ってると高確率で起きます。
ドライバがログを吐きすぎても起きる気がします。

開くものを最小限にしたり、SteamVRを終了したときにはUnityEditorも終了するなど対策が必要です。

## なんの手を基準にすればええねん
手の形を一から作るのは大変ですよね、SteamVR側もそれはわかっていて、EVRSkeletalReferencePoseという基準姿勢を用意しています。
ここには、基準姿勢、手を完全に閉じた状態、手を完全に開いた状態の情報が入っています。

Driverでは取得できないので、取得用のアプリケーションを作成するなどして取得してください。また、左右の手別に存在する(間違えると手がおかしくなる)ので、ご注意を。

https://github.com/ValveSoftware/openvr/wiki/SteamVR-Skeletal-Input

## 静的姿勢ズレてるんだけど... (続く)
で、VRSkeletalReferencePose_BindPoseを適用してみてから、VRSkeletalReferencePose_FistやVRSkeletalReferencePose_OpenHandを適用してみると、違和感に気づくと思います。

なんか手の位置が違わない...？
これ、Indexコントローラと比較してみると、BindPoseがズレていて、FistやOpenHandが正しいのですが、位置関係から見るとどう見てもBindPoseのほうが正しそうに見える。

結論から言うと、FistとOpenHandを使っておいてください。ただし、この違和感は正解です。後述。

## ゲームソフトに認識されない
### その1: Input ProfileとBinding
独自のモデル名とか設定して、独自のInput Profileを作るまではいいんです。
足りない項目はきちんと埋めればよいのです。

問題は、そんな新参コントローラをVRゲームソフトは一切知らないということです。

知らないコントローラの操作は受け付けてくれません。
なので、バインディングを誰かが作ることになります。
主な選択肢としては以下。

+ ユーザーが頑張ってSteamVR上でバインドする
+ ドライバ内蔵のバインディングとして各アプリをサポートする
+ 各アプリの製作者にお願いして組み込んでもらう

うーん、どれも辛い。

### その2: Legacy binding (主にVRChat)
SteamVR Action Input Systemは後からできたシステムです。
なので、先にできたゲームソフトには、古い入力システムで動いています。そのためにLegacy Bindingが必要なのは知ってのとおりです。(VRChatもこの古い入力システムです。)
なので、Legacy Bindingを設定してあげることで、ボタンや軸を使えます。

あれ？VRChatで使うとVIVE Controllerとして見えるぞ？指動かないぞ？

ところで、Skeletal Inputは、Action Input Systemの上で動いているものなので、Legacy bindingは対応していません。

あれ？

Legacy Bindingなのに、指が動くのはなぜでしょう。

正解は、Valve Indexの指検出は軸としても取得可能だからです。
つまり、トリガーなどと同じ扱いで取得できます。Skeletal Inputとは別に、です。
この場合は、Legacy Bindingでも扱うことができます。

「あー、じゃあ、自作ドライバでも軸として指を用意すればいいのね」
その通りです。でもそううまくは行きません。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/e2c2cc51-1231-e4a0-831a-2d20e25cb3a4.png)
(VMTも、軸とSkeletalの両方で指が用意されています。軸の方に親指がないですね？はい。Index Controllerに親指の軸というものが無いからです。)

### その3: Legacy bindingかつIndex専用仕様
Action Inputは、異なるコントローラの違いをSteamVR側で吸収するシステムです。そのためにBindingや、コントローラモデルの表示、指の扱いなどがあるわけです。

そうではないということは？
ゲーム側で違いを吸収しているということです。

ゲーム側で違いを吸収しているということは、ゲーム側で何かを読み込んでコントローラを区別して読み込む内容を決めているということになります。

指が読み込めるのはIndexだけです。
はい、つまり、Legacy bindingで指が動く = Index Controllerということです。
自前のコントローラにいくらLegacy bindingを設定しようと、不明なコントローラでしかありません。

**詰み** と思うかもしれませんが、これは案外簡単に回避できます。

DirverのInput Profileを以下のようにするだけです。

```json
  "jsonid" : "input_profile",
  "controller_type": "knuckles",
  "device_class" : "TrackedDeviceClass_Controller",
  "legacy_binding" :  "{indexcontroller}/input/legacy_bindings_index_controller.json",
  "input_bindingui_mode" : "controller_handed",
(略)
```

これにより、コントローラはIndex扱いになり、Legacy bindingもシステムにもとからあるのが使われます。

**注意: input_sourceもIndexと同じにする必要があります。つまりDriverのC++コードのレベルで修正が必要です。**

具体的にどのinput sourceを作らないといけないかは、[ご丁寧にOpenVR公式できちんと書いてあります](https://github.com/ValveSoftware/openvr/wiki/IVRDriverInput-Overview#:~:text=Existing%20input%20component%20paths)。

### その4: Steam上からダウンロードされる問題
「legacy bindingを自前で用意すれば、Input Sourceを合わせなくても良くない？」

はい、理論上はそうです。Input SourceはどうせLegacy bindingを経由して置き換わりますから、適切に書けば良いはず。
でもダメです。

なぜなら、BindingはローカルファイルよりもSteam上にあるものが優先されます。これはWebConsoleをきちんと観察するとわかりますし、[ドライバのものより優先されるとここに書いています](https://github.com/ValveSoftware/openvr/wiki/Input-Profiles#default-bindings)。

なので、システム内臓のlegacy bindingsで動くようにInput Sourceごと合わせる必要があります。

結果として「その1」の問題も解決します。なんたってIndex Controller扱いですからね。大抵対応してます。

### その5: 結局Indexのフリをする
負けた気分！
でも、RenderModelとか、設定画面とかは自前のままにできます。あくまでアプリケーションからの認識だけです。
もっとも、ボタンや軸の数も合わせないといけない(多く付けても読み込んでもらえない)ですけどね！

なので、VMTでは、通常モードと互換モードという形で使い分けできるようにしています。

## ボーン吹っ飛んでるんですけど
補助ボーンとかをきちんと実装しようと、そういえばIndexコントローラだとボーンってどうなっているんだろうと思うわけじゃないですか

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/7b4d369d-be8a-580d-9896-afa97a1dd25e.png)

ｷﾞｬｰ
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/36b17f0a-676b-6b5d-0932-1c7aa9275569.png)

これ、BindPoseに対してFistやOpenHandがズレてる分だけ、rootがおかしくなっています。
どうも、これで正常(???)の様子。というか、バグってるのが実装されているようです...

というのも、rootがズレていたり、AUXボーンが吹っ飛んでいても、描画にはあんまり影響しないからですね。
Index Controllerを正式実装した時におかしくなったのかな...

# おわりに: それでもOpenVRが好き
それでもコントローラを自由に実装できて、違う構造を吸収できて、骨格の共通仕様を用意しているのは(OpenXRを除いて)OpenVRのみだったと思います。
共通仕様があるということは、共通仕様を理解している者同士であれば正常に動くということ。

実際、多種多様なVRゲームではきちんと動くわけです。
SteamVR Pluginで開発されたソフトでも動くし、OpenXR HandTrackingとしてもきちんと動くようです。

やっぱOpenVRだな、と思うわけです。

# おまけ
指のIKが必要になったら、Final IKのFinger Rigが便利そうです

https://twitter.com/Seg_Faul/status/1614314125441761280?s=20&t=WSvUv5rpTFAcYMEP_BMx0VZpgD0iTT8frbtH97mofws




