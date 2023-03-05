# SEDSS - SimpleEncryptedDataSendSample
UnityでByte配列をデバイス間で送受信するだけのライブラリアセットを作りました。
https://github.com/gpsnmeajp/SimpleEncryptedDataSendSample

詳しい仕様は、[API仕様](https://github.com/gpsnmeajp/SimpleEncryptedDataSendSample/blob/master/doc/APISpecification.md)をご確認ください

#これは何なの？
byte[]配列をデバイス間で送りたい時に手軽な通信ライブラリです。
ファイルを経由せず、メモリ上で送受信します。送受信データは暗号化されています。

内部ではHTTPを利用していますが、利用する際は意識せずに利用できます。
Unity間で、byte[]配列1つ渡せればそれでいいのになー、サーバーとか通信とか考えるの面倒だなー、って時に使えます

#どうやって使うの？
1. UnityCipherを入れます。https://github.com/TakuKobayashi/UnityCipher
2. SEDSS_Server,SEDSS_Clientを適当なGameObjectにアタッチします
3. 以下のコードを入れます。ここではテキストを送受信しています。

注意点: 送受信側でPasswordは同じものである必要があります。アドレスやポートは適時設定してください。

```cs:クライアント側
    void download()
    {
        var client = GetComponent<SEDSS_Client>();
        client.SetAddress("127.0.0.1");
        client.SetPassword("1234");

        string request_id = "test message";
        Debug.Log("Download Start ID:" + request_id);
        client.Download(request_id, (data, id) =>
        {
            Debug.Log("Download OK ID:" + id);
            Debug.Log("Message:" + new System.Text.UTF8Encoding(false).GetString(data));
        }, (e, id) => {
            Debug.Log("Download Error ID:" + id);
            Debug.Log("Message:" + e);
        });
    }
//アップロードも同様にできます。ここでは省略
```

```cs:サーバー側
    void Start()
    {
        var server = GetComponent<SEDSS_Server>();
        server.SetPassword("1234");
        server.StartServer();

        server.OnDataUploaded = (data, id) => {
            Debug.Log("Server Received ID:" + id);
            Debug.Log("Message:" + new System.Text.UTF8Encoding(false).GetString(data));
        };
        server.OnDownloadRequest = (id) => {
            Debug.Log("Server Send ID:" + id);
            return new System.Text.UTF8Encoding(false).GetBytes("You're welcome");
        };
    }
```

# なぜ作ったの？
Unity内のオブジェクトを同期させたりと言った、同じアプリケーション間で通信するためのフレームワークってのはいっぱいありますよね。
でも、目的が違うアプリケーションで通信する場合はどうしましょうか？

例えば、スマホのセンサーアプリと、PCで3Dで出すアプリで、わざわざPUNとか使いませんよね。

そういう違うPCやデバイス間でUnityアプリケーション間で簡単に、ちょっとした通信をしようとすると、OSCが便利です。
私はuOSCを愛用しています。
https://github.com/hecomi/uOSC

そして軽量なデータ(1kB以内程度)のデータはまあ簡単にやり取りできるようになったとしましょう。
そこでこう思うことって結構無いですか？

- 起動時に設定ファイルを渡したい
- いくつか画像や音声データを渡したい
- PC側のデータとスマホ側のデータ同期させておきたいなぁ

こういうとき、大体数百kBや2～3MBくらいあったりするものです。さすがにUDPベースのOSCで送る気にはなりませんよね。
HTTPとか使いますよね。でもサーバー立てますか？面倒ですよね。サーバー立てるのも、サーバー機能を実装するのも。

というわけで作ったのが、SimpleEncryptedDataSendSampleです。
サーバー立てるのは避けられませんが、実装するのは楽になります。

# 作者は具体的にどういう場面で使うの？
私は、VRデバイスのモーションデータを他のソフトに送信して利用し合うVMC Protocolを最近活用しています。
https://sh-akira.github.io/VirtualMotionCaptureProtocol/

これを利用する際は、送信側と受信側の両方で同じVRMファイル(3Dアバターデータ)を読み込む必要があります。
同じPC内で使う場合は、同じファイルを自動的に読みに行く仕組みがあるので良いのですが、WindowsとMacとか、iPhoneとPCとか、そういう別デバイス間で使おうとすると、いちいちVRMファイルをコピーするのが面倒になってきたので作りました。

VRアバターデータをやり取りする仕組み上、同じネットワークに複数人居た時に「間違って送った/受け取った」が起きると嫌なので、暗号化処理を入れて、間違った相手とつながらない&平文でパケット見られても大丈夫にしています。

# どういう仕組なの？
図で表すとこんな感じです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/19841bd9-5192-bd37-89de-1c9a81c12005.png)

サーバーは.NetのHTTP Listenerを使っています。
クライアントは、UnityWebRequest(中身はcurlらしい？)です。

HTTP PUTを使って送受信しています。
通信が終わるとコールバックが発生します。
それだけです。

通信処理と暗号化処理を抽象化し、HTTPを意識すること無く使いやすくするというのがこのライブラリの目的です。
中身も別段複雑なものではないのですが、ハマりポイントは色々ありました。

先に記載の通り、単にHTTPで送受信するだけではなく、暗号化処理を入れています。
AES256(Rijndael)なので、パスワードが十分長ければ十分な暗号化が行われます。

ただ一方で、詳しくは説明しませんがプロトコルの作りが甘いため、いたずら防止程度の効果しかないと思ってください。
暗号化はあくまでおまけであり、LAN内やVPN上で使用する前提です。
また当然ですが、末端の送信者・受信者は信頼できる相手である前提です。
(メモリ上には普通に生データで置かれますので。その後どうするかは利用するプログラムによりますが)

#知っておくと良いこと
### IDについて
最初は本当にbyte[]配列1つだけの想定でしたが、そのうち複数の種類やり取りできる必要があるかも知れないと考えたので、idを追加しました。
idは単に文字列ですので付加情報として使うのも良いですし、ほしいファイル名を送っても良いです。
サーバーは単に無視しても良いです。

### Upload/Download
スマホって基本的にサーバーになることはないじゃないですか。クライアントですよね。
PCがサーバーになります。

しかし、やり取りしたいデータは方向が決まっているとは限りません。
あるときはスマホ→PCかもしれないし、あるときは逆かもしれない。

そういうことで、このシンプルな作りでも、アップロードとダウンロード両方に対応しています。

### push
残念ながらこの作りでは、サーバー側からプッシュはできません。
あくまでクライアント側が動作の起点になります。

しかしながら、pushしたくなる時があるかも知れません。
OSCなどでやり取りしているのであれば、クライアント側に別経路で要求を送るのも手ですが、
一番簡単な解決策としては、両方ともサーバーを立ててしまうという手もあります。
OSCとかそんな感じですし。


#おわりに
小さなアプリで気軽に使うためのライブラリです。
大容量だったり、頻繁だったりするデータのやり取りには向いていませんが、用途が合えばとても便利だと思います。

私の最近の目標に「複雑で使いにくいものを作るくらいなら、仕様を絞ってとにかく気軽に使えるようにしたい」というのがあります。
もし気になりましたらぜひ使ってみてください。

# 暗号化ライブラリ
こちらを利用させていただいております。

Unity(C#) で「正しい」暗号化処理をするライブラリを作成しました
https://qiita.com/taptappun/items/1a9dbc8dc62c072aabb5
