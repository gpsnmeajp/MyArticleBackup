VirtualBox Guest Additionをインストールしようとすると

```
vboxadd.sh: Building Guest Additions kernel modules.
Failed to set up service vboxadd, please check the log file
/var/log/VBoxGuestAdditions.log for details.
```

で止まる場合。

以下のようにインストール。(gccとかはインストールしてあること前提)

```
apt-get install build-essential module-assistant linux-headers-$(uname -r)
```

あとは普通に

```
su
sh VBoxLinuxAdditions.run
```

でOK。
