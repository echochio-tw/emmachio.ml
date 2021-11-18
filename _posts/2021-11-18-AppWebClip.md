---
layout: post
title: App Web Clip
date: 2007-06-28
tags: AppWebClip
---

AppWebClip:應用程序內容 webclip 到桌面


需要用到的工具:

Mac Appstore 搜索Apple Configurator，下載Apple Configurator 2這個應用

此處需要先填寫名稱，標識符，公司，描述，同意許可信息等內容

圖標為顯示在設備桌面上的圖標，建議大小1024*1024px，png格式，圖標會base64進生成的描述文件中，所以文件大小盡量小，推薦到https://tinypng.com/ 去壓縮

生成的文件描述文件實際上是壹個XML，使用sublime text等工具可以快捷標記，上面生成的範例文件如下，可手機點擊安裝

看是要開發者帳號的 SSL.....

如果保存直接發布會提示未簽名，下面就介紹壹下如何對描述文件進行簽名。

簽名有兩種方式，使用蘋果開發者賬號進行簽名，與 使用SSL證書進行簽名

SSL簽名

使用SSL簽名需要先有壹個註冊域名並且取得域名相關的SSL證書，

推薦免費獲取證書的 https://letsencrypt.org/

letsencrypt證書不能通過iOS驗證，但Mac驗證可通過，如需商用，建議購買商用SSL證書

可通過如命令進行簽名，註意，證書使用pem格式

```
openssl smime -sign -in ~/Desktop/IOS_WEBCLIP.mobileconfig -out ~/Desktop/iOSWebClip_signed.mobileconfig -signer ~/Desktop/we.me.public.pem -inkey ~/Desktop/we.me.private.pem -outform der -nodetach

security cms -S -N "Apple Development: XXX" -i iOSWebClip_signed.mobileconfig -o iOSWebClip_signed.mobileconfig.mobileconfig
```

有保存別人的資訊到 git

https://github.com/echochio-tw/webclip
