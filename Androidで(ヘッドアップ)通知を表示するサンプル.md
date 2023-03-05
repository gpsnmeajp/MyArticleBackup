#通知を出したい
通知を出したい
→ なんかいっぱい方法ある
　→ NotificationCompat.Builderを使うのが安定らしい

まず通知が表示されない
→ アイコンを設定する
→ NotificationChannelが必要(Android O以降のみ)

ヘッドアップ通知を出したい
→　でない
　→ 優先度を上げる
　→ バイブレーションを許可する
　→ 他色々
　→ NotificationChannelの設定が必要
　　→ アンインストールするまで設定が残っている

**API Levelごとに必要な情報が違いすぎてつらい**

```java
   private void notice()
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
                        .setContentText("processCommandApdu")
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

# Foreground Service
```kotlin
package com.example.servicetest

import android.app.Notification
import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.Service
import android.content.Context
import android.content.Intent
import android.os.Build
import android.os.IBinder
import androidx.core.app.NotificationCompat


class MyService : Service() {

    override fun onBind(intent: Intent): IBinder {
        TODO("Return the communication channel to the service.")
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val CHANNEL_ID = "sample_notification_channel"

        val mBuilder: NotificationCompat.Builder
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                CHANNEL_ID,
                "This is notificationChannel1",
                NotificationManager.IMPORTANCE_HIGH
            )
            channel.lockscreenVisibility = Notification.VISIBILITY_PUBLIC
            channel.enableVibration(true)
            val manager =
                getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
            manager.createNotificationChannel(channel)
            mBuilder = NotificationCompat.Builder(this, CHANNEL_ID)
        } else {
            mBuilder = NotificationCompat.Builder(this)
        }

        val notification = mBuilder
            .setSmallIcon(R.mipmap.ic_launcher)
            .setContentTitle("タイトル")
            .setContentText("通知")
            .build()

        Thread(
            Runnable {
                Thread.sleep(30000);
                stopForeground(true);
            }
        ).start()

        startForeground(1,notification)

        return START_STICKY
    }
}

```

#参考文献
[Android] Alarm をNotificationManager で通知する 
https://akira-watson.com/android/alarm-notificationmanager.html

Androidの通知チャンネルの振る舞いをプロジェクトのtargetSDKVersionを変えて少し試してみた
http://woshidan.hatenablog.com/entry/2017/08/23/083000

Androidの通知をカスタムする(API21以降)
https://qiita.com/sakebook/items/8cafc0766b4f8dc95994

【Android】通知をカスタマイズしよう！ 
http://www.eda-inc.jp/post-622/

Android Oの通知チャンネルを使ってみる 
http://blog.techium.jp/entry/2017/09/11/090000

Android 5.0 Lolipop以上で通知アイコンが白くなってしまう問題を解決する
https://qiita.com/syarihu/items/95788cbab9b63100c4fb

Androidアプリでステータスバーに通知を出すやりかた。
https://qiita.com/steroid66/items/27f5ce27eb32eae49732

Foreground Serviceの基本
https://qiita.com/naoi/items/03e76d10948fe0d45597
