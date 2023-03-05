# はじめに
とりあえずC言語でGUIを扱いたい人向け
ほぼ公式ドキュメントそのままですが、微妙に迷う可能性があったので記述

Raspberry Pi (初期のModel B)で動く軽量なGUIアプリを作りたいが、
Linuxデスクトップ環境より慣れたWindowsデスクトップ環境で扱いたかったため、試したきｒ

# MSYS2のインストール
https://www.msys2.org/

書いてある通りにインストールする。

# GTK4を導入

インストール直後、MSYS2のウィンドウが動いているはずである。

以下を打ち込む。(pacmanは、UbuntuやDebianでいうところのapt-getに相当する)

```bash
pacman -S mingw-w64-x86_64-gtk4
```

次に以下。実行後、どのグループを導入するかを聞かれるが、そのままEnterを押せば良い。
途中一部でダウンロード失敗することがあるが、何度か繰り返し入れると進む。

```bash
pacman -S mingw-w64-x86_64-toolchain base-devel
```

参考

https://www.gtk.org/docs/installations/windows/

# ソースをビルドする

MSYS2のウィンドウを閉じて、スタートメニューから **MSYS2 MinGW x64** を起動する。
※ここを開き直さずにMSYS2のまま実行すると、pkg-configがエラーになる

エクスプローラから以下のフォルダを開き、ユーザー名のフォルダを開く。
(Linuxでの/homeの位置がMSYSではここになる)

```
C:\msys64\home\
```

お試し用に以下をmain.cとして保存する

```c:main.c
static void print_hello(GtkWidget *widget, gpointer data){
	g_print("OK\n");
}

static void activate (GtkApplication* app, gpointer user_data)
{
	GtkWidget *window = gtk_application_window_new(app);
	gtk_window_set_title(GTK_WINDOW(window), "Window");
	gtk_window_set_default_size(GTK_WINDOW(window),640,480);

	GtkWidget *fixed = gtk_fixed_new();
	gtk_window_set_child(GTK_WINDOW(window),fixed);

	GtkWidget *button = gtk_button_new_with_label("Hello");
	gtk_widget_set_size_request(button,100,100);
	g_signal_connect(button,"clicked",G_CALLBACK(print_hello),NULL);

	gtk_fixed_put (GTK_FIXED(fixed), button, 100, 200);

	gtk_widget_show(window);
}

int main(int argc, char **argv)
{
	GtkApplication *app = gtk_application_new("org.gtk.example", G_APPLICATION_FLAGS_NONE);
	g_signal_connect (app,"activate", G_CALLBACK(activate),NULL);
	
	int status = g_application_run(G_APPLICATION(app), argc, argv);
	g_object_unref(app);

	return status;
}
```

MSYS2 MinGW x64のウィンドウで以下を打ち込む

```bash
gcc $( pkg-config --cflags gtk4 ) -o main.out main.c $( pkg-config --libs gtk4 )
```

正常にビルドが終了したら実行する

```bash
./main.out
```

正常にウィンドウが立ち上がればOK

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/191114/1c9c3f0f-c3d8-7947-f120-bac0bc313d39.png)

参考

https://docs.gtk.org/gtk4/getting_started.html


# おまけ: 別スレッドなどからのイベント処理
通常、メインループはGTK4に握られることになる。
そのため、GUIと関係ないイベント(通信など)を拾うにはどうすればよいか。

簡単なのは、別スレッドでイベントが発生したタイミングで、g_idle_addをコールする。
(なお、最低優先度で処理される。優先度を上げたい場合はg_idle_add_fullで優先度を指定する)

```c
//コールバック
static gboolean idlefunc(gpointer user_data){
	g_print("idle\n");
	return false; //falseの場合は単発、trueの場合は繰り返し呼ばれる

    //常にtrueにした場合、かなりCPUを食うので注意。
    //連続した処理が必要な場合を除いて基本的にfalseで使用すべき
}


//GTK4の関数は基本的に別スレッドからのコールが禁止されているが、
//g_idle_add含む一部の関数はスレッドセーフになっている。
//これを呼び出すことで、低い優先度のタスクとしてGTK4のキューに関数が積まれる。タイミングが来るとコールバックが呼ばれる。
g_idle_add(G_SOURCE_FUNC(idlefunc),NULL);
```

# おまけ: タイマー(定期処理、定期画面更新)
定期的なタイマー処理をしたい場合はg_timeout_addが便利。

```c
static gboolean timerfunc(gpointer user_data){
	g_print("timer\n");
	return true; //falseの場合は単発、trueの場合は繰り返し500msごとに呼ばれる
}

int main(int argc, char **argv)
{
    ~~~略~~~
	guint tag = g_timeout_add(500,timerfunc,NULL); //初回の呼び出しを500ms後に予約する
	//g_source_remove(tag); //removeで呼び出しを止めることもできる
    ~~~略~~~
}


```

参考

https://qiita.com/seshimaru/items/640749092864aba8f83c

https://irasya.hatenadiary.org/entry/20100601/1275400725

https://docs.gtk.org/glib/func.idle_add.html

https://docs.gtk.org/glib/func.timeout_add.html

http://web.archive.org/web/20100711002310/http://www.gnome.gr.jp/docs/gtk+faq.20040114.html#AEN500

https://www.comp.sd.keio.ac.jp/jp/lectures/sdex/event
