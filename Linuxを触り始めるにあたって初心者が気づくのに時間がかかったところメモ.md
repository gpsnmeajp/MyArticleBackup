#はじめに
最近、Linuxを本格的に勉強したくなったが、Windowsでのコマンド操作には慣れている自分が
Linuxを学ぶに当たってちょっと気づいたことなどを追記していこうと思う。

#DOS環境との違い
Windowsから、DOSに降り遊ぶのが好きな人だったので、考え方としてここがかなり参考になった。
[Linux Vs. MS-DOS (Yes, Seriously) - PCMech](https://www.pcmech.com/article/linux-vs-ms-dos-yes-seriously/)

#バックグラウンドプロセスの扱い方
以下のサイトが一番参考になった。特に「Important Commands」がまとまっていて便利。
[Multitasking from the Linux Command Line + Process Prioritization :: Chris Jean]( https://chrisjean.com/multitasking-from-the-linux-command-line-plus-process-prioritization/)

Ctrl + Zで後ろに下げたプロセスをfgで引っ張り出せることは知っていたが、
Ctrl + Zと、fg、bg、&、ジョブ番号の対応関係が頭のなかでばらばらになっていた。

実機で作業していると仮想コンソールの切り替えを使いがちだが、SSHで作業するときなどはこっちのほうが便利かもしれない。

#仮想コンソールの操作
Ctrl + Zで一時停止
Ctrl + CでBreak
Tabでコマンド補完
↑↓でコマンド履歴
は当たり前なので知っているが、

Shift + PageUp
Shift + PageDown
で仮想コンソール上の表示履歴をたどることができるのを知らなかった。
いっつも| moreを使っていた

他にも、
Ctrl + Rでコマンド履歴の検索
Ctrl + Aで行頭
Ctrl + Eで行末
Ctrl + uで行頭まで削除
Ctrl + kで行末まで削除
Ctrl + wで単語単位で削除
Ctrl + lで画面クリア (clearコマンドと同じ)

[作業が爆速になるターミナルのショートカットキーまとめbacchi.me]( https://bacchi.me/linux/terminal-tips/)
[＠IT：コンソールで画面をバックスクロールするには]( http://www.atmarkit.co.jp/flinux/rensai/linuxtips/606backscr.html)

#Screenコマンドめっちゃ便利
今までの苦労は何だったんだ...
やっぱりSSH経由で操作したほうが色々楽なのかなぁ、と思っていたら、これがあるとは。
シリアルポートに接続するアプリケーションだとばかり思っていた。

[Linux screenコマンド使い方 - Qiita](https://qiita.com/hnishi/items/3190f2901f88e2594a5f)

#素に近いLinux環境を触ってみる
ArchLinuxがwikiも環境も勉強になる。
[インストールガイド - ArchWiki]( https://wiki.archlinux.jp/index.php/%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%82%AC%E3%82%A4%E3%83%89)

MS-DOS環境の構築はやったことがあったので、それに近い、コマンドベースでの環境構築の仕方を知りたかった。
Ubuntuとかは「何も知らなくても使える」のが便利なのだが、表面をなでているだけな感じが強かったので、
Archのセットアップは「自分がやっている感」があり、とてもやりやすい。

インストールガイドとコマンドヘルプを参考にしていれば概ね扱えるが、それでも最初詰まるので、
(主に、パーティション操作と、grubのインストール)

以下のようなコマンド例を参考にすると良い。

[Arch Linuxで最も早くサーバー環境を立ち上げるまでの手順 : Arch Linuxのインストール](https://qiita.com/syui/items/7b2ccd01da14fd8f3209#arch-linux%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB)
[Arch Linuxを簡単インストール – 「楽しいさくらのクラウド」(17) | さくらのナレッジ](https://knowledge.sakura.ad.jp/6443/)

個人的に、パーティション操作はcfiskが一番分かりやすかった。

ちなみに、当たってくだけてで3回ほどやり直した。(ブートローダーぶっ壊したりなんだり)
初回はVirtual Boxでやるのが一番良いと思う。

#...
後々追記するかもしれません。
