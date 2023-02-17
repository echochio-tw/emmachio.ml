---
layout: post
title: GAS use telegram-bot connect ChatGPT API 
date: 2023-02-16
tags: ChatGPT
---
以下用問 ChatGPT 寫的

問 ChatGPT:

Google Apps Script  使用 telegram bot 連接  ChatGPT API 回應至  telegram 房間

ChatGPT 回：

1. 要使用 Google Apps Script 连接 ChatGPT API 并将回复发送到 Telegram 群组，您需要完成以下步骤：

2. 创建一个 Telegram bot 并获取 bot 的 token。

3. 在 Google Apps Script 编辑器中创建一个新项目，并将项目命名为您喜欢的名称。

在代码编辑器中添加以下代码，以使用 Telegram bot API 进行身份验证并从 Telegram 群组接收消息：
```
var token = 'YOUR_TELEGRAM_BOT_TOKEN';
var telegramUrl = 'https://api.telegram.org/bot' + token;

function getUpdates() {
  var url = telegramUrl + '/getUpdates';
  var response = UrlFetchApp.fetch(url);
  var content = JSON.parse(response.getContentText());
  if (content.ok == true) {
    var message = content.result[content.result.length-1].message;
    var chat_id = message.chat.id;
    var text = message.text;
    if (text) {
      return {text: text, chat_id: chat_id};
    }
  }
  return null;
}
```

4. 添加以下代码以连接到 ChatGPT API 并获取 ChatGPT 的回复：
```
function getResponse(text) {
  var url = 'https://api.openai.com/v1/engine/your_engine_name/completions';
  var apiKey = 'YOUR_OPENAI_API_KEY';
  var prompt = text + '\nGPT Response:';
  var data = {
    "prompt": prompt,
    "temperature": 0.7,
    "max_tokens": 100,
    "top_p": 1,
    "frequency_penalty": 0,
    "presence_penalty": 0
  };
  var options = {
    'method': 'post',
    'headers': {
      'Authorization': 'Bearer ' + apiKey,
      'Content-Type': 'application/json'
    },
    'payload': JSON.stringify(data)
  };
  var response = UrlFetchApp.fetch(url, options);
  var content = JSON.parse(response.getContentText());
  return content.choices[0].text.trim();
}
```

这将使用 OpenAI 的 API 进行自然语言处理，并返回 ChatGPT 的响应。

5. 最后，添加以下代码以向 Telegram 群组发送 ChatGPT 的回复：
```
function sendResponse(text, chat_id) {
  var url = telegramUrl + '/sendMessage?chat_id=' + chat_id + '&text=' + text;
  var response = UrlFetchApp.fetch(url);
  return response;
}

```

这将使用 Telegram bot API 向特定的 Telegram 群组发送回复。

6. 为了使脚本定期检查是否有新的消息需要回复，您可以使用 setInterval() 函数，例如：
```
function start() {
  setInterval(function() {
    var updates = getUpdates();
    if (updates) {
      var response = getResponse(updates.text);
      sendResponse(response, updates.chat_id);
    }
  }, 1000);
}
```
此函数将每隔 1 秒钟检查是否有新的消息，并自动回复。您可以根据需要更改此间隔时间。

7. 最后，单击“运行”按钮以启动您的脚本，并确保您已将其部署为 Web 应用程序（选择“发布”>“部署为 Web 应用程序”）。这将为您提供一个 URL，


這邊我加上改一下 （用房間號與前導來判斷）
```
function doPost(e) {
  var token = '1272333:AAH6d_w1222OtqZsdddZQ1Sh_vbj840';
  var chat_id = '-31712236';
  var chat = JSON.parse(e.postData.contents);
  // 是文字格式時
  if(chat.message) {
    // 房間號的正確時處理
    if (chat.message.chat.id == chat_id){
      // 切割字串
      var message = chat.message.text;
　    var messageArray = new Array();
      var messageArray = message.split(" ");
      // 前個為 /hi 時處理
      if (messageArray[0] == "/hi") {
        var msg = "收到訊息查詢第 : "+messageArray[1];
      }
    }
  }
}
```

自己組.... 這不寫了
