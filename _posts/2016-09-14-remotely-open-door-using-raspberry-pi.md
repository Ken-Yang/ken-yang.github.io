---
layout: post
title: "Remotely open door using Raspberry Pi"
description: ""
category: 
tags: [raspberry, pi, telegram, bot]
---
{% include JB/setup %}

<font size="3">

由於上一篇寫了一個Telegram Bot，最近就在幫它加feature，

1. 出build時會通知
2. 開會時會通知
3. 11:30提醒要吃飯了

總覺得少了點什麼，於是就想說來做個機器人自動開門。</br>
在開始之前，先準備以下材料：

1. Raspberry Pi 3: 1台
2. 杜邦線: 至少4條
3. 麵包板: 1塊
3. 漆包線: 可有可無
4. 一顆勇敢的心（因為你可能用壞，導致整個公司的人被反鎖）

下圖為完成品，

![Ticket]({{site.url}}/assets/2016-09-14-finish.jpg)


</br>

<!--more-->

---
### 1. 拆開電源座
---

大部分的公司，門旁邊應該都有一個開關，不管是感應式的或者是按鈕式的也好，只要一trigger，門就會打開。</br>
也就是說開關一定接著門的trigger，所以我們要做man in the middle hack...LOL...</br>
所以首先要把電源拆掉，然後拿出來（如下圖），

![Ticket]({{site.url}}/assets/2016-09-14-1.jpg)

拿出來以後會看到有二條線接在電源開關上，分別為一紅一黑，把這二條線從開關中拔出來，</br>
然後你要先找出你的正極跟負極分別為哪一條，我的黑色是正，紅色是負。

</br>

---
### 2. 接線 （backward compatible）
---

為了讓原本的開關也可以正常work，所以我們必須把上面拆出來的電源線接在麵包板上，然後在接另外2條杜邦線，</br>
一頭接在開關上面，另一頭接在麵包板上面，如下圖。

![Ticket]({{site.url}}/assets/2016-09-14-3.png)

</br>

---
### 3. 接上Raspberry Pi 3
---

接著要再接上2條杜邦線，一頭接在Pi上的GPIO，另一頭一樣接在麵包板上，</br>
如果不知道接在哪個Pin腳，可以[Google一下Raspberry Pi 3的GPIO圖](https://www.google.com.tw/search?q=raspberry+3+gpio&source=lnms&tbm=isch&sa=X&ved=0ahUKEwiJl-jJ3o7PAhXEj5QKHdpyC3QQ_AUICCgB&biw=1920&bih=916)，</br>
我自己是接在Pin 7 and 6上面，正極在7，負極在6，如下圖。然後分別把它接在對應的麵包板位置上。

![Ticket]({{site.url}}/assets/2016-09-14-4.jpg)

</br>

---
### 4. 寫Code
---

最後就要來寫code，我這裡是使用NodeJS加上Python，因為用NodeJS setup web server很快速簡單，</br>
然後用python控制GPIO也很簡單，所以就用這二種組合。</br>
首先先寫Python來透過GPIO來開門，由於現在的Raspbian預設就有Rpi.GPIO，所以你可以直接使用。

```python
import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BOARD)
GPIO.setup(7, GPIO.OUT)

GPIO.output(7, GPIO.LOW)
time.sleep(0.5)
GPIO.output(7, GPIO.HIGH)

GPIO.cleanup()
```

</br>

接著就來用NodeJS來建立Web Server，詳細的步驟就[參考上一篇](http://blog.kenyang.net/2016/09/06/how-to-create-a-telegram-bot)，</br>
如果建立完Web Server以後，我們得加裝另一個module，因為我們要call python script，

```bash
npm install --save python-shell
```

</br>
接著就可以在index.js裡面加入下面的code，

```javascript
const https     = require("https");
const express   = require("express");
const bodyParser    = require('body-parser');
const logger        = require('morgan');
const fs            = require('fs');
const request     = require('request');
const logStream     = fs.createWriteStream('/var/log/bot/access.log', {flags: 'a'});
var PythonShell = require('python-shell');

.... ignore ....

if (cmd == '/open') {
    PythonShell.run('open.py', {}, function (err, results) {
        if (err) {
	        console.log(err);
	     }
	 });
}
```

</br>
然後記得restart。

```bash
pm2 restart index
```

</br>

---
### 5. Test
---

完成上面的步驟以後，你就可以跟你的機器人講`/open`，他就會幫你開門了！

![Ticket]({{site.url}}/assets/2016-09-14-5.jpg)


</font>