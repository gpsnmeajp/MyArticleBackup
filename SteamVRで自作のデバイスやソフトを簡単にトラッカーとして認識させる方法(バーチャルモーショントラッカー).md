# はじめに(OpenVR開発の難しさ)
SteamVRでVRゲームを遊んだり、またVRソフトウェアを自作していて、一度は思ったことがあるのではないでしょうか。
"自作の機器をつないでみたい"、"自作のソフトを仮想トラッカーにできたら"と。

SteamVRの公開仕様であるOpenVRを使うと、それはできます。
誰でもHMDやコントローラ、トラッカーを作れるように、ライブラリや仕様が公開されているからです。

https://github.com/ValveSoftware/openvr

APIも公開されている、サンプルもある。使いやすいものです。
サンプルをビルドし、数行書き換えれば動くようになります。

が、簡単に手が出るもの**ではない**のが現状ではないでしょうか。
主な原因は以下だと思います。

+ ドライバはDLLとしてSteamVRのVR Serverに読み込ませる必要があり、デバッグが難しい
+ コア部分はC++で書かなければならない。
+ サンプルはあるが極めてシンプルな内容しか無い。
+ ネット上にもあまり情報が揃っていない
+ 古い情報と新しい情報が混在している
+ 公式ドキュメントにはあまり詳しい内容が書いていない(書かれていないこともあります)
+ 行列計算の知識が必要
+ ドライバー座標とルーム座標の変換、対応関係の処理の知識が必要
+ 様々な用途に使えるように作られているが、何を取捨選択したらよいかわからない

では、他に手段があるかというと、あまりありませんでした。
一部特殊ドライバも存在していますが、程度が違えど同様の問題がありました。


2021/06中身のお話

https://qiita.com/gpsnmeajp/items/9c41654e6c89c6b9702f

# VMT - Virtual Motion Tracker

https://gpsnmeajp.github.io/VirtualMotionTrackerDocument/

そこで、私はバーチャルモーショントラッカーというものを作りました。
これはドライバのコア部分は完成済みで、姿勢や入力情報を[OSC (Open Sound Control)](https://ja.wikipedia.org/wiki/OpenSound_Control)で入力することで、SteamVRに伝えることができるドライバです。

HMDやコントローラとしての機能はあえて殆ど載せず、シンプルにトラッカーとして簡単に使用できることを目指しました。
特殊な機能も乗っていないため、安定して動く(当社比)ことが売りの一つです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/b3596ad5-b405-dd36-6aaa-c9ae90257326.png)

これにより何が嬉しいか。他の言語からとても簡単に仮想のトラッカーを作り出し、制御できるようになります。

どのくらい簡単かというと、Unityで以下のコードをGameObjectにアタッチするだけで
仮想のトラッカーとして動かすことができるほどです。

※OSC通信アセット: uOSCを導入してください。
　ポートは39570です。

https://github.com/hecomi/uOSC

**あらかじめ説明書に従ってVMTのインストールと設定を済ませてください**

```cs:sendme.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
public class sendme : MonoBehaviour
{
    uOSC.uOscClient client;
    void Start()
    {
        client = GetComponent<uOSC.uOscClient>();
    }

    void Update()
    {
        client.Send("/VMT/Room/Unity", (int)0, (int)1, (float)0f,
            (float)transform.position.x,
            (float)transform.position.y,
            (float)transform.position.z,
            (float)transform.rotation.x,
            (float)transform.rotation.y,
            (float)transform.rotation.z,
            (float)transform.rotation.w
        );
    }
}
```

C++、通信、その他のことをほとんど考える必要がないということがおわかりいただけますか？

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/d08a8e01-d1de-0f9d-6935-ffad7155cf3b.png)
# VMTの仕組み
VMTは以下の構成になっています。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/aa9b976d-2dca-120e-3146-a0dba4402704.png)
## VMT Driver
C++製OpenVRドライバです。OSCでの仮想トラッカー姿勢の受信と、SteamVRへの姿勢の受け渡しを行います。
ルーム座標に合わせた座標変換や、コントローラ入力の処理、登録処理など行います。
仕組みはほぼOpenVRのサンプルどおりですが、OSC入出力が付いていることが特徴です。
## VMT Manager
C#製管理ツールです。ドライバのインストールやアンインストール、設定や調整、動作確認に使用します。
このツールにより、開発者はインストールなどの処理を考えなくて済むようになるのと、
動作確認による切り分け、ドライバ単体ではできない空間の設定ができます。
## User Application
姿勢や入力情報をOSCで送信するだけで簡単に作成できます。
OSCが送信できれば言語や環境は何でも構いません。

UnityでもUE4でも、C++でもC#でもJavaでも動くと思います。
同じホスト上にある必要もありません。マイコンなどから直接ドライバに送信することもできます。
## 補足
ManagerとDriverの間のプロトコルも含めてOSCであり、公開されていますので、
開発者が作成したプログラムでManagerを完全に代替することもできます。
(ただし、ドライバは応答を127.0.0.1に対して送信するため、Managerとして振る舞う場合は、
同一ホスト上で動作している必要があります。)
# プロトコル
現時点でのプロトコルの一部を抜粋します。
(今後変更される可能性がありますので、最新版は[github](https://github.com/gpsnmeajp/VirtualMotionTracker)を参照ください。)
開発者が知る必要があるプロトコルはほぼ以下のみです。
(他に、コントローラとして機能させるためのものもありますが、バインディングを別途行う必要があります。)

|識別子|型|内容|
|---|---|---|
|index|int| 識別番号。現在0～57まで利用できます。|
|enable|int| 有効可否。1で有効、0で無効(非接続・非トラッキング状態)|
|timeoffset|float| 補正時間。通常0です。|
|x,y,z|float| 座標|
|qx,qy,qz,qw|float| 回転(クォータニオン)|

## トラッカー制御
### /VMT/Room/Unity index, enable, timeoffset, x, y, z, qx, qy, qz, qw
Unityと同じ左手系、かつ、ルーム空間(ルーム空間変換あり)で仮想トラッカーを操作します。
通常はこれを使用します。  

# おわりに
これで、簡単に仮想トラッカーや自作トラッカーを作成することができるようになったと思います。
すでに、Webカメラによるキャプチャからフルトラッキングを実現するソフトなどが出ています。

OpenVRのTrackingOverrides機能を使用すると、HMDやコントローラの代替として振る舞わせることも可能です。

https://github.com/ValveSoftware/openvr/wiki/TrackingOverrides

また、今回作成したドライバはMITライセンスで公開しています。
雛形として利用しやすいかとも思いますので、ドライバ開発の参考としてもご活用ください。

# 参考
過去に書いた記事は以下です。
[OpenVR Driver開発メモ](https://qiita.com/gpsnmeajp/items/050b31d78d1b9757c9db)

開発には以下の記事やサイトを参考にさせていただきました。
[OpenVR driver を作るメモ](https://qiita.com/ousttrue/items/48010aab8aa220072395)
[Driver Documentation](https://github.com/ValveSoftware/openvr/wiki/Driver-Documentation)
[OpenVR-driver-for-DIY](https://github.com/r57zone/OpenVR-driver-for-DIY)
[polygraphene/ALVR](https://github.com/polygraphene/ALVR)
[ValveSoftware/driver_hydra](https://github.com/ValveSoftware/driver_hydra)
