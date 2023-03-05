# はじめに
こんなツールを作りました。
Oredayo4V (おれだよ for VTuber)です。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr"><a href="https://twitter.com/hashtag/Oredayo4V?src=hash&amp;ref_src=twsrc%5Etfw">#Oredayo4V</a> がどれだけ多機能かというと、DiscordとOredayo4V (とWaidayo)だけで、こんなビデオ通話が実現できます。<br>仮想Webカメラ機能、PostProcessing機能、背景画像機能のあわせ技です。<a href="https://t.co/oXtb2JiE7N">https://t.co/oXtb2JiE7N</a> <a href="https://t.co/aeAFr7I1g2">pic.twitter.com/aeAFr7I1g2</a></p>&mdash; Segmentation Fault (@Seg_Faul) <a href="https://twitter.com/Seg_Faul/status/1267097513435164673?ref_src=twsrc%5Etfw">May 31, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

私は、iOSのアバターフェイシャルキャプチャアプリ「waidayo」の作者ではありません。
赤の他人です。データや通信仕様の解析や問い合わせをしたわけでもありません。

+ ではなぜ、こういうものが作れたのか？
+ どういう仕組で連携できているのか？
+ 同様のものを作るにはどうすればいいのか？

について本記事では解説いたします。

# VMCProtocol
突然ですが、皆さんはVirtual Motion Capture Protocol (VMCProtocol)はご存知でしょうか？

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/00fa2c28-ab17-6a8d-4977-04cf25f53473.png)
https://sh-akira.github.io/VirtualMotionCaptureProtocol/

バーチャルモーションキャプチャープロトコルの名前の通り、バーチャルモーションキャプチャーと通信するためのプロトコルです。
通信して何をするのかというと、キャプチャしたモーション情報を送受信します。

もともとはバーチャルモーションキャプチャーからUnityにモーションを送信するために作られたものでしたが、元より仕様は公開されており、誰でも利用することができます。

サンプルや、リファレンスとなる実装も公開されており、現在15以上のアプリケーションが対応しています。

これにより、アバターの姿勢や表情などのトラッキングと、描画、センシングをそれぞれ別々のアプリケーションで行うことができるようになります。
そう、ここでFace IDの表記がある通り、nmちゃん氏のwaidayoもこのVMCProtocolに対応したアプリケーションです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/111f232b-a5ab-f075-cc3e-b2ac045c7eca.png)


# 具体的な実装
Oredayo4VはUnityで実装されています。
VMCProcolには代表的な2つのライブラリ実装があります。(どちらもMITライセンスです)

+ [EVMC4U](https://github.com/gpsnmeajp/EasyVirtualMotionCaptureForUnity) - Unity向けモーション受信アセット
+ [VMC4UE](https://github.com/HAL9HARUKU/VMC4UE) - Unreal Engine向けモーション受信プラグイン

今回は、先に述べたとおり、Unityで実装したため、EVMC4Uを使いました。
EVMC4Uには、ボーン情報、表情その他の受信状機能があらかじめ備わっており、UnityPackageをインポートするだけでもう動きます。

**あとはVRMを読み込み、waidayoの送信先をEVMC4Uのポートに設定するだけです。**
概ねの実装はこれだけで、あとはカメラの移動やその他を制御する仕組み、UIを延々載せていっただけです。

WPFとの通信は、sh-akira氏の[UnityMemoryMappedFile](https://github.com/sh-akira/UnityMemoryMappedFile)
ウィンドウ制御はsh-akira氏の[VMCUnityWindowExtensions](https://github.com/sh-akira?tab=repositories)
仮想Webカメラはschellingb氏の[UnityCapture](https://github.com/schellingb/UnityCapture)など、
ひたすらライブラリを組み合わせていった結果です。

VMC4UEを使えば、UnrealEngine4版も簡単に作れることと思いますし、既存のゲームなどのプロジェクトにこれらライブラリを導入すれば、簡単に背景付きの撮影環境を構築することができます。

もちろん、ゲーム自体に組み込んでしまうこともできます。
(VR飛行ゲームPilotXrossという例があります)


# ボーンの受信
EVMC4Uには様々な支援機能や最適化が含まれていますが、やっていることは単純です。
以下がVMCProtocol公式ページに記載のボーン・表情受信サンプルです。

waidayoから受信すればフェイシャルキャプチャになりますし、バーチャルモーションキャプチャーから受信すればフルボディトラッキングになります。単純ですね。

```C#
/*
 * SampleBonesReceive
 * gpsnmeajp
 * https://sh-akira.github.io/VirtualMotionCaptureProtocol/
 *
 * These codes are licensed under CC0.
 * http://creativecommons.org/publicdomain/zero/1.0/deed.ja
 */
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using VRM;

[RequireComponent(typeof(uOSC.uOscServer))]
public class SampleBonesReceive : MonoBehaviour
{
    public GameObject Model;
    private GameObject OldModel = null;

    Animator animator = null;
    VRMBlendShapeProxy blendShapeProxy = null;

    uOSC.uOscServer server;

    void Start()
    {
        server = GetComponent<uOSC.uOscServer>();
        server.onDataReceived.AddListener(OnDataReceived);
    }

    void Update()
    {
        if (blendShapeProxy == null)
        {
            blendShapeProxy = Model.GetComponent<VRMBlendShapeProxy>();
        }
    }

    void OnDataReceived(uOSC.Message message)
    {
        if (message.address == "/VMC/Ext/Root/Pos")
        {
            Vector3 pos = new Vector3((float)message.values[1], (float)message.values[2], (float)message.values[3]);
            Quaternion rot = new Quaternion((float)message.values[4], (float)message.values[5], (float)message.values[6], (float)message.values[7]);

            Model.transform.localPosition = pos;
            Model.transform.localRotation = rot;
        }

        else if (message.address == "/VMC/Ext/Bone/Pos")
        {
            //モデルが更新されたときのみ読み込み
            if (Model != null && OldModel != Model)
            {
                animator = Model.GetComponent<Animator>();
                blendShapeProxy = Model.GetComponent<VRMBlendShapeProxy>();
                OldModel = Model;
            }

            HumanBodyBones bone;
            if (Enum.TryParse<HumanBodyBones>((string)message.values[0], out bone))
            {
                if ((animator != null) && (bone != HumanBodyBones.LastBone))
                {
                    Vector3 pos = new Vector3((float)message.values[1], (float)message.values[2], (float)message.values[3]);
                    Quaternion rot = new Quaternion((float)message.values[4], (float)message.values[5], (float)message.values[6], (float)message.values[7]);

                    var t = animator.GetBoneTransform(bone);
                    if (t != null)
                    {
                        t.localPosition = pos;
                        t.localRotation = rot;
                    }
                }
            }
        }
        else if (message.address == "/VMC/Ext/Blend/Val")
        {
            string BlendName = (string)message.values[0];
            float BlendValue = (float)message.values[1];

            blendShapeProxy.AccumulateValue(BlendName, BlendValue);
        }
        else if (message.address == "/VMC/Ext/Blend/Apply")
        {
            blendShapeProxy.Apply();
        }
    }
}
```

# おわりに
VMCProtocolを使用すると、非常に安価に、かつ様々な資産を活用してソフトウェアや撮影環境を構築できます。
ぜひお試しください。
