Cythonを使いたい場合はpython-devも入れる。

```
sudo apt-get install python-dev
sudo apt-get install python3-dev
```

でなければ

```
fatal error: Python.h: そのようなファイルやディレクトリはありません
 #include "Python.h"
                    ^
compilation terminated.
```

で止まる

