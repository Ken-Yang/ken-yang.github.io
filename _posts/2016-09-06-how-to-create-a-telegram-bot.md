---
layout: post
title: "How to create a Telegram Bot"
description: ""
category: 
tags: []
---
{% include JB/setup %}


今天看到一個朋友寫了FacebookBot，還整了AWS Lambda、斷詞API，突然感到很興奮，<br />
但我沒打算花太多時間在Bot上，只打算做一個簡單的資訊查詢的bot，<br />
所以選擇了[Telegram Bot](https://core.telegram.org/bots/api)，原因是我們公司內部使用Telegram來溝通，<br />
所以就想做一個Bot讓公司的同事查詢一些公司資訊（ex: wifi, 統編）。


<br />

---
### 1. Add BotFather as friend
---

首先我們要使用telegram加`BotFather`這個user，透過`BotFather`去create bot。

![Ticket]({{site.url}}/assets/2016-09-06-add-bot-father.png)


<br />

---
### 2. Create a bot
---

成功加入`BotFather`以後，就可以透過指令`/newbot`去create bot，<br />
整個過程會以對話的方式進行（如下圖），會先問你的bot名稱要叫什麼以及它的username（類似ID，用來被search），<br />

![Ticket]({{site.url}}/assets/2016-09-06-create-bot.png)

<br />

建立成功以後，BotFather就會跟你說的你的bot token，這token很重要，是之後要用來request API用的。

```
Use this token to access the HTTP API:
2X2X2XXX0:XXXXXXXXXXXXXXXXXXXXXXX
```

<br />

<!--more-->


---
### 3. Send/Receive message from bot
---

Telegram在receive訊息有二種方式，一種是透過RESTful API去要，但是被動的方式，得透過long-pulling的方式去拿，<br />
另外一種就是透過web hook，是主動的方式，有任何訊息給bot，Telegram會主動的去打我們的Server，<br />
這裡的案例是使用webhook，而在send and receive訊息之前，<br />
我們必須先建立一個**HTTPS** server，Telegram規定一定必須為HTTPS，<br />
不過是允許用self-signed的，但前提是你得把public key上傳至telegram，<br />
稍後都會說明如何上傳。
這裡我選擇在ubuntu上，使用Node.js來建立HTTPS server，下面的步驟分為三個部分：<br />

1. Install Node.js
2. Create web project
3. Generate SSL certificate


```bash
# Install Node.js
$ wget https://nodejs.org/dist/v4.4.7/node-v4.4.7-linux-x64.tar.gz
$ tar -xvzf node-v4.4.7-linux-x64.tar.gz
$ export PATH=/home/ubuntu/node-v4.4.7-linux-x64/bin:$PATH

# Create bot web project
$ cd ~/
$ mkdir bot/
$ cd bot
$ npm init
$ npm install --save express
$ npm install --save body-parser
$ npm install --save request
$ npm install --save morgan
$ npm install -g pm2
$ mkdir ssl
$ cd ssl

# Generate SSL certificate
$ openssl req -newkey rsa:2048 -sha256 -nodes -keyout server.key -x509 -days 3650 -out server.pem -subj "/C=TW/ST=Taipei/L=Taipei/O=None/CN=bot.kenyang.net"
$ cd ../
$ mkdir -p /var/log/bot/
```

<br />
接著來寫我們的主程式`index.js`，記得把下面的`YOUR_TOKEN`換成你自己的token，

```javascript
const https     = require("https");
const express   = require("express");
const bodyParser    = require('body-parser');
const logger        = require('morgan');
const fs            = require('fs');
const request     = require('request');
const logStream     = fs.createWriteStream('/var/log/bot/access.log', {flags: 'a'});

const SERVER_CONFIG = {
    key:  fs.readFileSync('./ssl/server.key'),
    cert: fs.readFileSync('./ssl/server.pem')
};

var app = express();

app.use(logger('combined', {stream: logStream}));
app.use(logger('dev'));
app.use(bodyParser.json());

app.post('/receive', function(req,resp) {
    console.log(req.body);
    var body = req.body;
    if (!body) {
        resp.status(200).json({"STATUS":"No body"});
        return;
    }

    if (!body.message) {
        resp.status(200).json({"STATUS":"No message"});
        return;
    }

    var chatId = body.message.chat.id;
    var cmd    = body.message.text;
    cmd = cmd.toLowerCase();
    
    // 記得把YOUR_TOKEN換成你自己的
    var options = {
        url: 'https://api.telegram.org/botYOUR_TOKEN/sendMessage',
        form : {
            chat_id: chatId
        }
    };

    if (cmd == '/wifi') {
        options.form.text =  'Your Wi-Fi password is : !@#!$%^&*&$#';
    } else if (cmd == '/address' || cmd == '/addr' || cmd == '/office') {
        options.form.text =  '台北市信義區松X路XXX號XX樓';
    } else if (cmd == '/統編') {
        options.form.text =  'xxxxxxxxx';
    } else if (cmd == '/ken') {
        options.form.text =  'Ken is an engineer.';
    } else {
        options.form.text =  'I don\'t know what you are talking...';
    }

    request.post(options, function(err,httpResponse,body){
        if (err) {
            console.log('err' + err);
            resp.status(200).json({"STATUS":"FAIL"});
            return;
        }
        console.log(body);
        resp.status(200).json({"STATUS":"OK"});
    });

});

https.createServer(SERVER_CONFIG, app)
     .listen(443,function() { console.log("HTTPS sever started"); }
);

```

<br />
完成以後就用pm2把它run起來，

```
$ pm2 start index.js
```

<br />

---
### 4. Set webhook
---

接著我們得向Telegram註冊剛剛建立的API，讓bot收到訊息時，Telegram會主動通知我們，<br />
我們透過curl去註冊，同時會把剛剛create的certificate上傳上去Telegram，<br />
記得把下面參數換成你自己的。

```
$ curl -i -X POST -H "Content-Type: multipart/form-data" -F "certificate=@server.pem" -F "url=https://YOUR_DOMAIN/receive" https://api.telegram.org/botYOUR_TOKEN/setWebhook
```

<br />

---
### 5. Test
---

Set webhook成功以後，就可以跟你的bot進行互動測試了。

![Ticket]({{site.url}}/assets/2016-09-06-test.png)


