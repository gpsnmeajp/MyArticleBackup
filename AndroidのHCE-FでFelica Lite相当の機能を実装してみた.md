#はじめに
AndroidのHCE-FでFelica Lite相当の機能を実装してみました．
ただし簡単のため，Felica Liteと異なり，読み出し時も1ブロックのみしか受け付けません．
また，pollingが改めて飛んでくる場面を手元で再現できなかったため，pollingも実装していません．

IDmはランダムを想定しており，そのためIDmに関係なくそのIDmのカードとして反応します．
(制約があります．詳しくはSonyの**Host-based Card Emulation for NFC-F アプリケーション開発ガイドライン**を参照してください)

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/852a8d48-395a-1625-a40f-87e7e8193dca.png)

#Serviceのソース
HCEFService.java以外の部分は以下を参照してください．
[AndroidのHCE-Fについて調べてみたメモとサンプルソース](https://qiita.com/gpsnmeajp/items/e525bb5c13511c18dda5)

**[より使いやすくしたものはこちら](https://sabowl.sakura.ne.jp/gpsnmeajp/android/LazyNfcFHCELite/)**

```java:HCEFService.java
/*
zlib License
Copyright (c) 2018 GPS_NMEA_JP

This software is provided 'as-is', without any express or implied warranty. In no event will the authors be held liable for any damages arising from the use of this software.
Permission is granted to anyone to use this software for any purpose, including commercial applications, and to alter it and redistribute it freely, subject to the following restrictions:
1. The origin of this software must not be misrepresented; you must not claim that you wrote the original software. If you use this software in a product, an acknowledgment in the product documentation would be appreciated but is not required.
2. Altered source versions must be plainly marked as such, and must not be misrepresented as being the original software.
3. This notice may not be removed or altered from any source distribution.
*/

package com.example.gpsnmeajp.hce_test;

import android.nfc.cardemulation.HostNfcFService;
import android.os.Bundle;
import android.util.Log;
import android.widget.Toast;

public class HCEFService extends HostNfcFService {

    final int PACKET_LENGTH_POS = 0;
    final int PACKET_LENGTH_SIZE = 1;

    final int COMMAND_TYPE_POS = 1;
    final int COMMAND_TYPE_SIZE = 1;

    final int IDM_POS = 2;
    final int IDM_SIZE = 8;

    final int RESPONSE_SIZE = 1;
    final int STATUS_FLAG1_SIZE = 1;
    final int STATUS_FLAG2_SIZE = 1;

    final int BLOCK_NUM_SIZE = 1;
    final int BLOCK_SIZE = 16;

    final int ONE_BLOCK = 1;

    //---
    final byte CMD_POLLING = (byte)0x04;
    final byte RES_POLLING = (byte)0x05;
    final byte CMD_READ_WITHOUT_ENCRYPTION = (byte)0x06;
    final byte RES_READ_WITHOUT_ENCRYPTION = (byte)0x07;
    final byte CMD_WRITE_WITHOUT_ENCRYPTION = (byte)0x08;
    final byte RES_WRITE_WITHOUT_ENCRYPTION = (byte)0x09;

    //---
    final byte STATUS_FLAG1_SUCCESS = (byte)0x00;
    final byte STATUS_FLAG1_FAILED = (byte)0xFF;
    final byte STATUS_FLAG1_FELICA_LITE_ERROR = (byte)0x01;

    final byte STATUS_FLAG2_SUCCESS = (byte)0x00;
    final byte STATUS_FLAG2_READ_ERROR = (byte)0x70;
    final byte STATUS_FLAG2_READONLY = (byte)0xA8;
    final byte STATUS_FLAG2_NEED_AUTH = (byte)0xB1;
    final byte STATUS_FLAG2_SERVICE_NUM_ERROR = (byte)0xA1;
    final byte STATUS_FLAG2_BLOCK_NUM_ERROR = (byte)0xA2;
    final byte STATUS_FLAG2_SERVICE_CODE = (byte)0xA6;
    final byte STATUS_FLAG2_ACCESS_MODE = (byte)0xA7;

    final byte SERVICE_CODE_READ_ONLY = 0x000B;
    final byte SERVICE_CODE_READ_WRITE = 0x0009;


    @Override
    public void onCreate() {
        Log.d("HCEFService(NFCF)","onCreate");
        super.onCreate();
        Toast.makeText(this, "onCreate", Toast.LENGTH_LONG).show();
    }

    @Override
    public void onDestroy() {
        Log.d("HCEFService(NFCF)","onDestroy");
        super.onDestroy();
    }
    //onBindは作れません

    @Override
    public byte[] processNfcFPacket(byte[] commandPacket, Bundle extras)
    {
        Log.d("HCEFService(NFCF)","processNfcFPacket: Received NFCF");
        Toast.makeText(this, "processNfcFPacket", Toast.LENGTH_LONG).show();

        //必須情報より小さいならエラー
        if (commandPacket.length < (PACKET_LENGTH_SIZE+COMMAND_TYPE_SIZE+IDM_SIZE)) {
            Log.e("HCEFService(NFCF)","processNfcFPacket: Packet size too short");
            return null;
        }

        byte commandType = commandPacket[COMMAND_TYPE_POS];
        switch(commandType)
        {
            case CMD_POLLING: return null;//stub
            case CMD_WRITE_WITHOUT_ENCRYPTION: return WriteWithoutEncryption(commandPacket);//stub
            case CMD_READ_WITHOUT_ENCRYPTION: return ReadWithoutEncryption(commandPacket);//stub
            default: return null; //該当コマンドなし
        }
    }

    @Override
    public void onDeactivated(int reason) { //RF切断時
        if(reason == DEACTIVATION_LINK_LOSS)
        {
            Log.d("HCEFService(NFCF)","onDeactivated: DEACTIVATION_LINK_LOSS");
        }else{
            Log.d("HCEFService(NFCF)","onDeactivated: Unknown reason");
        }
    }

    private byte[] WriteWithoutEncryption(byte[] commandPacket)
    {
        Log.d("HCEFService(NFCF)","WriteWithoutEncryption");
        //response
        int len = PACKET_LENGTH_SIZE + RESPONSE_SIZE + IDM_SIZE + STATUS_FLAG1_SIZE + STATUS_FLAG2_SIZE;
        byte[] responsePacket = new byte[len];
        responsePacket[PACKET_LENGTH_POS] = (byte)len;
        responsePacket[COMMAND_TYPE_POS] = RES_WRITE_WITHOUT_ENCRYPTION;
        responsePacket[len-2] = STATUS_FLAG1_SUCCESS;
        responsePacket[len-1] = STATUS_FLAG2_SUCCESS;

        //IDmをセット
        responsePacket = setIDmToPacket(responsePacket,getIDm(commandPacket));

        //analyze
        int serviceNum = commandPacket[10]&0xFF;
        if(serviceNum != 1)
        {
            responsePacket[len-2] = STATUS_FLAG1_FAILED;
            responsePacket[len-1] = STATUS_FLAG2_SERVICE_NUM_ERROR;
            Log.d("HCEFService(NFCF)","SERVICE_NUM_ERROR");
            return responsePacket; //ERROR RES
        }

        int serviceCode = (commandPacket[11]&0xFF) | ((commandPacket[12]&0xFF)<<8);
        Log.d("HCEFService(NFCF)",String.format("Service code : %04X,",serviceCode));

        if(serviceCode != SERVICE_CODE_READ_WRITE )
        {
            responsePacket[len-2] = STATUS_FLAG1_FELICA_LITE_ERROR;
            responsePacket[len-1] = STATUS_FLAG2_SERVICE_CODE;
            Log.d("HCEFService(NFCF)","STATUS_FLAG2_SERVICE_CODE");
            return responsePacket; //ERROR RES
        }

        int blockNum = commandPacket[13]&0xFF;
        if(blockNum != 1)
        {
            responsePacket[len-2] = STATUS_FLAG1_FAILED;
            responsePacket[len-1] = STATUS_FLAG2_BLOCK_NUM_ERROR;
            Log.d("HCEFService(NFCF)","BLOCK_NUM_ERROR");
            return responsePacket; //ERROR RES
        }

        int blockAddress = -1;
        int blockHeadPos = -1;
        int blocklist_1 = commandPacket[14]&0xFF;
        if(blocklist_1 == 0x80)
        {
            //1byte
            blockAddress = commandPacket[15]&0xFF;
            blockHeadPos = 16;
        }else if(blocklist_1 == 0x00)
        {
            //2byte
            blockAddress = (commandPacket[15]&0xFF) | ((commandPacket[16]&0xFF)<<8);
            blockHeadPos = 17;
        }else{
            responsePacket[len-2] = STATUS_FLAG1_FAILED;
            responsePacket[len-1] = STATUS_FLAG2_ACCESS_MODE;
            Log.d("HCEFService(NFCF)","SERVICE_NUM_ERROR");
            return responsePacket; //ERROR RES
        }

        Log.d("HCEFService(NFCF)",String.format("Block Address : %04X,",blockAddress));

        byte data[] = new byte[BLOCK_SIZE];
        System.arraycopy(commandPacket, blockHeadPos,data , 0, BLOCK_SIZE);

        String debug="";
        for(int i=0;i<data.length;i++)
        {
            debug += String.format("%02X,",data[i]);
        }
        Log.d("HCEFService(NFCF)",debug);
        Log.d("HCEFService(NFCF)","SUCCESS");
        return responsePacket; //SUCCESS
    }
    private byte[] ReadWithoutEncryption(byte[] commandPacket)
    {
        Log.d("HCEFService(NFCF)","ReadWithoutEncryption");
        //response
        int len = PACKET_LENGTH_SIZE + RESPONSE_SIZE + IDM_SIZE + STATUS_FLAG1_SIZE + STATUS_FLAG2_SIZE + BLOCK_NUM_SIZE + BLOCK_SIZE;
        int headLen = PACKET_LENGTH_SIZE + RESPONSE_SIZE + IDM_SIZE + STATUS_FLAG1_SIZE + STATUS_FLAG2_SIZE;
        byte[] responsePacket = new byte[len];
        responsePacket[PACKET_LENGTH_POS] = (byte)len;
        responsePacket[COMMAND_TYPE_POS] = RES_READ_WITHOUT_ENCRYPTION;
        responsePacket[headLen-2] = STATUS_FLAG1_SUCCESS;
        responsePacket[headLen-1] = STATUS_FLAG2_SUCCESS;
        responsePacket[headLen] = ONE_BLOCK;

        //IDmをセット
        responsePacket = setIDmToPacket(responsePacket,getIDm(commandPacket));

        //analyze
        int serviceNum = commandPacket[10]&0xFF;
        if(serviceNum != 1)
        {
            responsePacket[len-2] = STATUS_FLAG1_FAILED;
            responsePacket[len-1] = STATUS_FLAG2_SERVICE_NUM_ERROR;
            Log.d("HCEFService(NFCF)","SERVICE_NUM_ERROR");
            return responsePacket; //ERROR RES
        }

        int serviceCode = (commandPacket[11]&0xFF) | ((commandPacket[12]&0xFF)<<8);
        Log.d("HCEFService(NFCF)",String.format("Service code : %04X,",serviceCode));

        if(serviceCode != SERVICE_CODE_READ_WRITE && serviceCode != SERVICE_CODE_READ_ONLY)
        {
            responsePacket[len-2] = STATUS_FLAG1_FELICA_LITE_ERROR;
            responsePacket[len-1] = STATUS_FLAG2_SERVICE_CODE;
            Log.d("HCEFService(NFCF)","STATUS_FLAG2_SERVICE_CODE");
            return responsePacket; //ERROR RES
        }

        int blockNum = commandPacket[13]&0xFF;
        if(blockNum != 1)
        {
            responsePacket[len-2] = STATUS_FLAG1_FAILED;
            responsePacket[len-1] = STATUS_FLAG2_BLOCK_NUM_ERROR;
            Log.d("HCEFService(NFCF)","BLOCK_NUM_ERROR");
            return responsePacket; //ERROR RES
        }

        int blockAddress = -1;
        int blocklist_1 = commandPacket[14]&0xFF;
        if(blocklist_1 == 0x80)
        {
            //1byte
            blockAddress = commandPacket[15]&0xFF;
        }else if(blocklist_1 == 0x00)
        {
            //2byte
            blockAddress = (commandPacket[15]&0xFF) | ((commandPacket[16]&0xFF)<<8);
        }else{
            responsePacket[len-2] = STATUS_FLAG1_FAILED;
            responsePacket[len-1] = STATUS_FLAG2_ACCESS_MODE;
            Log.d("HCEFService(NFCF)","SERVICE_NUM_ERROR");
            return responsePacket; //ERROR RES
        }

        Log.d("HCEFService(NFCF)",String.format("Block Address : %04X,",blockAddress));

        byte data[] = {(byte)0x00,(byte)0x01,(byte)0x02,(byte)0x03,(byte)0x04,(byte)0x05,(byte)0x06,(byte)0x07,(byte)0x08,(byte)0x09,(byte)0x0A,(byte)0x0B,(byte)0x0C,(byte)0x0D,(byte)0x0E,(byte)0x0F};
        System.arraycopy(data, 0,responsePacket , headLen + BLOCK_NUM_SIZE, BLOCK_SIZE);

        String debug="";
        for(int i=0;i<responsePacket.length;i++)
        {
            debug += String.format("%02X,",responsePacket[i]);
        }
        Log.d("HCEFService(NFCF)",debug);
        Log.d("HCEFService(NFCF)","SUCCESS");

        return responsePacket;
    }

    private byte[] getIDm(byte[] commandPacket)
    {
        byte[] IDm = new byte[8];
        System.arraycopy(commandPacket,2, IDm, 0, 8);

        return IDm;
    }

    private  byte[] setIDmToPacket(byte[] packet,byte[] IDm)
    {
        System.arraycopy(IDm, 0, packet, IDM_POS, IDM_SIZE);// NFCID2
        return packet;
    }
}

```

---

#LocalBroadCastを用いて，サービスと処理を分離

```java:hcef_service.java
package jp.ne.sakura.sabowl.gpsnmeajp.intent_hce;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.nfc.cardemulation.HostNfcFService;
import android.os.Bundle;
import android.support.v4.content.LocalBroadcastManager;
import android.util.Log;

public class hcef_service extends HostNfcFService {
    @Override
    public void onCreate() {
        super.onCreate();
        Log.d("HCEFService(NFCF)","onCreate");
        LocalBroadcastManager.getInstance(this).registerReceiver(mSendResponsePacketReceiver,
                new IntentFilter("sendResponsePacket"));
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d("HCEFService(NFCF)","onDestroy");
        LocalBroadcastManager.getInstance(this).unregisterReceiver(mSendResponsePacketReceiver);
    }

    @Override
    public byte[] processNfcFPacket(byte[] commandPacket, Bundle extras)
    {
        Log.d("HCEFService(NFCF)","processNfcFPacket: Received NFCF");

        //Forward
        Intent intent = new Intent("processNfcFPacket");
        intent.putExtra("packet", commandPacket);
        LocalBroadcastManager.getInstance(this).sendBroadcast(intent);
        return null;
    }

    @Override
    public void onDeactivated(int reason) { //RF切断時
        Log.d("HCEFService(NFCF)","onDeactivated");

        //Forward
        Intent intent = new Intent("onDeactivated");
        intent.putExtra("reason", reason);
        LocalBroadcastManager.getInstance(this).sendBroadcast(intent);
    }

    private BroadcastReceiver mSendResponsePacketReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            byte responsePacket[] = intent.getByteArrayExtra("packet");
            sendResponsePacket(responsePacket);
            Log.d("HCEFService(NFCF)","sendResponsePacket");
        }
    };
}
```

```java:MainActivity.java
package jp.ne.sakura.sabowl.gpsnmeajp.intent_hce;

import android.content.BroadcastReceiver;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.pm.PackageManager;
import android.nfc.NfcAdapter;
import android.nfc.cardemulation.NfcFCardEmulation;
import android.os.Bundle;
import android.support.v4.content.LocalBroadcastManager;
import android.support.v7.app.AppCompatActivity;

import android.util.Log;
import android.widget.Toast;

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

        //------- HCE-F -------
        LocalBroadcastManager.getInstance(this).registerReceiver(mProcessNfcFPacketReceiver,
                new IntentFilter("processNfcFPacket"));
        LocalBroadcastManager.getInstance(this).registerReceiver(mOnDeactivatedReceiver,
                new IntentFilter("onDeactivated"));

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
                "jp.ne.sakura.sabowl.gpsnmeajp.intent_hce", //自パッケージ名
                "jp.ne.sakura.sabowl.gpsnmeajp.intent_hce.hcef_service"); //自サービス名
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

    //------------------------------------------------

    final int PACKET_LENGTH_POS = 0;
    final int PACKET_LENGTH_SIZE = 1;

    final int COMMAND_TYPE_POS = 1;
    final int COMMAND_TYPE_SIZE = 1;

    final int IDM_POS = 2;
    final int IDM_SIZE = 8;

    final int RESPONSE_SIZE = 1;
    final int STATUS_FLAG1_SIZE = 1;
    final int STATUS_FLAG2_SIZE = 1;

    final int BLOCK_NUM_SIZE = 1;
    final int BLOCK_SIZE = 16;

    final int ONE_BLOCK = 1;

    //---
    final byte CMD_POLLING = (byte)0x04;
    final byte RES_POLLING = (byte)0x05;
    final byte CMD_READ_WITHOUT_ENCRYPTION = (byte)0x06;
    final byte RES_READ_WITHOUT_ENCRYPTION = (byte)0x07;
    final byte CMD_WRITE_WITHOUT_ENCRYPTION = (byte)0x08;
    final byte RES_WRITE_WITHOUT_ENCRYPTION = (byte)0x09;

    //---
    final byte STATUS_FLAG1_SUCCESS = (byte)0x00;
    final byte STATUS_FLAG1_FAILED = (byte)0xFF;
    final byte STATUS_FLAG1_FELICA_LITE_ERROR = (byte)0x01;

    final byte STATUS_FLAG2_SUCCESS = (byte)0x00;
    final byte STATUS_FLAG2_READ_ERROR = (byte)0x70;
    final byte STATUS_FLAG2_READONLY = (byte)0xA8;
    final byte STATUS_FLAG2_NEED_AUTH = (byte)0xB1;
    final byte STATUS_FLAG2_SERVICE_NUM_ERROR = (byte)0xA1;
    final byte STATUS_FLAG2_BLOCK_NUM_ERROR = (byte)0xA2;
    final byte STATUS_FLAG2_SERVICE_CODE = (byte)0xA6;
    final byte STATUS_FLAG2_ACCESS_MODE = (byte)0xA7;

    final byte SERVICE_CODE_READ_ONLY = 0x000B;
    final byte SERVICE_CODE_READ_WRITE = 0x0009;

    private BroadcastReceiver mProcessNfcFPacketReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            byte commandPacket[] = intent.getByteArrayExtra("packet");
            Log.d(debugMsgTitle,"processNfcFPacket");

            byte responsePacket[] = processNfcFPacket(commandPacket);
            //Forward
            Intent responceIntent = new Intent("sendResponsePacket");
            responceIntent.putExtra("packet", responsePacket);
            LocalBroadcastManager.getInstance(context).sendBroadcast(responceIntent);
        }
    };

    private BroadcastReceiver mOnDeactivatedReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            int reason = intent.getIntExtra("reason",0);
            Log.d(debugMsgTitle,"onDeactivated");
        }
    };

    public byte[] processNfcFPacket(byte[] commandPacket)
    {
        Log.d(debugMsgTitle,"processNfcFPacket: Received NFCF");
        Toast.makeText(this, "processNfcFPacket", Toast.LENGTH_LONG).show();

        Intent intent = new Intent("custom-event-name");
        intent.putExtra("message", "This is my message!");
        LocalBroadcastManager.getInstance(this).sendBroadcast(intent);


        //必須情報より小さいならエラー
        if (commandPacket.length < (PACKET_LENGTH_SIZE+COMMAND_TYPE_SIZE+IDM_SIZE)) {
            Log.e(debugMsgTitle,"processNfcFPacket: Packet size too short");
            return null;
        }

        byte commandType = commandPacket[COMMAND_TYPE_POS];
        switch(commandType)
        {
            case CMD_POLLING: return null;//stub
            case CMD_WRITE_WITHOUT_ENCRYPTION: return WriteWithoutEncryption(commandPacket);//stub
            case CMD_READ_WITHOUT_ENCRYPTION: return ReadWithoutEncryption(commandPacket);//stub
            default: return null; //該当コマンドなし
        }
    }

    private byte[] WriteWithoutEncryption(byte[] commandPacket)
    {
        Log.d(debugMsgTitle,"WriteWithoutEncryption");
        //response
        int len = PACKET_LENGTH_SIZE + RESPONSE_SIZE + IDM_SIZE + STATUS_FLAG1_SIZE + STATUS_FLAG2_SIZE;
        byte[] responsePacket = new byte[len];
        responsePacket[PACKET_LENGTH_POS] = (byte)len;
        responsePacket[COMMAND_TYPE_POS] = RES_WRITE_WITHOUT_ENCRYPTION;
        responsePacket[len-2] = STATUS_FLAG1_SUCCESS;
        responsePacket[len-1] = STATUS_FLAG2_SUCCESS;

        //IDmをセット
        responsePacket = setIDmToPacket(responsePacket,getIDm(commandPacket));

        //analyze
        int serviceNum = commandPacket[10]&0xFF;
        if(serviceNum != 1)
        {
            responsePacket[len-2] = STATUS_FLAG1_FAILED;
            responsePacket[len-1] = STATUS_FLAG2_SERVICE_NUM_ERROR;
            Log.d(debugMsgTitle,"SERVICE_NUM_ERROR");
            return responsePacket; //ERROR RES
        }

        int serviceCode = (commandPacket[11]&0xFF) | ((commandPacket[12]&0xFF)<<8);
        Log.d(debugMsgTitle,String.format("Service code : %04X,",serviceCode));

        if(serviceCode != SERVICE_CODE_READ_WRITE )
        {
            responsePacket[len-2] = STATUS_FLAG1_FELICA_LITE_ERROR;
            responsePacket[len-1] = STATUS_FLAG2_SERVICE_CODE;
            Log.d(debugMsgTitle,"STATUS_FLAG2_SERVICE_CODE");
            return responsePacket; //ERROR RES
        }

        int blockNum = commandPacket[13]&0xFF;
        if(blockNum != 1)
        {
            responsePacket[len-2] = STATUS_FLAG1_FAILED;
            responsePacket[len-1] = STATUS_FLAG2_BLOCK_NUM_ERROR;
            Log.d(debugMsgTitle,"BLOCK_NUM_ERROR");
            return responsePacket; //ERROR RES
        }

        int blockAddress = -1;
        int blockHeadPos = -1;
        int blocklist_1 = commandPacket[14]&0xFF;
        if(blocklist_1 == 0x80)
        {
            //1byte
            blockAddress = commandPacket[15]&0xFF;
            blockHeadPos = 16;
        }else if(blocklist_1 == 0x00)
        {
            //2byte
            blockAddress = (commandPacket[15]&0xFF) | ((commandPacket[16]&0xFF)<<8);
            blockHeadPos = 17;
        }else{
            responsePacket[len-2] = STATUS_FLAG1_FAILED;
            responsePacket[len-1] = STATUS_FLAG2_ACCESS_MODE;
            Log.d(debugMsgTitle,"SERVICE_NUM_ERROR");
            return responsePacket; //ERROR RES
        }

        Log.d(debugMsgTitle,String.format("Block Address : %04X,",blockAddress));

        byte data[] = new byte[BLOCK_SIZE];
        System.arraycopy(commandPacket, blockHeadPos,data , 0, BLOCK_SIZE);

        String debug="";
        for(int i=0;i<data.length;i++)
        {
            debug += String.format("%02X,",data[i]);
        }
        Log.d(debugMsgTitle,debug);
        Log.d(debugMsgTitle,"SUCCESS");
        return responsePacket; //SUCCESS
    }
    private byte[] ReadWithoutEncryption(byte[] commandPacket)
    {
        Log.d(debugMsgTitle,"ReadWithoutEncryption");
        //response
        int len = PACKET_LENGTH_SIZE + RESPONSE_SIZE + IDM_SIZE + STATUS_FLAG1_SIZE + STATUS_FLAG2_SIZE + BLOCK_NUM_SIZE + BLOCK_SIZE;
        int headLen = PACKET_LENGTH_SIZE + RESPONSE_SIZE + IDM_SIZE + STATUS_FLAG1_SIZE + STATUS_FLAG2_SIZE;
        byte[] responsePacket = new byte[len];
        responsePacket[PACKET_LENGTH_POS] = (byte)len;
        responsePacket[COMMAND_TYPE_POS] = RES_READ_WITHOUT_ENCRYPTION;
        responsePacket[headLen-2] = STATUS_FLAG1_SUCCESS;
        responsePacket[headLen-1] = STATUS_FLAG2_SUCCESS;
        responsePacket[headLen] = ONE_BLOCK;

        //IDmをセット
        responsePacket = setIDmToPacket(responsePacket,getIDm(commandPacket));

        //analyze
        int serviceNum = commandPacket[10]&0xFF;
        if(serviceNum != 1)
        {
            responsePacket[len-2] = STATUS_FLAG1_FAILED;
            responsePacket[len-1] = STATUS_FLAG2_SERVICE_NUM_ERROR;
            Log.d(debugMsgTitle,"SERVICE_NUM_ERROR");
            return responsePacket; //ERROR RES
        }

        int serviceCode = (commandPacket[11]&0xFF) | ((commandPacket[12]&0xFF)<<8);
        Log.d(debugMsgTitle,String.format("Service code : %04X,",serviceCode));

        if(serviceCode != SERVICE_CODE_READ_WRITE && serviceCode != SERVICE_CODE_READ_ONLY)
        {
            responsePacket[len-2] = STATUS_FLAG1_FELICA_LITE_ERROR;
            responsePacket[len-1] = STATUS_FLAG2_SERVICE_CODE;
            Log.d(debugMsgTitle,"STATUS_FLAG2_SERVICE_CODE");
            return responsePacket; //ERROR RES
        }

        int blockNum = commandPacket[13]&0xFF;
        if(blockNum != 1)
        {
            responsePacket[len-2] = STATUS_FLAG1_FAILED;
            responsePacket[len-1] = STATUS_FLAG2_BLOCK_NUM_ERROR;
            Log.d(debugMsgTitle,"BLOCK_NUM_ERROR");
            return responsePacket; //ERROR RES
        }

        int blockAddress = -1;
        int blocklist_1 = commandPacket[14]&0xFF;
        if(blocklist_1 == 0x80)
        {
            //1byte
            blockAddress = commandPacket[15]&0xFF;
        }else if(blocklist_1 == 0x00)
        {
            //2byte
            blockAddress = (commandPacket[15]&0xFF) | ((commandPacket[16]&0xFF)<<8);
        }else{
            responsePacket[len-2] = STATUS_FLAG1_FAILED;
            responsePacket[len-1] = STATUS_FLAG2_ACCESS_MODE;
            Log.d(debugMsgTitle,"SERVICE_NUM_ERROR");
            return responsePacket; //ERROR RES
        }

        Log.d(debugMsgTitle,String.format("Block Address : %04X,",blockAddress));

        byte data[] = {(byte)0x00,(byte)0x01,(byte)0x02,(byte)0x03,(byte)0x04,(byte)0x05,(byte)0x06,(byte)0x07,(byte)0x08,(byte)0x09,(byte)0x0A,(byte)0x0B,(byte)0x0C,(byte)0x0D,(byte)0x0E,(byte)0x0F};
        System.arraycopy(data, 0,responsePacket , headLen + BLOCK_NUM_SIZE, BLOCK_SIZE);

        String debug="";
        for(int i=0;i<responsePacket.length;i++)
        {
            debug += String.format("%02X,",responsePacket[i]);
        }
        Log.d(debugMsgTitle,debug);
        Log.d(debugMsgTitle,"SUCCESS");

        return responsePacket;
    }

    private byte[] getIDm(byte[] commandPacket)
    {
        byte[] IDm = new byte[8];
        System.arraycopy(commandPacket,2, IDm, 0, 8);

        return IDm;
    }

    private  byte[] setIDmToPacket(byte[] packet,byte[] IDm)
    {
        System.arraycopy(IDm, 0, packet, IDM_POS, IDM_SIZE);// NFCID2
        return packet;
    }
}

```
