---
layout: post
title: minecraft java Spigot伺服器 python3
date: 2021-12-20
tags: minecraft python3
---

以下其他能 如 linux 及 MacOS 的 都適用 請自行修改 .... 因為 java 跨平台

windows 可用 GUI .... 請往下找後面 ....

下載JAVA

```
java -version
java version "17.0.1" 2021-10-19 LTS
Java(TM) SE Runtime Environment (build 17.0.1+12-LTS-39)
Java HotSpot(TM) 64-Bit Server VM (build 17.0.1+12-LTS-39, mixed mode, sharing)
```

下載 BuildTools
```
https://hub.spigotmc.org/jenkins/job/BuildTools/lastSuccessfulBuild/artifact/target/BuildTools.jar
```

執行(現在 minecraft 最新是 1.18.1)
```
java -jar BuildTools.jar --rev 1.18.1
```

或是
```
java -jar BuildTools.jar --rev latest
```

裝完顯示
```
Success! Everything completed successfully. Copying final .jar files now.
Copying spigot-1.18-R0.1-SNAPSHOT-bootstrap.jar to C:\spigot1.18\.\spigot-1.18.jar
  - Saved as .\spigot-1.18.jar
```

設定一個啟動檔
```
java -Xms1024M -Xmx2048M -jar spigot-1.18.jar nogui
```

如果 出現 那再點一下 BuildTools 去執行會跑出 server.properties 及 eula.txt

```
[10:38:18] [ServerMain/ERROR]: Failed to load properties from file: server.properties
[10:38:18] [ServerMain/WARN]: Failed to load eula.txt
[10:38:18] [ServerMain/INFO]: You need to agree to the EULA in order to run the server. Go to eula.txt for more info.
```

要去編輯 eula.txt 把內容改 eula=true

再次起動 直到出現 type "help"
```
java -Xms1024M -Xmx2048M -jar spigot-1.18.jar nogui
......
......
[10:41:25] [Server thread/INFO]: Done (33.101s)! For help, type "help"
```

加管理者 admin
```
op admin
```

開啟 minecraft 1.18.1 多人建立 直連伺服器 127.0.0.1 或電腦IP

--------------------------------------------------------------------
windows 下載 GUI https://github.com/0uti/BuildToolsGUI/releases

下到 D:\

直接壓縮 後就下載 .....

後來執行 spigot-1.18.1.jar (直接點就好)

改 eula.txt 把內容改 eula=true

再執行 spigot-1.18.1.jar

-------------------------------------------------------------------------
接下來安裝 python3 模組

去下載 https://dev.bukkit.org/projects/raspberryjuice

我下載到 raspberryjuice-1.11.jar

放到 C:\spigot1.18\plugins

下載 https://www.spigotmc.org/resources/python-plugin-for-mcpi.81664/

我下載到 MCPI_plugin-0.0.3.jar

放到 %AppData%\.minecraft\mods 下面

(下載不到有放一份 https://github.com/echochio-tw/echochio-tw.github.io/tree/master/images 可去找找)


啟動 spigot 其中有
```
[15:21:08] [Server thread/INFO]: [RaspberryJuice] Enabling RaspberryJuice v1.11
[15:21:08] [Server thread/INFO]: [RaspberryJuice] Using port 4711
[15:21:08] [Server thread/INFO]: [RaspberryJuice] Using RELATIVE locations
[15:21:08] [Server thread/INFO]: [RaspberryJuice] Using RIGHT clicks for hits
[15:21:08] [Server thread/INFO]: [RaspberryJuice] ThreadListener Started
```
再檢查 port 
```
netstat -nat |findstr 4711
  TCP    0.0.0.0:4711           0.0.0.0:0              LISTENING       InHost
  TCP    [::]:4711              [::]:0                 LISTENING       InHost
```

再啟動 minecraft 連上去 spigot
```

python pip
```
pip install mcpi
pip install mcpi-e
```

python Hello world
```
from mcpi.minecraft import Minecraft
mc = Minecraft.create()
mc.postToChat("Hello world")
```

python 彩虹
```
import mcpi.minecraft as minecraft
import mcpi.block as block
from math import *

colors = [14, 1, 4, 5, 3, 11, 10]

mc = minecraft.Minecraft.create()
height = 60

mc.setBlocks(-64,0,0,64,height + len(colors),0,0)
for x in range(0, 128):
        for colourindex in range(0, len(colors)):
                y = sin((x / 128.0) * pi) * height + colourindex
                mc.setBlock(x - 64, y, 0, block.WOOL.id, colors[len(colors) - 1 - colourindex])
				
```
