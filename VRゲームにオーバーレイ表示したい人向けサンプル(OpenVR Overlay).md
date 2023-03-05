#はじめに
VRChatなどVRゲームを遊んでいるプログラマなら誰もが一度は思うこと。
「VRゲームの空間に自分の好きな画面を表示できたらいいのに」

それを叶えるツールはあります。
OVRDrop(Steamで1480円)
https://store.steampowered.com/app/586210/OVRdrop/

VR空間にデスクトップ画面のキャプチャを常時表示できるようにするツールで、かなり便利です。
が、人間欲が出てくるものでして「あそこが気に食わない」「自分の好きにコントロールしたい」「わざわざデスクトップ専有したくない」など出てくるものです。

で、「自分のアプリケーションでVR空間に表示できないかな」と考えます。
少し調べると、資料がいくつか出てくるのですが...なんか難しそう。
OVRDropのOSS時代のソースも出てきますが理解が難しい。

と思って数ヶ月。その間にシェーダーで遊んだりした経験のおかげか、久しぶりに調べると
今まで見つけられなかった資料を見つけることができ「行けるんじゃね？」という感覚になりました。
それで実装してみたら行けたので共有しておきます。

#どんなことができるの？
VR空間に割り込んで自分のアプリケーションのRenderTextureを出せます。
機能の仕組み上、表示は2Dです。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/4a845edd-343f-e66b-a614-085028970e45.png)

OVRDropのような入力機能は、うまく動作しないので省いています。
(1月あたりからまともに動かなくなったらしい。SDKの問題らしい？)

※Unity 2Dでも使えます。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/3b50b308-d172-ce97-5136-78d47c9e13eb.png)

**CameraのClear FlagsをSolid Colorに設定することで、背景を透過させることができます。**

#概念図
これを理解するのに無駄に時間がかかった。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/1a7b934f-3148-18a4-fcf0-a8f085a9d868.png)

#おしらせ
Scriptingが苦手な方でもインポートするだけで使えるスクリプトを作りました。
中身は変わらずシンプルに作成しており、サンプルとして使えます。
この記事では取り扱っていない、回転や鏡像反転、ルームスケールなどに対応しています。
2018-09-02追記: Dashboard版を追加しました。

https://sabowl.sakura.ne.jp/gpsnmeajp/unity/EasyOpenVROverlayForUnity/
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/a01834fd-8e1e-8936-0ae7-7e8a9de8c9d0.png)

#より高度なサンプルの紹介
VirtualMotionCaptureの作者のsh_akiraさんが、uGUIの操作に対応したオーバーレイスクリプトを作成されています。
VROverlay
https://github.com/sh-akira/VROverlay?files=1

#やり方
Unity2018.2.5f1 Personalで動作確認しています。
##Unityプロジェクトの準備
1. Unityを立ち上げ、3Dプロジェクトを新規作成します
2. Asset StoreからSteam VR Pluginをインポートします
3. SteamVR_Settingsが立ち上がるので、Accept All
4. Edit→Project Settings→Playerを開き、Inspectorの下のXR SettingsからVirtual Reality Supportedを**オフ**に
5. Edit→Preferencesを開き、SteamVRのAutomativally Enable VRを**オフ**に

##スクリプトの準備
1. Assetsを右クリックし、Create→C# Script
2. スクリプト名を「OverlaySampleScript」に
3. ダブルクリックして開き、以下のソースを流し込んで保存

