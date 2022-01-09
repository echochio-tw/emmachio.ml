---
layout: post
title: minecraft java edition craftbukkit Server 與 python3 
date: 2022-01-10
tags: minecraft java craftbukkit python3
---

1. 到 https://getbukkit.org/ 下載 craftbukkit 我現在是 craftbukkit-1.18.1
2. 開一個目錄 如 D:\Craftbukkit-python-1.18.1  把 craftbukkit-1.18.jar 丟進去
3. 建立一個 run.bat (改你jar 的名字 與記憶體大小 如2G）
```
java -jar -Xms2G -Xmx2G craftbukkit-1.18.1.jar nogui
pause
```
4. 執行 run.bat
5. 改 server.properties (直接複製或請自行上網找參數）
```
#Minecraft server properties
enable-jmx-monitoring=false
rcon.port=25575
gamemode=creative
enable-command-block=false
enable-query=false
level-name=PythonCraft
motd=A Minecraft Server
query.port=25565
pvp=true
difficulty=easy
network-compression-threshold=256
require-resource-pack=false
max-tick-time=60000
use-native-transport=true
max-players=20
online-mode=true
enable-status=true
allow-flight=true
broadcast-rcon-to-ops=true
view-distance=10
server-ip=
resource-pack-prompt=
allow-nether=true
server-port=25565
enable-rcon=false
sync-chunk-writes=true
op-permission-level=4
prevent-proxy-connections=false
hide-online-players=false
resource-pack=
entity-broadcast-range-percentage=100
simulation-distance=10
rcon.password=
player-idle-timeout=0
debug=false
force-gamemode=false
rate-limit=0
hardcore=false
white-list=false
broadcast-console-to-ops=true
spawn-npcs=false
spawn-animals=false
function-permission-level=2
level-type=flat
text-filtering-config=
spawn-monsters=false
enforce-whitelist=false
resource-pack-sha1=
spawn-protection=16
max-world-size=299999
``

6. 改 eula.txt 為 true
```
#By changing the setting below to TRUE you are indicating your agreement to our EULA (https://account.mojang.com/documents/minecraft_eula).
#Sun Jan 09 23:23:28 CST 2022
eula=true
```

7. 再執行 run.bat
8. 開完後下 stop 停止服務器
9. 下載 https://github.com/zhuowei/RaspberryJuice/blob/master/jars/raspberryjuice-1.12.1.jar
10. 把 raspberryjuice-1.12.1.jar 放到 D:\Craftbukkit-python-1.18.1\plugins 下面
11. python3 就自行安裝 (我習慣用 3.7其他版本也可）
12. python3 加 pip install mcpi 與 pip install mcpi_e
13. 開 minecraft java edition 1.18.1 (要配合 craftbukkit-1.18.1）
14. 執行 run 看到最後有 RaspberryJuice 才是正常的
```
[00:38:06] [Server thread/INFO]: [RaspberryJuice] Enabling RaspberryJuice v1.12.1
[00:38:06] [Server thread/INFO]: [RaspberryJuice] Using host:port - 0.0.0.0:4711
[00:38:06] [Server thread/INFO]: [RaspberryJuice] Using RELATIVE locations
[00:38:06] [Server thread/INFO]: [RaspberryJuice] Using RIGHT clicks for hits
[00:38:06] [Server thread/INFO]: [RaspberryJuice] ThreadListener Started
[00:38:06] [Server thread/INFO]: Server permissions file permissions.yml is empty, ignoring it
```
 
15. minecraft 多人連 127.0.0.1 (看你Server 裝到哪台IP 就連哪台....多人時用playerName 可分辨的)
16. 執行個 Hello_word.py
```
import mcpi.block as block
from mcpi_e.minecraft import Minecraft

serverAddress="127.0.0.1" # change to your minecraft server
pythonApiPort=4711 #default port for RaspberryJuice plugin is 4711
playerName="echochio" # change to your username

mc = Minecraft.create(serverAddress,pythonApiPort,playerName)
pos = mc.player.getPos()
basex = int(pos.x) + 0
basey = int(pos.y) + 6
basez = int(pos.z) + 0

data = [
  "#  # ### #   #    ##   #   #  ##  ###  #   ### ",
  "#  # #   #   #   #  #  # # # #  # #  # #   #  #",
  "#### ### #   #   #  #  # # # #  # ###  #   #  #",
  "#  # #   #   #   #  #  # # # #  # # #  #   #  #",
  "#  # ### ### ###  ##    # #   ##  #  # ### ### "
]

for y, line in enumerate(data):
  for x, c in enumerate(line):
    if c == "#":
      mc.setBlock(basex + x, basey - y, basez, block.DIAMOND_BLOCK.id)
```

ps: 重建世界就把 D:\Craftbukkit-python-1.18.1\PythonCraft 目錄刪除

<img src="/images/posts/minecraft/a.png">
