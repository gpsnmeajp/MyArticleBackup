#はじめに
Visual Studio 2015のC++でHello Worldする手順を丁寧に説明します。
画面を追えばできるはずです。
ただし、インストールとアクティベーションの工程の説明は省きます。

#VisualStudioを起動します。
スタートメニューから起動してください

#プロジェクトの新規作成
ファイル→新規作成→プロジェクト　をクリック
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/7f2b3a2d-32a4-0c81-7218-288d61bafe86.png)

#Win32コンソールアプリケーション
左のテンプレートより、「Visual C++」内の「Win32」をクリックし、
中央で「Win32 コンソールアプリケーション」をクリックして選択。

画面下部の「名前」にプロジェクトの名前を
「場所」にプロジェクトを作りたい場所を選択してください。(右側の「参照(B)...」を使用してください)
終わったらOKをクリックしてください。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/3be5e606-a216-31fc-7f34-cfc152a0ffd2.png)

#Win32アプリケーションウィザード
次へをクリック。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/36414f48-ab0e-62e3-3d62-a03ff144b4f9.png)

「追加のオプション」の「空のプロジェクト」にチェックを入れ、
「Security Development Lifecycle(SDL)チェック」のチェックを外して、
「完了」をクリック。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/180dc064-45cb-596f-3989-93a8308687c3.png)

#ソースファイルの追加
ソリューションエクスプローラ(右側あるいは左側)の「ソースファイル」を右クリックし、
「追加(D)」→「新しい項目(W)...」をクリック。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/9bb020ef-ba53-b752-d90c-b342210ca259.png)

「C++ファイル(.cpp)」が選択されているのを確認したら、「名前」に適当な名前を入力するか、あるいはそのまま、
「追加」をクリック
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/decd0d9c-31b7-49b4-9ecf-228b6c6d09f3.png)

#Hello World
あとは自動で開くので(開かない場合は、ソリューションエクスプローラ内の先ほど作成したファイルをダブルクリック)

Hello Worldしましょう。
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/a95bc928-bcaf-7495-3e43-af3da469f5bd.png)

```cpp:Source.cpp
#include <stdio.h>
int main()
{
	printf("Hello World\n");
	return 0;
}
```

#ビルド
終わったら、Ctrl + F5
あるいは、画面上方の「デバッグ」→「デバッグ無しで実行」をクリックすると
ビルドの後、実行されます。

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/e2c6794b-53dd-b0c0-f809-61d65eeb2523.png)

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/ac50e747-c592-bc7e-30b9-983f36952212.png)
