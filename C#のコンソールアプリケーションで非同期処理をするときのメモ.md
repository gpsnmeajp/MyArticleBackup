・コンパイラ > Asyncなメソッドから呼び出せ
・解説サイト > Taskは使うな
・現実 > やってみたら即終了するじゃねぇか！

なお、例外の際デバッガの行表示はMainメソッドに戻ってきてしまう。
呼び出し履歴や、例外の行番号を確認すること

使い方

```C#
using System;
using System.IO;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApplication2
{
    class Program
    {
        static void Main(string[] args)
        {
            //コンソールアプリケーションからAsyncを呼び出す大元はTaskを使用する
            Task task = MainAsync();
            //終了を待つ
            task.Wait();
        }

        //AsyncなMain。ここでは非同期処理をawaitを使って同期的処理のように扱うことができる
        static async Task MainAsync()
        {
            await xxxxxxxxAsync();
        }
    }
}
```

#戻り値が欲しい場合
```C#
using System;
using System.IO;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApplication2
{
    class Program
    {
        static int Main(string[] args)
        {
            //コンソールアプリケーションからAsyncを呼び出す大本はTaskを使用する;
            return MainAsync().Result;
        }

        //AsyncなMain
        private static async Task<int> MainAsync()
        {
            await xxxxxxxxAsync();
            return 0;
        }
   }
}
```

#参考文献
Taskを極めろ！async/await完全攻略
https://qiita.com/acple@github/items/8f63aacb13de9954c5da

できる！C#で非同期処理(Taskとasync-await)
https://www.kekyo.net/2016/12/06/6186
