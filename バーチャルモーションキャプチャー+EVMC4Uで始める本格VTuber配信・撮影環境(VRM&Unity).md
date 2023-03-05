<img src="https://sabowl.sakura.ne.jp/gpsnmeajp/title3.png"></img>
# はじめに
VR機材で3D VTuberの配信環境を作成するには、今まで様々な技術が必要でした。
モデル単体での撮影や、ゲームとの合成にはバーチャルモーションキャプチャーを始めとした様々なソフトが有りましたが、
Unityといった3D開発環境と組み合わせるものは殆どないか、専用ハードウェアとの組み合わせが必要であったり、自前で構築する必要がありました。

代表的なのが以下の記事で、IKの設定からボタンの設定、割り当て、調整など様々な作業と知識が要求され、その上技術の変化に対応するための追加作業が必要となっていました。

UniVRM+SteamVR+Final IKで始めるVTuber
https://qiita.com/sh_akira/items/81fca69d6f34a42d261c

せっかくVR機材を持っているのであれば、もっと簡単できないか？多少の制約があっても、もっと気軽にできないか？
という思いから、EasyVirtualMotionCaptureForUnityを開発しました。

こんな撮影ができる環境を作れます。([シネスイッチャー](https://booth.pm/ja/items/1654878)も利用)
<img src="https://sabowl.sakura.ne.jp/gpsnmeajp/orange_pv/HDRP.gif"></img>

# EVMC4U - EasyVirtualMotionCaptureForUnity
## [ダウンロード(booth)](https://booth.pm/ja/items/1801535)
## [ダウンロード(github)](https://github.com/gpsnmeajp/EasyVirtualMotionCaptureForUnity)

**かんたんにばもきゃとUnityつなぐやつ**という名前の通り、
本ツールは

+ SteamVRとのやり取り
+ ボタン入力
+ MIDI入力
+ VRMの諸々の処理
+ トラッカーの扱いや補正
+ 10点トラッキング含めた処理
+ カメラ移動
+ ボーンのあれこれ

を**すべてバーチャルモーションキャプチャーに丸投げ**し、得られた「おいしいところ」だけ自分のUnityに持ってこれるツールです。

MIT licenceです。

# 使い方
Unity側で準備するべきことは、モデルを動かすのに必要なことだけとなります。
Unity初心者でも10分あれば動かせます。

簡単には以下の手順です。
先に、動かしたいVRMモデルと、[バーチャルモーションキャプチャーv0.36～](https://sh-akira.github.io/VirtualMotionCapture/download.html)をダウンロードします。
バーチャルモーションキャプチャーは先行リリース版をおすすめします。

1. Unityを準備する
2. ExternalReceiverPackをダウンロードして新しい3Dプロジェクトに入れる
3. 読み込みたいVRMファイル入れて、ExternalReceiverSceneを開いて配置する(あるいはExternalReceiverプレハブを配置する)
4. Scene ViewでExternalReceiverに、読み込んだVRMのGameObjectを「Model」に割り当てる
5. 再生して実行開始
6. VirtualMotionCaptureを起動して、OSCモーション送信を有効にする
　(v0.36以降のみ存在します)
7. VirtualMotionCaptureでキャリブレーションする。

より詳しい手順は[説明書](https://github.com/gpsnmeajp/EasyVirtualMotionCaptureForUnity/wiki/How-to-use)を御覧ください。
[動画版の説明もあります。](https://www.youtube.com/watch?v=L5dkdnk5c9A&feature=youtu.be)

# 仕組み
やっていることとしてはシンプルです。

+ バーチャルモーションキャプチャーからボーン情報を送る
+ EVMC4Uでボーン情報を受け取り、モデルに反映する

## バーチャルモーションキャプチャー側
**バーチャルモーションキャプチャーはオープンソースソフトウェア**ですので、まずこちらにボーンやBlendShapeをOSCで送信する機能を実装。
[プルリクエストして組み込んでいただきました。](https://github.com/sh-akira/VirtualMotionCapture/pull/12)
10点トラッキングやキャリブレーションなどのノウハウが多数詰まっている処理の結果だけをOSCで送ります。

今となっては様々な機能が追加されていますが、基本となるコードはこれだけです。
https://github.com/sh-akira/VirtualMotionCapture/pull/12/commits/9597423472a9441f791b9b44e565ffa1a6005605

見て分かる通り、VRMからAnimatorを経由してUnity Humanoid Boneを取り出し、それをOSCで送信しているだけです。
これにより、受け取る側はOSCで受信して、ボーンに適用するだけで、トラッキングなどの処理は一切考えること無く処理できるようになります。

## EVMC4U側
上記の通り、Unity側ではOSCで受信してボーンに入れるだけです。
ので、初期はこんな感じのシンプルなコードでした。(今は様々な処理が入っています)

https://github.com/gpsnmeajp/EasyVirtualMotionCaptureForUnity/commit/8cb977ca98545cc9d9dcc5e8c5fe2d88670c86a4

BlendShapeの制御やボーンの正規化などは、すべて[UniVRM](https://github.com/vrm-c/UniVRM)におまかせしていますし、
OSC通信は[uOSC](https://github.com/hecomi/uOSC)におまかせしています。どちらも良くできており、扱いがとても楽です。

シンプルなスクリプトであり環境依存する要素も少ないため、Windows環境以外でも動作するようです。(iOSで動かしてる人もいました)

## プロトコル
プロトコルは公開されています。

これに沿って送受信できるアプリケーションであれば、なんでも利用できます。
ネットワークやInternet経由でも送受信できます。
https://github.com/gpsnmeajp/EasyVirtualMotionCaptureForUnity/wiki/Protocol-V2

# より応用的な使い方
## Unityでアセットと組み合わせる
CinemachineやPlaymakerといったUnityのアセットと組み合わせることで、高度な撮影や、ゲーム的なものを実装することもできます。
既存のスクリプトとの強豪も起きづらいため、自作ゲームに組み込むことも可能です。

Unityバージョンは5.6.3p1から2019.3まで動作確認しています。

[シネスイッチャー / Cine Switcher](https://orange3134.booth.pm/items/1654878)を使えば、プロのカメラワークのような映像も撮れます。
https://www.youtube.com/watch?v=GURL1CDgAiw

<img src="https://sabowl.sakura.ne.jp/gpsnmeajp/orange_pv/CineSwitcher.gif"></img>

## スクリプトを書く
EVMC4Uは、基本のスクリプトに様々なインターフェースを用意しており、拡張して利用することができます。
https://github.com/gpsnmeajp/EasyVirtualMotionCaptureForUnity/wiki/Option

例えば、物を持ったり、ワープ移動したりといったスクリプトはサンプルに同梱されています。
トラッカーの座標などの情報も取得できますし、以下のようなコードを書けば、バーチャルモーションキャプチャーのキー入力・コントローラ入力、MIDI入力などの情報を受け取ることもできます。

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace EVMC4U
{
    public class InputTesting : MonoBehaviour
    {
        public InputReceiver receiver;
        private void Start()
        {
            receiver.ControllerInputAction.AddListener(ControllerInputEvent);
            receiver.KeyInputAction.AddListener(KeyInputEvent);

            receiver.MidiNoteInputAction.AddListener(MidiNoteEvent);
            receiver.MidiCCValueInputAction.AddListener(MidiCCValEvent);
            receiver.MidiCCButtonInputAction.AddListener(MidiCCButtonEvent);
        }

        public void KeyInputEvent(EVMC4U.KeyInput key)
        {
        }

        public void ControllerInputEvent(EVMC4U.ControllerInput con)
        {
        }

        public void MidiNoteEvent(EVMC4U.MidiNote note)
        {
        }

        public void MidiCCValEvent(EVMC4U.MidiCCValue val)
        {
        }

        public void MidiCCButtonEvent(EVMC4U.MidiCCButton bit)
        {
        }
    }
}
```

# おわりに
EVMC4Uは以下のプロダクトに依存しています。
商用利用の際はUnityおよび各プロダクトのライセンスにご注意ください。

[Unity](https://unity.com/ja)
[バーチャルモーションキャプチャー](https://sh-akira.github.io/VirtualMotionCapture/)
[UniVRM-0.53.0_6b07 - UniVRM, UniGLTF, UniHumanoid, MToon(MIT Licence)](https://github.com/vrm-c/UniVRM)
[uOSC v0.0.2(MIT Licence)](http://tips.hecomi.com/entry/2017/08/20/193823)

UnrealEngine版を作成されている方もいます。
https://github.com/HAL9HARUKU/VMC4UE
