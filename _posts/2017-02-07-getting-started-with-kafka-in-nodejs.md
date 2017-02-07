---
layout: post
title: "Getting started with Kafka in Node.js"
description: ""
category: 
tags: []
---
{% include JB/setup %}

<font size="3">

之前有講過[Kafka的用途以及如何安裝](http://blog.kenyang.net/2015/06/25/apache-kafka-distributed-messaging)，這篇會講如何用`Node.js`去連結`Kafka`，至於安裝Kafka的部分就請去[上一篇](http://blog.kenyang.net/2015/06/25/apache-kafka-distributed-messaging)來看。


</br>

<!--more-->

---
### 1. Create a topic
---

首先，先用CLI去建立一個topic，這topic等等會被Node.js subcribe。

```bash
$ cd $YOUR_PATH/kafka_2.11-0.10.1.0
$ bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic open
```

可以用下面的指令去看topic是否建立成功：

```bash
$ bin/kafka-topics.sh --list --zookeeper localhost:2181
```

</br>

---
### 2. Install kafka-node
---

這一步驟要安裝kafka的Node.js module，這裡選用的是`kafka-node`（其實選擇也不多）。

```bash
$ mkdir /opt/kafka-test
$ cd /opt/kafka-test
$ npm install kafka-node --no-optional --save
```

</br>

---
### 3. Connect to Zookeeper and Send message (Producer.js)
---

接著就可以在Node.js中使用`kafka-node`，首先要先connect到你自己的Zookeeper，再來就可以對我們剛剛建立的topic進行傳送訊息。</br>
傳訊息是透過`producer.send`這方式，成功以後我會去做`client.close()`的動作，因此這隻node.js每次執行完就會關閉了。

```javascript
var kafka = require('kafka-node'),
    Producer = kafka.Producer,
    client = new kafka.Client('${YOUR_HOST}:2181'),
    producer = new Producer(client),
    payloads = [
        { topic: 'open', messages: ['hello world from nodejs'] }
    ];

producer.on('ready', function () {
    producer.send(payloads, function (err, data) {
        console.log(err || data);
        client.close();
    });
});

producer.on('error', function (err) {})
```

</br>

---
### 4. Connecto to Kafka Server and Receiver message (Consumer.js)
---

接著就是implement `consumer.js`，基本上跟上面的Producer沒有太大差別，比較需要說明的參數是`autoCommit`，一旦設為true，在你的consumer收到這則message時，就會被標記已讀的概念，接下來其他的consumer就不會再收到。

```javascript
var kafka = require('kafka-node'),
    Consumer = kafka.Consumer,
    client = new kafka.Client('${YOUR_HOST}:2181'),
    consumer = new Consumer(
        client,
        [
            { topic: 'open', partition: 0 }
        ],
        {
            autoCommit: true
        }
    );

consumer.on('message', function (message) {
    console.log(message);
});

```

</br>

這裡我有踩到一個雷，由於我是把Kafka放在EC2上，然後EC2綁了一個elastic IP，但我的consumer並不是放在EC2上，而是放在某個local VM裡面，在連接的時候，一直遇到下面的error message，解決方式很簡單，只要在你的`/etc/hosts`裡面增加一筆mapping就好了。

```
Error: getaddrinfo ENOTFOUND ip-xxx-xxx-xxx-xxx ip-xxx-xxx-xxx-xxx:9092
```

</br>

---
### 5. Testing
---

先在放置consumer.js的機器上執行它，

```bash
$ node consumer.js
```

接下來就執行producer.js，由於上面第三步驟有提到，在send完以後會把close client，所以執行完producer以後，這隻js就會被關閉。可以一直不斷執行這隻檔案，你會在consumer那台機器上不斷看到訊息傳遞過來。

```bash
$ node producer.js
```

</br>

---
### 6. Advance
---

kafka server在連zookeeper時，default的IP and Port是存在`config/server.properties`裡面的`zookeeper.connect`。
</br>
</br>
由於我的EC2 ram很小，所以在launch Kafka時，一直發生OOM，解決方式是去`bin/kafka-server-start.sh`改`KAFKA_HEAP_OPTS`，

```
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
    export KAFKA_HEAP_OPTS="-Xmx500M -Xms500M"
fi
```






</font>