# EasyOpenVRUtil

かゆいところに微妙に手が届かないSteam VR Pluginの仕様を見て、
補助ライブラリを作りたくなりました。

OpenVRに直接アクセスして、
Steam VR 2.0で色々おもしろいことになった入力システムを無視して
コントローラやトラッカーの座標を取得したり、
識別しにくいトラッカーをシリアル番号で識別できたり、
バッテリー残量を取得できたり、
VR内スクリーンショットを勝手に撮影したり、
デバイス一覧を取得したり、
非VRアプリケーションだけどトラッカーやコントローラーの姿勢を取得したりできます。

3時間で作ったのでデバッグ不足なところがあるかも知れません。不具合報告お願いします。

**↓ダウンロード**
EasyOpenVRUtil
https://github.com/gpsnmeajp/EasyOpenVRUtil

CC0ライセンスです。
バグ報告などは、コメント、メール、もしくはDiscord:https://discord.gg/QSrDhE8まで

# 使い方
詳細はこちらをご参照ください。
https://github.com/gpsnmeajp/EasyOpenVRUtil/wiki

例えば特定のシリアル番号を持つVIVEトラッカーの位置を
装着位置や角度をリセットした上でGameObjectに反映する処理は以下のように書けます。

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using EasyLazyLibrary;

public class VRTracker : MonoBehaviour {
    public GameObject Tracker;
    public string serialNumber;
    EasyOpenVRUtil eou;
    EasyOpenVRUtil.Transform offset;

    void Start () {
        eou = new EasyOpenVRUtil();
    }

    void Update () {
        eou.Init(); //初期化処理(初期化後は特に何もしない)
        eou.AutoExitOnQuit(); //VRシステムの終了を検知して自動終了する

        //シリアル番号を指定して、姿勢情報を取得
        var t = eou.GetTransformBySerialNumber(serialNumber);
        if (offset == null)
        {
            //一番最初に取得された位置情報を、初期姿勢とする
            offset = t;
        }
        //初期姿勢からの変化を、GameObjectに反映する
        eou.SetGameObjectLocalTransformWithOffset(ref Tracker, t, offset);
    }
}
```

