#はじめに
Steam VRとOpen VRを使って色々*作ってきたのですが、Open VRはC++向け(というかC言語)のもののためか
色々とお世辞にも使いやすい形をしていません。

で、それを手助けしようとするライブラリを作ろうと思ってふと疑問になり、
Steam VR Pluginのスクリプトを読んだら、今までの苦労がなんだったんだというくらい
整理されたものを見つけてしまいました。

検索能力の低さゆえか、Steam VR Pluginのドキュメントというのを見つけられていないので
ソースを読んで気づいたことを逐次ここに記入していきます。

*以下
VaNiiPopS - Virtual Notification Popup Sytem
2DソフトウェアからVR空間へ、スマートフォンの通知のような「プッシュ通知」を実現する橋渡しツール
https://sabowl.sakura.ne.jp/gpsnmeajp/unity/vanii/

EasyOpenVROverlayForUnity.cs
OepnVRを用いたオーバーレイ表示の支援スクリプト
https://sabowl.sakura.ne.jp/gpsnmeajp/unity/EasyOpenVROverlayForUnity/

publicで外から使えるものだけを載せていく予定です。
Steam VR Plugin 2.0を確認しています。

#SteamVR.cs
Valve.VR.SteamVR
なお、private SteamVR()を読むと、VRで表示するための一連の流れがわかる。

##static bool active 
SteamVRのインスタンスが生成されているかチェック

##static bool enabled
VRシステムとSteamVRが有効かをチェックしている。
trueにすると初期化、falseにするとインスタンス破棄。
ゲームの設定画面での有効無効の切り替えを想定している様子。
初期値はtrue

##public static SteamVR instance
インスタンスを返す。インスタンスがない場合はシングルトンを生成する。
(VRコンポジターの初期化や、オーバーレイの初期化などが行われる)

##static void Initialize(bool forceUnityVRMode = false)
インスタンスがない場合はシングルトンを生成する。
forceUnityVRMode = trueで呼び出すとSteamVR_Behaviour.InitializeSteamVR(true)を呼び出す。
enabled=trueならばSteamVR_Behaviour.Initialize()を呼び出す

##static bool usingNativeSupport
UnityがXRをネイティブサポートしているならtrue
falseの場合はSteam VRの初期化に失敗する。

##static SteamVR_Settings settings
???

##CVRSystem hmd
ネイティブ機能OpenVRのインスタンス
OpenVRの各種機能を呼び出すのに必要になる

##CVRCompositor compositor
ネイティブ機能VRコンポジッターのインスタンス
映像処理や、画面のフェードなどに直接アクセスするのに使用する

##CVROverlay overlay
ネイティブ機能オーバーレイのインスタンス
ゲーム外に表示処理をする時に必要になる。

##static bool initializing
tracking status

##static bool calibrating
tracking status

##static bool outOfRange
tracking status

##static bool[] connected
デバイスの数だけ存在する
tracking status

##float sceneWidth
render values

##float sceneHeight
render values

##float aspect
render values

##Vector2 tanHalfFov
render values

##VRTextureBounds_t[] textureBounds
render values

##SteamVR_Utils.RigidTransform[] eyes
render values

##ETextureType textureType
render values

##string hmd_TrackingSystemName
hmd properties

##string hmd_ModelNumber
hmd properties

##string hmd_SerialNumber
hmd properties

##float hmd_SecondsFromVsyncToPhotons
hmd properties

##float hmd_DisplayFrequency
hmd properties

##string GetTrackedDeviceString(uint deviceId)
Prop_AttachedDeviceId_Stringを取得する

##string GetStringProperty(ETrackedDeviceProperty prop, uint deviceId = OpenVR.k_unTrackedDeviceIndex_Hmd)
指定されたデバイスの文字列プロパティを取得する(デバイスのシリアル番号など)

##float GetFloatProperty(ETrackedDeviceProperty prop, uint deviceId = OpenVR.k_unTrackedDeviceIndex_Hmd)
指定されたデバイスの数値プロパティを取得する(バッテリー残量など)

##void ShowBindingsForEditor()
UnityEditor利用時のみ有効
controllerbinding.htmlに情報を出力する。

##static string GetResourcesFolderPath(bool fromAssetsDirectory = false)
Steam VRリソースフォルダのパスを取得する

##const string defaultUnityAppKeyTemplate

##const string defaultAppKeyTemplate

##static string GenerateAppKey()

##static string GetManifestFile()
VRmanifestファイルの場所を取得する

##static void IdentifyApplication()

##void Dispose()

##static void SafeDispose()

#SteamVR_TrackedObject.cs
トラッカーやコントローラーなどの位置情報を管理する。
GameObjectに付加すると自動で位置を合わせてくれる。
新しい姿勢になったというイベントを元にOnNewPosesが呼び出され、自動で取得している

##enum EIndex
デバイスID情報(ただし固定されているのはHmdくらい)

##EIndex index
選択されている位置情報デバイス

##Transform origin
現在の位置情報(自動で更新される)

##bool isValid
有効か

##void SetDeviceIndex(int index)
デバイスIDを設定する

#SteamVR_Fade.cs
VRコンポジターによるフェード(画面のホワイトアウトやブラックアウトなど)を管理する

##static void Start(Color newColor, float duration, bool fadeOverlay = false)
フェードを開始する

##static void View(Color newColor, float duration)

##void OnStartFade(Color newColor, float duration, bool fadeOverlay)
