# はじめに
Raspberry Piも高価、電子部品としての大画面タッチパネルはもっと高価な今日このごろ。

電子部品感覚で使えそうな、安いWindowsタブレットとか無いかと探してみたところ、[Windows TabletのWT8-A / VT484 の状態が良い中古が5000円～7000円程度でたくさん投げ売りされています。](https://www.google.com/search?q=wt8-a)
Windows 8.1がデフォで入っており、RAM 2GB, eMMC 32GB, Atom Z3740 クアッドコアプロセッサというもので、だいたい中古で買うとWindows 10が入っています。

まあ意外とWindows 10でも使えなくはないのですが、せっかくなので軽いLubuntuを入れてみます。
より快適になりますし、色々いじりがいがあります。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/ac9a9f1f-1ea7-c6a3-63f7-e0a68d826209.png)

# インストールの準備(ハード)

まず、Windowsが正常に動作するかは確認しておきましょう。

その上で、

+ microUSB-USBハブ (Android向けUSBホストハブなど) セルフパワー推奨
+ USBキーボード
+ USBマウス
+ USBメモリ

を用意し、接続し、正常に動作することを確認してください。

あと、100%まで充電しておいてください。(後述の充電機能付きハブを使う場合を除く)

# インストールの準備(ソフト)

+ [Lubuntuの22.04.1 LTSのISOイメージ](https://lubuntu.me/)
+ [debian(multi-arch)のISOイメージ](http://ftp.jaist.ac.jp/debian-cd/11.5.0/multi-arch/iso-cd/)
+ [rufus](https://forest.watch.impress.co.jp/library/software/rufus/)

まず、LubuntuのISOイメージを、rufusを使ってISOモードでUSBメモリに焼きます。

これで通常のPCはUSBメモリからブートできるようになりますが、WT8-A / VT484では起動できません。
なぜかというと、32bit UEFI環境(CPUは64bit)だからです。

そのため、起動用のEFIブートローダを、debianから取り出します。
debian-11.5.0-amd64-i386-netinst.isoを適当なツール(7-zipなど)で開き

```
/EFI/boot/bootia32.efi
/EFI/boot/grubia32.efi
```

を、USBメモリ内の同じパスにコピーします。

また、

```
/boot/grub/grub.cfg
```

を

```
/EFI/debian/grub.cfg
```

にコピーします。

# インストール

以下どちらかの方法で、USBメモリから起動します。

+ Windowsの回復メニューから、USBメモリで起動する
+ 電源ボタンを10秒押して完全に電源を切ってから、音量ボタン+を押しながら電源ボタンを押しつづけ、UEFIブートメニューを開く(メニューが出るまで離さないこと)

すると、grubのメニューが出るので、Enterを押して起動します。(機種によっては、先にUEFIからSecure bootをオフにする必要があるかもしれません)

しばらくするとLive CD版デスクトップが表示されるはずです。

あとは、Install Lubuntu 22.04 LTSのアイコンから、インストールウィザードに従ってインストールしてください。
タッチ操作も使用できます。

再起動後は、特に何事もないかのように普通に起動します。
シャットダウン時のPlease remove medium... は、Enterキーまたは、本体の音量-キーで進むことができます。

# grub.cfgをコピーし忘れた場合

筆者が最初にインストールしたときの手順。

USBメモリから起動すると、grubのコマンド画面に飛ばされます。

```
grub>
```

と表示されていると思います。

```
ls
```

を実行すると、下記のように出ると思います。

```
(proc) (hd0) (hd0,msdos1) (hd1) ...
```

hdの後の番号が違う場合は、適時読み替えてください。

ここで、msdos1となっているのが現在差し込んでいるUSBメモリです。

ここから、以下のコマンドを1行ずつ入力します。
USキーボード扱いなので、JISキーボードを使用している場合は(,),=などが刻印と異なる記号が入力されます。
対応表を見るなり、色々打ち込んでみるなりして、適切な記号を入力してください。

なお、TAB補完が使用できます。

```
set root=(hd0,msdos1)
linux /casper/vmlinuz file=/cdrom/preseed/lubuntu.seed quiet splash --- 
initrd /casper/initrd
boot
```

2,3行目がキーとなるコマンドですが、これはlubuntuのISO内の /boot/grub/grub.cfg に記載されているコマンドです。
別ディストリビューションでは、それぞれのgrub.cfgを使うことで同様に実行できると思います。(適切に配置するれば自動で読み込む可能性あり。)

bootを打ってEnterを押すと、UEFIの画面に戻り、しばらくするとLive CD版デスクトップが表示されるはずです。

# おまけ: スピーカーから音が出ない
Windows 10の場合、メーカー公式サイトからプラットフォームドライバをインストールしてください。

Lubuntuの場合、ミキサーからBuilt-in Audio Headphones + Speakersの「ポート」をSpeakersに切り替えることで、音が出るようになります。

# おまけ: タスクバーのタッチが効かない
タッチパネルのドライバの実装か何かに問題があるらしく、一部のアプリケーションがうまく操作をハンドリングできなくなるタイミングがあるようです。
対策は調査中

→ LXDEに変えましょう

その後自動ログインのセッションも変更
https://www.palm84.com/entry/20150425/1429958648

# おまけ: USBホストを接続しながら充電(低速)したい場合
価格.comのページ(URL忘れました)にて、USBハブを改造することで実現できるという情報がありました。

どうも、mico USBのID端子(ホストモード切替)を、スレーブ状態にしたまま電源を供給し、信号線をUSB Aメスに接続することで充電しながら周辺機器を繋げられるようです。

実際手元で作成して試してみたところ動作しました。
(ほぼこの記事のものと同等: https://ascii.jp/elem/000/001/019/1019243/2/ )

安さを求める場合は、安いUSBハブに、microBケーブルをぶった切ったものを繋げ、AC 5Vをどうにかして(USB充電器でもなんでも)供給すればよいかと思います。

~~なお、既製品でも同様の機能を持ったハブが販売されているようです。~~
→買ってみましたが動きません。自作したものが確実でした。

# おまけ: 画面の要素が小さい
下記に従って、DPI設定をしてみてください。

https://askubuntu.com/questions/1135874/how-to-change-display-scaling-settings-in-lubuntu

QT_SCALE_FACTOR, GDK_DPI_SCALEは1.5
XCURSOR_SIZEは48
DPIは144

くらいにするとちょうどよくタブレットっぽくなります。

# おまけ: ソフトウェアキーボード
Live CDだとソフトウェアキーボードが自動で起動するのですが、どうもない模様？

```
sudo apt-get install onboard
```

でいい感じのソフトウェアキーボードが入ります。
スタート → ユニバーサル・アクセス → Onboard
で起動します。
適当にカスタマイズすると入力しやすくなります。

# おまけ: 起動・終了時のsplashは、音量-ボタンでコンソールに切り替わります
起動や終了が妙に遅いときは、音量-を押すとコンソールに切り替わるので原因がわかります。

# おまけ: 電源を切るときは、電源ボタンを押す
すぐにシャットダウンが始まります。(設定で変えられる？)

# おまけ: 二本指ジェスチャーなど使いたい
toucheggを導入するとできます。

# おまけ: マウスカーソルを追従させたくない
/usr/share/X11/xorg.conf.d/40-libinput.conf で
touchscreenだけDriverをevdevにすると、マウスと別になります。

# おまけ: タッチを回転させたい
多分これでできます。(未検証)
https://blog.goediy.com/?p=441

または

https://www.gechic.com/ja/raspberry-pi-touch-monitor-rotate-touch-screen-rotate-settings/

# おまけ: タッチスクリーンのデバッグしたい
evtest /dev/input/event0 を使用する


# 参考文献
https://blog.goediy.com/?p=440
https://urotasm.hatenablog.com/entry/2018/01/12/232111
https://szymonkrajewski.pl/how-to-boot-system-from-usb-using-grub/
https://www.express.nec.co.jp/linux/distributions/knowledge/system/grub.html
http://lubuntuhowto.blogspot.com/2014/10/on-screen-virtual-keyboard-on-lubuntu.html
