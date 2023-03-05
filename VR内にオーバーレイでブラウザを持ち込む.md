#注意
**Embedded Browserは任意のサイトを見るための作りではないようです。**
**「信頼できないコードを読み込まないように」とドキュメントに書かれていますし、Chromiumのバージョンは古く、Googleにアクセスすると更新するように表示が出ます。**
**自作のHTMLやjavascriptを読み込む、信頼できるサイトのみを読み込むようにしてください。**

#紹介
ちなみに、別のアセットを使った完成品を配布している方がいます。
こちらはIE11です。
VROverlayWebView
https://github.com/Narazaka/VROverlayWebView

#手順
1 この記事のとおりに作る

空間タップできるVRオーバーレイアプリケーションをUnityで作る【コーディングなし】
https://qiita.com/gpsnmeajp/items/3b67223f7f11bb6d93c3

2 このアセットを買う
ZEN FULCRUM LLC - Embedded Browser
https://assetstore.unity.com/packages/tools/gui/embedded-browser-55459

3 Assets/ZFBrowser/Prefabs/Browser (GUI)を、Canvasに突っ込む

4 以下のコードを書き、Browser (GUI)のPointer UIGUI(Script)を置き換える。
この際、Inspector上で
・Enable Touch Inputをオンに
・EasyOpenVROverlayには、OverlaySystemを突っ込む

```cs:MyBrowserUI
//gpsnmeajp 2019
//These codes are licensed under CC0.
//http://creativecommons.org/publicdomain/zero/1.0/deed.ja
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using ZenFulcrum.EmbeddedBrowser;

public class MyBrowserUI : PointerUIGUI
{
    public EasyOpenVROverlayForUnity EasyOpenVROverlay;
    public Vector2 PointerPositionLeft = Vector2.zero;
    public bool IsClickLeft = false;
    public Vector2 PointerPositionRight = Vector2.zero;
    public bool IsClickRight = false;

    protected override void FeedTouchPointers()
    {
        if (EasyOpenVROverlay.LeftHandU >= 0 && EasyOpenVROverlay.LeftHandV >= 0)
        {
            PointerPositionLeft = new Vector2(EasyOpenVROverlay.LeftHandU, EasyOpenVROverlay.LeftHandV);
        }
        IsClickLeft = EasyOpenVROverlay.tappedLeft;

        if (EasyOpenVROverlay.RightHandU >= 0 && EasyOpenVROverlay.RightHandV >= 0)
        {
            PointerPositionRight = new Vector2(EasyOpenVROverlay.RightHandU, EasyOpenVROverlay.RightHandV);
        }
        IsClickRight = EasyOpenVROverlay.tappedRight;

        FeedPointerState(new PointerState
        {
            id = 10 + 0,
            is2D = true,
            position2D = PointerPositionLeft,
            activeButtons = IsClickLeft ? MouseButton.Left : 0,
        });
        FeedPointerState(new PointerState
        {
            id = 10 + 1,
            is2D = true,
            position2D = PointerPositionRight,
            activeButtons = IsClickRight ? MouseButton.Left : 0,
        });
    }
}
```

5 実行して、VR空間にブラウザが現れればOK

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">VRの中にブラウザを持ち込むの自分も作ってみている。<br>前にEmbedded Browserというアセットを買ったのでそれを使用。デスクトップとは別に動いている。<br>(※セキュリティ上難があるため、このままの機能で公開はしません)<a href="https://t.co/S7QZyqCWKp">https://t.co/S7QZyqCWKp</a> <a href="https://t.co/xsLdNC8Xp3">pic.twitter.com/xsLdNC8Xp3</a></p>&mdash; Segmentation Fault (@Seg_Faul) <a href="https://twitter.com/Seg_Faul/status/1155382705321738241?ref_src=twsrc%5Etfw">July 28, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
