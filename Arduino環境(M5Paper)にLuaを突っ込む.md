# 概要
Arduino環境で開発できるメモリたっぷりなマイコン(例えばESP32)に手軽にLua突っ込んで俺環境を構築したいときに見るもの。

すでに有志がLua環境なりMicroPythonなり用意しているのは知っているが、独自機能つけたり、組み込み言語のあれこれを習得したくてやっている。
(LuaはFlashAirで使い慣れていたのもあって、使いたい)

とりあえず筆者はM5Paperのためにやっている。

# 動くようにする手順
## Luaのsourceをダウンロードする
https://www.lua.org/

## ビルドできるようにする
Makefileが使えない。
ので、lua-5.4.1/srcの中身を一旦全て、スケッチフォルダ直下に突っ込む。
これで一旦全部がビルド対象になる。

## ビルドエラー対処
lua.c(スタンドアロン), luac.c(コンパイラ)は不要なので消す。
OS系関数が存在しないため、loslib.cを消す。

linit.cで、

```
  {LUA_OSLIBNAME, luaopen_os},
```

を削って、OSライブラリを読み込まないようにする。

## 実行
こんな感じで実行する

```cpp:lua.ino
#define LUA_32BITS
#include "lua.hpp"

void setup() {
  Serial.begin(115200);
  Serial.println("Start...");

  char buff[256] = "print(\"Hello World\")\nprint(1+1)";
  int error;
  lua_State *L = luaL_newstate();
  luaL_openlibs(L);

  error = luaL_loadbuffer(L, buff, strlen(buff), "line") || lua_pcall(L,0,0,0);
  if(error){
    Serial.println(lua_tostring(L,-1));
    lua_pop(L,1);
  }
  lua_close(L);

  Serial.println("Done");
}

void loop() {

}
```

buffに突っ込まれているのが、Luaコードである。

```lua:buff
print("Hello World")
print(1+1)
```

# 実行結果
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/92a50e2f-155b-abe2-a876-5ac0f86ee9c4.png)

ちなみにこれでこのくらい

```
最大6553600バイトのフラッシュメモリのうち、スケッチが407705バイト（6%）を使っています。
最大4521984バイトのRAMのうち、グローバル変数が15428バイト（0%）を使っていて、ローカル変数で4506556バイト使うことができます。
```

# 感想
Luaってすげえ。移植性がすごい

# 補足
さすがにスケッチフォルダが散らかって嫌ですよね。

Document/libraries/lua-5.4.1というフォルダを掘って、そこに上記の変更を加えたlua-5.4.1/srcの中身を突っ込んでおくと、Arduinoがライブラリとして認識するのでビルド対象となり、スケッチと同じフォルダに入れなくて良くなります。

要るのかよくわかりませんが、library.propertiesは以下です。

```library.properties
name=lua-5.4.1
version=5.4.1
author=
maintainer=
sentence=
paragraph=
category=
url=
architectures=
includes=lua.hpp
depends=
```

# 参考文献
Programming in Lua プログラミング言語Lua公式解説書
http://asciimw.jp/search/mode/item/cd/A0904200
