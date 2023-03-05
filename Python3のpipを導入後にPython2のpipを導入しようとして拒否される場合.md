Python3はpipとpip3が使えますが、Python2でpipを導入後にpip3を導入してしまうと
上書きされることがあります。

```
$ sudo python get-pip.py
Requirement already up-to-date: pip in /usr/local/lib/python2.7/dist-packages
```

--ignore-installedをつけると無理やり入る。

```
$ sudo python get-pip.py --ignore-installed
Collecting pip
  Using cached pip-9.0.1-py2.py3-none-any.whl
Installing collected packages: pip
Successfully installed pip-9.0.1
```

```
$ pip -V
pip 9.0.1 from /usr/local/lib/python2.7/dist-packages (python 2.7)
$ pip3 -V
pip 9.0.1 from /usr/local/lib/python3.5/dist-packages (python 3.5)

```