```Csharp:OverlaySampleScript.cs
/**
 * OpenVR Overlay samlpe by gpsnmeajp v0.2
 * 2018/08/25
 * 
 * v0.1 公開
 * v0.2 エラーチェックが不完全だった問題を修正。RenderTextureが無効なままセットしていた問題を修正
 * 
 * 2DのテクスチャをVR空間にオーバーレイ表示します。
 * 現在動作中のアプリケーションに関係なくオーバーレイすることができます。
 * 
 * 入力機能は正常に動作していないようなので省いています。
 * ダッシュボードオーバーレイは省略しています。
 * 
 * 各メソッドの詳細はValveSoftwareのIVROverlay_Overviewを確認してください。
 * https://github.com/ValveSoftware/openvr/wiki/IVROverlay_Overview
 *
 * These codes are licensed under CC0.
 * http://creativecommons.org/publicdomain/zero/1.0/deed.ja
 */

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Valve.VR; //Steam VR

public class OverlaySampleScript : MonoBehaviour
{
    //エラーメッセージの名前
    const string Tag = "[OverlaySample]";

    //グローバルキー(システムのオーバーレイ同士の識別名)。
    //ユニークでなければならない。乱数やUUIDなどを勧める
    const string OverlayKey = "[OverlaySample]";

    //ユーザーが確認するためのオーバーレイの名前
    const string OverlayFriendlyName = "OverlaySampleApplication";

    //オーバーレイのハンドル(整数)
    ulong overlayHandle = 0;

    //OpenVRシステムインスタンス
    CVRSystem openvr;

    //Overlayインスタンス
    CVROverlay overlay;

    //オーバーレイに渡すネイティブテクスチャ
    Texture_t overlayTexture;

    //上下反転フラグ
    int textureYflip = 1;

    //HMD視点位置変換行列
    HmdMatrix34_t pose;

    //取得元のRenderTexture
    public RenderTexture renderTexture;

    void Start()
    {
        var openVRError = EVRInitError.None;
        var overlayError = EVROverlayError.None;

        //OpenVRの初期化
        openvr = OpenVR.Init(ref openVRError, EVRApplicationType.VRApplication_Overlay);
        if (openVRError != EVRInitError.None)
        {
            Debug.LogError(Tag + "OpenVRの初期化に失敗." + openVRError.ToString());
            ApplicationQuit();
            return;
        }

        //オーバーレイ機能の初期化
        overlay = OpenVR.Overlay;
        overlayError = overlay.CreateOverlay(OverlayKey, OverlayFriendlyName, ref overlayHandle);
        if (overlayError != EVROverlayError.None)
        {
            Debug.LogError(Tag + "Overlayの初期化に失敗. " + overlayError.ToString());
            ApplicationQuit();
            return;
        }

        //オーバーレイの大きさ設定(幅のみ。高さはテクスチャの比から自動計算される)
        var width = 2.0f;
        overlay.SetOverlayWidthInMeters(overlayHandle, width);
        //オーバーレイの透明度を設定
        var alpha = 0.5f;
        overlay.SetOverlayAlpha(overlayHandle, alpha);

        //オーバーレイに渡すテクスチャ種類の設定
        var isOpenGL = SystemInfo.graphicsDeviceVersion.Contains("OpenGL");
        if (isOpenGL)
        {
            //pGLuintTexture
            overlayTexture.eType = ETextureType.OpenGL;
            //上下反転しない
            textureYflip = 1;
        }
        else
        {
            //pTexture
            overlayTexture.eType = ETextureType.DirectX;
            //上下反転する
            textureYflip = -1;
        }
        Debug.Log(Tag + "初期化完了しました");
    }

    void Update()
    {
        //初期化失敗するなどoverlayが無効な場合は実行しない
        if (overlay == null)
        {
            return;
        }

        //オーバーレイを表示する
        overlay.ShowOverlay(overlayHandle);

        //オーバーレイを非表示にする
        //overlay.HideOverlay(overlayHandle);

        //オーバーレイが表示されている時
        if (overlay.IsOverlayVisible(overlayHandle))
        {
            //HMD視点位置変換行列に書き込む。
            //ここでは回転なし、平行移動ありのHUD的な状態にしている。
            var wx = -0f;
            var wy = -0f;
            var wz = -10f;

            pose.m0 = 1; pose.m1 = 0; pose.m2 = 0; pose.m3 = wx;
            pose.m4 = 0; pose.m5 = textureYflip; pose.m6 = 0; pose.m7 = wy;
            pose.m8 = 0; pose.m9 = 0; pose.m10 = 1; pose.m11 = wz;

            //回転行列を元に、HMDからの相対的な位置にオーバーレイを表示する。
            //代わりにSetOverlayTransformAbsoluteを使用すると、ルーム空間に固定することができる
            uint indexunTrackedDevice = 0;//0=HMD(他にControllerやTrackerにすることもできる)
            overlay.SetOverlayTransformTrackedDeviceRelative(overlayHandle, indexunTrackedDevice, ref pose);

            //RenderTextureが生成されているかチェック
            if (!renderTexture.IsCreated())
            {
                Debug.Log(Tag + "RenderTextureがまだ生成されていない");
                return;
            }

            //RenderTextureからネイティブテクスチャのハンドルを取得
            try
            {
                overlayTexture.handle = renderTexture.GetNativeTexturePtr();
            }
            catch (UnassignedReferenceException e)
            {
                Debug.LogError(Tag + "RenderTextureがセットされていません");
                ApplicationQuit();
                return;
            }

            //オーバーレイにテクスチャを設定
            var overlayError = EVROverlayError.None;
            overlayError = overlay.SetOverlayTexture(overlayHandle, ref overlayTexture);
            if (overlayError != EVROverlayError.None)
            {
                Debug.LogError(Tag + "Overlayにテクスチャをセットできませんでした. " + overlayError.ToString());
                ApplicationQuit();
                return;
            }
        }


    }

    void OnApplicationQuit()
    {
        //アプリケーション終了時にOverlayハンドルを破棄する
        if (overlay != null)
        {
            overlay.DestroyOverlay(overlayHandle);
        }
        //VRシステムをシャットダウンする
        OpenVR.Shutdown();
    }

    //アプリケーションを終了させる
    void ApplicationQuit()
    {
#if UNITY_EDITOR
        UnityEditor.EditorApplication.isPlaying = false;
#else
        Application.Quit();
#endif
    }
}
```

