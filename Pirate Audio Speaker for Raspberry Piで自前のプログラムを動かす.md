# はじめに
Pirate Audio: Speaker for Raspberry Piというボードを千石電商の店頭で見て買ってしまいました。
これは、Raspberry Pi Zeroに被せて使うDAC+スピーカー+カラー液晶+ボタンのついたボードです。

https://shop.pimoroni.com/products/pirate-audio-mini-speaker

もともと、音楽プレイヤーを作るためのキットとなっており、書きを実行するだけで音楽プレイヤーとして機能するようになっています。

```
git clone https://github.com/pimoroni/pirate-audio
cd pirate-audio/mopidy
sudo ./install.sh
```

が、このボードを買う人の大半は、自分好みの画面を出して自分のプログラムで制御したい人ではないでしょうか。

そのための手順をメモしておきます。

# OSのセットアップ
Raspberry Pi Zeroにピンヘッダをはんだ付けし、接続できたら、
Raspberry Pi ImagerでRaspberry Pi OSをセットアップしましょう。

書き込む前に、歯車マークからSSHの有効化とWi-Fiの設定などを済ませておきましょう。
※Wi-Fiは2.4GHz帯である必要があるので注意です。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/ab3693cd-9c35-6068-5997-5013e1f52505.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/40249439-8d3e-6455-7171-de228a957a75.png)

# 初回起動
起動時にはしばらく時間がかかります。数分待ちましょう。

しばらく経ったあと、[Advanced IP Scanner](https://forest.watch.impress.co.jp/library/software/advipscanner/)などを使ってRaspberry Pi ZeroのIPアドレスを探します。

見つかったらSSHで接続しましょう

```
ssh pi@192.168.1.3
```

# セットアップ

まず下記を実行してセットアップを行います

```
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install -y python3-rpi.gpio python3-spidev python3-pip python3-pil python3-numpy libopenjp2-7
sudo raspi-config nonint do_spi 0
sudo pip3 install st7789
```

次に音声デバイスの有効化を行います。

```
sudo nano /boot/config.txt
```

で/boot/config.txtを開いたあと、下記を最終行に追加して保存

```
gpio=25=op,dh
dtoverlay=hifiberry-dac
```

再起動します。

```
sudo reboot
```

SSHに再接続し、音声の設定をします。

```
aplay -l
```

下記のようにcard0 に HifiBerry DAC HiFi pcm5102a-hifi-0 が認識されていればOKです。

```
**** List of PLAYBACK Hardware Devices ****
card 0: sndrpihifiberry [snd_rpi_hifiberry_dac], device 0: HifiBerry DAC HiFi pcm5102a-hifi-0 [HifiBerry DAC HiFi pcm5102a-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: vc4hdmi [vc4-hdmi], device 0: MAI PCM i2s-hifi-0 [MAI PCM i2s-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

下記で音がなるはずです。(鳴らない場合はHDMI接続していませんか？そちらで鳴っているかもしれません)

```
aplay /usr/share/sounds/alsa/Front_Left.wav
```

音量が大きすぎる場合はalsamixerコマンドで音量調整してください。

```
alsamixer
```

# サンプルを動かす

画面を映す

```
cd ~
git clone https://github.com/pimoroni/pirate-audio
cd pirate-audio/examples
python3 rainbow.py
```

ボタン入力

```
cd pirate-audio/examples
python3 buttons.py
```

バックライト制御

```
cd pirate-audio/examples
python3 backlight-pwm.py
```

ボタン入力

```
cd pirate-audio/examples
python3 buttons.py
```
