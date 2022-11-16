---
layout: post
title: WSL 裝 ssh 讓 internet 可以連
date: 2022-11-16
tags: windows wsl wsl2 centos ubuntu ssh
---

先安裝 WSL + systemd 去啟動 sshd

https://echochio.ml/2022/08/wsl-docker-minikube/

去啟動 sshd 與安裝設定就不說了
```
apt install ssh -y
systemctl enable ssh
systemctl start ssh

```

然後去 windows administrator cmd
```
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=22 connectaddress=127.0.0.1 connectport=22
netsh advfirewall firewall add rule name=SSHD dir=in action=allow protocol=TCP localport=22
```

上面那個 advfirewall 是由 RDP 抄出來的.......
```
netsh advfirewall firewall add rule name=RDP dir=in action=allow protocol=TCP localport=3389
```
