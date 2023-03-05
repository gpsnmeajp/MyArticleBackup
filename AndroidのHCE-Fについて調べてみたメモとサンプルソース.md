#はじめに
2016年からAndroid 7にてFelicaのカードエミュレーション機能が使えるようになりました．
が，情報があんまりない気がしますので，とりあえずまとめてみました．

こんな感じのアプリが作れるようになります．
<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">Android 7.0から搭載されたHost-based Card Emulation NFC-F (HCE-F Felica)を使ったポイントカードアプリ的なデモを作りました。<a href="https://t.co/lL35QnZRyq">https://t.co/lL35QnZRyq</a><br>なお効果音はフリー音楽素材MusMus( <a href="https://twitter.com/t_watson?ref_src=twsrc%5Etfw">@t_watson</a> 様)より、アイコンはICOOON MONOよりお借りしています。 <a href="https://t.co/xMBdKqEAGH">pic.twitter.com/xMBdKqEAGH</a></p>&mdash; Segmentation Fault (@Seg_Faul) <a href="https://twitter.com/Seg_Faul/status/962743731295170560?ref_src=twsrc%5Etfw">2018年2月11日</a></blockquote>


#HCEについて
・Host Card Emulation NFC-F
・Host-based Card Emulation NFC-F
・ホストカードエミュレーション(FeliCa)
・ホストベースドカードエミュレーション(FeliCa)
と呼ばれる．

**注記:これで「SIMフリースマホでFeliCaが使える！Suicaが使える！」という言説が流行ったようですが、以下の理由で望みは薄いです。
・既存のFeliCa決済システムとは違うシステムが必要になる上、下記の通りSonyは決済に向いてるとは考えていない。
(HCEの利点はカードリーダーが使いまわせる程度です。うまく作れば微量の改修で済みますが、セキュリティは...)
・HCEは遅いので、ms単位にこだわってFeliCaの使用に関わったJRが許すとは到底思えない。
(情報量次第ですが0.5-1秒と見積もってください。Suicaは0.1-0.2秒です。)
以上**

本来，セキュアエレメントですべて処理していたNFCカード側処理ですが，セキュアエレメントに関する事業者間の足の引っ張り合いの結果，NFCが中々普及しないので，いっそのことセキュアエレメント無しで直接アプリケーションからカードとして通信してしまおうというもの．

これにより，多少の制約はつくものの，アプリからカードとして振る舞うことが許されるようになった．
一方，高セキュリティな通信には向いていないので，トークナイゼーションなどを併用する必要がある．

NFC Type-Aでは2014年にすでに実装されている．
2016年より，FeliCaでも利用できるようになった．

ロンドンで見た世界の最新NFC事情――新技術「HCE」と公共交通機関の進展に注目 (1/2) - ITmedia Mobile 
http://www.itmedia.co.jp/mobile/articles/1408/12/news028.html

NFCのカード側機能の開発者への解放
https://weekly.ascii.jp/elem/000/000/240/240556/
https://weekly.ascii.jp/elem/000/000/240/240557/
https://weekly.ascii.jp/elem/000/000/240/240567/


NFC Type-Aでは，ISO 14443-4上にAPDU(ISO7816-4)を流す形である．(NFC_HCE_APDU)
NFC Tyep-Fでは，Felicaのコマンドを直接やり取りする形のようである(pollingは自動で処理されるが) (NFC_HCE_NFCF)

Android 7.0のNFC HCE-Fの実装について
https://qiita.com/eggman/items/7462e7e993742db88798

HCEでなんちゃってType4のNDEFタグをつくる #android #hce 
https://www.slideshare.net/hieroadgjmptw/hcetype4

Enjoy RFID: HCEでおしゃべり 
http://enjoy-rfid.blogspot.com/2015/12/hce.html?spref=tw

