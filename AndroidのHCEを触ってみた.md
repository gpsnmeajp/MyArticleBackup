#概要
Android 4.4から提供されているHCE (Host Card Emulation)を触ってみたのでコードを張っておきます。

#HCE-Fとの違い
FeliCaを使うHCE-F(Android 7から提供)との違いは以下です

+ NFC Type-A/Type-B規格(MIFARE DESfire系と考えて良い)を使うので世界中で使える
+ ISO7816-4に沿って通信するのでその辺の知識が必要(カードとの互換性を気にしなければ、気にせず適当に実装することも可能)
+ ISO7816-4に従って通信するため、かなり長時間の待ち時間をもたせることもできるようだ
+ AID(アプレット識別子)できちんと識別されるので、FeliCaのように独自の識別を行う必要性が低い
+ AID(アプレット識別子)で分離されるので複数のアプリと共存できる
+ バックグラウンドでも受け付けられる
+ ロック画面でも通信できる(スリープ解除が必要)
+ Windows Phoneも対応しているらしい

HCE-F(FeliCa)についてはこちらを参照ください
[AndroidのHCE-Fについて調べてみたメモとサンプルソース](https://qiita.com/gpsnmeajp/items/e525bb5c13511c18dda5)

#実装
HCE-Fと異なり、Activityからの操作は不要。
アプリのインストール完了時点から待受開始される。

android:requireDeviceUnlock="true"にすると、ロック解除しないと通信が成立しなくなる。

android:category="other"は自由な通信。ポイントカードなど、独自規格で送受信対象が限定されている場合など。
android:category="payment"にすると、ユーザーがAndroidの設定画面で支払手段として選択しないと動作しなくなる。
これはクレジットカードアプリ同士の干渉・多重支払いを防止するため。
paymentの場合は、apduServiceBannerが別途必要になる。(メニューに表示するアイコン)

aid-filterが、カード内のアプリケーションを識別する識別子。
Android端末は1つのスマートカードとして動作し、アプリはその中の1つのアプリケーションとして動作する。

クレジットカードなどはすでに定義済み。
大抵は独自に定義することになるだろう。16Byteで設定しておくのが無難。

AID(application identifier)は以下のルールで定められている．
最初の4bitが

A:国際的に登録されたAID
D:国内用に登録されたAID
F:登録無しで自由に使用できるID
参考:[Android HCE: are there rules for AID?](https://stackoverflow.com/questions/27533193/android-hce-are-there-rules-for-aid)
空間が広いため，よっぽどのことがないかぎり衝突することはないだろう．

通信手順は、AIDのSELECTは必須だが、それ以降はアプリにお任せになる。
ので、もう一度AIDのSELECTを行う以外の操作は、一切自由に解釈・実行できるらしいので、独自プロトコルを定義しても多分問題ない。
(この辺よく理解できていないので、誤った理解をしている可能性もあるのでご注意を)

```xml:res/xml/apduservice.xml
<host-apdu-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:description="@string/servicedesc"
    android:requireDeviceUnlock="false">
    <aid-group android:description="@string/aiddescription"
        android:category="other">
        <aid-filter android:name="F0010203040506"/>
    </aid-group>
</host-apdu-service>
```

```xml:res/values/strings.xml
<resources>
    <string name="app_name">HCE_APDU_test</string>
    <string name="servicedesc">HCE_APDU_Service</string>
    <string name="aiddescription">HCE_APDU_Service AID</string>
</resources>
```

```xml:AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="jp.ne.sakura.sabowl.gpsnmeajp.hce_apdu_test">

    <!-- HCE APDU -->
    <uses-feature
        android:name="android.hardware.nfc.hce"
        android:required="true" />

    <uses-permission android:name="android.permission.NFC" />
    <uses-permission android:name="android.permission.VIBRATE"/>

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <!-- HCE APDU -->
        <service
            android:name=".ApduService"
            android:exported="true"
            android:permission="android.permission.BIND_NFC_SERVICE">
            <intent-filter>
                <action android:name="android.nfc.cardemulation.action.HOST_APDU_SERVICE" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>

            <meta-data
                android:name="android.nfc.cardemulation.host_apdu_service"
                android:resource="@xml/apduservice" />
        </service>
    </application>

</manifest>
```

```java:ApduService.java
package jp.ne.sakura.sabowl.gpsnmeajp.hce_apdu_test;

import android.app.Notification;
import android.app.NotificationChannel;
import android.app.NotificationManager;
import android.app.Service;
import android.content.Context;
import android.content.Intent;
import android.graphics.Color;
import android.nfc.cardemulation.HostApduService;
import android.os.Build;
import android.os.Bundle;
import android.os.Handler;
import android.os.IBinder;
import android.support.v4.app.NotificationCompat;
import android.support.v4.app.NotificationManagerCompat;
import android.support.v4.content.ContextCompat;
import android.util.Log;
import android.widget.Toast;

public class ApduService extends HostApduService {
    String _TAG = "ApduService";

    class APDUdatas{
        public byte CLA;
        public byte INS;
        public byte P1;
        public byte P2;
        public byte[] raw;
    }

    private byte[] RESPONSE_SUCCESS = new byte[]{(byte)0x90,(byte)0x00};

    private byte[] RESPONSE_EXECUTION_ERROR = new byte[]{(byte)0x64,(byte)0xFF};
    private byte[] RESPONSE_INVALID_COMMAND = new byte[]{(byte)0x69,(byte)0xFF};

    private byte[] RESPONSE_FILE_NOT_FOUND = new byte[]{(byte)0x6A,(byte)0x82};
    private byte[] RESPONSE_WRITE_ERROR = new byte[]{(byte)0x65,(byte)0x00};

    private byte[] RESPONSE_AUTHENTICATION_ERROR = new byte[]{(byte)0x63,(byte)0x00};
    private byte[] RESPONSE_FILE_FULL = new byte[]{(byte)0x63,(byte)0x81};

    private byte[] RESPONSE_NO_FUNCTION = new byte[]{(byte)0x6A,(byte)0x81};
    private byte[] RESPONSE_COMMAND_ERROR = new byte[]{(byte)0x69,(byte)0x85};
    private byte[] RESPONSE_INVALID_INS = new byte[]{(byte)0x6D,(byte)0x00};
    private byte[] RESPONSE_INVALID_CLA = new byte[]{(byte)0x6E,(byte)0x00};
    private byte[] RESPONSE_INVALID_LC_LE = new byte[]{(byte)0x67,(byte)0x00};

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(_TAG,"onCreate");


    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(_TAG,"onDestroy");
    }

    @Override
    public byte[] processCommandApdu(byte[] apdu, Bundle extras) {
       Log.d(_TAG,"processCommandApdu");

       //SELECT FILE
       if(apdu[0]==(byte)0x00 && apdu[1]==(byte)0xA4 && apdu[2]==(byte)0x04 && apdu[3]==(byte)0x00)
       {
           Log.d(_TAG,"processCommandApdu : SELECT FILE");
           return RESPONSE_SUCCESS;
       }
           //0x00,0xA4,0x04,0x00

        notice("processCommandApdu");

        showhex(apdu);
       return RESPONSE_SUCCESS;
    }
    @Override
    public void onDeactivated(int reason) {
        Log.d(_TAG,"onDeactivated");
    }


    private void showhex(byte[] dat){
        String dbg="";
        for(int i=0;i<dat.length;i++)
        {
            dbg += String.format("%02X ",dat[i]);
        }
        Log.d(_TAG,dbg);
    }

    private void notice(String msg)
    {
        final String CHANNEL_ID = "sample_notification_channel";
        final int ID = 0;
        NotificationCompat.Builder mBuilder;

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel channel = new NotificationChannel(CHANNEL_ID,"This is notificationChannel1",NotificationManager.IMPORTANCE_HIGH);
            channel.setLockscreenVisibility(Notification.VISIBILITY_PUBLIC);
            channel.enableVibration(true);

            NotificationManager manager = (NotificationManager)getSystemService(Context.NOTIFICATION_SERVICE);
            manager.createNotificationChannel(channel);
            mBuilder = new NotificationCompat.Builder(this, CHANNEL_ID);
        }else{
            mBuilder = new NotificationCompat.Builder(this);
        }

        mBuilder.setSmallIcon(R.drawable.ic_stat_name)
                        .setContentTitle("ApduService")
                        .setContentText(msg)
                        .setColor(Color.rgb(0,255,0))
                        .setDefaults(Notification.DEFAULT_ALL)
                        .setAutoCancel(true)
                        .setWhen(System.currentTimeMillis())
                        .setPriority(Notification.PRIORITY_HIGH)
                        .setVibrate(new long[]{100, 0, 100, 0, 100, 0});

        NotificationManagerCompat manager = NotificationManagerCompat.from(this);
        manager.notify(ID, mBuilder.build());
    }
}
```

```java:MainActivity.java
package jp.ne.sakura.sabowl.gpsnmeajp.hce_apdu_test;

import android.app.Notification;
import android.app.NotificationChannel;
import android.app.NotificationManager;
import android.app.PendingIntent;
import android.app.TaskStackBuilder;
import android.content.Context;
import android.content.Intent;
import android.graphics.Color;
import android.os.Build;
import android.support.v4.app.NotificationCompat;
import android.support.v4.app.NotificationManagerCompat;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }


    @Override
    protected void onResume()
    {
        super.onResume();


    }
}
```

#PC側の通信
開発中のライブラリ(未公開)を使用しているので参考程度だが、以下のような感じで通信する。
通信にはPaSoRiを使用してもうまくできない(PC/SCの場合。FeliCa/NFC SDKを使えばできるらしい)
そのため、ACR1251を購入して試している。

NTTコミュニケーションズが出しているACR1251CLで良い。(現在2780円)
[yodobashi.comで購入できる](https://www.yodobashi.com/product/100000001003065479/)

余談だが、ACR1251はこちらはこちらでFeliCaのタイムアウトが厳しすぎて、HCE-Fとの通信には使い物にならない。
(試してみた限り、タイムアウト時間の調整も一切できない)
Type-Aは何ら問題なく、タイムアウトせず延々と待機してくれる。


```cpp:main.cpp
#include <stdio.h>
#include "PCSCACR1251CLlib.h"
using LazyPCSCACR1251C::PCSCACR1251CL;

int main()
{
	try
	{
		PCSCACR1251CL rw = PCSCACR1251CL(true);
		rw.openService();

		while ( 1 )
		{
			rw.connectDirect();
			rw.setLedIndicator(0x7F);
			rw.setPolling(true);

			//AndroidがデフォルトでP2P接続に使うFeliCaを引っ掛けると面倒なためフィルタしている。
			rw.setPollingSelect(PCSCACR1251CL::DETECT_TYPE_AB);

			rw.disconnectCard();

			rw.waitForSetCard();
			rw.connectCard();
			rw.readATR();

			uint8_t dat[16] = {0x01,0x10,0x00};
			uint8_t aid[] = {0xF0,0x01,0x02,0x03,0x04,0x05,0x06};

			//カード内アプリを選択
			rw.selectFile(aid, sizeof(aid));
			//読み取りコマンドを発行
			rw.readBinary(0,dat,0,0);

			rw.disconnectCard();
			Sleep(250);
			getchar();
		}
	} catch ( std::runtime_error e )
	{
		printf("%s\n", e.what());
	}
}
```
