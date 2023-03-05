```powershell
#ランダムなパスワードを生成
Add-type -AssemblyName System.Web;[System.Web.Security.Membership]::GeneratePassword(32,0) | %{$_ -replace "[^a-z0-9]",""}
```

```sh
#ubuntu16.04を自動インストール

#捨てランダムパスワードで管理者パスワードを設定

#シリアルコンソールでログイン
sudo su

#ポートを全閉鎖
ufw enable
ufw default DENY
ufw reload
ufw status

#ufwを有効にするために既設のiptablesをふっとばす
mv /etc/iptables/iptables.rules /var/tmp
#既設のiptablesの無効化とufwの有効化
reboot

#外界からの一切の通信を受け付けない状態になりました
#シリアルコンソールでログイン
sudo su

#ユーザーがいないかチェック
w

#システムのアップデート
apt-get update
apt-get upgrade

#grubのアップデートはKeepに

#iptstate
apt-get install iptstate

#SSHのポート変更、rootログイン禁止変更
vi /etc/ssh/sshd_config
> Port 1111 # イイ感じの数字に変える
> PermitRootLogin no # yesをnoにする

#UFWポート開放
ufw allow 1111 #SSHを開放する
ufw reload # リロードする
ufw status # 開いてるポートを確認する

#適用
reboot

#SSHでログイン
sudo su

#ユーザーがいないかチェック
w

#作業用ユーザー追加・パスワード設定
adduser username

#ユーザーにsudo権限を付与
gpasswd -a username sudo

#作業用ユーザーでログインできることの確認。
#以降、作業用ユーザーで作業を開始

#ubuntuアカウントのロックアウト
usermod -L ubuntu

#再起動
reboot

#作業用ユーザーに公開鍵ログインを設定
https://help.sakura.ad.jp/hc/ja/articles/206208161

#SSH公開鍵をホームにアップロード
mkdir .ssh 
chmod 700 .ssh 
cat id_rsa.pub > .ssh/authorized_keys 
chmod 600 .ssh/authorized_keys 
rm -f id_rsa.pub

#パスワードログインの禁止
sudo vi /etc/ssh/sshd_config
> PasswordAuthentication no

sudo service sshd restart

#一通りの設定が完了
```

```
w
ログイン中ユーザー一覧が見れる

sudo iptstate
通信接続状態を一覧で表示できる

sudo tailf /var/log/ufw.log
ファイアーウォールの遮断ログが見れる

```

参考文献

http://kazmax.zpp.jp/linux/account_lock.html

https://qiita.com/0x50/items/05c89333ae046dc6fa0f

https://help.sakura.ad.jp/hc/ja/articles/206208181?_ga=2.80060179.589210644.1540011313-1788467906.1527353702&_bdld=2C9DGp.mqfQYyB

http://blog.mogmet.com/how-to-enable-ufw-on-sakura-vps/
