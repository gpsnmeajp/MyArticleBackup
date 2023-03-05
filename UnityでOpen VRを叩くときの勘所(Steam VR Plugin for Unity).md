#はじめに
Unityで、例えばVRアプリケーションではないものからOverlayを使いたいときや、コントローラーの詳細情報が欲しいときなどは、Open VRの機能に直接アクセスしたほうが早いことがあります。
私もそうして知見が溜まってきたので、メモを残しておきます。


#メモ
##公式のWikiはアテになったりならなかったりする
とりあえず困ったら、まずはAPI Documentationを見ましょう。
https://github.com/ValveSoftware/openvr/wiki/API-Documentation

しかしながら、情報が不足していたり古かったりします。
openvr_api.csや、Steam VR Pluginのソースの中にヒントや答えがあることがあります。
新機能がしれっと増えていたりします。

他に当てになる情報は、正直ないです。
他の人の作ったサンプルソースを探してさまよう事が多いです。

##だいたい必要な定数は
OpenVR.k_～で定義されている

HMDのindexとか
k_unTrackedDeviceIndex_Hmd = 0;
デバイスの最大接続数とか
k_unMaxTrackedDeviceCount = 64;

##シングルトンなんだよなぁ
OpenVR.System
OpenVR.Overlay
などのクラスは、内部でシングルトンになっているので、引っ張り回す必要なく軽率に使える。

逆にこれらはstaticに確保されているようで、アプリケーションを終了する(UnityEditorで動作させている場合はUnityEditorを終了する)まで、変な挙動をし続けることがある。

なお、OpenVR.Systemがnullを返してくる場合は、OpenVRが適切に初期化されていない証拠となる。

##HMD持ってるユーザーか調べたい
IsHmdPresentを使うと、高速に判定できる。
ただしTrueだからといって本当に使える環境とは限らない

##初期化の必要性
OpenVR.Initは、すでに同じアプリケーション内にVRを動作させているやつがいる場合はやってはいけない。
一方、誰も居ない場合、明示的に呼び出してやらないと恐ろしく不安定な動作になる。
基本的にこれを手動で呼び出す場合は、ApplicationTypeはOverlayだろう。

ここで帰ってくるCVRSystemは、OpenVR.Systemで取得できるものと同じものだが、nullが帰ってくる場合は異常。

##明示的なshutdownは不要
基本的に不安定になるので不要。
きっちりオブジェクトが開放されていればそれで問題ないはず。
