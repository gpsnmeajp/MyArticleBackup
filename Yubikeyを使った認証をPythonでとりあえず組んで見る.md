#はじめに
[Yubikeyをオフラインで使う方法を考えてみる](http://gpsnmeajp.sblo.jp/article/181685678.html)で、色々検討してみたわけですが、
実際実装してみないと特性わからないなぁ、と思い、とりあえず組んでみることにしました。

#⚠注意
あくまで試しに実装してみた形です。
セキュリティに関しては詳しくないため、誤りが含まれるかもしれません。
実務等にこれらをそのまま使うことは避けてください。

#実装
##1. Static password
ただの簡易的なパスワード認証です。
恥ずかしながらそれさえ組んだことがなかったので、とりあえず試しに組んでみました。

```shell-session:事前準備
pip install bcrypt
```

```python:register.py
# coding: utf-8
import getpass
import bcrypt

def main():
    print("Password Registration")
    password = getpass.getpass('password: ').encode('utf8')
    verify_password = getpass.getpass('verify password: ').encode('utf8')

    print("Please wait...")
    salt = bcrypt.gensalt(rounds=10, prefix=b'2a')
    pwhash = bcrypt.hashpw(password, salt)

    if(bcrypt.checkpw(verify_password,pwhash)):
        print("Verify OK")
    else:
        print("Verify NG")
        print("Registration failed")
        return

    try:
        with open('password.txt', 'wb') as f:
            f.write(pwhash)
        print("Registration succeed")
    except Exception as e:
        print(e)
        print("Registration error")



if __name__ == '__main__':
    main()
```


```python:verify.py
# coding: utf-8
import getpass
import bcrypt

def main():
    print("Enter password")
    password = getpass.getpass('password: ').encode('utf8')

    print("Please wait...")

    try:
        with open("password.txt","r") as f:
            pwhash = f.read().encode('utf8')

        if(bcrypt.checkpw(password,pwhash)):
            print("OK")
        else:
            print("NG")
    except Exception as e:
        print(e)
        print("Error")

if __name__ == '__main__':
    main()
```

##2. Public ID (先頭12文字)
Yubikeyのワンタイムパスワードの先頭12文字を切り出してパスワードにするスタイルです。
本来Public IDと呼ばれ、Yubico OTPにおけるIDに相当します。

LastPassのオフライン設定等で利用されています。
お手軽、しかし、あまり強くはないかと...

```shell-session:事前準備
pip install bcrypt
```

```python:register.py
# coding: utf-8
import bcrypt

def main():
    print("Yubikey Registration")
    password = input('Touch Yubikey: ')[0:12].encode('utf8')
    verify_password = input('verify Touch Yubikey: ')[0:12].encode('utf8')

    print("Please wait...")
    salt = bcrypt.gensalt(rounds=10, prefix=b'2a')
    pwhash = bcrypt.hashpw(password, salt)

    if(bcrypt.checkpw(verify_password,pwhash)):
        print("Verify OK")
    else:
        print("Verify NG")
        print("Registration failed")
        return

    try:
        with open('password.txt', 'wb') as f:
            f.write(pwhash)
        print("Registration succeed")
    except Exception as e:
        print(e)
        print("Registration error")



if __name__ == '__main__':
    main()

```


```python:verify.py
# coding: utf-8
import bcrypt

def main():
    password = input('Touch Yubikey: ')[0:12].encode('utf8')

    print("Please wait...")

    try:
        with open("password.txt","r") as f:
            pwhash = f.read().encode('utf8')

        if(bcrypt.checkpw(password,pwhash)):
            print("OK")
        else:
            print("NG")
    except Exception as e:
        print(e)
        print("Error")

if __name__ == '__main__':
    main()

```

##3. Yubico OTP(online)
YubiCloudのYubico OTP認証を利用します。
ここで同等のテストができます。
https://demo.yubico.com/

yubico-clientは非公式のクライアントライブラリですが、
ローカル認証の機能も内包した公式のよりよっぽど導入が楽です。(Windowsにおいて)
https://yubico-client.readthedocs.io/en/latest/

認証を利用するにはYubico公式サイトでAPIキーを取得する必要があり、
そのためには開発者は最低1本のYubikeyを持っている必要があります。
https://upgrade.yubico.com/getapikey/

```shell-session:事前準備
pip install yubico-client
```

登録はYubico側で工場出荷時に行われているため、ユーザー側での特別の登録は不要です。
(ユーザーIDとの関連付けは必要ですが、今回は対象外としています。)

もし、パーソナライゼーションツールでYubico OTP情報を書き換えた場合は、
以下のURLで再登録することができます。
https://upload.yubico.com/

```python:verify.py
from yubico_client import Yubico

def main():
    password = input('Touch Yubikey: ')
    client = Yubico('アプリケーションID', 'シークレット')

    print("KeyID:"+password[0:12])

    try:
        if(client.verify(password)):
            print("OTP valid (OK)")
        else:
            print("Unknown Error")
    except Exception as e:
        print(e)
        print("Error")

if __name__ == '__main__':
    main()
```

##以下、開発中...
メーカーが提供するライブラリがバグってる...
