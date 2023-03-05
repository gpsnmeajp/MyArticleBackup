#はじめに
ドライバ開発を試行している最中のメモ
随時更新。

参考サイト
https://github.com/ValveSoftware/openvr
https://github.com/terminal29/Simple-OpenVR-Driver-Tutorial/wiki
https://github.com/ValveSoftware/driver_hydra

途中経過
https://github.com/gpsnmeajp/SegsVRControllerDriverSample

#トラブル1
https://github.com/terminal29/Simple-OpenVR-Driver-Tutorial/wiki
にそって進めていたところ、WatchdogDriver.cppを追加したところでWebConsoleの挙動がおかしくなった。
関連コンポーネントからのログが出なくなり、ドライバに関するメッセージが出なくなる。

WatchdogDriverかServerDriverの領域確保を止めると治ることから、コンストラクタかそれ以前のどこかからおかしくなることがわかった。
しかしながらコード上なんら処理はしておらず、呼び出しすべてを止めても問題は解決しなかった。

結果として、Visual Stuido Community 2017でビルドするのではなく、Visual Stuido Community 2013でビルドすると治った。
原因は、CRTのバージョン不整合ではないかと思われる。

Potential Errors Passing CRT Objects Across DLL Boundaries
https://docs.microsoft.com/ja-jp/cpp/c-runtime-library/potential-errors-passing-crt-objects-across-dll-boundaries?view=vs-2019

というのも、もともと使用しているOpenVRのプロジェクトバージョンではVS2013ベースだったが、それをアップグレードして使用していたため。
やはり開発環境のバージョンアップは下手に行うとハマるようだ。

→2020年6月追記: プロジェクト設定を注意深くやったところ、普通にVS2019で動いている

#トラブル2
https://github.com/terminal29/Simple-OpenVR-Driver-Tutorial/wiki
仮想コントローラの実装あたりで古くて使えなくなるようだ...
SteamVR側でコントローラ関係の実装に大きな変更が入っているので、その影響かと思われる。

#メモ
binの方もチェックしないといろいろファイルが足りないぞ気をつけろ
https://github.com/ValveSoftware/openvr/tree/60eb187801956ad277f1cae6680e3a410ee0873b/samples/bin/drivers/sample

Renderモデルについて
https://github.com/ValveSoftware/openvr/issues/150

#動作
適当にコントローラ動かすのは、ここの流れがわかりやすい。
TrackedDevicePoseUpdatedが必要。
https://github.com/terminal29/Simple-OpenVR-Driver-Tutorial/wiki/3---Adding-Tracked-Controllers

# サンプルドライバを動かすときの注意
・TrackedDevicePoseUpdatedが必要。
　nullドライバにはこれがないのでデバイスの位置は一生動かない

・アイコンがうまく読み込めていないことがある。
　コントローラロールのままだと、HMDのコントローラと見分けがつかなくて悩むので注意。
　さっさとトラッカーロールに変えてしまうか、HMDを引っこ抜くとわかる。

・マルチスレッド　MT, MTdである点に注意。何を言ってるのかわからない人はVCのリンカ画面を開く。
　DLLではない。作ってるのはDLLだけど

#新Inputのドライバ版について
https://github.com/ValveSoftware/openvr/wiki/IVRDriverInput-Overview
プロファイルの書き方
https://github.com/ValveSoftware/openvr/wiki/Input-Profiles

どうやら、ViveのBinding jsonをそのままつかい、ドライバ内でも同じ名前で割り当ててやれば、Viveコントローラと同じ割当でいい感じにやってくれるようだ。

#参考
すでに動いている公開されたドライバは参考になる
https://github.com/HipsterSloth/PSMoveSteamVRBridge/blob/master/src/main/cpp/driver/ps_move_controller.cpp

#座標が合わない
https://github.com/ValveSoftware/driver_hydra/blob/master/drivers/driver_hydra/driver_hydra.cpp
ドライバの原点は、通常どちらかのベースステーションになる。(bに設定した方？)
これはアプリケーション側で取得できるTrackingUniverseRawAndUncalibratedなドライバ座標系の原点と同じ。

通常取得するルーム座標系や座位座標系は、ここからシャペロンシステムによりルーム中心かつフロア高さが適用された状態で補正されたもの。
https://github.com/ValveSoftware/openvr/wiki/IVRSystem::GetDeviceToAbsoluteTrackingPose


#共有メモリ
直接は関係ないが使うことにはなる
http://chokuto.ifdef.jp/urawaza/commonmem1.html
https://www.wabiapp.com/WabiSampleSource/windows/shared_memory.html
https://kagasu.hatenablog.com/entry/2017/05/03/003922

https://docs.microsoft.com/en-us/windows/desktop/api/winbase/nf-winbase-createfilemappinga
https://docs.microsoft.com/en-us/windows/desktop/api/memoryapi/nf-memoryapi-mapviewoffile
https://docs.microsoft.com/en-us/windows/desktop/api/memoryapi/nf-memoryapi-unmapviewoffile
https://docs.microsoft.com/en-us/windows/desktop/api/handleapi/nf-handleapi-closehandle
