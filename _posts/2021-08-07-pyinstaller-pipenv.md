---
layout: post
title: pyinstaller打包肥大
date: 2021-08-07
tags: pyinstaller
---

pyinstaller打包肥大

最下面那個 code 打包變成 27M 有點肥.....

用些方法變成 13M 還可接後

建立虛擬環境 (如裝過看下面有寫如何移除舊環境)
```
pipenv install
```

進去 虛擬環境 安裝所需模块 之後編譯
```
pipenv shell
pip install pyinstaller
pip install oss2
pip install pyqrcode
pyinstaller -Fw 9.py
```

如裝過 pipenv 要先移除舊環境
```
pipenv --rm
```

進去看到是 空的
```
D:\python>pipenv --rm
Removing virtualenv (C:\Users\echochio\.virtualenvs\python-oyZFM5cL)...

D:\python>pipenv shell
Creating a virtualenv for this project...
Pipfile: D:\python\Pipfile
Using C:/Python/python.exe (3.7.4) to create virtualenv...
[====] Creating virtual environment...created virtual environment CPython3.7.4.final.0-32 in 805ms

  creator CPython3Windows(dest=C:\Users\chio\.virtualenvs\python-oyZFM5cL, clear=False, global=False)

  seeder FromAppData(download=False, pip=latest, setuptools=latest, wheel=latest, via=copy, app_data_dir=C:\Users\echochio\AppData\Local\pypa\virtualenv\seed-app-data\v1.0.1)

  activators BashActivator,BatchActivator,FishActivator,PowerShellActivator,PythonActivator,XonshActivator

Successfully created virtual environment!
Virtualenv location: C:\Users\echochio\.virtualenvs\python-oyZFM5cL
Launching subshell in virtual environment...
Microsoft Windows [版本 10.0.19043.1110]
(c) Microsoft Corporation. 著作權所有，並保留一切權利。

(python-oyZFM5cL) D:\python>pip list
Package    Version
---------- -------
pip        20.1.1
setuptools 47.1.1
wheel      0.34.2

(python-oyZFM5cL) D:\python>
```

code 如下
```
#!/usr/bin/env python 
# -*- coding:utf-8 -*-
# 用途：使用GUI上传阿里云OSS
import tkinter as tk
import tkinter.ttk as ttk
from tkinter import messagebox  # 弹窗
# from tkinter import filedialog  # 选择单独文件
from tkinter.filedialog import askdirectory # 选择文件夹
import oss2,os,sys,time  #引入zipfile解压
import requests
import pyqrcode
from tkinter import *
window = tk.Tk()
window.title("將文件上傳到国内阿里雲OSS")   # 窗体的标题
def generate_QR():
    mmm = 'itms-services://?action=download-manifest&url=https://raw.githubusercontent.com/HHDDXX/XXXXXSSSS/master/test.plist'
    global qr,img
    qr = pyqrcode.create(mmm)
    img = BitmapImage(data = qr.xbm(scale=5))
    img_lbl.config(image = img)
def confirm():
    result = messagebox.askokcancel("請確認", '''本地文件是：%s\n對應OSS路徑是：%s''' % (filename,region1.get())) # 前面是弹窗主题，后面是弹窗内容
    Des_path = region1.get()

    if result is True:
        ak = "XXXXXXXXXXXXXXXXXXXXX"
        sk = "XXXXXXXXXXXXXXXXXXXXXXX"   # 秘钥
        auth = oss2.Auth(ak, sk)   # 鉴权
        ossBucket = oss2.Bucket(auth, 'http://oss-XXXXXXXXXX.aliyuncs.com',Des_path)   # 定义ossBucket

        # 文件夹上传
        def uploadFile(file):
            out=""
            
            #remoteName = file[file.rfind('/')+1:len(file)]
            remoteName = 'test.ipa'
            out = out + "上傳檔案 : "+ file + "\n目的檔案 : "+remoteName+"\n"
            with open(oss2.to_unicode(file), 'rb') as f:
                result = ossBucket.put_object(remoteName, f)
            out = out +'http 狀態 : '+str(result.status)+"\n"
            return (out)

        out = uploadFile(filename)   # 开始上传
        out = out + "\n目標文件夾裡所有文件上傳完畢！"
        result = messagebox.askokcancel("處理結果",'''%s\n''' % (out))

def chooseZip():
    global filename
    filename = tk.filedialog.askopenfilename()    # 选择单独的文件
    if filename != '':
        lb.config(text="您選擇的文件是：" + filename)
    else:
        lb.config(text="您沒有選擇任何文件")

lb = tk.Label(window, text='')
lb.grid(row=0, column=1, sticky=tk.W, padx=10, pady=5)
dout = "選擇要上傳的文件"
btn = tk.Button(window, text=dout, command=chooseZip)
btn.grid(row=0, column=0, sticky=tk.E, padx=10, pady=5)

lb2 = tk.Label(window, text='請選擇上傳地址')
lb2.grid(row=2, column=0, sticky=tk.W, padx=10, pady=5)
number = tk.StringVar()
region1 = ttk.Combobox(window,width=35,textvariable=number,state='readonly')    #下拉列表设置成为只读模式
region1['values'] = ('Download')   #下拉列表里面具体的元素
region1.grid(row=2,column=1)
region1.current(0)
tk.Button(window, text='顯示 QR code', width=10, command=generate_QR).grid(row=3, column=2, sticky=tk.W, padx=10, pady=5)
tk.Button(window, text='上傳', width=10, command=confirm).grid(row=3, column=0, sticky=tk.W, padx=10, pady=5)
tk.Button(window, text='退出', width=10, command=window.quit).grid(row=3, column=1, sticky=tk.E, padx=10, pady=5)
img_lbl = tk.Label(window,text='')
img_lbl.grid()
tk.mainloop()
```