#HCE-Fの制約
・Android 7系であり，かつNFC機能にカードエミュレーション機能が搭載されていれば使える．
・FEATURE_NFC_HOST_CARD_EMULATION_NFCFが提供されていれば使える
・**Type-A/Bではバックグラウンドでサービスが動作できたが，Felicaでは禁止されている**
　**→ポイントカードアプリのバーコードスキャンの置き換えのようなのを狙っている？**
　(アクティビティが画面の最前面から外れると待受を強制的に解除される．
　RF受信中は通信を持続するがRFが切れると終了する)
・システムコードは0x4000～0x4FFF(但し0x4xFFを除く)しか使用できない
・IDmは0x2FExxxxxxxxxxxしか利用できない．
　**→いわゆるIDm偽装や既存のカード偽装はできない**

Host-based Card Emulation for NFC-F アプリケーション開発ガイドラインより
https://www.sony.co.jp/Products/felica/business/tech-support/index.html#NFC

おそらくだが，IDmとシステムコードの制約は，IDmで認証するような脆弱なシステムの保護のためだと思われる．
(Androidカーネルをリコンパイルすれば解除できそうだが，カジュアルな悪用は防止できるだろう)
※PaSoRiではこういった制約なくカードエミュレーションできるので，IDmでの認証は本当にダメ

#バックグラウンドでの実行禁止は理由不明
##HCE Type-A/Bの仕様
※この記事を書いている人はまだType-A/Bを触ったことがありません

上述の通り，Type-A/BにおけるHCEではバックグラウンドサービスで起動しっぱなしになるため，
いちいちアプリを立ち上げる必要のあるHCE-Fとは異なり，ロック画面でも利用できる．

