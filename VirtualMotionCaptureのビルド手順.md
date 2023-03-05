#はじめに
VirtualMotionCaptureをたまにビルドしていじるんですが、手順を忘れるのでメモします。
(2019/10/05時点)

#元
以下のページの手順が問題なく理解できる場合はこのメモを読む必要はありません
https://github.com/sh-akira/VirtualMotionCapture/tree/v0.22basefix

#手順
**1.Unity 2018.1.6f1、Visual Studio 2017を準備します。(2019.1.6fではありません)**
ダウンロードはここから
https://unity3d.com/get-unity/download/archive

Visual Studio 2017(Community)も準備します。
すでに古いバージョンとなっているので、Visual Studio Dev Essentialsからダウンロードが必要です。
https://visualstudio.microsoft.com/ja/dev-essentials/

**2.リポジトリをクローンします**
https://github.com/sh-akira/VirtualMotionCapture/tree/v0.22basefix

Gitの扱いに慣れてない私のような人向け: チェックアウトするブランチはv0.22basefixです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/97a7fca6-f53d-ae6d-496a-18c64dc3d757.png)

**チェックアウトするブランチ**と**サブモジュールに再帰**

**2.追加: ColorPickerWPFフォルダが空の場合ColorPickerWPFをダウンロードします**
**サブモジュールに再帰**を入れ忘れた場合。
zipでダウンロードして突っ込む。
無いと、VirtualMotionCaptureControlPanelのビルドに失敗します。
https://github.com/sh-akira/ColorPickerWPF/tree/c6cd9113069cb805a7d37cd43cbc4341d5a834fb

**3.ProjectSettingsフォルダを別の場所にコピーします。**
アセット導入の過程で変更されるためです。

**4.Unity 2018.1.6f1でプロジェクトを開きます**

**4.以下のアセットを導入します。**
Final IK 1.8(アセットストア、有料)

~~SteamVR 1.2.3~~
~~https://github.com/ValveSoftware/steamvr_unity_plugin/releases/tag/1.2.3~~
→現在1.2.3を入れるとラッパーと衝突して起動しません

~~SteamVR Unity Plugin v2.3.2 (sdk 1.4.18)(配布停止)~~
SteamVR Unity Plugin v2.2.0(再生のたびに警告が出ますが動きます)
https://github.com/ValveSoftware/steamvr_unity_plugin/releases/tag/2.2.0
**SteamVR Unity Plugin v2.4.5 (sdk 1.7.15)**ではありません！
ファイル処理が変わっていてエラーが出ます。

UniVRM-0.53.0_6b07.unitypackage
https://github.com/vrm-c/UniVRM/releases

Oculus Lipsync Unity 1.30.0
https://developer.oculus.com/downloads/package/oculus-lipsync-unity/1.30.0/

uOSC v0.0.2
https://github.com/hecomi/uOSC/releases/tag/v0.0.2

MidiJack #71a66c7
https://github.com/keijiro/MidiJack

**5.ExternalPluginsフォルダにアセットを移動します。**
エラーは気にせず。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/f6491994-cfd6-1050-ce2a-268b55cd11c8.png)

**5.追加: Assets\Scripts\EyeTrackingフォルダを削除します**
現時点でアイトラッキング環境なんて持ってる人そんなにいないと思うので。
残ってると、今のアセット構成では動作しません。

**6.Unityを終了します**

**7.ProjectSettingsフォルダをコピーしておいたもので上書きします。**

**8.ControlWindowWPF/ControlWindowWPF.slnをVisual Studio 2017で開きます**

**9.VirtualMotionCaptureControlPanelプロジェクトのプロパティを開きデバッグのコマンドライン引数を/pipeName VMCTestにする。**
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/e2eb639f-144a-0366-284f-a8d511d1a189.png)

**10.実行します**
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/634303f1-2dee-9640-df7c-543a896be11c.png)

起動を確認したら終了します。(Unityとの連携に必要なファイルが生成されます)

**11.Unityを起動します**

**12.起動します**
ScenesフォルダのVirtualMotionCaptureシーンを開き、Unityを再生
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/47ab0ef4-a817-6cd4-fbec-e61d11bb7b5a.png)

VisualStudioでVirtualMotionCaptureControlPanelを実行
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/634303f1-2dee-9640-df7c-543a896be11c.png)

注意: boothで配布されているものと比較し、設定ファイルがなかったり、初期値になっていたりするので、
それが困る場合は配布版から設定を持ってきましょう
