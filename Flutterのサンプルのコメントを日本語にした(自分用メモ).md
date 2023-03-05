Flutterのサンプルのコメントを日本語にした(自分用メモ)

```Dart:main.dart
import 'package:flutter/material.dart';

//flutter runで実行

//エントリポイント(起動したときの定義)
void main() => runApp(MyApp());

//アプリ自体の定義(状態を持たないStatelessWidget)
class MyApp extends StatelessWidget {
  //描画・UI構築処理
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      //アプリ自体のタイトル
      title: 'Flutter Demo',
      //タイトルバーのテーマ
      theme: ThemeData(
        primarySwatch: Colors.blue, //色
      ),
      //画面を表す要素を内包する
      home: MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

//画面の定義1(状態を持つStatefulWidget)
class MyHomePage extends StatefulWidget {
  //コンストラクタの定義
  MyHomePage({Key key, this.title}) : super(key: key);
  //状態の定義(ここではタイトル)
  final String title;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

//状態の定義
class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  //ボタンをタップされた時のイベント定義
  void _incrementCounter() {
    //状態の変化を通知するにはsetStateを使う。これによりbuildが再コールされる
    setState(() {
      _counter++;
    });
  }

  //描画・UI構築処理(setStateが呼び出されるたびに呼ばれる)
  //フレームワークで管理されているため、更新が必要なものだけ更新される
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        // MyHomePage objectから値をとってきてタイトルにセット
        title: Text(widget.title),
      ),
      body: Center(
        //レイアウト構築(1つしかないので中央に置かれる)
        child: Column(
          //垂直に並べる
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            //テキストを配置
            Text(
              'You have pushed the button this many times:',
            ),
            //テキストを配置($変数で動的に表示)
            Text(
              '$_counter',
              //スタイルを適用
              style: Theme.of(context).textTheme.display1,
            ),
          ],
        ),
      ),
      //フローティングアクションボタンを追加
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ), // This trailing comma makes auto-formatting nicer for build methods.
    );
  }
}
```

#参考文献
Flutter でサンプルプログラムを理解する
https://qiita.com/kurararara/items/73deac522ad9fea36abc
