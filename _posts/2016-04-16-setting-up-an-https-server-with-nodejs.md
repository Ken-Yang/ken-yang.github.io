---
layout: post
title: "Setting up an HTTPS server with Node.js"
description: ""
category: 
tags: [node.js, ssl, csr, nodejs]
---
{% include JB/setup %}



這篇要講怎麼用Node.js建立一個HTTPS的server，如果你的certificate不是self-signed的，</br>
那設定HTTPS並不難，產生CSR給CA provider，然後就會有certificate，把它放進去就好，</br>
但如果是self-signed，那過程就有點麻煩，每次要弄的時候，都還是會有點忘記，所以乾脆把過程記錄下來好了。</br>


</br>

---
### 1. Creating a private key and CSR
---

在create certificate之前，必須要先有`private key`以及`CSR` (certificate signing request)，</br>
所以我們要先generate出private key以及CSR。

```bash
# generate private key
$ openssl genrsa -des3 -passout pass:kenyang -out server.pass.key 2048
$ openssl rsa -passin pass:kenyang -in server.pass.key -out server.key
$ rm server.pass.key

# generate csr
$ openssl req -new -key server.key -out server.csr
```

</br>

<!--more-->

---
### 2. Creating a certificate
---

有了key和CSR以後，我們就可以issue一張certificate出來，

```bash
$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

</br>


---
### 3. Configuring SSL in Node.js
---

有了certificate以後，就可以把它放在Node.js中使用，</br>
這裡我搭配的`express`+`https`這二個module，https這個module，default就有了，</br>
所以我們只需要安裝`express`就好。

```bash
$ npm install --save express
```

</br>
接著就把下面的內容貼入到`index.js`當中，

```javascript
const https     = require("https");
const express   = require("express");
const fs        = require("fs");

const SERVER_CONFIG = {
    key:  fs.readFileSync('server.key'),
    cert: fs.readFileSync('server.crt')
};

var app = express();
app.get('/', function(req,resp) {
    resp.status(200).json({"STATUS":"OK"});
});

https.createServer(SERVER_CONFIG, app)
     .listen(443,function() { console.log("HTTPS sever started"); }
);
```

</br>
靠`fs`去讀取certificate，然後再餵給`createServer`，這樣就完成了https server。</br>

</br>
</br>
</br>
</br>

