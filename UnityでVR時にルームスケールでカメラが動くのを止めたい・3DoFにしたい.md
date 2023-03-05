#はじめに
カメラの制御は基本的に切れないので、親オブジェクトで、
カメラの座標や回転を打ち消して無理やり止めるしか無い

**...なんてことはない。**
(昔はそうだったのかもしれない？)

#カメラ制御を切る(0DoF)
位置追従を切る。ただし最初に一瞬追従が働いて上書きされるので注意。
https://docs.unity3d.com/ScriptReference/XR.XRDevice.DisableAutoXRCameraTracking.html

```cs
XRDevice.DisableAutoXRCameraTracking(GetComponent<Camera>(), false);
transform.localPosition = Vector3.zero;
transform.localRotation = Quaternion.identity;
```

#ルーム位置制御のみを切る(3DoF)
360度動画向けに、頭の位置追従を切る機能がある。
https://docs.unity3d.com/ScriptReference/XR.InputTracking-disablePositionalTracking.html

```cs
InputTracking.disablePositionalTracking = true;
transform.localPosition = Vector3.zero;
transform.localRotation = Quaternion.identity;
```

#位置や回転だけほしい
TrackedPoseDriverという標準のスクリプトを使えば、任意のデバイスの回転・位置、あるいはそのどちらかだけ取れる。
なおUnity2019ではXR.LegacyInputHelpersに入っているらしい。

スクリプトの場合は以下のようにできる。

```cs
transform.localPosition = InputTracking.GetLocalPosition(XRNode.Head);
transform.localRotation = InputTracking.GetLocalRotation(XRNode.Head);
```

**注意: カメラ制御を切って、スクリプトからHMDの回転を取得してカメラを制御することはおすすめしない。**
**遅延により微妙に気持ち悪くなる。**

#参考文献
[Unity] VRのカメラトラッキングを切る
https://qiita.com/Urad/items/f6ea27682c3db86a2c04

Unity標準のVR機能（UnityEngine.XR）メモ
https://framesynthesis.jp/tech/unity/xr/
