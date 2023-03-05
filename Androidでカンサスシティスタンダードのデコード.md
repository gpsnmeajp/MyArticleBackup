まともに整理していませんが、とりあえず貼っておきます。
正直汚いソースです。

おまけでArduinoから送信するサンプルも付いてます

#仕様
カンサスシティスタンダードのため、

+ FSK変調
+ 2400Hzの8サイクルで1
+ 1200Hzの4サイクルで0です。

よって300bpsとなります。

py-kcsで生成したデータで動作を確認しています。
http://www.dabeaz.com/py-kcs/index.html

wikipediaをご参照ください

カンサスシティスタンダード
https://ja.wikipedia.org/wiki/%E3%82%AB%E3%83%B3%E3%82%B5%E3%82%B9%E3%82%B7%E3%83%86%E3%82%A3%E3%82%B9%E3%82%BF%E3%83%B3%E3%83%80%E3%83%BC%E3%83%89

#処理の流れ

0. ランタイムパーミッションのチェックと獲得をする(手抜きで、権限処理後は必ずアプリが再起動するようにしている)
1. AudioRecordで音声取り込み
2. FIFOバッファに音声バッファを突っ込む
3. 立てておいたThreadを叩き起こす
4. 処理ThreadはFIFOバッファのサイズをチェック、あれば起きる、なければ寝る
5. 音声を1 word単位で処理。立ち上がりエッジから周波数測定
6. 周波数からビット抽出、バイト判定、組み立て
7. 片っ端から画面に出力

#作ってて思ったこと
+ AudioRecordのonPeriodicNotificationはUI Threadで走る。
+ さらに、毎回呼び出されるというよりは、readで一旦止まるっぽい。(止まってる間は別の処理が回る)
+ 1 word単位で処理してればバッファの大きさ気にしなくて良くて楽ですね
+ 音声処理スレッドが、時折超遅になる。原因不明。アプリ再起動で治る。
+ FIFOにはバッファごと突っ込むのが楽でいいですね
+ wait, notifyを初めて使ったが、楽で便利
+ Thread.interrupt()の利用価値が初めてわかった。javaの例外機構便利ですねぇ
+ 意外とマイク&スピーカーでも通信が成立する
+ 結構波形は歪む。

#Androidのマイク入力ケーブルを自作する場合の注意事項

+ マイク端子は4.7kΩくらいの抵抗を並列に繋がないと、マイクがつながっている判定にならないため、入力が切り替わらないことがある。
+ PCでUSBデバッグしながら実行する場合、PCにマイクを繋ぐと、見事にグラウンドループが生じてノイズだらけになる
+ Nexus 5Xでの検証だが、ケーブルかソフトウェアか不明だが、まずい状態になるとオーディオシステムが再起動しまくる。その際は他のアプリも落ちる
+ (マイク端子への入力が過大だと落ちるようである。1mV～10mV程度に抑えること)



#参考にしたページ
http://tadaoyamaoka.hatenablog.com/entry/2015/03/28/214647
https://groups.google.com/forum/#!topic/android-group-japan/JfLrtG3QfxE
https://firespeed.org/diary.php?diary=kenz-1821

##ソース

