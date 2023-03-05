#はじめに
Steam VR Input Systemについての読み解きメモです。
参考になるかもしれませんし、ならないかもしれません。
調べながら書いているので、誤りが多分に含まれる可能性があります。

普通にUnityから使いたい人は関連記事をご参照ください。
こちらはどちらかというと、APIレベルの話です。
(オーバーレイアプリケーションや、その他から使う場合を想定)

#関連記事
SteamVR Plugin 2.0 Input取得・入力関連スクリプト読み解き 
https://qiita.com/kyourikey/items/232f7810769c7727c9bd

UniVRM+SteamVR+Final IKで始めるVTuber 
https://qiita.com/sh_akira/items/81fca69d6f34a42d261c

最近のSteamVRにおけるViveTracker挙動が変な理由と対処法(1538437695) 
https://qiita.com/neon-izm/items/39417171bfda5eba807e

SteamVR Plugin 2.0 セットアップ・サンプルシーンを動かすまでの備忘録
https://qiita.com/kyourikey/items/5967018e613b2fd49a1f

#参考文献
SteamVR Input (ValveSoftware/openvr)
https://github.com/ValveSoftware/openvr/wiki/SteamVR-Input

Action manifest (ValveSoftware/openvr)
https://github.com/ValveSoftware/openvr/wiki/Action-manifest

Input Profiles (ValveSoftware/openvr)
https://github.com/ValveSoftware/openvr/wiki/Input-Profiles

OpenXR Overview
https://www.khronos.org/openxr

［GTC 2018］Khronosが語る「Vulkan 1.1」。VR＆AR向けAPI「OpenXR」の最新動向も - 4GamerNews
https://www.4gamer.net/games/293/G029343/20180330081/

#用語集
##Input Profiles
ドライバが扱える、HMDやコントローラの入出力情報をJSON形式で表したもの。
InputSource(出力含む)を定義する。
古いドライバなど、プロファイルを持たない場合はSteam VR側である程度のものが生成される。
また、アクション形式に対応しない古いアプリケーションのためのデフォルトバインディングも定義できる。

以下の形式は、Input Profilesで定義されるものである。

```
/input/xxx/yyy
/input/trigger
/input/pose/raw
/input/skeleton/left
/output/xxx/yyy
/output/haptic
/pose/xxx
```

###InputSource
入出力系統と訳すのが正しそうという印象を受ける。

例えば、HMD、左コントローラ、右コントローラなど。
しかしながら、実際はデバイス単位というよりは「ユーザーがどこに装着しているか」をベースに関することになっているようである。
(記述、およびUnity Pluginの形式から)

これにより、どの手から操作が来たのかなどを判別・フィルタできる。

以下の形式は、InputSourceと定義されるものである

```
/user/head
/user/hand/left
/user/hand/right
/user/hand/gamepad
```

これが組み合わさって、以下のようなinputおよびoutputのpathとなる。

```
/user/hand/left/input/xxx/yyy
/user/hand/left/input/trigger
/user/hand/left/input/pose/raw
/user/hand/left/input/skeleton/left
/user/hand/left/output/xxx/yyy
/user/hand/left/output/haptic
```

##Action manifest
アプリケーションが期待する入出力情報をJSON形式で表したもの。
この情報を元にVRシステムは、ユーザーにコントローラとの紐付けをしてもらう。

単に入出力のパス・型情報のほか
・各アクションがどういう意味を持つかをローカライズした説明
・割り当て要求: 必須/推奨/オプション
・割当スタイル: 単一コントローラにのみ/左右のコントローラに個別
・一般的なコントローラに対する既定の設定
を提供する。

一方、ユーザーは説明を見ながら、ボタンを割り当てなおしたり、振動デバイスを振り分けたり、全く新規のデバイスに割り当てることができる(Controller bindings)。

以下の形式は、Action manifestで定義されるものである。

```
/actions/(action set)/in/(action name)
/actions/(action set)/out/(action name)

/actions/default
/actions/default/in/pose
/actions/default/out/haptic

/actions/buggy/in/throttle
/actions/buggy/in/brake
```

