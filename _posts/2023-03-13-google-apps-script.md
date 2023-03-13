---
layout: post
title: google apps script get api sent telegram
date: 2023-03-13
tags: script
---

google sheet 開 SheetName 'data' 資料如下
```
test1验证码_余额:	I5_I1889099	testpass1
test2验证码_余额:	I4_I1889099	testpass2
```

在 sheet 開 Apps Script
```
function getSheetDataAndSendToTelegram() {
  var telegramBotToken = "telegramBotToken";
  var telegramChatId = "telegramChatId";
  var url = 'https://test.com/balance/json';
  var headers = {'Content-Type': 'application/json'};
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("data");
  var data = sheet.getDataRange().getValues();
  for (var i = 0; i < data.length; i++) {
	  var requestData = {'account': data[i][1],'password': data[i][2]};
    var options = {
    'method' : 'post',
    'headers' : headers,
    'payload' : JSON.stringify(requestData),};
    var response = UrlFetchApp.fetch(url, options);
    var result = response.getContentText();
    var final = JSON.parse(result);
    var message = data[i][0] + final.balance;
    Logger.log(message);
    var telegramUrl = "https://api.telegram.org/bot" + telegramBotToken + "/sendMessage?chat_id=" + telegramChatId + "&text=" + encodeURIComponent(message);
    UrlFetchApp.fetch(telegramUrl);
  }
}
```
