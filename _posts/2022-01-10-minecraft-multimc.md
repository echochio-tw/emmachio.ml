---
layout: post
title: minecraft java edition 啟動器 multimc 與 java server 
date: 2022-01-10
tags: minecraft java craftbukkit python3
---

說到啟動器我首推 https://multimc.org/

那 java server 工業模組推薦 All-of-Fabric-5-Server （TeamAOF 做的或許下次就有6,7,8, ....)
https://www.curseforge.com/members/teamaof/projects

那 java server 光影推薦 Valhelsia+Enhanced+Vanilla-1.0.4-SERVER （ValhelsiaTeam 做的下次或許就換名)
https://www.curseforge.com/members/ValhelsiaTeam/projects


小 server 就用 Craftbukkit 吧
https://getbukkit.org/download/craftbukkit

常見的Server文件在
https://www.spigotmc.org/wiki/what-is-spigot-craftbukkit-bukkit-vanilla-forg/

安裝client 就用 multimc 再選 curseforge 然後搜尋所要的套件安裝就完成了 .... 夠簡單吧

這安裝紀錄一下....這樣就啥都可下載安裝了
```
@ECHO OFF
SETLOCAL

:BEGIN
CLS
COLOR 3F >nul 2>&1
SET MC_SYS32=%SYSTEMROOT%\SYSTEM32
REM Make batch directory the same as the directory it's being called from
REM For example, if "run as admin" the batch starting dir could be system32
CD "%~dp0" >nul 2>&1

:CHECK
REM Check if serverstarter JAR is already downloaded
IF NOT EXIST "%cd%\serverstarter-2.1.1.jar" (
	ECHO serverstarter binary not found, downloading serverstarter...
	%SYSTEMROOT%\SYSTEM32\bitsadmin.exe /rawreturn /nowrap /transfer starter /dynamic /download /priority foreground https://github.com/TeamAOF/ServerStarter/releases/download/v2.1.1/serverstarter-2.1.1.jar "%cd%\serverstarter-2.1.1.jar"
   GOTO MAIN
) ELSE (
   GOTO MAIN
)

:MAIN
java -jar serverstarter-2.1.1.jar
GOTO EOF

:EOF
pause
```

```
DO_RAMDISK=0
if [[ $(cat server-setup-config.yaml | grep 'ramDisk:' | awk 'BEGIN {FS=":"}{print $2}') =~ "yes" ]]; then
    SAVE_DIR=$(cat server.properties | grep 'level-name' | awk 'BEGIN {FS="="}{print $2}')
    mv $SAVE_DIR "${SAVE_DIR}_backup"
    mkdir $SAVE_DIR
    sudo mount -t tmpfs -o size=2G tmpfs $SAVE_DIR
    DO_RAMDISK=1
fi
	if [ -f serverstarter-2.1.1.jar ]; then
			echo "Skipping download. Using existing serverstarter-2.1.1.jar"
         java -jar serverstarter-2.1.1.jar
               if [[ $DO_RAMDISK -eq 1 ]]; then
               sudo umount $SAVE_DIR
               rm -rf $SAVE_DIR
               mv "${SAVE_DIR}_backup" $SAVE_DIR
               fi
               exit 0
	else
			export URL="https://github.com/TeamAOF/ServerStarter/releases/download/v2.1.1/serverstarter-2.1.1.jar"
	fi
		echo $URL
		which wget >> /dev/null
		if [ $? -eq 0 ]; then
			echo "DEBUG: (wget) Downloading ${URL}"
			wget -O serverstarter-2.1.1.jar "${URL}"
   else
			which curl >> /dev/null
			if [ $? -eq 0 ]; then
				echo "DEBUG: (curl) Downloading ${URL}"
				curl -o serverstarter-2.1.1.jar "${URL}"
			else
				echo "Neither wget or curl were found on your system. Please install one and try again"
         fi
      fi
java -jar serverstarter-2.1.1.jar
if [[ $DO_RAMDISK -eq 1 ]]; then
    sudo umount $SAVE_DIR
    rm -rf $SAVE_DIR
    mv "${SAVE_DIR}_backup" $SAVE_DIR
fi
```