##Render Textureの準備
1. Assetsを右クリックし、Create→Render Texture。「New Render Texture」ができる。
2. HierarchyのMain Cameraの「Target Texture」に、「New Render Texture」をドラッグアンドドロップしてセット

##OverlaySampleScriptの設定
1. Hierarchyを右クリックしてCreate Empty
2. 出来上がったGameObjectにOverlaySampleScriptをドラッグアンドドロップ
3. GameObjectをクリック
4. InspectorのOverlaySampleScriptのRender Textureに「New Render Texture」をドラッグアンドドロップしてセット

#動作確認
Unityの再生ボタンをクリック。
Consoleに

```
[OverlaySample]初期化完了しました
```

と表示され、HMD内に先ほどまでのUnityの画面みたいなのが中央に出ていればOK。
MainCameraはディスプレイ出力をしなくなるため、「Display 1 No cameras rendering」と出るが問題ない。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/f477b8cd-a3de-e34e-5cc4-de07781d946b.png)

#解説
なし。ソースコード内のコメントを参照してください。

#参考文献
###ValveSoftware/openvr/samples/helloworldoverlay/ - Github
Valve公式のOpenVRサンプル。ただしC++&Qt向けである。
https://github.com/ValveSoftware/openvr/tree/master/samples/helloworldoverlay

###IVROverlay_Overview - Github
Valve公式のOpenVRマニュアル。結局はここの内容がいちばん大切。
なのだが、ここの内容だけで開発できたらすごいと思います。
C++向けのためUnityで扱う場合は一部相違がある他、乗っていないものも割とあるし、HandleControllerOverlayInteractionAsMouseに至っては廃止されている。
https://github.com/ValveSoftware/openvr/wiki/IVROverlay_Overview

###BenWoodford/OVROverlayManager - Github
Unity上でOpenVR Overlayを扱うシンプルなサンプルソース
ただし2年前のもののため、動作させるために一部修正が必要。
https://github.com/BenWoodford/OVROverlayManager

###Unity＋Viveで開発する - Qiita
UnityでSteamVRを使い始めるための入門解説
https://qiita.com/ShirakawaMaru/items/056a336ab817b60ed875

###Unityでプレビュー時に毎回SteamVRが立ち上がるのをやめさせたい - シバニャンだニャン！
SteamVRが自動で立ち上がらないようにするための設定について
http://shiba6v.hatenablog.com/entry/2018/02/17/034450

###【Unity】ViveのHMDなしにTrackerを使う - てんちょーの技術日誌
OpenVRを手動で初期化する際のやり方、コントローラーのいち取得など
http://shop-0761.hatenablog.com/entry/2018/01/08/034418

###Unity＋HTC Vive開発メモ - FRAME SYNTHESIS
SteamVRでトリガーやボタンの入力を取得したりといった基本的な事項に関する解説
https://framesynthesis.jp/tech/unity/htcvive/


###Unity SteamVR Pluginのソースを読む Part2 - KazumaLab.
HmdMatrix34_tのmの意味を調べているとたどり着いた。
http://kazumalab.hatenablog.com/entry/2017/07/04/065540

###【Unity道場スペシャル 2017博多】クォータニオン完全マスター - SlideShare
行列を用いた座標変換について解説されている。
HmdMatrix34_tの3x4行列の意味を理解するために。4x4行列の省略形である。
https://www.slideshare.net/UnityTechnologiesJapan/unity-2017

###OpenVR API Documentation - Github
Valve公式のOpenVRマニュアル
VR_Initはここ
https://github.com/ValveSoftware/openvr/wiki/API-Documentation

###When I call PollNextOverlayEvent() function, VREvent_MouseMove, VREvent_MouseButtonDown, VREvent_MouseButtonUp events don't work. #712
https://github.com/ValveSoftware/openvr/issues/712

###UnityのSteamVRでGripMove的に位置移動するスクリプト - 備忘ノート
コントローラの生データが欲しい時に
https://note.spage.jp/archives/category/steamvr

###Unityでスタートボタン、ゲーム終了ボタンのUIを作成する - Unityを使った３Dゲームの作り方（かめくめ）
Unityアプリケーションの終了のさせ方
https://gametukurikata.com/ui/startbuttonui
