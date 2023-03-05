#はじめに
職場でちょっとツール作りたいけど、使う人のことも考えてGUIにしたい、C言語で組みたい、ライセンスとかも考えてgccで作りたい、ライブラリ色々覚えたくない、そんなときにさっと使える手順です

#MinGWのインストール
https://ja.osdn.net/projects/mingw/

左の「download/installer」でインストーラが手に入ります。
実行してインストール。

インストール後のInstallation Manager
左のBasic Setupを選び

+ mingw32-base-bin
+ mingw32-gcc-g++-bin

をクリックしてMark

左のAll Packages→MinGW→MinGW Contributedを選び

+ mingw32-tcl-bin
+ mingw32-tk-bin

をクリックしてMark

左上のInstallation
Apply Changes
でインストールを実行

#MinGWにPATHを通す
システムのPATHにC:\MinGW\binを登録します。

こちらの記事など参考にどうぞ

Windows10で実行ファイルへのパスを通す手順
https://qiita.com/shuhey/items/7ee0d25f14a997c9e285

#ソースを作る
適当なフォルダに以下のように作ります。

C言語のソースが以下です。

```c:main.c
/*
 * These codes are licensed under CC0.
 * http://creativecommons.org/publicdomain/zero/1.0/deed.ja
 */
#include <tcl.h>
#include <tk.h>

//独自コマンドの作成
int proc(ClientData clientData, Tcl_Interp* interp, int argc, const char* argv[])
{
	//argc:引数の数
	//argv[0]:自分自身(コマンド名)
	//argv[1]～:引数

	if ( argc != 1 )
	{
		char *retmsg = "argument error";
		Tcl_SetResult(interp, retmsg, TCL_VOLATILE);
		return TCL_ERROR;
	}
	printf("Call!\n");

	char *retvalue = "0";
	Tcl_SetResult(interp, retvalue, TCL_VOLATILE);
	return TCL_OK;

	//終了コードの種類
	//TCL_OK、TCL_ERROR、 TCL_RETURN、TCL_BREAK、またはTCL_CONTINUE

	//応答の種類
	//TCL_STATIC ... データが静的な領域(static char)にあって、関数を抜けてもその領域が上書きされない場合
	//TCL_VOLATILE ... データが揮発する(auto char)スタック領域にあり、関数を抜けると上書きされる場合
	//TCL_DYNAMIC ... データがAPI Tcl_Allocで確保された領域にある場合
}

int main(int argc, char* argv[])
{
	Tcl_Interp *interp = Tcl_CreateInterp(); //インタプリタを作成
	Tcl_FindExecutable(argv[0]); //初期化前準備
	if ( Tcl_Init(interp) == TCL_ERROR ) //Tclを初期化(スタンダートライブラリinit.tclをロード)
	{
		const char *errmsg = Tcl_GetStringResult(interp); //エラーメッセージの取得
		printf("インタプリタの初期化に失敗: %s\n",errmsg);
		return -1;
	}
	if ( Tk_Init(interp) == TCL_ERROR ) //Tkを初期化(GUIライブラリをロード)
	{
		const char *errmsg = Tcl_GetStringResult(interp); //エラーメッセージの取得
		printf("GUIライブラリの初期化に失敗: %s\n", errmsg);
		return -1;
	}

	//コマンドを作る
	Tcl_CreateCommand(interp,"testproc",proc,NULL,NULL);

	if ( Tcl_EvalFile(interp, "script.tcl") == TCL_ERROR )//インタプリタで実行
	{
		const char *errmsg = Tcl_GetStringResult(interp); //エラーメッセージの取得
		printf("実行失敗: エラー内容:%s\n", errmsg);
		return -1;
	}else{
		const char *errmsg = Tcl_GetStringResult(interp); //エラーメッセージの取得
		printf("実行成功:%s\n", errmsg);
	}

	Tk_MainLoop();//Tkのイベントループへ

	Tcl_DeleteInterp(interp); //インタプリタのお片付け
	return 0;
}
```

TCL言語のソースが以下です。(GUI定義ファイルのようなものと考えてください)

```tcl:script.tcl
button .b1 -text "Hello World" -command {puts "Hello World"}
button .b2 -text "Call proc" -command {testproc}
pack .b1 .b2
```


#コンパイルする
ソースのあるフォルダでコマンドプロンプトを開き、以下を実行します。

```cmd
gcc main.c -o test.exe -ltcl86 -ltk86
```

#実行する
test.exeを実行します。
上のボタンを押すとHello Worldが表示され、
下のボタンを押すとC言語のproc関数が実行されます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/6993cc88-32e0-157d-963c-bdbca33e0ed5.png)

#配布時の注意
一見EXEファイルとTCLファイルで動くように思えますが、実際にはMinGWフォルダ内のファイルに依存しています。
配布するには、以下のようなフォルダ構成にして配布してください。

```cmd
配布フォルダ
├─bin
|  ├─test.exe
|  ├─script.tcl
|  ├─libgcc_s_dw2-1.dll ←C:\MinGW\binにあるファイル
|  ├─tcl86.dll ←C:\MinGW\binにあるファイル
|  └─tk86.dll　←C:\MinGW\binにあるファイル
└─lib
    ├─tcl8.6 ←C:\MinGW\libにあるフォルダ
    └─tk8.6 ←C:\MinGW\libにあるフォルダ
```

Tcl/tkのライセンスにも目を通しておいてください
https://www.tcl.tk/software/tcltk/license.html

#おわりに
Tcl/Tkは、見た目は素朴ですがかなり簡単にGUIが書けるツールキット&言語です。
C言語に組み込む前提で作られているため、このようにかなり簡単に使用できます。

TCL言語自体の使い方については以下などを参照してください。

Tcl/Tk GUI Programming
http://www.nct9.ne.jp/m_hiroi/tcl_tk.html

Tcl/Tkのススメ
https://sites.google.com/site/gpsnmeajp/tcl-tk/tcl-tknosusume

Tcl/tk簡易リファレンス
https://sites.google.com/site/gpsnmeajp/tcl-tk/tcl-tk-jian-yirifarensu

Tcl/TkでWindowsプログラミング
http://web.archive.org/web/20160116031827/http://homepage3.nifty.com/kaku-chan/tcl_tk/index.html

