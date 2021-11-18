---
layout: post
title: docker macOS
date: 2021-11-18
tags: docker macOS
---

windows 11 + docker (windows 版) +  WSL Ubuntu 20.04

如何使用 Docker-OSX 在 QEMU + KVM 中安裝 macOS (OSX)

使用基本恢復映像運行 macOS Big Sur（將 WIDTH 和 HEIGHT 值更改為要用於 macOS Big Sur 虛擬機的屏幕分辨率；我在下面使用 1600x900）：

以下是在 WSL Ubuntu 20.04 操作的 (當然要裝 docker client 連 windows docker
```
docker run -it \
    --device /dev/kvm \
    -e DEVICE_MODEL="iMacPro1,1" \
    -e WIDTH=1600 \
    -e HEIGHT=900 \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -e "DISPLAY=${DISPLAY:-:0.0}" \
    -e GENERATE_UNIQUE=true \
    -e MASTER_PLIST_URL=https://raw.githubusercontent.com/sickcodes/osx-serial-generator/master/config-custom.plist \
    sickcodes/docker-osx:big-sur
```
 
接下來，在磁盤工具工具中，從 2 QEMU HARDDISK Media 中檢查左側最大的硬盤，然後單擊最大的（在我用於測試的版本中為 XXX GB）。

提示：要讓 QEMU 虛擬機釋放鼠標，請按 Ctrl + Alt + g（在某些情況下，這只是 Ctrl + g）。

現在單擊“磁盤工具”工具欄中的“擦除”按鈕以格式化此硬盤：
 
為磁盤設置一個名稱，例如 “macOS”，並將其他選項保留為默認值

（它們是 Format: APFS for macOS Big Sur 和 Mac OS Extended (Journaled) for macOS Catalina，Schema 為 GUID Parition Map for both）。

完成擦除最大的 QEMU HDD 後，關閉“磁盤工具”對話框。

在虛擬機中安裝macOS。

一段時間後，mac OSX 虛擬機將重新啟動。 在啟動時選擇 macOS 安裝程序條目：
 
安裝完成後，此引導條目將變為“macOS”，並允許您在虛擬機內引導到新的 macOS 安裝：
 
現在您需要選擇您所在的國家或地區，可選擇使用您的 Apple ID 登錄，同意條款和條件等，以及創建您的計算機帳戶（用戶名和密碼）。

請注意，安裝macOS並首次啟動後，我不得不選擇兩次macOS條目（第一次選擇它後重新啟動）。 此外，在 macOS 安裝過程中存在鼠標延遲（和一般延遲），

但是一旦您運行已安裝的 macOS 虛擬機，這種情況就不再發生（或者不太明顯；這取決於您的計算機硬件）。

啟動新安裝的 macOS 虛擬機。當您想啟動新安裝的 macOS 虛擬機時，請運行 docker ps -a 以查看容器 ID 和/或名稱：

docker ps -a

然後啟動容器：

docker start Container_ID_or_Name

將 Container_ID_or_Name 替換為使用上一個命令獲得的容器 ID 或名稱。

您可能還喜歡：Portainer：用於遠程或本地使用的基於 Web 的 Docker GUI

如何刪除 Docker-OSX

所以你已經決定要刪除 Docker-OSX。 首先運行以下命令以獲取 Docker 容器名稱和 ID 的列表：

docker container ls -a

使用此命令，確定要刪除的容器。 現在使用以下命令停止並移除容器：

docker container stop ID_or_Image

docker container rm ID_or_Image

將 ID_or_Image 替換為使用上一個命令獲得的容器 ID 或圖像。

要刪除 Docker-OSX 鏡像，首先使用以下命令列出現有的 Docker 鏡像：

docker image ls

接下來，使用以下命令刪除 Docker-OSX 映像：

docker image rm Image_ID

將 Image_ID 替換為您使用上一個命令獲得的圖像 ID。
