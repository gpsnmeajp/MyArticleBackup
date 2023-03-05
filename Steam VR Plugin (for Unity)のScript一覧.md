#はじめに
どこかにドキュメント無いんですかね...
[Behaviour]が着いているものはGameObjectにアタッチして使用する

Steam VR Plugin 2.0

#Scripts系
##SteamVR.cs
SteamVRのシステムの初期化や、デバイスへのアクセス、ネイティブハンドルなど。
VR機能の有効化・無効化(ゲームの設定画面)などもここの様子。
地味に便利なデバイスのシリアル番号などを取得するメソッドも含まれている。

##[Behaviour]SteamVR_Behaviour.cs
これをアタッチするとSteam VRが初期化される。
スクリプトから初期化したりするのに使うようだ。

##[Behaviour]SteamVR_Camera.cs
すでに存在するカメラに対してSteam VRサポートを提供する

##SteamVR_CameraFlip.cs
廃止済み

##SteamVR_CameraMask.cs
廃止済み

##[Behaviour]SteamVR_Ears.cs
スピーカー使用時の音声補正(?)
カメラの回転に対してオフセットを適用している。

##SteamVR_EnumEqualityComparer.cs
内部的に使用されているEnum比較器

##SteamVR_Events.cs
Steam VR内のイベントシステム
SteamVR_Events.DeviceConnected.Listen
SteamVR_Events.DeviceConnected.Remove
を使用することで、イベントを受信できる。
詳細はファイル内のコメントの使用説明を参照。

##[Behaviour]SteamVR_ExternalCamera.cs
外部カメラのレンダリング(客観視点?)

##[Behaviour]SteamVR_Fade.cs
SteamVR_Cameraに対して、視界のフェードイン・フェードアウトを行う。
(ブラックアウト、ホワイトアウトなど)
VRコンポジターを用いた完全なフェードも対応している。
詳細はファイル内のコメントの使用説明を参照。

##[Behaviour]SteamVR_Frustum.cs
視野を示すメッシュを生成する(?)
デバイスから物理視野情報を取得して、メッシュに反映する様子。

##[Behaviour]SteamVR_IK.cs
シンプルな2ボーンIK

##[Behaviour]SteamVR_LoadLevel.cs
切り替え対象のシーンを指定すると、指定した背景と挿絵を使ったプログレスバー付きロード画面を出す。
(いわゆる白世界)
ロードが終わるとシーンが切り替わる。

##[Behaviour]SteamVR_Menu.cs
SteamVR_Cameraのオーバーレイ機能を利用した、OnGUIでのメニューのサンプル

##[Behaviour]SteamVR_Overlay.cs
仮想スクリーンを利用した2Dコンテンツの表示
事実上のOpenVR Overlayのサンプルスクリプトであり、ライブラリ。
なおオーバーレイ機能のため、Steam VRのメニューと同等の優先度で表示される。

私がこれを作った意味とは...
https://qiita.com/gpsnmeajp/items/421e3853465df3b1520b

##[Behaviour]SteamVR_PlayArea.cs
キャリブレーションされた範囲(あるいは指定の範囲)に基づき、プレイエリアを表示します。

##[Behaviour]SteamVR_Render.cs
設定ファイルに基づき、すべてのVRカメラのレンダリングを処理する。
外部カメラのレンダリング処理も行う。

##[Behaviour]SteamVR_RenderModel.cs
コントローラーなどのtracked objectの描写を行う。

##SteamVR_RingBuffer.cs
内部的リングバッファ実装

##SteamVR_Settings.cs
エディタ拡張

##[Behaviour]SteamVR_Skybox.cs
VRコンポジターに対してCubemapを設定する。
おそらくロード画面などの背景が変更される。

##[Behaviour]SteamVR_SphericalProjection.cs
オブジェクトに球形投影シェーダーを適用する
360°写真用？

##SteamVR_TrackedCamera.cs
HMDに搭載されたカメラの映像を取得することができる。
詳細はファイル内のコメントの使用説明を参照。

##[Behaviour]SteamVR_TrackedObject.cs
指定したデバイスIDの姿勢をGameObjectにリアルタイムで反映する

##SteamVR_Utils.cs
内部で利用されている様々な処理
・数学演算・行列演算
・変換行列からスケールや回転・位置を取り出す処理
・OpenVRの初期化
・ステレオ画像によるスクリーンショットの撮影
・\と/の変換
など

#Input系
将来的には右手左手以外も扱えるようにしたい傾向が見受けられる

##[Behaviour]SteamVR_ActivateActionSetOnLoad.cs
##[Behaviour]SteamVR_Behaviour_Boolean.cs
右手左手のボタンの押下などのイベント処理

##[Behaviour]SteamVR_Behaviour_Pose.cs
右手左手の位置や角度などの姿勢情報をオブジェクトに自動反映する。
速度などの高度な推定情報も内部で自動処理する。

##[Behaviour]SteamVR_Behaviour_Single.cs
右手左手の単一アクション変化のイベント処理

##[Behaviour]SteamVR_Behaviour_Skeleton.cs
右手左手の指まで含めた動きを再現する

##[Behaviour]SteamVR_Behaviour_SkeletonCustom.cs
右手左手の指まで含めた動きを再現する。
使いたい関節情報だけ使用することができる

##[Behaviour]SteamVR_Behaviour_Vector2.cs
右手左手のVector2変化のイベント処理

##[Behaviour]SteamVR_Behaviour_Vector3.cs
右手左手のVector3変化のイベント処理

##SteamVR_～Action～.cs
##SteamVR_Input.cs
アクション設定ファイルの読み込みやその他初期化処理

##SteamVR_Input_ActionFile.cs
##SteamVR_Input_ActionScopes.cs
##SteamVR_Input_ActionSetUsages.cs
##SteamVR_Input_Generator_Names.cs
##SteamVR_Input_References.cs
##SteamVR_Input_Source.cs
##SteamVR_Input_Sources.cs
##SteamVR_UpdateModes.cs

#OpenVR系
##openvr_api.cs

#おまけ
・VRゲームにオーバーレイ表示したい人向けサンプル(OpenVR Overlay)
Overlay関係の資料の無さに嘆いて作成
https://qiita.com/gpsnmeajp/items/421e3853465df3b1520b

・EasyOpenVROverlayForUnity.cs
Unity用OpenVRオーバーレイ表示支援スクリプト
DashboardOverlayもある。
https://sabowl.sakura.ne.jp/gpsnmeajp/unity/EasyOpenVROverlayForUnity/

・🐰VaNiiPopS - Virtual Notification Popup Sytem
VR空間通知中継システム(2Dのソフトウェアから簡単にVR空間へ)
https://sabowl.sakura.ne.jp/gpsnmeajp/unity/vanii/

・VRツール一覧まとめwiki(VR Tool Wiki)
VRソフトウェアや補助ツールなどをまとめている
http://pc-vr.game-info.wiki/d/VR%a5%bd%a5%d5%a5%c8%a5%a6%a5%a7%a5%a2
