---
layout: post
title: python pyinstaller 包 py 以外的檔案
date: 2021-08-01
tags: pyinstaller
---

想說用pyinstaller 把 jpg 包在 exe 內
```
pyinstaller --onefile --clean --add-data "QRcode.jpg;." put.py --noconsole
```

原來要加這個不然路徑會不正確
```
import os
def resource_path(relative_path):
    """ Get absolute path to resource, works for dev and for PyInstaller """
    try:
        # PyInstaller creates a temp folder and stores path in _MEIPASS
        base_path = sys._MEIPASS
    except Exception:
        base_path = os.path.abspath(".")

    return os.path.join(base_path, relative_path)
```

把以前程式改一改 ......把IPA 上傳裡面還有 QRcode 方便多了
```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
import tkinter as tk
import tkinter.ttk as ttk
from tkinter import messagebox  # 弹窗
from tkinter.filedialog import askdirectory # 选择文件夹
import os,sys,time  #引入zipfile解压
from paramiko import SSHClient
from scp import SCPClient
import paramiko
import PIL.ImageTk
import PIL.Image
import os


def resource_path(relative_path):
    """ Get absolute path to resource, works for dev and for PyInstaller """
    try:
        # PyInstaller creates a temp folder and stores path in _MEIPASS
        base_path = sys._MEIPASS
    except Exception:
        base_path = os.path.abspath(".")

    return os.path.join(base_path, relative_path)

window = tk.Tk()
window.title("將文件上傳到172.16.0.99") # 窗体的标题
window.resizable(0, 0)

def confirm():
    result = messagebox.askokcancel("請確認", '''本地文件是：%s\n路徑是：%s''' % (filename,region1.get()))
    Des_path = region1.get()

    if result is True:
        # 文件夹上传
        def uploadFile(file):
            ssh = SSHClient()
            ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
            ssh.connect(hostname='172.16.0.99', 
                        port = '22',
                        username='root',
                        password='echochio',
                        compress=True)
            scp = SCPClient(ssh.get_transport())
            scp.put(file, '/var/www/html/app.ipa')
            out=""
            remoteName = file[file.rfind('/')+1:len(file)]
            out = out + "上傳檔案 : "+ file + "\n目的檔案 : "+Des_path+"\n"
            return (out)

        out = uploadFile(filename) # 开始上传
        out = out + "\n文件上傳完畢！"
        result = messagebox.askokcancel("處理結果",'''%s\n''' % (out))

def chooseZip():
    global filename
    filename = tk.filedialog.askopenfilename()    # 选择单独的文件
    if filename != '':
        lb.config(text="您選擇的文件是：" + filename)
    else:
        lb.config(text="您沒有選擇任何文件")

def confirmstatus(i):
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh.connect(hostname='172.16.0.99', port = '22',
                username='root',
                password='echochio',
                compress=True)
    ssh_stdin, ssh_stdout, ssh_stderr = ssh.exec_command('systemctl '+i+' nginx')
    out = '完成'
    result = messagebox.askokcancel("處理nginx "+i,'''%s\n''' % (out))
def start():
    confirmstatus('start')
def stop():
    confirmstatus('stop')

lb = tk.Label(window, text='')
lb.grid(row=0, column=0, sticky=tk.W, padx=10, pady=5)
dout = "選擇要上傳的文件"
btn = tk.Button(window, text=dout, command=chooseZip)
btn.grid(row=0, column=0, sticky=tk.E, padx=10, pady=5)
lb2 = tk.Label(window, text='上傳地址檔案')
lb2.grid(row=2, column=0, sticky=tk.W, padx=10, pady=5)
number = tk.StringVar()
region1 = ttk.Combobox(window,width=60,textvariable=number,state='readonly')    #下拉列表设置成为只读模式
region1['values'] = ('/var/www/html/app.ipa')   #下拉列表里面具体的元素
region1.grid(row=2,column=1)
region1.current(0)
#nginx 停止與啟動
tk.Button(window, text='nginx start', width=10, command=start).grid(row=3, column=0, sticky=tk.W, padx=10, pady=10)
tk.Button(window, text='nginx stop', width=10, command=stop).grid(row=3, column=1, sticky=tk.W, padx=10, pady=10)
#上傳與退出
tk.Button(window, text='上傳', width=10, command=confirm).grid(row=4, column=0, sticky=tk.W, padx=10, pady=5)
tk.Button(window, text='退出', width=10, command=window.quit).grid(row=4, column=1, sticky=tk.E, padx=10, pady=5)
#顯示 QR code
bg_img = PIL.ImageTk.PhotoImage(PIL.Image.open(resource_path('QRcode.jpg')))
bg = ttk.Label(window, image=bg_img)
bg.grid(column=0, row=100)
tk.mainloop()
```