###Action
デジタルボタン入力、アナログスティック入力、振動出力といった、入出力1単位にアプリケーション独自の名前をつけたもの。

冗長なボタンを用意しておき、既定では割り当てておかないが、ユーザーが割り当てたいときには割り当てられるようにすることもできる。

また、振動など、最終的に同じ出力にバインドされるものでも、別々に分けておくことができ、将来的なデバイスの拡張に備えておくことができる。


###Action Set
Actionをまとめたもの。

入力の取得の際にはこれが用いられるため、例えばいつでも使うボタン達、運転時にしか使わないボタンたちなどをまとめておき、モードなどに合わせて取得単位を切り替えることができる。

これはバインディングUIにも反映される。

##Controller bindings
Input Profilesで定義された物理的な入出力と、Action manifestで定義されたアプリケーション側の入出力を、結合(binding)するためのJSONファイル。

Steam VRのUIが生成するため中身を直接触ることは基本無い。
簡単に言えば、pathとactionを結合する形になっている。

outputは直結なことが多いが、inputの場合は、アナログ入力をしきい値を設けてデジタル入力にするなどのことも行っている。

```
output: "/actions/default/out/haptic"
path:"/user/hand/left/output/haptic"
```

サポートしたい各コントローラ(おそらくInput Profiles)に対して1ファイル作り、Action manifestとともに製品に同梱して配布することとなる。
これをDefault binding fileという。

また、一方でユーザーは、Default bindingをベースに説明を見ながら、ボタンを割り当てなおしたり、振動デバイスを振り分けたり、全く新規のデバイスに割り当てることができる。

開発者も、新しいコントローラが出た際にはbinding fileを追加するだけで対応ができる。

#Steam VR Input Systemとは
OpenXRは、「アプリケーション側の入出力と、デバイス側の入出力を抽象化して共通化し、開発者の負担を減らし、将来的なデバイスの拡張にも対応しやすくする」ことを目的として制定されています。
Steam VR Input Systemは、まさにこのOpenXR規格の手法を用いた入出力の抽象化レイヤーのようです。
(規格に準拠しているかは不明ですがかなり似ています)

#抽象化の構造
**入力プロファイル**
ドライバが、VRシステム側に対して、入出力の情報を定義する

**コントローラバインディング**
VRシステム側がコントローラ入出力を、どうVRアプリケーションのアクションに変換するかを定義する。
(これは、VRアプリケーションがデフォルトを用意するが、ユーザー側がGUIで編集でき、
　開発時において未知のコントローラや追加機器も手動で割り当てて拡張できる)

**アクションマニフェスト**
VRアプリケーション側が、各入出力アクションがどういうものかを定義し、
入出力バインディング設定に必要な情報を提供する。(型・ローカライズされた説明など)

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/897c188a-1eef-463b-9bc9-9f84e178bdf7.png)


アクションは、ボタン、ジョイスティック、タッチ位置などの入力のほか、
振動などの出力も扱う。

この際、割り当てに関して「必須」「推奨」「オプション」が指定できる。
つまり、ボタン数の多いコントローラであればいろいろ機能を割り当てることもできるし、ボタン数の少ないコントローラなら必要最低限割り当てて実行することもできるようになっている。

また、出力に関して、現状は振動源はコントローラしか無いが、複数の意味を別のアクションにして、出力先はコントローラへの出力にまとめておくことができる。これは、将来的に振動ベストなどの異なった振動源デバイスが追加された際、ユーザー側で銃の振動であればコントローラ、銃撃を受けたときなら振動ベスト、のように割り当て直すことを可能にする。

#開発の手順
1. Steam VRの開発者設定から、入力バインディングに関するデバッグオプションを有効にする
2. アクションマニフェストファイルを作成しておく(UnityならSteam VR Pluginから作成できる)
3. アプリケーションからOpenVRにマニフェストファイルを読み込ませる
4. アプリケーションを実行する
5. Steam VRからバインディングを作成する。(VR内あるいはブラウザ: 関連記事を参照)
6. 作成後、「デフォルトバインディングの置き換え」を選択すると、アクションマニフェストと同じフォルダにデフォルトバインディングが生成される
7. アクションマニフェストのデフォルドバインディング指定に、生成されたデフォルドバインディングを指定する。
8. 他のタイプのコントローラに対する割り当ての分、5～7を繰り返す。
9. ソフトのリリース時に、アクションマニフェストとデフォルドバインディングをアプリケーションと共に配布する。

