あくまで個人的メモを整形しただけなので、参考になるかは不明です。

#デバッグにはとりあえずLog
Log.d("識別名","メッセージ")
Log.d("識別名","メッセージ",例外オブジェクト)


#XML Oncilck
ボタンのクリックといった基礎的なものは，わざわざプログラム中で定義しなくとも，
XML(GUIデザイナ)上で設定できる．

...が，うまくいかないのはなぜ？

#Activity Lifecycle
##onCreate()
初回起動時・アプリ再起動時に呼び出し
画面等の初期化の準備をする

##onResume()
画面初期化時に呼び出し
データの呼び出しをする
キー，データの組み合わせを保存したい場合は，Preference
http://techbooster.jpn.org/andriod/application/468/
(イベントリスナーの書き方が参考になる)

まとまったデータを保存したい場合は，ローカルファイル
データベース等もあるらしい

##onPause()
画面消滅前に呼び出し
データの保存をする

##intent
明示的インテント(送信先の決まっているメッセージ)
画面の切り替えや，機能の呼び出し

暗黙的インテント(送信先の決まっていないメッセージ)
ブラウザの立ち上げなど？
http://qiita.com/ymotongpoo/items/d8a054f6fc93d069cb37

#バックグラウンド
##Service
裏で延々走る操作に利用
いろいろ？

##intentService
アプリケーションの動作に関係なく別スレッドで動作する
UIとは独立して動作する．
コールバックを受けるContextとIntentServiceのContextは完全に別物
ファイルのダウンロードなど？
完全に非同期にしたいとき便利？

##Asynctask
UIをフリーズさせずに処理を行う．
重い処理など．UIに逐次反映しやすい．プログレスバーを表示して行いたい場合などに最適．
ActivityContextに紐づく．
多くの場合コールバックはActivityContextと共にその寿命を迎えるので、ActivityContextが生きているかどうかに注意しなければならなくなる．
UIが消えると動作が強制終了したりするんだろうか？
同期化を頑張ってくれるらしい．同期的に非同期したいときに便利？

##違い
Activity は「画面を持つアプリケーションのコンポーネント」であり、
その有用性はユーザーと対話をしている間（onResume()～onPause()）にあります。
ユーザーとの対話が終われば破棄されやすくなります。

Service は「画面を持たないアプリケーションのコンポーネント」であり、
startService() で開始された場合には、Service 自身が処理を完了したときに stopSelf() を呼び出すか、
Service を呼び出したコンポーネントが処理を中断させるために、stopService() を呼び出すまで生存します。

Application は全てのコンポーネントを統括する存在です。
「設定＞アプリケーション＞強制停止」でApplication プロセスがキルされると、
その子コンポーネントである Activity や Service は同時に終了します。

#他のアプリとの連携
バインドされたサービス
バインドされたサービスは、クライアントサーバー インターフェースにおけるサーバーになります。
バインドされたサービスでは、コンポーネント（アクティビティなど）をサービスにバインドしたり、
送信を要求したり、受信を応答したり、さらにはプロセス間通信（IPC）を実行したりできるようにします。
通常、バインドされたサービスは他のアプリケーション コンポーネントを提供している間だけ機能し、
バックグラウンドで無期限に実行されることはありません。
　 
　 
#マルチウィンドウの場合
onPauseだが，画面は表示され操作できるというややこしい状況になっている．
この場合は，onStopで検出することになる．　 
　
そもそも，アプリ画面が消えたのを検出するのはonStop．
onPauseはダイアログやポップアップなどで阻害された場合．ただし，onStopが呼び出される保証はない．
onPauseは保証されている
　 
+細かいデータの書き込みはonPause
+大容量のデータの書き込みはonStop
+リソースの開放はonStop
 
#リンク集

アクティビティのライフサイクル
http://andante.in/i/android%E3%82%A2%E3%83%97%E3%83%AAtips/activity%E3%81%AE%E3%83%A9%E3%82%A4%E3%83%95%E3%82%B5%E3%82%A4%E3%82%AF%E3%83%AB%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%E8%80%83%E3%81%88%E3%82%8B/

キーボード表示時にActivityを伸縮したい（フルスクリーンに注意）
タイトルバーの消し方
http://d.hatena.ne.jp/nkawamura/20130824/1377328515

TIMERを使って定期実行する
(複数ボタンのハンドリング，UI Threadへの投げ方などの参考にもなる)
http://techbooster.org/android/application/934/

複数ボタンをメソッド分けて実装する
http://qiita.com/t2low/items/8ac683c7ebf4b6dd1b41

android textview 装飾
https://www.google.co.jp/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=android+textview+%E8%A3%85%E9%A3%BE&*

android textview スクロール
https://www.google.co.jp/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=android+textview+%E3%82%B9%E3%82%AF%E3%83%AD%E3%83%BC%E3%83%AB&*
