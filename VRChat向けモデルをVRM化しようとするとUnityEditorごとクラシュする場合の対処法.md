#はじめに
私が遭遇したがネット上に情報が見つけられなかったパターンを記す

#ボーンの自動認識が失敗している
##結論
左手のボーンがまるごと割り当てられていないのに気づかず、UniVRMからExportしようとしたら、
Unity Editorごとクラッシュした。修正すると正常に完了する

##トレース
まず普通に変換しようとしてクラッシュ

UniVRMを古いバージョン(0.4X系)も新しいバージョン(0.5X系)も試したし、
Unityも5.6.3p1から2017, 2018まで色々試したがどれも同じようにクラッシュする。

Blenderでfbxを吐き直してみてもダメ、調整しようとしてみてもダメ。

Editor.logを確認してみたところ、以下のようなトレースログが出ており、
ボーン関係ではないかと推測。

```
0x0000000141719C47 (Unity) Animator::GetBoneTransform
0x00000001417A4885 (Unity) Animator_CUSTOM_GetBoneTransformInternal
0x00000000505D8F45 (Mono JIT Code) (wrapper managed-to-native) UnityEngine.Animator:GetBoneTransformInternal (int)
0x00000000505D8DB7 (Mono JIT Code) [C:\buildslave\unity\build\artifacts\generated\common\modules\Animation\AnimatorBindings.gen.cs:1153] UnityEngine.Animator:GetBoneTransform (UnityEngine.HumanBodyBones) 
0x0000000050622627 (Mono JIT Code) [E:\UnityProjectFrom201902\tovrm\Assets\VRM\Scripts\Format\VRMExporter.cs:77] VRM.VRMExporter:_Export (UniGLTF.glTF,VRM.VRMExporter,UnityEngine.GameObject) 
0x000000005061F8A3 (Mono JIT Code) [E:\UnityProjectFrom201902\tovrm\Assets\VRM\Scripts\Format\VRMExporter.cs:32] VRM.VRMExporter:Export (UnityEngine.GameObject) 
0x00000000505D63E9 (Mono JIT Code) [E:\UnityProjectFrom201902\tovrm\Assets\VRM\Scripts\Format\VRMExporSettings.cs:271] VRM.VRMExportSettings:Export (string,System.Collections.Generic.List`1<UnityEngine.GameObject>) 
0x00000000505D5844 (Mono JIT Code) [E:\UnityProjectFrom201902\tovrm\Assets\VRM\Scripts\Format\VRMExporSettings.cs:235] VRM.VRMExportSettings:Export (string) 
0x00000000505D559F (Mono JIT Code) [E:\UnityProjectFrom201902\tovrm\Assets\VRM\Scripts\Format\Editor\VRMExporterMenu.cs:42] VRM.VRMExporterWizard:OnWizardCreate () 
```

humanoidボーンの割当を確認してみたところ、
・左手のボーンが認識されていない
・頭のボーンが変な認識になっている
ことがわかり、そこを手動で割り当て直したところ正常にVRMが生成できるようになった。

なお、これらのボーンの割当をあえて中途半端にしてみたところ、
UniVRMがNullReferenceExceptionを吐き始めた(Crushはしない)ので、すべてきちんと治す必要があるらしい。

#遭遇したアバター
以下

【VRChat想定３Dモデル】傀儡人形 キョンシーくん【Quest版対応】
https://booth.pm/ja/items/1567182

なお、こちらのアバターはVRChat向けにはあらかじめ製作者側でセットアップ済みのUnitypackageが同梱されているため、
VRChatでは極めて正常に動作していた。

fbxを読み込み直す必要があるのは改変時や他形式に変換する必要がある場合のみだったため、
自動認識に失敗していることに気がつくのが遅れた。(なおなぜか右手は正常に認識されるのが謎である)

本モデル特有の問題ではなく、Unityがhumanoidボーンの自動認識に失敗しうるモデルであれば同様の問題が発生すると思われる。