#API
https://github.com/ValveSoftware/openvr/wiki/SteamVR-Input
の概要をメモ。詳しくは上記のURLへ。

###SetActionManifestPath(actionmanifestpath)
manifestpath: アクションマニフェストの絶対パス

この呼び出しは、IVRInput::UpdateActionStateまたはIVRSystem::PollNextEventが最初に呼び出される前に行う必要がある。
(おそらく、これらが先に呼び出されると互換性のためLegacy inputになる)

呼び出し以降、GetControllerStateが使えなくなるようである。
GetControllerStateが使えないと、従来のやり方ではボタンもスティックもタッチパッドも取れなくなる。
GetDeviceToAbsoluteTrackingPoseは使えるため、HMDの姿勢は取れるし、コントローラの姿勢などもこちらから普通に取れる。

なお、Steamパートナーサイトにてアクションマニフェストに関する設定がある場合はこの設定は無視される。
Steamパートナーサイトでの設定と合わない場合は、エラーが返る。

###GetActionSetHandle(name,handle)
name: アクション**セット**のパス(/actions/xxx)
handle: ハンドルの返却

使用するアクションセットの登録となる。アクションセットごとに呼び出し、ハンドルを保管する。
アクションを取得する際は、このアクションセットが1単位となる

###GetActionHandle(name,handle)
name: **アクション**のパス(/actions/xxx/in/yyy)
handle: ハンドルの返却

使用するアクションの登録となる。アクションごとに呼び出し、ハンドルを保管する。
ボタン・スティックの1つ1つに相当する

###GetInputSourceHandle(source,handle)
source: 入力ソースへのパス(/user/hand, /user/gamepadなど)
handle: ハンドルの返却

使用する入力ソースの登録となる。アクションごとに呼び出し、ハンドルを保管する。
右手、左手、頭などを区別するために使用する。

###UpdateActionState(sets,size,count)
sets: このフレームで有効にしたいVRActiveActionSet_tの配列へのポインタ
size: sizeof(sets)
count: sets.length

指定したアクション**セット**を更新する。これにより後述のGet*ActionDataが更新される。
1フレームに1回呼び出されることが想定されている。

###VRActiveActionSet_t
```C#
struct VRActiveActionSet_t
{
	VRActionSetHandle_t ulActionSet;
	VRInputValueHandle_t ulRestrictedToDevice;
	VRActionSetHandle_t ulSecondaryActionSet;
};
```

ulActionSet: アクティブにしたい**アクションセット**のハンドル
ulRestrictedToDevice: 入力ソースのハンドルを設定すると、特定のソースからのみ反応するようになる。右手のみなど。
　k_ulInvalidInputValueHandleで無効にする。
ulSecondaryActionSet: ulRestrictedToDeviceが設定されているとき、別の入力ソースで使う**アクションセット**。

###GetDigitalActionData(action,actionData,actionDataSize)
action: デジタルアクションのハンドル
actionData: InputDigitalActionData_tへのポインタ
actionDataSize: sizeof(InputDigitalActionData_t)

InputDigitalActionData_tでは、
ボタンが存在するか、どの入力から最後に入力されたか(入力ソースハンドル)、現在の状態、状態が変化したか、最後に状態が変化してからの時間が取得できる。
※この入力Sourceハンドルは、自分が登録したものとは別になる。
　おそらくGetOriginTrackedDeviceInfoを使って詳細を取得してやる必要がある。

###GetAnalogActionData(action,actionData,actionDataSize)
action: アナログアクションのハンドル
actionData: InputAnalogActionData_tへのポインタ
actionDataSize: sizeof(InputAnalogActionData_t)

InputAnalogActionData_tでは、
アナログ入力が存在するか、どの入力から最後に入力されたか(入力ソースハンドル)、現在の状態、状態が変化したか、最後に状態が変化してからの時間が取得できる。

