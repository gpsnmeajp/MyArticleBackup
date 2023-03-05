#概要
今までWindowsでFelica Liteを読み書きしたりしてましたが、やっぱモバイル端末は外せないなということで。
Felica LiteアクセスのためにWindowsで [改造したライブラリ](https://sites.google.com/site/gpsnmeajp/tools/felicalib_segfix) と同等の機能を実装しました。
といっても、他の人とそんなにやってること変わりません。

<img width="300" alt="68c8bafc-5099-52c2-ee5b-8aaf43cbd1ba.png" src="https://qiita-image-store.s3.amazonaws.com/0/191114/68c8bafc-5099-52c2-ee5b-8aaf43cbd1ba.png">

#目的
AndroidでFelicaにアクセスしてるいろいろは見かけるんですが、ちょっと煩雑だったので。
それと個人的に単なるFelica Liteカードではなく、Felica PlugやFelica Linkとかのデバイス
[例えばこれ](https://flashair-developers.com/ja/nfc/) とアクセスしたいので、既存のサンプルの1Byte長アドレスアクセスでは足りません。
ので、2Byte長アドレスにも対応しています。

#大まかな使い方
1. インテントやReaderCallBack等でTag情報を得たら、setTagでタグ情報をセット
2. cardCaptureで通信開始。
3. pollingでカードの情報を確認(IDm、PMm、System Codeを確認できます)
4. readWithoutEncryptionやwriteWithoutEncryptionを好きなだけ行う
5. cardReleaseで通信終了

IDmも通信ハンドルも全部掴んで副作用だらけなので、とりあえず適当にアクセスするには便利ですが、
あるメソッドの処理中に別のメソッドを呼び出したりした場合の動作は保証しません。

#使い方の例
0x1780への1ブロック読み込み

```java:read
//tagにNFCインテントあるいはReaderModeのコールバックで得られたタグ情報を入れる
   nfcReader.setTag(tag);
   nfcReader.cardCapture();
   nfcReader.polling(nfcReader.FELICA_LITE);
   byte res[] = nfcReader.readWithoutEncryption(0x1780, nfcReader.FELICA_SERVICE_RO);
   nfcReader.cardRelease();
//resに16Byteのデータが帰る
```

0x1780への1ブロック書き込み

```java:write
//tagにNFCインテントあるいはReaderModeのコールバックで得られたタグ情報を入れる
//datに16Byteのデータを用意しておく
   nfcReader.setTag(tag);
   nfcReader.cardCapture();
   nfcReader.polling(nfcReader.FELICA_LITE);
   nfcReader.writeWithoutEncryption(0x1780, nfcReader.FELICA_SERVICE_RW, dat);
   nfcReader.cardRelease();
```

#メソッド
####void setTag(Tag _tag)
タグ情報をセットします。インテントやコールバックの際のTagを投げ入れてください。
**これがなければすべての操作に失敗します。**
それらに関係ないタイミングで通信したかったためにこうなりました。
####void cardCapture()
NFC通信を開始します。
####void cardRelease()
NFC通信を終了します
####int polling(int systemCode_in)
ポーリングを行います。同時にIDm、PMm、Systtem Codeを取得します。
Felica Liteの場合はSystemCodeにはFELICA_LITE(0x88B4)を指定するのをおすすめします。
Felica Liteはコスト削減のため上記以外のシステムはありません。
別名として、NFC-F(0x12FC)が利用できる場合があります。
Felica Plugは0xFEE1です。
####byte[] readWithoutEncryption(int addr, int serviceCode)
認証なし読み出しを行います。
簡単にするため1ブロックのみのアクセスです。
FELICA_SERVICE_RO(0x000B)を用いた場合はアクセス可能な全領域、
FELICA_SERVICE_RW(0x0009)を用いた場合は書込み可能な領域を読み出すことができます。
Felica Liteには上記以外のサービスはありません。これ以外はカード側エラーで失敗します。
####void writeWithoutEncryption(int addr, int serviceCode, byte data[])
認証なし書き込みを行います。
簡単にするため1ブロックのみのアクセスです。
必ずFELICA_SERVICE_RW(0x0009)を指定してください。それ以外はカード側エラーで失敗します。
addrには0-65535まで指定できます。
dataには16Byte格納してください。それ以外は例外を吐きます。

####byte[] getIDm()
pollingで取得したIDmが格納されています。

####byte[] getPMm()
pollingで取得したPMmが格納されています。

#ダウンロード
[ダウンロードはこちらから](https://sabowl.sakura.ne.jp/gpsnmeajp/android/lazynfcreader/)

AndroidもJavaも正直初心者レベルなので、色々と間違った実装があるかもしれません。ご了承ください。

大まかな実装は、 [AndroidでFelica(NFC)のブロックデータの取得](http://qiita.com/nshiba/items/38f94d61c020a17314b6) をかなり参考にさせていただきました。
(pollingやread/write処理の大部分)

#サンプル
サンプルのアクティビティです。
Android標準のNFC検出時の効果音が気に入らなかったため、enableReaderModeを使ってデフォルトのサウンドを消音しています。
そのため、無駄にSoundPoolで音を鳴らす処理が入っています。

タグをかざしていない時にボタンを押すと、次かざしたときのコールバックで処理が実行されます。
また、かざしながらボタンを押した場合でも、直接処理が実行されます。
そのため、いちいちかざし直す手間が省けて個人的に便利です。

```java:MainActivity.java
package com.example.seg.nfc_test;

import android.media.AudioAttributes;
import android.media.AudioManager;
import android.media.SoundPool;
import android.nfc.NfcAdapter;
import android.nfc.Tag;
import android.nfc.TagLostException;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.view.View;
import android.os.Handler;
import android.widget.Toast;

import java.io.IOException;

public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    private LazyNfcReader nfcReader = new LazyNfcReader();

    private AudioAttributes audioAttributes;
    private SoundPool mSoundPool;
    private int Sound_Success,Sound_Fail,Sound_Found;

    private String debugMsgTitle = "NFC_TEST_APP";

    private int tagFoundAction = 0;
    private static final int TAG_ACTION_NOTING = 0;
    private static final int TAG_ACTION_READ = 1;
    private static final int TAG_ACTION_WRITE = 2;

    Handler mHandler = new Handler(); //UIスレッドハンドラ
    TextView mHello;
    Button mBtRead,mBtWrite,mBtCancel;
    EditText mEdWrite,mEdRead;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d(debugMsgTitle,"Create");
        setContentView(R.layout.activity_main);

        mHello = (TextView) findViewById(R.id.text1);
        mEdRead = (EditText) findViewById(R.id.et_Read);
        mEdWrite = (EditText) findViewById(R.id.et_Write);

        mBtRead = (Button)   findViewById(R.id.bt_read);
        mBtRead.setOnClickListener(this);
        mBtWrite = (Button)   findViewById(R.id.bt_write);
        mBtWrite.setOnClickListener(this);
        mBtCancel = (Button)   findViewById(R.id.bt_cancel);
        mBtCancel.setOnClickListener(this);

        //オーディオシステム準備
        audioAttributes = new AudioAttributes.Builder().setUsage(AudioAttributes.USAGE_GAME).setContentType(AudioAttributes.CONTENT_TYPE_SPEECH).build();
        setVolumeControlStream(AudioManager.STREAM_MUSIC);
    }

    @Override
    protected void onResume() {
        super.onResume();
        Log.d(debugMsgTitle,"Resume");

        //オーディオ環境準備
        mSoundPool = new SoundPool.Builder().setAudioAttributes(audioAttributes).setMaxStreams(3).build();
        //オーディオ読み込み
        Sound_Success = mSoundPool.load(getApplicationContext(), R.raw.success, 0);
        Sound_Fail = mSoundPool.load(getApplicationContext(), R.raw.fail, 0);
        Sound_Found = mSoundPool.load(getApplicationContext(), R.raw.found, 0);

        //NFC取得設定
        NfcAdapter adapter = NfcAdapter.getDefaultAdapter(this);
        if(adapter == null) {
            mHello.setText("NFC unsupported");
        }else if (!adapter.isEnabled()) {
            mHello.setText("NFC disabled");
        }else {
            //NFCリーダーモードに設定。NFC-Fにのみ反応、消音、NFEF解析をしない。コールバック設定
            adapter.enableReaderMode(this,new CustomReaderCallback(),NfcAdapter.FLAG_READER_NFC_F | NfcAdapter.FLAG_READER_NO_PLATFORM_SOUNDS| NfcAdapter.FLAG_READER_SKIP_NDEF_CHECK,null);
        }
    }

    @Override
    protected void onPause() {
        super.onPause();

        //オーディオ開放
        mSoundPool.release();

        // NFCの読み込みを無効化
        //mAdapter.disableForegroundDispatch(this);
        Log.d(debugMsgTitle,"Pause");

    }

    public void onClick(View v) {
        Log.d(debugMsgTitle,"?");

        Button btn = (Button)v;

        switch( btn.getId() ){

            //ボタンが押されたとき
            case R.id.bt_read:
                tagFoundAction = TAG_ACTION_READ;
                mHello.setText("Read wait...");
                tagCheck();
                break;
            case R.id.bt_write:
                tagFoundAction = TAG_ACTION_WRITE;
                mHello.setText("Write wait...");
                tagCheck();
                break;
            case R.id.bt_cancel:
                tagFoundAction = TAG_ACTION_NOTING;
                mHello.setText("Cancel");
                break;

            default:
                Log.d("MyApp","Button not found");
                break;

        }
    }

    public void tagCheck()
    {
        //タグと通信できるかとりあえず試してみる。
        //かざしっぱなしでもこれで通信できる
        try {
            nfcReader.cardCapture();
            onTagFound();
        }catch (IOException e){
            //Not Found
        }
        return;
    }

    public void onTagFound() {
        Log.d(debugMsgTitle,"Tag Found Callback");
//        mHello.setText("Tag Found");
//        mSoundPool.play(Sound_Found,1.0F, 1.0F, 0, 0, 1.0F);
        switch(tagFoundAction) {
            case TAG_ACTION_READ: {
                try {
                    nfcReader.cardCapture();
                    nfcReader.polling(nfcReader.FELICA_LITE);
                    byte res[] = nfcReader.readWithoutEncryption(0x1780, nfcReader.FELICA_SERVICE_RO);
                    nfcReader.cardRelease();

                    String str = new String(res);
                    mEdRead.setText(str);

                    mSoundPool.play(Sound_Success, 1.0F, 1.0F, 0, 0, 1.0F);
                    Toast.makeText(this, "読み取り成功", Toast.LENGTH_SHORT).show();

                    tagFoundAction = TAG_ACTION_NOTING; //完了で取り消し
                    mHello.setText("OK");
                } catch (IOException e) {
                    Toast.makeText(this, "読み取り失敗", Toast.LENGTH_LONG).show();
                    Log.d(debugMsgTitle, "Connection Error", e);
                    mSoundPool.play(Sound_Fail, 1.0F, 1.0F, 0, 0, 1.0F);
                }
                break;
            }
            case TAG_ACTION_WRITE: {
                try {
                    String str = mEdWrite.getText().toString();
                    byte[] sbyte = str.getBytes();
                    byte[] dat = new byte[16];
                    for (int i = 0; i < 16; i++) {
                        if (i < sbyte.length) {
                            dat[i] = sbyte[i];
                        } else {
                            dat[i] = 0;
                        }
                    }

                    nfcReader.cardCapture();
                    nfcReader.polling(nfcReader.FELICA_LITE);
                    nfcReader.writeWithoutEncryption(0x1780, nfcReader.FELICA_SERVICE_RW, dat);
                    nfcReader.cardRelease();

                    mSoundPool.play(Sound_Success, 1.0F, 1.0F, 0, 0, 1.0F);
                    Toast.makeText(this, "書き込み成功", Toast.LENGTH_SHORT).show();

                    tagFoundAction = TAG_ACTION_NOTING; //完了で取り消し
                    mHello.setText("OK");
                } catch (IOException e) {
                    Toast.makeText(this, "書き込み失敗", Toast.LENGTH_SHORT).show();
                    Log.d(debugMsgTitle, "Connection Error", e);
                    mSoundPool.play(Sound_Fail, 1.0F, 1.0F, 0, 0, 1.0F);
                }
                break;
            }
            default:{
                mSoundPool.play(Sound_Found, 1.0F, 1.0F, 0, 0, 1.0F);
                break;
            }
        }
    }

    private class CustomReaderCallback implements NfcAdapter.ReaderCallback {
        @Override
        public void onTagDiscovered(Tag tag) {
            // タグの情報が渡される
            nfcReader.setTag(tag);

            //UIスレッドで処理実行
            mHandler.post(new Runnable() {
                @Override
                public void run() {
                    onTagFound();
                }
            });
        }
    }
}

```

#追記
外部に移動し、zlibライセンスに変更しました。
また、NfcReaderだとかぶってたようなので、LazyNfcReaderに名称を変更しました。
