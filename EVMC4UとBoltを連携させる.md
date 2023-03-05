# はじめに
拙作のEVMC4UとBoltを連携させたいというツイートを見かけました。
Boltは、最近Unity Technologiesに買収され無償化された、ビジュアルスクリプティング環境です

私自身Boltを触ったことがあまりないのですが、手探りで試してみましたので記録しておきます。

# 環境
- Unity 2020.1.2f1
- EVMC4U v3.7
- Bolt Version 1.4.13 - September 28, 2020

# EVMC4Uのセットアップ
EVMC4U v3.7は以下からダウンロードして使用します。

booth
https://booth.pm/ja/items/1801535
もしくはgithub
https://github.com/gpsnmeajp/EasyVirtualMotionCaptureForUnity

EVMC4U自体のセットアップは、UnityPackageを導入した直後に出るチュートリアルに従ってください。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/c49d7f3f-3d35-53ed-6a89-8aca76bb5f85.png)


# Bolt のセットアップ
## 導入
Boltは、AssetStoreで導入後、PackageManagerから導入します。

Download後、Importできなくて困ったのですが、Toolsから「Install Bolt」をクリックで導入されるようです。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/15a8facf-899f-e6df-b45c-e2b0d52c86e1.png)

その後、セットアップに入るのですが、**重要な手順がいくつかあります**。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/04bed051-a71d-39b8-ceb7-5f26d1750af6.png)
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/b514b3f3-03ba-0de0-052f-7cc6c843b7f9.png)

##  Programmer Namingを選択してください。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/a2d3b82a-505e-e67e-e113-1d7fe83e8e91.png)

## Assembly Optionsは特に変更ありません

## Type Options にてEVMC4U関係のTypeを追加してください
※これは一例です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/79154660-a4ac-41e1-6968-964cf516a159.png)

## 完了

# 作例
## MIDI CC値を取り出す
直接取り出せます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/0a035c3a-5e5f-dfa9-fa6c-22b4fddf1771.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/91cffe13-ed4c-ac5e-3482-2babb12b16d6.png)


## キャリブレーション状態を取り出す
直接取り出せます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/7c01c6b0-fafd-3ec6-16cb-13d8306a5f50.png)


## トラッカーの姿勢をTransformにセットする
直接取り出せますが、少し工夫が必要です。
どのデバイスが、どの位置に格納されるかは一定ではないため、リストをスキャンして、予め設定したシリアル番号に一致する姿勢にセットしています。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/fcd50e75-f575-a318-7ae4-4ae89d380851.png)


## コントローラー情報を取り出す
以下の補助スクリプトを導入してください。
https://github.com/gpsnmeajp/EVMC4U-Bolt-Bridge/blob/main/InputReceiverForBolt.cs

適当なオブジェクトにアタッチし、ReceiverにEVMC4UのInputReceiverのGameObjectを登録します。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/9cfc8e88-3732-9dec-3059-3129e52a5d43.png)

Boltに登録し、認識させます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/1aa32927-5bfa-a7a9-468e-6b0f44947375.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/7b96fbf8-8d37-b4ae-a114-4e41c516ba13.png)

そして以下のように組むと、スティック操作でオブジェクトが動きます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/636d2eb8-c201-bff8-861b-e925afff303f.png)

<blockquote class="twitter-tweet"><p lang="und" dir="ltr"><a href="https://twitter.com/hashtag/EVMC4U?src=hash&amp;ref_src=twsrc%5Etfw">#EVMC4U</a> <a href="https://t.co/PHJCjcHXKQ">pic.twitter.com/PHJCjcHXKQ</a></p>&mdash; Segmentation Fault (@Seg_Faul) <a href="https://twitter.com/Seg_Faul/status/1325282305804435456?ref_src=twsrc%5Etfw">November 8, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

入力状態は以下のようにInspectorから確認できます。
執筆時点のバーチャルモーションキャプチャーの動作に合わせて作成しています。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/1193b086-d3db-2d1a-5ad6-e97edf96a018.png)