Type-A/BにおけるHCEでは，5～16ByteのAID(アプレット識別子)で識別される．
~~そのため**既存のカードのエミュレーションもできるように設計されている**．~~
([その点を突くとこのようなこともできるようである．](https://www.forbes.com/sites/thomasbrewster/2015/02/18/android-app-clones-cards/#6e81f27adb39))

AIDは以下のルールで定められている．
最初の4bitが

+ A:国際的に登録されたAID
+ D:国内用に登録されたAID
+ F:登録無しで自由に使用できるID

参考:[Android HCE: are there rules for AID?](https://stackoverflow.com/questions/27533193/android-hce-are-there-rules-for-aid)
空間が広いため，よっぽどのことがないかぎり衝突するつことはないだろう．

支払い用とその他が別れており，支払用のカードはAndroidの設定画面から変更する必要がある．
ユーザーが誤って別のカードで決済しないため & 同じ決済システムの違うカード同士の衝突防止だろう．

支払用以外のカードのAIDが衝突した場合は，ユーザーにどれを使うかを問い合わせる画面が出るようだ．

~~結論として，HCE-A/Bはアプリの立ち上げが不要な上，既存のカードとして振る舞うこともできることから，
カードの仮想化・置き換えを目的としていると考えられる．~~

##HCE Type-Fの仕様
一方，Type-FにおけるHCEであるHCE-Fでは，2Byteのシステムコードで識別される．
その上システムコード空間自体が制限されているため，アプリの識別に使用できる空間は4079しかない．

このシステムコード空間はHCE-Fのために割り当てられた専用の領域であり，既存のカードと衝突しない．
**つまりシステムの改修なく既存のカードをエミュレーションすることはできない．**

その上，0x4000以外はSonyの管理する領域(カードとHCEを併用する業者用)であり，自由に使うことはできない．
よって，数多くのHCEを用いたシステムはシステムコード0x4000を使うと考えられる．

つまり，システムコードをアプリの識別に使うことがそもそも有用ではない．
上記アプリケーション開発ガイドラインでも以下のように述べている．

>HCE-F アプリケーションは、先述の制限の範囲内で任意のシステムコードおよびNFCID2を設定することができます。このため、他のアプリケーションとシステムコードが重複する可能性があります。その前提でリーダ／ライタを設計してください。例えば、システムコード以外の値を併用してアプリケーションを区別するなどの手段が考えられます。

であることから，複数のアプリケーション間で干渉しあってまともに使えない事態を防止するため
**フォアグラウンドのアプリケーションの1サービスのみ待ち受けられる**としたのではないかと考えられる．

結論として，HCE-Fはかならずアプリを立ち上げておく必要が有るため，
バーコードやQRコードでやり取りするアプリの置き換えなどを狙っていると考えられる．

#HCEを選ぶポイント
##HCE(A/B)を選ぶ理由としては...

+ MIFARE DESfire系に対応しているシステムやカードリーダーを使用している
+ ISO7816-4の知識がある
+ バックグラウンドやロック画面で動作する必要がある
+ 国外のシステムに相乗りする必要がある

##HCE-Fを選ぶ理由としては...

+ FeliCa系専用のシステムやカードリーダーを使用している(PCならRC-S370以前のPaSoRiを使用しているなど)
+ FeliCaの通信コマンドに関する知識がある
+ アプリ起動中のみ動作すれば良い
+ 国内のシステムで完結する


#暗号通信は不可？
また
> 具体的なアプリケーションの想定として、「最初はクーポンやポイント、IDカード、回数チケットなど、FeliCaの高いセキュリティを必要としないアプリケーションで、お使いいただければと考えています」と、同開発部 3課 統括課長 中津留勉氏は話す。

と言っていることから，Felica Lite相当(暗号化なし通信のみ)を想定していると思われる．
画面のバーコードを読み取ってポイントカードとしているようなアプリにおいては有効だろう。

>HCE-Fは決済手段として使うこともできなくはないが、セキュリティをどう担保するかが課題であり、「原理的にはトークナイゼーションのシステムを構築することも可能ですが、そのシステムをお客様が構築することを考えると、SEを使った方がトータルなコストは安く済むのではないかと考えています」と同氏は続ける。

>日本や香港では既存のシステムが整っているため、HCE-Fを活用した決済アプリケーションを新規に構築する必要性は薄いとしているが、「NFCフォーラムは、HCEはセキュアエレメントなしでサービスが構築できると注目しています。アジアなどでこれから新たにシステムが構築される国であれば、メリットが感じられるかもしれません」と竹澤氏は考えを述べる。

とのことから、日本においてはセキュアエレメントの搭載が前提であるようである。
QRコード決済が普及している/しようとしている国へのアプローチと見るのが良いだろう。

引用元: ソニーのFeliCaの現状、「HCE-F」の想定アプリケーションは？ 
https://www.paymentnavi.com/paymentnews/62571.html


**追記**
実際の仕様を見てみると，Androidアプリ側で全く好きなように通信内容を生成できる．
カードのエミュレーションをしてもいいし，独自のプロトコルを利用しても良さそうな感じがある．

また，Javaアプリ内でカードの通信内容を生成するのであれば，暗号化なし通信相当しかできない(デコンパイルなどの危険性を考えると)が，いわゆるQRコードを用いた決済に使うトークナイゼーションを使えば決済までできそう．

また，Javaアプリ内はカードの通信内容を素通しするだけの処理をし，通信内容は実際にはクラウド上で生成するクラウド型セキュアエレメントの構成であれば，暗号化通信もできそうである．(通信速度に影響されるが)

#実装で詰まった点
6時間位格闘してわかったこと

・PC/SCでは反応してくれない...！？
　→システムコードが強制0xFFFFになるため無理．
・felicalibでは反応する
・サービスを自分で起動する必要はない．HCE-Fサービスから起動してくれる
・カードなら使える，システムコードのワイルドカード(0xFFFF)は使えない．
　~~0x4FFFでは反応しない~~→**当たり前．ワイルドカードは1バイト単位．0x40FFで反応した．**
　0x4000決め打ちである必要がある．0xFFFFはNFC-DEPが反応する．
・**サンプルそのまんまでお行儀が悪いと**サービスがよく落ちる(NFC service dead)
　落ちている場合は再度enableServiceが必要．
・~~かざしながらアプリを起動するとたいていサービスが落ちる~~~
　→お行儀良く作り直したら落ちにくくなった．たまに落ちる
・反応しないときはアプリを再起動
・リソースが間違ってるとうんともすんとも言わない

**※~~RFにかざしたまま~~ PC/SCでNFC-FをポーリングしているPaSoRiにかざしたまま，最後に通信してから(あるいはかざしてから)10秒以上経つとNFCサービスが落ちて反応がなくなる．
9秒以内に通信をするとまた9秒間は待ってくれるが，10秒経つと落ちる．(Nexus 5Xにて)**

→PC/SCのNFC-Fポーリングを無効にしたらかざし続けてもサービス落ちなくなった．
また，FelicaアプリケーションがPaSoRiを掴んでる間はかざし続けても落ちないっぽい．

#整理されていない例
Android 8.1.0のNexus 5Xで動作確認を行った．

※Felica Liteの再現をしてみた方のHCEFService.javaはこちら
[AndroidのHCE-FでFelica Lite相当の機能を実装してみた](https://qiita.com/gpsnmeajp/items/317064e357989fc1a62b)

**[より使いやすくしたものはこちら](https://sabowl.sakura.ne.jp/gpsnmeajp/android/LazyNfcFHCELite/)**

```xml:AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.gpsnmeajp.hce_test">

    <!-- HCE-F -->
    <uses-sdk
        android:minSdkVersion="24"
        android:targetSdkVersion="24" />

    <uses-feature
        android:name="android.hardware.nfc.hcef"
        android:required="true" />

    <uses-permission android:name="android.permission.NFC" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name"
            android:theme="@style/AppTheme.NoActionBar">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>

        </activity>

        <!-- HCE-F -->
        <service
            android:name=".HCEFService"
            android:exported="true"
            android:permission="android.permission.BIND_NFC_SERVICE">
            <intent-filter>
                <action android:name="android.nfc.cardemulation.action.HOST_NFCF_SERVICE" />
            </intent-filter>
            <meta-data
                android:name="android.nfc.cardemulation.host_nfcf_service"
                android:resource="@xml/host_nfcf_service"/>
                <!-- 別途res/xml/host_nfcf_service.xmlというファイルが必要-->
        </service>
    </application>

</manifest>
```

```xml:res/xml/host_nfcf_service.xml
<?xml version="1.0" encoding="utf-8"?>
<host-nfcf-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:description="@string/service_desc"><!-- ←自分でstring.xmlに定義する -->
    <system-code-filter android:name="4000" /> <!--4000以外はSonyに割当申請する必要あり？ -->
    <nfcid2-filter android:name="02FE000000000000" /> <!-- "random"でもよい -->
    <t3tPmm-filter android:name="00F0000020600300"/>
</host-nfcf-service>
```

```xml:strings.xml
<resources>
    <string name="app_name">HCE_Test</string>
    <string name="action_settings">Settings</string>
    <string name="service_desc">This is description</string>
</resources>

```

```java:MainActivity.java
package com.example.gpsnmeajp.hce_test;

import android.content.ComponentName;
import android.content.pm.PackageManager;
import android.nfc.NfcAdapter;
import android.nfc.cardemulation.NfcFCardEmulation;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;

import android.util.Log;

public class MainActivity extends AppCompatActivity{
    private String debugMsgTitle = "NFCF-TEST";

    private NfcAdapter nfcAdapter;
    private NfcFCardEmulation nfcFCardEmulation;
    private ComponentName componentName;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d(debugMsgTitle, "Create");
        setContentView(R.layout.activity_main);
        
        //------- HCE-F --------
        PackageManager pm = getPackageManager();
        if (!pm.hasSystemFeature(PackageManager.FEATURE_NFC_HOST_CARD_EMULATION_NFCF)) {
            Log.e(debugMsgTitle,"This device not supports HCE-F.");
            finish();
        }else {
            Log.d(debugMsgTitle, "This device supports HCE-F.");
        }

        nfcAdapter = NfcAdapter.getDefaultAdapter(this);
        nfcFCardEmulation = NfcFCardEmulation.getInstance(nfcAdapter);
        componentName = new ComponentName(
                "com.example.gpsnmeajp.hce_test", //自パッケージ名
                "com.example.gpsnmeajp.hce_test.HCEFService"); //自サービス名
    }

    @Override
    protected void onResume() {
        super.onResume();
        Log.d(debugMsgTitle,"Resume");

        Log.d(debugMsgTitle,"Enable service");
        nfcFCardEmulation.enableService(this, componentName);
    }

    @Override
    protected void onPause() {
        super.onPause();
        Log.d(debugMsgTitle,"Pause");

        Log.d(debugMsgTitle,"Disable service");
        nfcFCardEmulation.disableService(this);
    }
}
```

```java:HCEFService.java
package com.example.gpsnmeajp.hce_test;

import android.app.Notification;
import android.app.NotificationManager;
import android.app.PendingIntent;
import android.app.Service;
import android.content.Intent;
import android.nfc.cardemulation.HostNfcFService;
import android.os.Bundle;
import android.os.IBinder;
import android.util.Log;
import android.widget.Toast;

import java.util.Arrays;

public class HCEFService extends HostNfcFService {

    @Override
    public void onCreate() {
        Log.d("HCEFService","onCreate NFCF");
        super.onCreate();
        Toast.makeText(this, "onCreate", Toast.LENGTH_LONG).show();
    }

    @Override
    public void onDestroy() {
        Log.d("HCEFService","onDestroy NFCF");
        super.onDestroy();
    }
    //onBindは作れません

    @Override
    public byte[] processNfcFPacket(byte[] commandPacket, Bundle extras)
    {
        Log.d("HCEFService","onDeactivated: Received NFCF");
        Toast.makeText(this, "onDeactivated", Toast.LENGTH_LONG).show();

        if (commandPacket.length < 1 + 1 + 8) {
            Log.e("HCEFService","processNfcFPacket: packt size error");
            return null;
        }
        byte[] mMyNfcid2 = {(byte)0x02,(byte)0xFE,0x00,0x00,0x00,0x00,0x00,0x00};
        byte[] nfcid2 = new byte[8];
        System.arraycopy(commandPacket,2, nfcid2, 0, 8);
        if (!Arrays.equals(mMyNfcid2,nfcid2)) {
            return null;
        }
        if (commandPacket[1] == (byte)0x04) { //pollingコマンド
            byte[] resp = new byte[1 + 1 + 8 + 1];
            resp[0] = (byte)11;// LEN
            resp[1] = (byte)0x05;// polling Response
            System.arraycopy(nfcid2, 0, resp, 2, 8);// NFCID2
            resp[10] = (byte)0;// Mode
            return resp;
        } else
        {
            return null;//応答しない
        }

    }

    @Override
    public void onDeactivated(int reason) { //RF切断時
        Log.d("HCEFService","onDeactivated NFCF");
        /*
        if(reason == DEACTIVATION_LINK_LOSS)
        {
            Log.d("HCEFService","onDeactivated: DEACTIVATION_LINK_LOSS");
        }else{
            Log.d("HCEFService","onDeactivated: Unknown reason");
        }
        */
    }

}
```


#うまく動いている場合のログ(HCEFでフィルタ)

```
RegisteredNfcFServicesCache: Service unchanged, not updating

02-07 21:30:38.393 21114-21136/? D/RegisteredT3tIdentifiersCache: Resolved to: NfcFService: ComponentInfo{com.example.gpsnmeajp.hce_test/com.example.gpsnmeajp.hce_test.HCEFService}, description: NFC-F, System Code: 4000, NFCID2: 02FE000000000001, T3T PMM:FFFFFFFFFFFFFFFF
02-07 21:30:38.393 21114-21136/? D/HostNfcFEmulationManager: Binding to service ComponentInfo{com.example.gpsnmeajp.hce_test/com.example.gpsnmeajp.hce_test.HCEFService}
02-07 21:30:38.399 21114-21136/? D/HostNfcFEmulationManager: Waiting for new service.
02-07 21:30:38.506 21114-21114/? D/HostNfcFEmulationManager: Service bound
02-07 21:30:38.507 21114-21114/? D/HostNfcFEmulationManager: Sending data to service
02-07 21:30:38.751 21114-21136/? D/RegisteredT3tIdentifiersCache: Resolved to: NfcFService: ComponentInfo{com.example.gpsnmeajp.hce_test/com.example.gpsnmeajp.hce_test.HCEFService}, description: NFC-F, System Code: 4000, NFCID2: 02FE000000000001, T3T PMM:FFFFFFFFFFFFFFFF
02-07 21:30:38.751 21114-21136/? D/HostNfcFEmulationManager: Sending data to service
02-07 21:30:39.272 21114-21136/? D/HostNfcFEmulationManager: Unbinding from service ComponentInfo{com.example.gpsnmeajp.hce_test/com.example.gpsnmeajp.hce_test.HCEFService}
02-07 21:32:49.201 28345-28345/? D/RegisteredNfcFServicesCache: Service unchanged, not updating

02-07 21:10:58.019 16633-16633/com.example.gpsnmeajp.hce_test E/NFC: NFC service dead - attempting to recover
                                                                    android.os.DeadObjectException
                                                                        at android.os.BinderProxy.transactNative(Native Method)
                                                                        at android.os.BinderProxy.transact(Binder.java:764)
                                                                        at android.nfc.INfcAdapter$Stub$Proxy.getState(INfcAdapter.java:407)
                                                                        at android.nfc.NfcAdapter.isEnabled(NfcAdapter.java:699)
                                                                        at android.nfc.NfcAdapter.getNfcFCardEmulationService(NfcAdapter.java:624)
                                                                        at android.nfc.cardemulation.NfcFCardEmulation.recoverService(NfcFCardEmulation.java:475)
                                                                        at android.nfc.cardemulation.NfcFCardEmulation.enableService(NfcFCardEmulation.java:328)
                                                                        at com.example.gpsnmeajp.hce_test.MainActivity.onResume(MainActivity.java:85)
                                                                        at android.app.Instrumentation.callActivityOnResume(Instrumentation.java:1355)
                                                                        at android.app.Activity.performResume(Activity.java:7107)
                                                                        at android.app.ActivityThread.performResumeActivity(ActivityThread.java:3556)
                                                                        at android.app.ActivityThread.handleResumeActivity(ActivityThread.java:3621)
                                                                        at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2862)
                                                                        at android.app.ActivityThread.-wrap11(Unknown Source:0)
                                                                        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1589)
                                                                        at android.os.Handler.dispatchMessage(Handler.java:106)
                                                                        at android.os.Looper.loop(Looper.java:164)
                                                                        at android.app.ActivityThread.main(ActivityThread.java:6494)
                                                                        at java.lang.reflect.Method.invoke(Native Method)
                                                                        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:438)
                                                                        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:807)
                                                                        
                                                                        
02-07 22:08:08.898 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onCreate NFCF
02-07 22:08:08.937 20885-20885/? D/HostNfcFEmulationManager: Service bound
02-07 22:08:08.937 20885-20885/? D/HostNfcFEmulationManager: Sending data to service
02-07 22:08:08.949 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onDeactivated: Received NFCF
02-07 22:08:09.033 20885-20908/? D/HostNfcFEmulationManager: Unbinding from service ComponentInfo{com.example.gpsnmeajp.hce_test/com.example.gpsnmeajp.hce_test.HCEFService}
02-07 22:08:09.083 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onDeactivated NFCF
02-07 22:08:09.084 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onDestroy NFCF
02-07 22:08:09.208 20885-20908/? D/RegisteredT3tIdentifiersCache: Resolved to: NfcFService: ComponentInfo{com.example.gpsnmeajp.hce_test/com.example.gpsnmeajp.hce_test.HCEFService}, description: This is description, System Code: 4000, NFCID2: 02FE000000000002, T3T PMM:00F0000020600300
02-07 22:08:09.209 20885-20908/? D/HostNfcFEmulationManager: Binding to service ComponentInfo{com.example.gpsnmeajp.hce_test/com.example.gpsnmeajp.hce_test.HCEFService}
02-07 22:08:09.213 20885-20908/? D/HostNfcFEmulationManager: Waiting for new service.
02-07 22:08:09.214 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onCreate NFCF
02-07 22:08:09.232 20885-20885/? D/HostNfcFEmulationManager: Service bound
02-07 22:08:09.232 20885-20885/? D/HostNfcFEmulationManager: Sending data to service
02-07 22:08:09.245 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onDeactivated: Received NFCF
02-07 22:08:09.627 20885-20908/? D/HostNfcFEmulationManager: Unbinding from service ComponentInfo{com.example.gpsnmeajp.hce_test/com.example.gpsnmeajp.hce_test.HCEFService}
02-07 22:08:09.951 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onDeactivated NFCF
02-07 22:08:09.952 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onDestroy NFCF
02-07 22:08:39.719 22075-22075/com.example.gpsnmeajp.hce_test E/NFC: NFC service dead - attempting to recover
                                                                    android.os.DeadObjectException
                                                                        at android.os.BinderProxy.transactNative(Native Method)
                                                                        at android.os.BinderProxy.transact(Binder.java:764)
                                                                        at android.nfc.INfcAdapter$Stub$Proxy.getState(INfcAdapter.java:407)
                                                                        at android.nfc.NfcAdapter.isEnabled(NfcAdapter.java:699)
                                                                        at android.nfc.NfcAdapter.getNfcFCardEmulationService(NfcAdapter.java:624)
                                                                        at android.nfc.cardemulation.NfcFCardEmulation.recoverService(NfcFCardEmulation.java:475)
                                                                        at android.nfc.cardemulation.NfcFCardEmulation.enableService(NfcFCardEmulation.java:328)
                                                                        at com.example.gpsnmeajp.hce_test.MainActivity.onResume(MainActivity.java:88)
                                                                        at android.app.Instrumentation.callActivityOnResume(Instrumentation.java:1355)
                                                                        at android.app.Activity.performResume(Activity.java:7107)
                                                                        at android.app.ActivityThread.performResumeActivity(ActivityThread.java:3556)
                                                                        at android.app.ActivityThread.handleResumeActivity(ActivityThread.java:3621)
                                                                        at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2862)
                                                                        at android.app.ActivityThread.-wrap11(Unknown Source:0)
                                                                        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1589)
                                                                        at android.os.Handler.dispatchMessage(Handler.java:106)
                                                                        at android.os.Looper.loop(Looper.java:164)
                                                                        at android.app.ActivityThread.main(ActivityThread.java:6494)
                                                                        at java.lang.reflect.Method.invoke(Native Method)
                                                                        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:438)
                                                                        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:807)
                                                                        
02-07 22:08:41.572 22472-22495/? D/RegisteredT3tIdentifiersCache: Resolved to: NfcFService: ComponentInfo{com.example.gpsnmeajp.hce_test/com.example.gpsnmeajp.hce_test.HCEFService}, description: This is description, System Code: 4000, NFCID2: 02FE000000000002, T3T PMM:00F0000020600300
02-07 22:08:41.573 22472-22495/? D/HostNfcFEmulationManager: Binding to service ComponentInfo{com.example.gpsnmeajp.hce_test/com.example.gpsnmeajp.hce_test.HCEFService}
02-07 22:08:41.576 22472-22495/? D/HostNfcFEmulationManager: Waiting for new service.
02-07 22:08:41.583 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onCreate NFCF
02-07 22:08:41.602 22472-22472/? D/HostNfcFEmulationManager: Service bound
02-07 22:08:41.602 22472-22472/? D/HostNfcFEmulationManager: Sending data to service
02-07 22:08:41.628 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onDeactivated: Received NFCF
02-07 22:08:41.970 22472-22495/? D/HostNfcFEmulationManager: Unbinding from service ComponentInfo{com.example.gpsnmeajp.hce_test/com.example.gpsnmeajp.hce_test.HCEFService}
02-07 22:08:41.971 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onDeactivated NFCF
02-07 22:08:41.974 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onDestroy NFCF
02-07 22:08:42.142 22472-22495/? D/RegisteredT3tIdentifiersCache: Resolved to: NfcFService: ComponentInfo{com.example.gpsnmeajp.hce_test/com.example.gpsnmeajp.hce_test.HCEFService}, description: This is description, System Code: 4000, NFCID2: 02FE000000000002, T3T PMM:00F0000020600300
02-07 22:08:42.142 22472-22495/? D/HostNfcFEmulationManager: Binding to service ComponentInfo{com.example.gpsnmeajp.hce_test/com.example.gpsnmeajp.hce_test.HCEFService}
02-07 22:08:42.145 22472-22495/? D/HostNfcFEmulationManager: Waiting for new service.
02-07 22:08:42.155 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onCreate NFCF
02-07 22:08:42.169 22472-22472/? D/HostNfcFEmulationManager: Service bound
02-07 22:08:42.170 22472-22472/? D/HostNfcFEmulationManager: Sending data to service
02-07 22:08:42.189 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onDeactivated: Received NFCF
02-07 22:08:42.563 22472-22495/? D/HostNfcFEmulationManager: Unbinding from service ComponentInfo{com.example.gpsnmeajp.hce_test/com.example.gpsnmeajp.hce_test.HCEFService}
02-07 22:08:42.563 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onDeactivated NFCF
02-07 22:08:42.568 22075-22075/com.example.gpsnmeajp.hce_test D/HCEFService: onDestroy NFCF
```


#検証ツール

使用しているライブラリ
[felicalib_segfix](https://sites.google.com/site/gpsnmeajp/tools/felicalib_segfix)

```cpp:main.cpp
void hexdump(uint8 *addr, int n)
{
	for (int i=0;i<n;i++)
		printf("%02X ", addr[i]);
	printf("\n");
}

int main()
{
	// --- PaSoRiを開く --- 
	pasori *p = pasori_open();
	if (p == NULL)
	{
		printf("Felicaライブラリがオープンできませんでした\n");
		return -1;
	}

	// --- PaSoRiを初期化 --- 
	if (pasori_init(p) != 0)
	{
		printf("PaSoRiが開けませんでした．(使用中or未接続)\n");
		return -1;
	}

	// --- カードを検出 --- 
	felica *f = NULL; //fハンドル
	while ( !f )
	{
		printf("カードを探しています...\n");
		f = felica_polling(p, 0x4000, 0);
	}

	// --- pollingで得られた情報 --- 
	printf("カードが見つかりました\n");
	printf("# IDm: "); hexdump(f->IDm,8);

	// --- 読み出し --- 
	UINT8 readData[16]={0};
	int readCode = felica_read_without_encryption03(f, SERVICECODE_RO, 0x1780, readData);
	if ( !readCode )
	{
		printf("# Read: "); hexdump(readData, 16);
	}else{
		printf("# Read Error\n");
		print_response_error(readCode);
	}

	// --- 書き込み --- 
	UINT8 writeData[16] = {"Hello World"};
	int writeCode = felica_write_without_encryption03(f, SERVICECODE_RW, 0x1780, writeData);
	if ( !writeCode )
	{
		printf("# Write: "); hexdump(writeData, 16);
	} else
	{
		printf("# Write Error\n");
		print_response_error(writeCode);
	}

	// --- 終了処理 --- 
	felica_free(f);
	pasori_close(p);
	return 0;

}
```
