PythonでQRコードを生成して直接Tkinterに表示する。

ファイルに保存せずに直接表示するには、PIL.ImageTk.PhotoImageが鍵でした。
ただし、Tkの初期化終了後じゃないと実行できないので注意。

```
pip install pillow qrcode colorama
```

```Python
#!/usr/bin/env python
# -*- coding: utf8 -*-

import sys
import tkinter as tk
import PIL.ImageTk
import qrcode #qrcodeを起動

msg = "Hello World"
w=800
h=500

def makeQR(s):
    qr = qrcode.QRCode(
        version=None, #QRコードの大きさ(バージョン)。Noneで自動設定(1～22)。数値指定しても必要なら自動で大きくなる
        error_correction=qrcode.constants.ERROR_CORRECT_M, #誤り訂正レベルL,M,Q,H
        box_size=3, #サイズを何倍にするか。1ならdot-by-dot(px)
        border=8,#認識用余白(最低4)
    )
    qr.add_data(s)
    qr.make(fit=True)
    return qr.make_image()

root = tk.Tk()
canvas = tk.Canvas(root,width=w,height=h,bg='white')

f = ('FixedSys, 14')
imgtk = PIL.ImageTk.PhotoImage(makeQR(msg))

canvas.create_image(w/2,h/2,image = imgtk,anchor = tk.CENTER)
canvas.create_text(w/2,50,text = "Header",font=f)
canvas.create_text(w/2,450,text = "Footer",font=f)
canvas.pack()

root.mainloop()
```
![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/e7271f19-a7a1-3547-5722-7e0d00a67e60.png)


以下のように書き換えると、ウィンドウサイズがぴったりになります。

```python
imgtk = PIL.ImageTk.PhotoImage(makeQR(msg))
canvas = tk.Canvas(root,width=imgtk.width(),height=imgtk.height(),bg='white')
canvas.create_image(0,0,image = imgtk,anchor = tk.NW)
canvas.pack()
```

![image.png](https://qiita-image-store.s3.amazonaws.com/0/191114/2ff3faba-4032-67d8-f8c6-1154a2c09b47.png)

#参考
[Pythonの「qrcode」を使ってQRコードを生成する](https://qiita.com/ak1procom/items/c9da5d5f1039adea9704)
[qrcode 5.3](https://pypi.python.org/pypi/qrcode)
[PythonのTkinterを使ってみる](https://qiita.com/nnahito/items/ad1428a30738b3d93762)
[【Python】Tkinterのcanvasを使ってみる](https://qiita.com/nnahito/items/2ab3ad0f3adacc3314e6)
[Python/Tkinter プログラミング講座 イメージとファイルの選択](http://bacspot.dip.jp/virtual_link/www/si.musashi-tech.ac.jp/new_www/Python_IntroTkinter/03/index-2.html)
[Tkinter (Canvas)の内容をPILでキャプチャ](http://boxheadroom.com/2009/05/29/tkinter-canvas%E3%81%AE%E5%86%85%E5%AE%B9%E3%82%92pil%E3%81%A7%E3%82%AD%E3%83%A3%E3%83%97%E3%83%81%E3%83%A3)