###GetPoseActionData(action,eOrigin,seconds,actionData,actionDataSize)
action: 位置アクションのハンドル
eOrigin: トラッキング原点(ルームスケールなど)
seconds: 予測時間(直近の最新の情報であれば0)
actionData: InputPoseActionData_tへのポインタ
actionDataSize: sizeof(InputPoseActionData_t)

InputPoseActionData_tでは、
位置入力が存在するか、どの入力から最後に入力されたか(入力ソースハンドル)、現在の状態が取得できる。

※理由は不明だが、HMDがスリープ状態に入ると取得できなくなる。
デジタルボタンやアナログ軸はスリープ状態でも動作する。

###TriggerHapticVibrationAction(action,startseconds,duration,frequency,amplitude)
action: 振動アクションのハンドル
startseconds: 開始時間
duration: 持続時間
frequency: 周波数
amplitude: 振幅(0.0～1.0)

振動させる

###GetActionOrigins(actionset,digitalActuion,originsout,originsOutCount)
actionset: アクションセットのハンドル
digitalActuion: デジタルアクションハンドル
originsout: VRInputValueHandle_t配列へのポインタ
originsOutCount VRInputValueHandle_t配列のサイズ

配列に収まるだけの、アクションのアクションソースハンドルを取得します。
もしソースが少ない場合は、余りにはk_ulInvalidInputValueHandleが格納されます。

(例えば、発射アクションに割り当てられた、右手トリガー、左手トリガーなど)

###GetOriginLocalizedName(origin, name, namesize)
origin: アクションソースハンドル
name: ローカライズされたアクションソースの名前文字列配列
namesize: 名前文字列配列のサイズ

アクションソースハンドルのローカライズされた名前を取得します。
「device controller_type input_source」の形式であり、例として「Right Hand Knuckles Controller Trigger」が帰ります。

(ゲーム内での操作説明用？)

###GetOriginTrackedDeviceInfo(略)
入力ソースハンドルから関連付けられた、ポジショントラッキングデバイスに関する情報を帰します。

入力ソースハンドル、トラッキングデバイスインデックス(OpenVRのデバイスID)、レンダーモデル部品名

これにより、GetDigitalActionData等により得られた内部入力ソースハンドルを、自分が取得した入力ソースハンドルに変換できるため、照合するとどの入力ソースから取得できたかがわかる。

また、ここでのレンダーモデル名は、部品単位であり、例えばトリガー由来だとtriggerと帰る。(アニメーション用と思われる)

###ShowActionOrigins(略)
指定されたアクションに対する現在のバインディングを、ヘッドセッド内で表示します。
現在はバインディングUIが表示されますが、将来的に変更される可能性があります。

###ShowBindingsForActionSet(略)
指定されたアクションセットに対する現在のバインディングを、ヘッドセッド内で表示します。
現在はバインディングUIが表示されますが、将来的に変更される可能性があります。


#実際に使ってみる
Unityでバインディング関係の画面を開く一番手っ取り早い方法は

1. 適当なGameObjectに「Steam VR_Behaviour_Boolean」をアタッチする。
2. 「Boolean Action」を選択し、「Add...」をクリックする

ここで開く「SteamVR Input」が、Action manifestの設定画面。
「Save and generate」をクリックで、プロジェクトフォルダに「actions.json」と、Default Bindingが生成される。
また、C#からつかいやすいクラスも自動生成される。
これに関しては関連記事を参照。

ここでSteamVRを起動し、「Open binding UI」をクリックするとブラウザが立ち上がる。
これが「Controller bindings」の設定画面。

注意点としては、設定したいデバイスは接続状態じゃないと出てこないことと、アプリケーションを再生状態にしないまま設定を開始するとLegacy Inputに対する設定になること。


#執筆中...

#メモ
愚直に実装するだけで、素直に動いてくれた。
裏でよしなにやってくれるSteamVR Pluginに対して、OpenVRを直接操作するのは面倒な箇所が多いが、
その分挙動が素直に見えて面白くもある。

#ライブラリ作りました
EasyOpenVRActionInput
https://github.com/gpsnmeajp/EasyOpenVRActionInput
