ちょっと迷いましたが、難しくはありませんでした。
調べるほうが困った。

Visual Studio 2015をC++を含めてインストールしてある前提です。

# Luaのソースコードをダウンロードする
公式サイトの「Source」からダウンロードしましょう。
https://www.lua.org/download.html

適当なディレクトリに展開しておきます。

# コンパイルしてstatic libを生成する
いろいろな方法を試しましたが、これが一番楽でした。

1. スタートメニューから「開発者コマンド プロンプト for VS2015」を起動
2. Luaソースコードのsrcディレクトリに移動
3. 以下のコマンドを実行

```bat
cl /MD /O2 /c /DLUA_BUILD_AS_DLL *.c
ren lua.obj lua.o
ren luac.obj luac.o
link /DLL /IMPLIB:lua5.3.0.lib /OUT:lua5.3.0.dll *.obj 
link /OUT:lua.exe lua.o lua5.3.0.lib 
lib /OUT:lua5.3.0-static.lib *.obj
link /OUT:luac.exe luac.o lua5.3.0-static.lib
```

フォルダ内に大量のobjファイルと、lib、static.lib、luac.lib、lua.exe、luac.exeが生成されます。

参考: How to compile Lua 5.3.0 for Windows
https://blog.spreendigital.de/2015/01/16/how-to-compile-lua-5-3-0-for-windows/

#プログラムに組み込む
1. VC2015を立ち上げる
2. ファイル→新規作成→新しいプロジェクト
3. Win32コンソールアプリケーションを選択して次へ
4. 「空のプロジェクト」にチェックを入れ、「SDLチェック」のチェックを外して完了
5. 「ソースファイル」を右クリックして、「追加」→「新しい項目」
6. C++ファイルを適当に生成(ここではSource.cpp)
7. 以下の記述をする

```C++:Source.cpp
#include <stdio.h>
#include <stdlib.h>
#include "lua.hpp"
#pragma comment(lib,"lua5.3.0-static.lib")

int main()
{
	lua_State* L = luaL_newstate();
	luaL_openlibs(L);

	if ( luaL_dostring(L, "print('Hello World'") )
	{
		printf("%s\n", lua_tostring(L, -1));
	}

	lua_close(L);
}

```

その後、「Source.cpp」と同じフォルダに、Luaソースコードのsrcディレクトリから以下のものをコピーする

+ lualib.h
+ luaconf.h
+ lua.hpp
+ lua.h
+ lauxlib.h
+ lua5.3.0-static.lib

ビルドして実行し、Hello Worldが出ればOK

参考: WindowsでVisual StudioでLua
http://tima620.hatenablog.com/entry/2014/03/07/051052

#メモ

```
'MSVCRT' は他のライブラリの使用と競合しています。/NODEFAULTLIB:library を使用してください。
```

が出た場合、とりあえずReleaseビルドにすると治ることがある。
(Debugビルドでlibを生成していないためと思われる、が、一度Releaseビルドにすると、Debugビルドに切り替えてもエラーが出なくなる。)

#その他
Luaの実行環境を作りたい人はこっち
http://qiita.com/gpsnmeajp/items/486a77d3c3e92e61ed7c

#リンク

参考: 第 5 回: Lua を組み込み用の言語として利用する方法 (関数編)
http://www.ie.u-ryukyu.ac.jp/~e085739/lua.hajime.5.html

