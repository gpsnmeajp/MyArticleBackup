#OpenVR-InputEmulatorと連携するアプリケーション作りたい
やってみたらビルド設定が結構大変！
のでマニュアル化

#1. ファイルの準備
以下のファイルを用意します。

OpenVR-InputEmulatorのソース
https://github.com/matzman666/OpenVR-InputEmulator
→zip展開するかpullしておく

OpenVRのソース
https://github.com/ValveSoftware/openvr
→zip展開するかpullしておく

Boost 1.63 Binaries (boost_1_63_0-msvc-14.0-64.exe)
https://sourceforge.net/projects/boost/files/boost-binaries/1.63.0/
→exeファイルをいい感じのところにインストール

Visual Studio 2015 Community
「Visual Studio Dev Essentials」で探して！
→C++を有効にしてインストール

#2.ライブラリのビルド
アプリケーションのビルドには以下のライブラリが必要になります。
・libvrinputemulator.lib
・boostのアレソレ
・openvr_api.lib
・openvr_api.dll

このうち、libvrinputemulator.libは存在しないのでビルドする必要があります。

OpenVR-InputEmulator-1.3\lib_vrinputemulatorにあるlib_vrinputemulator.vcxprojをVS2015で開き、
プロジェクトのプロパティを開いて、「すべての構成」に切り替え。
「VC++ディレクトリ」の「インクルードディレクトリ」に

```
D:\boost_1_63_0;E:\OpenVR-InputEmulator-1.3\lib_vrinputemulator\include;E:\openvr-master\headers;$(IncludePath)
```

みたいな感じで入れる。(boost_1_63_0と、OpenVR-InputEmulator-1.3と、openvr-masterを追加する)

「ライブラリディレクトリ」に

```
D:\boost_1_63_0\lib64-msvc-14.0;E:\OIEtest;$(LibraryPath)
```

みたいな感じに入れる。(boost_1_63_0)を追加。

ReleaseとDebugの両方でビルドする。
OpenVR-InputEmulator-1.3のフォルダに、DebugとReleaseが増えて、libファイルが入っているはず。

#3.プロジェクトの設定
Win32コンソールアプリケーションで空のプロジェクトを作る。
参考: https://qiita.com/gpsnmeajp/items/1905a74419f8055484d5

ソースファイルとして、
・client_commandline.cpp
・client_commandline.h
・main.cpp
を入れるとわかりやすい。

プロジェクトの構成はx64にすること。

先程のと同じプロジェクトの設定をした上で、
「リンカー」→「入力」→「追加の依存ファイル」を「libvrinputemulator.lib;openvr_api.lib;%(AdditionalDependencies)」にする。


ソースファイルが入っているのと同じ場所に
・libvrinputemulator.lib (構成がDebugならDebugフォルダから、ReleaseならReleaseフォルダから取り出したものを入れる)
・openvr_api.lib
を入れる。

コンパイル成功後、EXEファイルと同じ場所にopenvr_api.dllを入れること。

