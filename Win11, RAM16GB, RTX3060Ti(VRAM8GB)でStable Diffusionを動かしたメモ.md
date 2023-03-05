# 概要
Win11, RAM16GB, RTX3060Ti(VRAM8GB)でStable Diffusionを動かしたメモ

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/093e4377-5789-8614-1f5b-6abe63151e84.png)

# WSLのインストール
通常通り行う。

```
wsl --install
```

PCを再起動して、初回起動セットアップ後、

```
sudo apt-install python3-pip
```

するのを忘れずに。

# スワップの拡張
RAM16GBの環境だと、WSLは8GBになる。
どっちにしろ、ものすごいRAMを食うのでStable Diffusionを動かしてもOOMで止められてしまう。

```
C:\Users\{ユーザー名}\.wslconfig
```

を新規作成し、以下を書き込む

```
[wsl2]
swap=32GB
```

```
wsl --shutdown
```

する。

参考文献

https://zenn.dev/d_pontaro/articles/wsl-config-swap

# 環境構築する

以下のサイトを参照
https://touch-sp.hatenablog.com/entry/2022/08/23/222916

下記をExplorerで開いてファイルを入れると良い
```
\\wsl.localhost\Ubuntu\home\{UNIXユーザー名}\stable-diffusion
```

しかし、環境構築が終わって実行しても、Out of Memoryで止まると思う。

# Out of Memory対策

下記のサイトの通り、text2img.pyを修正して、モデルをfp16にする

https://td2sk.hatenablog.com/entry/2022/08/24/001630

# 実行する

```
python3 scripts/txt2img.py --prompt "a photograph of an astronaut riding a horse" --plms --ckpt sd-v1-4.ckpt --n_samples 1
```

以下のフォルダに出力される。

```
\\wsl.localhost\Ubuntu\home\{UNIXユーザー名}\stable-diffusion\outputs\txt2img-samples
```

150秒くらい掛かる。
