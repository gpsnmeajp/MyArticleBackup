#概要
普段からアプリ開発している人はともかくとして、電子工作などでBluetooth Low Energyを
「ちょっとかじってみたい」と言った人(主に私)には、AndroidのBLEのAPIはちょっと使いにくく感じます。

色々できるのが難しい原因かなと思い、簡単に極力シンプルに使えるライブラリを目指して開発しました。

※操作を簡単にするライブラリであり、BLEに関する基礎知識は必要です。
　[Bluetooth Low Energyをはじめよう (Make:PROJECTS) Kevin Townsend](https://www.amazon.co.jp/dp/4873117135/ref=cm_sw_r_tw_dp_x_YWG7zbPN0QYK0)を読むことをおすすめします。

※慣れている方は[RxAndroidBLE](https://qiita.com/tadaogi/items/59cfb042d81fd0bcedeb)や[Bletia: AndroidのBLE周りをモダンなAPIで扱う](https://qiita.com/izumin5210/items/374d532a252c15aeb484)をおすすめします。
そちらのほうが多分よっぽどきれいでバグもないです。

...あれ、なんでこんなライブラリ作ったんだろう？

#どんなライブラリ？
Android APIのBLE関係の非同期処理を、全部**同期処理にしました。**
本来はpromiseなどを使うべきだったのでしょうが、簡単さを優先しました。

ので、例えば、BBC micro:bitと通信してみたい場合には以下のようなコードでできます。
(実際には、UIとは別のThreadで実行する必要があります。)

```java
    private LazyBLEWrapper ble = new LazyBLEWrapper();

    private final String LEDServiceUUID            = "E95DD91D-251D-470A-A062-FA1922DFA9A8";
    private final String LEDTextCharacteristicUUID = "E95D93EE-251D-470A-A062-FA1922DFA9A8";
    private final String ButtonServiceUUID         = "E95D9882-251D-470A-A062-FA1922DFA9A8";
    private final String ButtonACharacteristicUUID = "E95DDA90-251D-470A-A062-FA1922DFA9A8";
    private final String UARTServiceUUID           = "6E400001-B5A3-F393-E0A9-E50E24DCCA9E";
    private final String UARTTxCharacteristicUUID  = "6E400002-B5A3-F393-E0A9-E50E24DCCA9E";
    private final String UARTRxCharacteristicUUID  = "6E400003-B5A3-F393-E0A9-E50E24DCCA9E";
    private final int timeout = 30*1000;

    //スキャン。前方一致です
    BluetoothDevice device = ble.scanDevice(this,"BBC micro:bit", timeout);
    //接続
    ble.connect(this,device, timeout);

    //切断時コールバック(任意)
    ble.setDisconnectCallback(new LazyBLEWrapper.DisconnectCallback() {
        @Override
        public void onDisconnect()  {
            Log.e(TAG,"Disconnected!");
        }
    });

    //LEDに"Hello World"と書き込み
    ble.writeData("Hello World",LEDServiceUUID,LEDTextCharacteristicUUID, BluetoothGattCharacteristic.WRITE_TYPE_DEFAULT,timeout);
    //ボタンの押下状態を取得して表示
    Integer dat = ble.readDataInt(ButtonServiceUUID,ButtonACharacteristicUUID,BluetoothGattCharacteristic.FORMAT_UINT8,0,timeout);
    Log.d(TAG, dat.toString());

    //Notificationを設定
    ble.setNotify(ButtonServiceUUID,ButtonACharacteristicUUID,true,timeout);
    //Indicationを設定
    ble.setIndicate(UARTServiceUUID,UARTTxCharacteristicUUID,true,timeout);
    //Callbackを設定。この中はUIThreadで実行されます。
    ble.setNotificationCallback(new LazyBLEWrapper.NotificationCallback() {
        @Override
        public void onNotification(BluetoothGattCharacteristic characteristic) {
            if(ble.isMatchCharacteristicUUID(characteristic,ButtonACharacteristicUUID)) {
                Integer ButtonA = characteristic.getIntValue(BluetoothGattCharacteristic.FORMAT_UINT8, 0);
                Log.d(TAG, ButtonA.toString());
            }
            if(ble.isMatchCharacteristicUUID(characteristic,UARTTxCharacteristicUUID)) {
                Log.d(TAG, characteristic.getStringValue(0));
            }
        }
    });

    //UART送信
    ble.writeData("Hello UART\n",UARTServiceUUID,UARTRxCharacteristicUUID,BluetoothGattCharacteristic.WRITE_TYPE_DEFAULT,timeout);
    //切断
    ble.disconnect(timeout);
    //ble.forceDisconnect();
```

深入りした機能を使うならそもそも標準API使えばいい、ということで機能を絞って
**よく使う機能**だけを中心にシンプルに実装しました。

のでユースケースは

+ アクセスしたいサービスやキャラクタリスティックのUUIDがわかっている
+ デバイスの名前がわかっているか、あるいはボンディング済み
+ スマートフォンがCentralかつGATTクライアントで、周辺機器はPeripheralかつGATTサーバー

のみです。

同期メソッドなので、シンプルな流れで書くことができます。(スレッドが必須になりますが)
非同期ではないため、待ち時間の発生するメソッドにはすべてタイムアウト指定がついています。

Android 6.0以上でBLEを使うときに必要になる詳細位置情報権限の取得処理なども、おまけで入っています。

Notification時のキャラクタリスティックの操作(setValue, getValue等)などは含まれていないので、
そこはAndroid API標準のメソッドを利用します。

基本的に、AndroidのBLE APIのオブジェクトをそのまま返すように設計しているため、
細かいことをしたい場合にはその辺のAPIが使えます。


#ダウンロード
[ダウンロードはこちら](https://sabowl.sakura.ne.jp/gpsnmeajp/android/lazyblewrapper/)
シンプルにするため、Android 5.0以降の未対応です。
ライセンスはzlibライセンスです。

Nexus 5X (Android 8.0)でのみ動作確認しています。

#メソッド一覧
##端末設定・許可
シンプルに実装できるようにするため、端末の設定やランタイムパーミッション関係も用意してあります。
これらのメソッドはUIスレッドで実行しても問題ありません。(待ち時間無しで完了します。)

v0.13: Contextを内部で保持するのを止めたため、コンテキストやアクティビティを渡す必要があります。

```java
//端末のBLE機能が有効かチェック。通信前などに。
boolean isBluetoothFeatureEnabled() 

//端末のBluetooth機能を有効にするリクエストをする。
void requestTurnOnBlueTooth(AppCompatActivity activity)

//端末がBLE機能をサポートしているかをチェック。
boolean isBluetoothFeatureSupported(Context context) 

//詳細位置情報パーミッションがあるかをチェックする。
//BLEスキャン時に必要となる。
boolean isPermitted(Context context) 

//詳細位置情報パーミッションをリクエストする。
//本来はActivityの方で行うものであり非推奨だが、とりあえず試作で作るときなど面倒なときに使えるようにしてある。
//本来はインテントを受け取って処理する必要があるが、なくても動くので実装していない。Activity側で実装してください。
void permissionRequest(AppCompatActivity activity)
```

##スキャン
とりあえずこれだけあれば困らないかな、という以下の3種類に絞りました。

+ 無差別スキャン
+ 名前検索
+ ボンディング済み一覧から探す

v0.13: Contextを内部で保持するのを止めたため、コンテキストやアクティビティを渡す必要があります。
　　　　また、内部的にUIスレッドでBLE関係の処理を呼び出すように変更しました。

```java
//周辺のペリフェラルをスキャンする。
//タイムアウトするまでスキャンを続け、見つかった順にArrayList<BluetoothDevice>に格納して返す。
//見つからなかった場合は0個のリストを返す。
ArrayList<BluetoothDevice> scanDevice(Context context, int timeOut) throws IOException 

//周辺のペリフェラルをスキャンする。
//デバイス名を前方一致でチェックし、一番最初に一致したデバイスを即座に返す。
//見つからなかった場合は例外を発生させる。
BluetoothDevice scanDevice(Context context, String deviceName, int timeOut) throws IOException 

//端末のボンディング(ペアリング)済みペリフェラル一覧から検索する。
//デバイス名を前方一致でチェックし、一番最初に一致したデバイスを即座に返す。
//見つからなかった場合はnullを返す。
BluetoothDevice scanBondedDevice(String deviceName)
```

##接続・切断
GATTオブジェクトの扱いが面倒そう(コールバックがたくさん)なので、そこを抱え込みました。
ので、シンプルで戻り値の無いものとなっています。

v0.13: Contextを内部で保持するのを止めたため、コンテキストやアクティビティを渡す必要があります。
　　　　また、内部的にUIスレッドでBLE関係の処理を呼び出すように変更しました。
　　　　切断処理を丁寧にするようになり、待ち時間が発生するようになりました。
　　　　アプリケーション終了時の切断などは待ち時間なしのforceDisconnectをおすすめします。
　　　　内部に保持できるGATT接続はインスタンスあたり1つのみです。
v0.14: 接続済みの時に接続しようとすると例外を吐くように。

```java
//Deviceに接続し、サービススキャンを行う。GATTオブジェクトは内部で保持する。
//(切断処理などを自動で行うため)
//接続に失敗した場合は例外を発生させる。
void connect(Context context, BluetoothDevice device, int timeOut) throws IOException 

//Deviceから切断する。
void disconnect(int timeOut)

//Deviceから強制切断する(旧バージョンでの切断処理)
//これはUIスレッドで実行して良い
void forceDisconnect()
```

##基本キャラクタリスティック読取・書込・通知
単なるキャラクタリスティックの読み書きなら、より単純にできるようにv0.11で改善しました。
サービスの探索・キャラクタリスティックの探索・値の代入などを1行で行います。

読み取りがやたら多いですが、AndroidのAPIに合わせました。取得可能な形式はすべて対応しています。
一方、書き込みの方はfloatを省略しました。というのも、ちょっと使いにくそうなスタイルだったのと、
浮動小数点を書き込む機会はそう多くなさそうという判断です。

```java
//**********読み取り***********

//指定されたサービスUUID文字列、キャラクタリスティックUUID文字列からサービス・キャラクタリスティックを取得し、
//その最新のデータをペリフェラルから取得、指定された書式・オフセットで解釈して、整数型で返す。
//通信に失敗した際は例外を発生させる。
int readDataInt(String ServiceUUID,String CharacteristicUUID,int formatType,int offset,int timeOut) throws IOException 

//指定されたサービスUUID文字列、キャラクタリスティックUUID文字列からサービス・キャラクタリスティックを取得し、
//その最新のデータをペリフェラルから取得、指定された書式・オフセットで解釈して、浮動小数点型で返す。
//通信に失敗した際は例外を発生させる。
float readDataFloat(String ServiceUUID,String CharacteristicUUID,int formatType,int offset,int timeOut) throws IOException 

//指定されたサービスUUID文字列、キャラクタリスティックUUID文字列からサービス・キャラクタリスティックを取得し、
//その最新のデータをペリフェラルから取得し、バイト列として返す。
//通信に失敗した際は例外を発生させる。
byte[] readDataValue(String ServiceUUID,String CharacteristicUUID,int timeOut) throws IOException 

//指定されたサービスUUID文字列、キャラクタリスティックUUID文字列からサービス・キャラクタリスティックを取得し、
//その最新のデータをペリフェラルから取得、指定されたオフセットで解釈して、文字列として返す。
//通信に失敗した際は例外を発生させる。
String readDataString(String ServiceUUID,String CharacteristicUUID,int offset,int timeOut) throws IOException 


//**********書き込み***********

//指定されたサービスUUID文字列、キャラクタリスティックUUID文字列からサービス・キャラクタリスティックを取得し、
//指定した書き込み方式で、指定した文字列を内容としてペリフェラルに設定。
//通信に失敗した際は例外を発生させる。
void writeData(String string,String ServiceUUID,String CharacteristicUUID,int writeType,int timeOut) throws IOException 

//指定されたサービスUUID文字列、キャラクタリスティックUUID文字列からサービス・キャラクタリスティックを取得し、
//指定した書き込み方式で、指定したバイト列を内容としてペリフェラルに設定。
//通信に失敗した際は例外を発生させる。
void writeData(byte[] value,String ServiceUUID,String CharacteristicUUID,int writeType,int timeOut) throws IOException 

//指定されたサービスUUID文字列、キャラクタリスティックUUID文字列からサービス・キャラクタリスティックを取得し、
//指定した書き込み方式で、指定した整数を内容としてペリフェラルに設定。
//通信に失敗した際は例外を発生させる。
void writeData(int num,int formatType,int offset,String ServiceUUID,String CharacteristicUUID,int writeType,int timeOut) throws IOException 
```

formatTypeは以下です。

```java
BluetoothGattCharacteristic.FORMAT_FLOAT  //(32-bit float)
BluetoothGattCharacteristic.FORMAT_SFLOAT //(16-bit float)
BluetoothGattCharacteristic.FORMAT_SINT16
BluetoothGattCharacteristic.FORMAT_SINT32
BluetoothGattCharacteristic.FORMAT_SINT8
BluetoothGattCharacteristic.FORMAT_UINT16
BluetoothGattCharacteristic.FORMAT_UINT32
BluetoothGattCharacteristic.FORMAT_UINT8
```

writeTypeは以下です。

```java
BluetoothGattCharacteristic.WRITE_TYPE_DEFAULT
BluetoothGattCharacteristic.WRITE_TYPE_NO_RESPONSE
BluetoothGattCharacteristic.WRITE_TYPE_SIGNED
```

##通知(Notification・Indication)
通知の設定も定形ですので、省略的な書き方ができるようにしました。
setNotifyはペリフェラル側の設定、setCallbackはAndroid側の設定ですので、両方必要です。

ただし、setCallbackで登録できるコールバックは1つのみで、内部でキャラクタリスティックを
確認して分岐する形ですので、setCallbackはアプリの最初の一回のみで済みます。

v0.12より、Indicationに対応しました。

```java
//指定されたサービスUUID文字列、キャラクタリスティックUUID文字列からサービス・キャラクタリスティックを取得し、
//そのキャラクタリスティックについての、ペリフェラル側のNotification設定を有効・無効にする。
//通信に失敗した際は例外を発生させる。
void setNotify(String ServiceUUID,String CharacteristicUUID,boolean enable,int timeOut) throws IOException 

//指定されたサービスUUID文字列、キャラクタリスティックUUID文字列からサービス・キャラクタリスティックを取得し、
//そのキャラクタリスティックについての、ペリフェラル側のIndication設定を有効・無効にする。
//通信に失敗した際は例外を発生させる。
void setIndicate(String ServiceUUID,String CharacteristicUUID,boolean enable,int timeOut) throws IOException
```

##コールバック
v0.14: コールバック解除はnullを設定してください。

```java
//ペリフェラルから通知を受信した際のコールバックを設定する。
void setNotificationCallback(final NotificationCallback callback)

//切断された場合のコールバック
void setDisconnectCallback(final DisconnectCallback callback){

//キャラクタリスティックのUUIDと文字列UUIDが位置するかをチェックする
//一致している場合はture、していない場合はfalse
//NotificationやIndicationで送られてきたキャラクタリスティックの判別に使用できる。
boolean isMatchCharacteristicUUID(BluetoothGattCharacteristic characteristic,String CharacteristicUUID)

//通知用コールバックインターフェース
interface NotificationCallback {
    void onNotification(BluetoothGattCharacteristic characteristic);
}

//切断時コールバックインターフェース
interface DisconnectCallback {
    void onDisconnect();
}
```

##応用キャラクタリスティック読取・書込・通知
v0.10で提供していた方のキャラクタリスティック操作です。
生のキャラクタリスティックが得られますので、詳細な操作をしたい場合はこちらをご利用ください。

単に読み書きするだけだと、サービスを掴んでから、キャラクタリスティックを掴んで、
操作して、発行の順で、最低4行必要になるので、ちょっと煩雑です。

v0.13: 内部的にUIスレッドで実行するようになりました。nullチェックを強化しました。

```java
//検索済みのサービス情報を使い、UUID文字列からサービスを取得する。
//見つからなかった場合はnullを返す。
BluetoothGattService getService(String serviceUUID) 

//検索済みのサービス情報を使い、UUID文字列からキャラクタリスティックを取得する。
//同じUUIDを持つキャラクタリスティックを判別するため、getServiceで取得したBluetoothGattServiceが必要。
//見つからなかった場合はnullを返す。
BluetoothGattCharacteristic getCharacteristicInService(BluetoothGattService service, String CharacteristicUUID) 

//現在のキャラクタリスティックの内容をペリフェラルに反映させる。
//通信に失敗した際は例外を発生させる。
void writeCharacteristic(BluetoothGattCharacteristic characteristic, int timeOut) throws IOException 

//最新のキャラクタリスティックの内容をペリフェラルから取得し、返す。
//(この際、元のキャラクタリスティックは多分変更されない)
//通信に失敗した際は例外を発生させる。
BluetoothGattCharacteristic readCharacteristic(BluetoothGattCharacteristic characteristic, int timeOut) throws IOException 
```

##ディスクリプタ操作
ディスクリプタの操作。Notificationの設定に内部的に使用している。
あまりテストしていないので注意。

v0.13: 内部的にUIスレッドで実行するようになりました。nullチェックを強化しました。

```java
//ディスクリプタをペリフェラルに書き込む。
//通信に失敗した際は例外を発生させる。
void writeDescriptor(BluetoothGattDescriptor descriptor, int timeOut) throws IOException 

//ディスクリプタをペリフェラルから読み込む。
//通信に失敗した際は例外を発生させる。
BluetoothGattDescriptor readDescriptor(BluetoothGattDescriptor descriptor, int timeOut) throws IOException 

//特定のキャラクタリスティックの、ペリフェラル側のNotification設定を有効・無効にする。
//通信に失敗した際は例外を発生させる。
void setNotification(BluetoothGattCharacteristic characteristic,boolean enable,int timeOut) throws IOException

//特定のキャラクタリスティックの、ペリフェラル側のIndication設定を有効・無効にする。
//通信に失敗した際は例外を発生させる。
void setIndication(BluetoothGattCharacteristic characteristic,boolean enable,int timeOut) throws IOException
```

##その他

```java
//内部に保持しているBluetoothGattを渡す。nullのときは切断されている。
//通信切断時などは勝手に無効になるので注意。
BluetoothGatt getGatt() 

//非常に冗長なログの有効無効を設定する。
//既定で有効である(プロトタイピングで使うことが多いと思うので)
//falseを渡すことで、致命的なログ以外は出さないようになる。
void setDebug(boolean f)

//実行中のメソッドがあるかチェック
//boolean getLockState()

//強制開放
//void forceUnlock()
```

##Android側API
ライブラリのメソッドではないが、よく使う奴です。

```java
//指定したデバイスとのボンディングをOSに依頼する。
//どういったボンディングが行われるかは、ペリフェラルの設計に依存する。
device.createBond();

//キャラクタリスティックに値をセットする。
//writeCharacteristicと併用する。
characteristic.setValue("Hello World");

//キャラクタリスティックから指定した書式で値を取り出す。
//readCharacteristicと併用する。
characteristic.getIntValue(BluetoothGattCharacteristic.FORMAT_UINT8, offset);
```

#最後に
Android初心者でjava初心者な人が、より初心者な人向けに作成したものですが、
何分初心者ですので、色々変なところがあるかと思います。

もっと便利なライブラリあるぞとか、ここおかしいぞ、というのがあれば、コメント等でお知らせください。

#参考文献
以下のサイトを大変参考にさせていただきました。
<li><a href="http://bril-tech.blogspot.jp/2015/05/bluetoothsmartmbed-5.html">BRILLIANTSERVICE TECHNICAL BLOG: BluetoothSMARTデバイスをmbed で開発する(5)</a></li>
<li><a href="http://mslgt.hatenablog.com/entry/2015/05/17/212257">Android5.0〜でBLEを使う(Central編) - vaguely</a></li>
<li><a href="http://yife.hateblo.jp/entry/2012/10/29/203330">getSystemServiceをActivityでないclassで使う - yifeの日記</a></li>
<li><a href="http://d.hatena.ne.jp/sankumee/20120329/1333021847">Handlerクラスの正しい使い方（Androidでスレッド間通信）- ちくたく</a></li>
<li><a href="http://yamataka.hatenablog.com/entry/2015/02/24/210233">BLEでペリフェラルを操作する - yamatakaの手帳</a></li>
<li><a href="https://qiita.com/maki02/items/27d5a6f50016097875ae">Android で GATT 通信アプリを作ってみる on @Qiita</a></li>
<li><a href="http://fantom1x.blog130.fc2.com/blog-entry-91.html">【Java】interface を使って簡単なコールバック機能を作る </a></li>
<li><a href="http://www.ne.jp/asahi/hishidama/home/tech/java/throw.html">Java 例外の投げ方メモ - ひしだま</a></li>					

開発で味わったいろいろなつらさは、2年前にすでに書かれていた。
[kyobashi.dexでAndroidのBLEがつらい話してきた #kyobashidex](http://izumin.hateblo.jp/entry/2015/09/23/165945)

あとここ。
[5 TIPS FOR BLUETOOTH LOW ENERGY (BLE) ON ANDROID](http://www.kodira.de/2016/08/tips-ble-android/)

#サンプルアクティビティ
Nexus 5X(Android 8.0)と、BBC micro:bitで動作を確認しています。
micro:bit側の準備についてはこちらのサイトを参考にしてください。
[Web Bluetooth API を使ってブラウザだけでMicro:bitとBLE通信してみる。](https://shimz.me/blog/microbit/5456)

```XML:AndroidManifest.xml
    <uses-permission android:name="android.permission.BLUETOOTH" />
　  <uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-feature android:name="android.hardware.bluetooth_le" android:required="true"/>
    <uses-feature android:name="android.hardware.location.gps" />
```

```java:MainActivity.java
package jp.ne.sakura.sabowl.gpsnmeajp.bletest;
import android.bluetooth.BluetoothDevice;
import android.bluetooth.BluetoothGattCharacteristic;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;

import java.util.ArrayList;
import java.util.UUID;

public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    ...

    final String LEDServiceUUID = "E95DD91D-251D-470A-A062-FA1922DFA9A8";
    final String LEDTextCharacteristicUUID = "E95D93EE-251D-470A-A062-FA1922DFA9A8";
    final String ButtonServiceUUID = "E95D9882-251D-470A-A062-FA1922DFA9A8";
    final String ButtonACharacteristicUUID = "E95DDA90-251D-470A-A062-FA1922DFA9A8";

    private LazyBLEWrapper ble = new LazyBLEWrapper(this);

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ...

        ble.setDebug(true);
        if(!ble.isBluetoothFeatureSupported(this))
        {
            Toast.makeText(this, "Bluetoothが利用できない端末です", Toast.LENGTH_LONG).show();
            finish();
        }
        if(!ble.isPermitted(this)){
           ble.permissionRequest(this);
        }

    }

    void connect()
    {
        try {
            // デバイスの検出.

            //ArrayList<BluetoothDevice> result = ble.scanDevice(this,5000); //全デバイスをスキャンする
            //BluetoothDevice device = ble.scanDevice(this,"BBC micro:bit", 5000); //前方一致で名前スキャン
            BluetoothDevice device = ble.scanBondedDevice("BBC micro:bit"); //ボンディング済み一覧から探す。(詳細位置権限不要)
            //device.createBond();//ボンディング

            //接続
            ble.connect(this,device, 30000);
            ble.setDisconnectCallback(new LazyBLEWrapper.DisconnectCallback() {
                @Override
                public void onDisconnect()  {
                    Log.e(TAG,"Disconnected!");
                }
            });

            //LED書き込み
            ble.writeData("Hello",LEDServiceUUID,LEDTextCharacteristicUUID, BluetoothGattCharacteristic.WRITE_TYPE_DEFAULT,30000);
            //ボタン取得
            Integer dat = ble.readDataInt(ButtonServiceUUID,ButtonACharacteristicUUID,BluetoothGattCharacteristic.FORMAT_UINT8,0,30000);
            Log.d(TAG, dat.toString());

            //ボタン通知設定
            ble.setNotify(ButtonServiceUUID,ButtonACharacteristicUUID,true,30000);
            //UART受信設定
            ble.setIndicate(UARTServiceUUID,UARTTxCharacteristicUUID,true,30000);
            ble.setNotificationCallback(new LazyBLEWrapper.NotificationCallback() {
                @Override
                public void onNotification(BluetoothGattCharacteristic characteristic) {
                    if(ble.isMatchCharacteristicUUID(characteristic,ButtonACharacteristicUUID)) {
                        Integer ButtonA = characteristic.getIntValue(BluetoothGattCharacteristic.FORMAT_UINT8, 0);
                        Log.d(TAG, ButtonA.toString());
                        mText.setText(ButtonA.toString());
                    }
                    if(ble.isMatchCharacteristicUUID(characteristic,UARTTxCharacteristicUUID)) {
                        Log.d(TAG, characteristic.getStringValue(0));
                    }
                }
            });
            //UART送信
            ble.writeData("Hello UART\n",UARTServiceUUID,UARTRxCharacteristicUUID,BluetoothGattCharacteristic.WRITE_TYPE_DEFAULT,30000);

        }catch (Exception e){
            Log.e(TAG,"connect",e);
            ble.forceDisconnect();
        }
    }

    void disconnect(){
        try {
//            ble.setNotify(ButtonServiceUUID,ButtonACharacteristicUUID,false,30000);
//            ble.setDisconnectCallback(null);
//            ble.setNotificationCallback(null);
            ble.disconnect(30*1000);
//            finish();
        }catch (Exception e){
            Log.e(TAG,"disconnect",e);
            ble.forceDisconnect();
        }
    }

    public void onClick(View v) {
        Button btn = (Button)v;
        Log.i(TAG,"onClick");

        switch( btn.getId() ){
            //ボタンが押されたとき
            case R.id.buttonConnect:
                //BLE処理
                if(ble.isBluetoothFeatureEnabled())
                {
                    //ロックするのでメインスレッドで走らせないこと(一生帰ってこなくなる))
                    new Thread(new Runnable() {
                        @Override
                        public void run() {
                            connect();
                        }
                    }).start();
                }else{
                    ble.requestTurnOnBlueTooth(this);
                }
                mText.setText("Wow!");

                break;
            case R.id.buttonDisconnect:
                //ロックするのでメインスレッドで走らせないこと(一生帰ってこなくなる))
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        disconnect();
                    }
                }).start();
                break;

            default:
                break;
        }
    }
```