```java:kcs.java
package com.example.seg.audiotest;

import android.content.Intent;
import android.content.pm.ActivityInfo;
import android.content.pm.PackageManager;
import android.media.AudioFormat;
import android.media.AudioRecord;
import android.media.MediaRecorder;
import android.os.Handler;
import android.support.v4.app.ActivityCompat;
import android.support.v4.content.ContextCompat;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.support.v7.view.menu.ExpandedMenuView;
import android.util.Log;
import android.view.WindowManager;
import android.widget.Button;
import android.widget.SeekBar;
import android.widget.TextView;
import android.widget.Toast;
import android.view.View;

import java.util.Collections;
import java.util.concurrent.Semaphore;
import java.util.LinkedList;
import java.util.Queue;

public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    private static final int AUDIO_SAMPLE_FREQ = 44100;
    private static final int AUDIO_BUFFER_SIZE = AudioRecord.getMinBufferSize( //最小限のバッファサイズ
            AUDIO_SAMPLE_FREQ, AudioFormat.CHANNEL_IN_MONO,
            AudioFormat.ENCODING_PCM_16BIT); //13倍でおおよそ528msくらい
    private static final int FRAME_BUFFER_SIZE = AUDIO_BUFFER_SIZE / 2 ;

    private AudioRecord record = null;
    private long AudioFrameCount = 0;
    private long AudioFrameProcessdCount = 0;

    private int levels=0;

    Handler mHandler = new Handler(); //UIスレッドハンドラ

    Object AudioLock = new Object();
    LinkedList<short[]> AudioFrameQueue = new LinkedList<short[]>();

    Thread processorThread;
    String bits = "";

    Button btn,btn2;
    TextView txt,txt2;
    SeekBar seekBar;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d("AudioTest","onCreate");
        setContentView(R.layout.activity_main);

        //画面消灯禁止
        getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
        //画面回転禁止
        setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);

        txt = (TextView) findViewById(R.id.textView);
        txt2 = (TextView) findViewById(R.id.textView2);
        btn = (Button)   findViewById(R.id.button);
        btn.setOnClickListener(this);
        btn2 = (Button)   findViewById(R.id.button2);
        btn2.setOnClickListener(this);
        seekBar = (SeekBar)findViewById(R.id.seekBar5);

        //---------------------
        //マイク権限のチェックと獲得
        if(!checkAndRequestRuntimePermission())
        {
            return;
        }
    }

    @Override
    protected void onResume(){
        super.onResume();
        Log.d("AudioTest","onResume");

        //startRecording();

        //音声処理を別スレッドで行う
        processorThread = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    try {
                        //起こされるのを待つ。関係ないやつに起こされてもいいように条件チェックをする
                        while (AudioFrameQueue.size() < 1) {
                            synchronized(AudioLock) {
                                AudioLock.wait();
                            }
                        }
//                        Log.d("processSound", String.format("%d", AudioFrameQueue.size()));

                        short buf[];
                        synchronized (AudioFrameQueue) {
                            buf = AudioFrameQueue.poll();//バッファごと取り出す
                        }

                        for (int i = 0; i < FRAME_BUFFER_SIZE; i++) {
                            processSound(buf[i]);
                        }

                        mHandler.post(new Runnable() {
                            @Override
                            public void run() {
                                txt2.setText("Current Value:"+seekBar.getProgress());
                                levels = seekBar.getProgress();
                                txt.setText(bits);
                            }
                        });
//                        Log.d("processSound DATA", bits);
//                        bits="";

                    } catch (InterruptedException e) {
                        //終了処理
                        return;
                    }
                }
            }
        });
        processorThread.start();
    }

    @Override
    protected void onPause(){
        super.onPause();
        Log.d("AudioTest","onPause");

        synchronized (AudioFrameQueue) {
            AudioFrameQueue.clear();//キューをクリアする
        }
        stopRecording();
        processorThread.interrupt(); //InterruptedExceptionを発生させる
    }

    public void onClick(View v) {
        Button btn = (Button)v;

        switch( btn.getId() ){
            //ボタンが押されたとき
            case R.id.button:
                startRecording();
                bits="";
                break;
            case R.id.button2:
                stopRecording();
                break;
            default:
                break;

        }
    }

    //-----------マイク処理-------------
    int stat=0;
    double Hz=0;
    int high_cnt=0;
    int low_cnt=0;
    long sample_cnt=0;

    boolean bit_high = false;
    boolean bit_low = false;
    char bitbuf=0;
    int bitcnt=0;
    boolean started = false;

    private void processSound(short in)
    {
        boolean gap=false;
        // Log.d("processSound",String.format("%d",in));
        if((in < -levels) && (stat==1))
        {
            stat = 0;
            gap=true;
        }
        if((in > levels) && (stat==0))
        {
            stat = 1;
//            gap=true;
        }
        sample_cnt++;

        if(gap){
            double time = (double)sample_cnt/(double)AUDIO_SAMPLE_FREQ;
            double Hz = 1./time;
            sample_cnt=0;
            bit_high = false;
            bit_low = false;

            if((Hz > 1800) && (Hz < 3000))
            {
                high_cnt++;
                if(high_cnt == 8)
                {
                    //bits += "1";
                    bit_high = true;
                    high_cnt=0;
                    low_cnt=0;
                }
                if(low_cnt > 2)
                {
                    //bits += "0";
                    bit_low = true;
                    low_cnt=0;
                }
            }
            if((Hz > 600) && (Hz < 1800))
            {
                low_cnt++;
                if(low_cnt == 4)
                {
                    //bits += "0";
                    bit_low = true;
                    low_cnt=0;
                    high_cnt=0;
                }
                if(high_cnt > 4)
                {
                    //bits += "1";
                    bit_high = true;
                    high_cnt=0;
                }
            }
            //--------bit to byte-----
            if(bit_low && !started)
            {
                started=true;
                bitbuf=0;
                bitcnt=0;
            }

            if(bit_high && started)
            {
                //ストップビット
                if(bitcnt > 8) {
                    bits += String.format("%c", bitbuf);
                    started=false;
                }else {
                    bitbuf >>= 1;
                    bitbuf |= 0x80;
                    bitcnt++;
                }
            }
            if(bit_low && started)
            {
                bitbuf >>= 1;
                bitbuf |= 0x00;
                bitcnt++;
            }
        }
    }

    private void startRecording()
    {
        if(record != null)
        {
            return;
        }

        Log.d("AudioTest",String.format("FRAME_BUFFER_SIZE: %d[word]\nTIME: %f[ms]",FRAME_BUFFER_SIZE,1000.*(double)FRAME_BUFFER_SIZE/(double)AUDIO_SAMPLE_FREQ));

        //優先度をオーディオレベルに上げる
        android.os.Process.setThreadPriority(android.os.Process.THREAD_PRIORITY_URGENT_AUDIO);

        record = new AudioRecord(MediaRecorder.AudioSource.MIC,
                AUDIO_SAMPLE_FREQ, AudioFormat.CHANNEL_IN_MONO,
                AudioFormat.ENCODING_PCM_16BIT, AUDIO_BUFFER_SIZE);
        record.setRecordPositionUpdateListener(new AudioRecord.OnRecordPositionUpdateListener() {
            // オーディオフレームごとの処理(UIスレッドなことに注意！！)
            @Override
            public void onPeriodicNotification(AudioRecord recorder) {
                short tmp_buf[] = new short[FRAME_BUFFER_SIZE];
                recorder.read(tmp_buf, 0, FRAME_BUFFER_SIZE); //ここで次のFrameまでロックされる？

                long start = System.nanoTime();
                synchronized (AudioFrameQueue) {
                    AudioFrameQueue.offer(tmp_buf);//バッファごとFIFOに突っ込む
                }
                synchronized(AudioLock) {
                    //Threadを叩き起こす
                    AudioLock.notifyAll();
                }
            }

            @Override
            public void onMarkerReached(AudioRecord recorder) {
                //原則使わないらしい
            }
        });

        record.setPositionNotificationPeriod(FRAME_BUFFER_SIZE);

        // 録音開始
        record.startRecording();
        //1回は読んでおかないと以後呼び出しが発生しない
        short record_buffer[] = new short[FRAME_BUFFER_SIZE];
        record.read(record_buffer, 0, FRAME_BUFFER_SIZE);
    }
    private void stopRecording()
    {
        try {
            record.stop();
            record.release();
            record = null;
        }catch (Exception e)
        {
            //無視
        }
    }

    //-----------権限処理-------------
    //許可されていればTrue、されていなければFalse
    //自動でアプリを再起動するため、Falseの場合はreturnするだけで良い
    private Boolean checkAndRequestRuntimePermission()
    {
        if (ContextCompat.checkSelfPermission(
                this,android.Manifest.permission.RECORD_AUDIO)== PackageManager.PERMISSION_GRANTED){
            // 許可されている時の処理
            return true;
        }else{
            // 拒否されている時の処理
            if (ActivityCompat.shouldShowRequestPermissionRationale(this, android.Manifest.permission.RECORD_AUDIO)) {
                //拒否された時 Permissionが必要な理由を表示して再度許可を求めたり、機能を無効にしたりします。
                Toast.makeText(this, "音声の録音が許可されていません！\n本体設定のアプリ権限設定から\n許可してください！", Toast.LENGTH_LONG).show();
                finish();
                return false;
            } else {
                //まだ許可を求める前の時、許可を求めるダイアログを表示します。
                ActivityCompat.requestPermissions(this, new String[]{android.Manifest.permission.RECORD_AUDIO}, 0);
                return false;
            }
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
        switch (requestCode) {
            case 0: { //ActivityCompat#requestPermissions()の第2引数で指定した値
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    //許可された場合の処理
                    Toast.makeText(this, "マイクが利用可能になりました\n再起動します...", Toast.LENGTH_LONG).show();

                    //アプリを再起動
                    Intent intent = new Intent(this, MainActivity.class);
                    intent.setAction(Intent.ACTION_MAIN);
                    intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
                    startActivity(intent);

                    finish();
                }else{
                    //拒否された場合の処理
                    Toast.makeText(this, "マイクの利用を拒否されました\n実行を継続できません", Toast.LENGTH_LONG).show();
                    finish();
                }
                break;
            }
        }
    }
}
```


