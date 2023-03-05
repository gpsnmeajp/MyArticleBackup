VirtualMotionCaptureのトラッキングの工夫やボーン対応、キャリブレーションには並々ならぬものがあるため、
VRMで姿勢に連動させて絵を作りたい場合などは、VirtualMotionCaptureをベースに改造したほうが楽にいい感じの結果を得られます。

そんなVirtualMotionCaptureをいじる際のファイルの知見

#ビルド
こちらを参照
https://qiita.com/gpsnmeajp/items/c7ba0ab9e188d3f22f9c

#設定ファイルなど
配布版のVirtualMotionCaptureには、default.jsonや手のポーズやショートカットなどのプリセット、WebCameraドライバなどが同梱されているが、Github版をビルドしただけではこれらは入らないため、配布版からコピーして適切に配置する必要がある。
これを忘れると、「手の角度が変」とか「表情が設定できない」といったことになる。

なお、VRoid Hub連携などオープンでない機能は配布版でのみ利用可能である。
同様にFinal IK(有料アセット)は自分で購入する必要がある。

#ControlWPFWindow.cs
コントロールパネルとの通信部分
ここで通信結果の受け取りなどをしている。
トップレベルの処理(VRMの読み込みその他)もここにあるので、まずこのファイルを見るべき


```cs:ControlWPFWindow.cs
    public Action<GameObject> ModelLoadedAction = null;
    ModelLoadedAction?.Invoke(CurrentModel);
```

があるので、以下のように外部からモデル読み込み時に現在のモデルを取得できる。

```cs:ExternalSender.cs
        window = GameObject.Find("ControlWPFWindow").GetComponent<ControlWPFWindow>();

        window.ModelLoadedAction += (GameObject CurrentModel) =>
        {
            this.CurrentModel = CurrentModel;
            animator = CurrentModel.GetComponent<Animator>();
            vrik = CurrentModel.GetComponent<VRIK>();
            blendShapeProxy = CurrentModel.GetComponent<VRMBlendShapeProxy>();
        };
```


保護レベルをpublicにする必要があるが、こんな感じのコードを書くとコントロールパネル相当の操作を注入できる。

```cs
            DataReceivedEventArgs e1 = new DataReceivedEventArgs(typeof(PipeCommands.LoadCurrentSettings), "", new PipeCommands.LoadCurrentSettings());
            WPFwin.Server_Received(null, e1);

            DataReceivedEventArgs e = new DataReceivedEventArgs(typeof(PipeCommands.ChangeCamera), "", new PipeCommands.ChangeCamera()
            {
                type = CameraTypes.Back
            });
            WPFwin.Server_Received(null, e);
            e = new DataReceivedEventArgs(typeof(PipeCommands.ChangeCamera), "", new PipeCommands.ChangeCamera()
            {
                type = CameraTypes.Front
            });
            WPFwin.Server_Received(null, e);
            e = new DataReceivedEventArgs(typeof(PipeCommands.ChangeCamera), "", new PipeCommands.ChangeCamera()
            {
                type = CameraTypes.PositionFixed
            });
            WPFwin.Server_Received(null, e);
            e = new DataReceivedEventArgs(typeof(PipeCommands.ChangeCamera), "", new PipeCommands.ChangeCamera()
            {
                type = CameraTypes.Free
            });
            WPFwin.Server_Received(null, e);


            DataReceivedEventArgs e2 = new DataReceivedEventArgs(typeof(PipeCommands.Calibrate),"", new PipeCommands.Calibrate()
            {
                CalibrateType = PipeCommands.CalibrateType.Default
            });
            WPFwin.Server_Received(null, e2);
```

#AnimationController.cs
アニメーション処理

#Calibrator.cs
キャリブレーション。超大作。
トラッカーの認識や、FinalIKのボーン認識のための操作などがけっこう大変なことに。
なお、キャリブレーション時はVRIKが生成・破棄を繰り返すため、事前に取得した参照は無効になるので注意。

外部から参照する場合は以下のようにすると良い。

```cs:ExternalSender.cs
	void Update () {
        if (CurrentModel != null && animator != null && uClient != null)
        {
            //Root
            if (vrik == null)
            {
                vrik = CurrentModel.GetComponent<VRIK>();
                Debug.Log("ExternalSender: VRIK Updated");
            }
```

#CameraLookTarget.cs
常に注目点を見るカメラ

#CameraMirror.cs
カメラを反転させる処理
これをいじると大体のカメラに影響するが、一部例外がある。

#CameraMouseControl.cs
カメラをマウス操作で操作する。フリーカメラなどがこれで制御される。
フリーカメラをスクリプトなどで制御したい場合は、これを無効にすることをおすすめする。

#CameraRenderCopyToTexture.cs
カメラをRenderTextureにコピー

#DynamicOVRLipSync.cs
リップシンク

#FaceController.cs
まばたきや表情などBlendSharpのController

#HandController.cs
指の角度などのController

#HandTracking
Index対応部分

#JsonSerializer.cs
jsonを見やすくシリアライズする

#KeyboardAction.cs
キーボードの解釈

#NativeMethods.cs
WindowsAPIで画面を透過させたりタイトルを変えたりする

#Photo.cs
好きな解像度のpngで撮影する

#ScalePositionOffset.cs
現実の体の大きさとVRMの体の大きさを変換する？

#SteamVRWrapper
多分使われていない。
SteamVR Pluginにあるものとほぼ同等

#SteamVRWrapper2.0
SteamVR2.0の入力を、Oculus準拠のUnityキー入力に変換する？

#VRMMetaImporter.cs
VRMのメタ情報を読み込む

#WristRotationFix.cs
手首がねじれすぎて破綻しないように肩と肘をいい感じに回す

#Asset/ExternalSender/ExternalSender.cs
OSCで外部に姿勢情報、Blendsharp、経過時間などを送信する。

受信側は
https://github.com/gpsnmeajp/EasyVirtualMotionCaptureForUnity
を参照。
