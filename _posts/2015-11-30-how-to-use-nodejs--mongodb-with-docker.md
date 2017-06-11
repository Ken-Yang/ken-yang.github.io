---
layout: post
title: "How to use NodeJS & MongoDB with Docker?"
description: "Running NodeJS+MongoDB on Docker"
category: 
tags: [docker, nodejs, mongodb, express, mongoose]
---
{% include JB/setup %}



這篇會講如何在Docker上部署一個NodeJS+MongoDB的application，  
所以會分二個部分來講，

1. 在Docker上部署MongoDB
2. 在Docker上部署NodeJS+Express

### 1. 在Docker上部署MongoDB

要在Docker上部署MongoDB，首先要先在local machine上建立一個folder，  
這個folder是待會要mapping到container裡面的volume，  
然後MongoDB上的data都會存在這個volume上。

```bash
$ mkdir ~/monogo-data
```

<br />
#### Create a container
---
接著就可以把MongoDB的image抓下來以及run。  

```bash
$ docker pull mongo:latest
$ docker run -v ~/mongo-data:/data \ 
             --name ken-mongo \
             -d \
             mongo \
             mongod --smallfiles
```

<!--more-->

<br />
#### Connect to a container
---
那麼怎麼連進去這個MongoDB呢？  
有二種方式，

1. 透過另外一個container，用`link`的方法
2. 透過`docker exec`

我們先講第二種方法，  
因為第一種方法，待會會透過NodeJS的container來示範。  
我們透過`exec`，然後用參數`-it`與ken-mongo進行互動，  
進去以後在再試試看create db以及新增一筆資料至table student之中。

```bash
$ docker exec -it ken-mongo bash
$ mongo
$ use school
$ db.student.insert({"name": "ken"})
$ db.student.find()
```

<br />
### 2. 在Docker上部署NodeJS+Express

首先先建立一個folder，用來放置source code以及json等檔案。

```bash
$ mkdir test-app
$ cd test-app
```

<br />
#### Package.json
---
接著在`test-app`這個folder底下，建立一個`package.json`，  
`package.json`是用來定義這個app的dependcies以及一些基本資料。  

```json
{
  "name": "node-centos",
  "private": true,
  "version": "0.0.1",
  "description": "Node.js on CentOS using docker",
  "author": "Ken Yang <ken@kenyang.net>",
  "dependencies": {
    "express": "3.2.4",
    "mongoose": "4.2.8"
  }
}
```

從上面的`package.json`中，我們在`dependencies`裡有二個packages，  
分別是

1. `express`  : Web framework
2. `mongoose` : MongoDB client driver

<br />
#### Schema.js
---
接著一樣在`test-app`底下，建立一個`Schema.js`，   
`Schema.js`是用來定義你的MongoDB的schema。

```javascript
var mongoose = require( 'mongoose' );
var Schema   = mongoose.Schema;
 
var Student = new Schema({
    name    : String
});

mongoose.model('Student', Student);

```


<br />
#### Index.js
---
接著一樣在`test-app`底下，建立一個`index.js`，  
`index.js`就是待會這個app的進入點。  

```javascript
var express = require('express');
var mongoose= require('mongoose');
require( './schema' );

var app     = express();
var Student = mongoose.model('Student');
app.use(express.bodyParser());

app.configure('dev', function(){
    mongoose.connect( 'mongodb://db:27017' );
});
app.configure('production', function(){
    mongoose.connect( 'mongodb://xxx.xxx.xxx.xxx:27017' );
});

app.get('/', function (req, res) {
  res.send('Hello world\n');
});

app.post('/insert', function(req, resp) {
    var body = req.body;
    var s = new Student({
              name : body.name
    });

    s.save(function(err,item){
        resp.redirect('/');
    });
});

var PORT = 8080;
app.listen(PORT);
console.log('Running on http://localhost:' + PORT);
```

上面的範例有個比較重要的地方，需要說明一下，  
首先看到`mongoose.connect( 'mongodb://db:27017' );`，  
我們是connect至`db`這個host去，  
這個`db`是怎麼來的？  
是待會會透過`--link`的指令指定的，  
一旦你指定了，就會在container中的`/etc/hosts`裡面增加一筆record。

<br />
#### DockerFile
---

接著建立`DockerFile`，  
`DockerFile`就像是`makefile`一樣，用來定義如何build這個image。

```
FROM    centos:centos6

# Enable Extra Packages for Enterprise Linux (EPEL) for CentOS
RUN     yum install -y epel-release
# Install Node.js and npm
RUN     yum install -y nodejs npm

# Install app dependencies
COPY package.json /src/package.json
RUN cd /src; npm install

# Bundle app source
COPY . /src

ENV NODE_ENV dev

EXPOSE  8080
CMD ["node", "/src/index.js"]
```
裡面比較需要注意的是， `ENV NODE_ENV dev`，  
這個是用來設置環境變數，因為我們設定了`dev`，  
所以在`index.js`中，才會去configure dev。


<br />
#### Build image & Run
---

有了`DockerFile`以後，就可以來build以及run了。

```
$ docker build -t ken-yang/centos-nodejs:v1 .
```
<br />
Build完以後，就可以用下面的指令去把container run起來。  
注意`--link ken-mongo:db`，  
意思就是把現在正在running的`ken-mongo`與現在這個新的container連起來。  
以及給它一個alias `db`。


```bash
$ docker run -d \
             --name ken-node \
             -p 80:8080 \
             --link ken-mongo:db \
             ken-yang/centos-nodejs:v1
```

<br />
#### Test
---

最後用`curl`來發request測試，

```bash
$ curl -X POST -d 'name=kenyang' 192.168.99.100/insert
```

然後可以再去看是否有新增成功，

```bash
$ docker -exec -it ken-mongo bash
$ mongo
$ db.students.find()
```