#Arduinoから送信するソース
java側プログラムのせいもあるし、適当回路のせいもあるのだが、極性によって一部のビットが化けることがある。
可能であれば、きちんと交流化して使用すべき

```cpp:Arduino_kcs.cpp
#define high_us 200 //412
#define low_us 400 //829
#define DATA_PIN 5

void setup() {
  pinMode(DATA_PIN,OUTPUT);
}

void loop() {
  data_send("Hello World! ");
  for(int i=0;i<800;i++)
      high(); 
}

void data_send(char str[])
{
  int adr=0;
  while(1)
  {
    if(str[adr]=='\0')
      break;
    unsigned char c = str[adr];
    adr++;
    //send
    low();
    for(int i=0;i<8;i++)
    {
      if(c&0x01)
      {
        high();  
      }else{
        low();
      }
      c >>=1;
    }
    high();  
    high();  
  }  
}

void high()
{
  digitalWrite(DATA_PIN, HIGH);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, LOW);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, HIGH);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, LOW);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, HIGH);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, LOW);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, HIGH);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, LOW);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, HIGH);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, LOW);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, HIGH);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, LOW);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, HIGH);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, LOW);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, HIGH);
  delayMicroseconds(high_us);
  digitalWrite(DATA_PIN, LOW);
  delayMicroseconds(high_us);  
}
void low()
{
  digitalWrite(DATA_PIN, HIGH);
  delayMicroseconds(low_us);
  digitalWrite(DATA_PIN, LOW);
  delayMicroseconds(low_us);
  digitalWrite(DATA_PIN, HIGH);
  delayMicroseconds(low_us);
  digitalWrite(DATA_PIN, LOW);
  delayMicroseconds(low_us);
  digitalWrite(DATA_PIN, HIGH);
  delayMicroseconds(low_us);
  digitalWrite(DATA_PIN, LOW);
  delayMicroseconds(low_us);
  digitalWrite(DATA_PIN, HIGH);
  delayMicroseconds(low_us);
  digitalWrite(DATA_PIN, LOW);
  delayMicroseconds(low_us);  
}
```

#ライセンス
BSD-3にしときます、とりあえず。

```txr:LISENCE
Copyright (c) 2017, GPS_NMEA_JP, All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are
met:

1. Redistributions of source code must retain the above copyright notice,
this list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright
notice, this list of conditions and the following disclaimer in the
documentation and/or other materials provided with the distribution.

3. Neither the name of the project nor the names of its contributors
may be used to endorse or promote products derived from this software
without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
"AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR
CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```
