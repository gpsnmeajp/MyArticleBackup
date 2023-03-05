Raspberry Pi ZeroはUSBスレーブとして動作することができ、仮想Ethernetアダプタとして動作させることができます。
ので、Raspberry Pi Zero Wを買ったけど、microUSBケーブルしか無い！という場合のセットアップ。
microUSBケーブルでPCからSSH接続しつつ、テザリングスマホに繋いでアップデート諸々してみます。

#Ethernetドライバの準備
Windowsならば以下のドライバをインストールする。
(これを入れないとCOMポートとして認識される。されてしまった場合はデバイスマネージャから削除)

Acer Incorporated. - Other hardware - USB Ethernet/RNDIS Gadget	Windows 7,Windows 8,Windows 8.1 and later drivers
http://www.catalog.update.microsoft.com/Search.aspx?q=Acer%20Incorporated.%20-%20Other%20hardware%20-%20USB%20Ethernet%2FRNDIS%20Gadget

CABファイルを解凍し、「RNDIS.inf」を右クリックして「インストール」。

#Bonjourの導入
Windows 10は標準搭載されている為不要です。
それ以外(あるいはWindows 10でもうまくいかない場合)は、
iTunesかbonjour for windowsを導入してください
https://support.apple.com/kb/DL999?locale=ja_JP&viewlocale=ja_JP

ファイアーウォールの設定が必要な場合があります。

Bonjour for Windows不要！Windows10マシンに".local"でアクセスしよう！ - もぐてっく http://moguno.hatenablog.jp/entry/2015/09/12/100231

#SSHクライアントの導入
個人的にはTeratermの導入をおすすめします
https://ja.osdn.net/projects/ttssh2/

#Raspberry Piのイメージを書き込む
お好きな方法でどうぞ

#USB OTGでのEthernetの設定
##config.txtの書き換え
bootパーティションを開きconfig.txtを開く。

```
dtoverlay=dwc2
```

以下は私のRaspberry pi Zero Wの例

```text:config.txt
～～～省略～～～
# Uncomment this to enable the lirc-rpi module
#dtoverlay=lirc-rpi

# Additional overlays and parameters are documented /boot/overlays/README

# Enable audio (loads snd_bcm2835)
dtparam=audio=on
dtoverlay=dwc2
```

を末端に追加

##cmdline.txtの書き換え
bootパーティションを開きcmdline.txtを開く。
cmdline.txt

```
modules-load=dwc2,g_ether
```

をrootwaitのあとに追記。
改行せずスペース区切りで続けて記述する。

以下は私のRaspberry pi Zero Wの例

```text:cmdline.txt
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=PARTUUID=cedfec53-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait modules-load=dwc2,g_ether quiet splash plymouth.ignore-serial-consoles
```

#SSHの準備
bootパーティションにSSHという空ファイルを作成

#Raspberry Pi Zero WとPCを接続
刺すのはPWRではなく、USBのポートなので注意
(HDMIポートよりの方のmicro USB端子)

暫く待つと、Ethernetアダプタとして認識されます。

#teratermで接続

```
raspberrypi.local
```

に接続。BonjourかmDNSが正常に機能していて、Ethernetが正常に認識されていれば、接続されます。

改行コードがおかしくなってる場合は、端末設定から設定

#Wi-Fi接続

```
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

以下の記述を追加

```
network={
    ssid="SSID"
    psk="パスワード"
}
```

それから接続を反映する

```bash
sudo ifdown wlan0
sudo ifup wlan0
```


#raspberrypi.localで繋がらなくなったとき
raspi-configでホスト名を変更しませんでしたか？
うっかりミスでやりましたのでご注意を

#PC経由でネットに繋げたい場合
PCからインターネットに出たい場合(ネカフェWiFiや会社のLAN等)は、PCをブリッジにするとでき...ると思います。
(どうやらIPv6でつながってるっぽいのでわからん)

Raspberry PiとMac or Windows PCを有線で直接繋いでさくっとSSH接続する http://qiita.com/mascii/items/7d955395158d4231aef6

#参考文献
Raspberry Pi ZeroをUSBケーブル1本で遊ぶ 
http://www.raspi.jp/2016/07/pizero-usb-otg/
(ほとんどここの引き写しですスミマセン)

Raspbian 2016-11-25リリース 
http://www.raspi.jp/2016/12/raspbian-2016-11-25-release/

Raspberry Pi Zero W: fast headless Wifi configuration with SSH
http://www.albertosarullo.com/blog/raspberry-pi-zero-w-headless-wifi-ssh-configuration-10-minutes


シリアルポートとして使う
http://qiita.com/mt08/items/84d3d3053bee92609c2d#_reference-b4829a49ece9984a4726

ってか、似たような内容がいくつもあった...

その他の使用例
Turning your Raspberry PI Zero into a USB Gadget
https://cdn-learn.adafruit.com/downloads/pdf/turning-your-raspberry-pi-zero-into-a-usb-gadget.pdf

Raspberry PiのUSB OTGを試す by @matoken #kagolug #raspberrypi https://www.slideshare.net/matoken/raspberry-piusb-otg
