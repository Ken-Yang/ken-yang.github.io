---
layout: post
title: "JCConf Taiwan 2015   Day 1"
description: ""
category: 
tags: [JCConf, java]
---
{% include JB/setup %}




![Ticket]({{site.url}}/assets/2015-12-04-ticket.jpg)

這篇只是紀錄Java Community Conference 2015的第一天筆記，  
今天聽了以下幾個session，

1. JRuby
2. Asynchronous and Non-blocking in Scala
3. 使用zookeeper打造軟體式負載平衡
4. workshop動手玩Java專案建置工具：以Gradle與Docker為例
5. Akka Cluster in Java
6. 使用Java的Future/Promise API來撰寫非同步程式

<!--more-->

### 1. JRuby 
---

 - Charles Oliver Nutter
 - headius@headius.com
 - blog.headius.com
 - works at RedHat

#### ruby 

- everything is Object. no primitive integer.
- invokeinterface
	- Runnable.run();
- invokespecial
   - super.equals();
- invokevirtual  
- invokestatic
	- System.currentMill();
- invokedynamic
	- *Released in Java7
- Ruby == Unix, but java == java everywhere
	* Ruby has getpwd(), getpid()...etc.
	* but java has no these capability.
* JNI 
	* Speaker doesn't like it.
	* you need to implement a feature for each platform (win, mac, linux)
  		* ex: if you want to get pid() in java.
* JNR
	* Speaker use JNR.
	* there are many impelmentations in JNR.
	* POSIX posix = POSIXFactory.getPostfix(xxxxx);  
	* posix.getpid()

#### Project panama

* Panama in JVM for java 9, api in java 10


#### why use JVM?
someone ask why not use standard ruby?
author answer: hard to implement JIT.

</br>

----
### 2. Asynchronous and Non-blocking in Scala
---

- I/O is very slow, so we need aysnc and non-blocking.

#### NIO
- not use thread to do operation. have a subsystem will handle it.

#### pure function
- no side effects
- no write db or file
- the output will be the same at each time

#### val vs var
- val is immutable
- var is mutable

#### case object
- singleton in Java. there is no static in Scala.

</br>

---
### 3. 使用zookeeper打造軟體式負載平衡
---

這個session主要在介紹zookeeper，由於時間很短，只有15分鐘。  
所以都是講很high level，並沒有講到detail的實作。

</br>

---
### 4. workshop動手玩Java專案建置工具：以Gradle與Docker為例
---

這個session在介紹他們team是怎麼用`docker`建立test env，  
然後再透過`Geb`去進行automation，  
主要提供了以下二個docker images讓我們練習測試，

1. client：vvoyer/docker-selenium-firefox-chrome
2. main：groovy

由於我對於`docker`算是瞭解，因此這個session沒有全部聽完就先去趕場了。  
簡單的說，整個流程就是先透過docker叫起`client`，  
之後再叫起`main`來對`client`進行測試。

### Resource

* [Git repo] (https://github.com/TrunkWorkshop/jcconf-2015-java-docker)
* [Hackpad] (https://hackpad.com/JCConf-Taiwan-2015-Workshop-lKcJEMyjraR)

#### docker-swarm
- cluster

#### docker-compose
- 簡化docker指令

#### geb
- http://learngeb.readbook.tw/
- functional test

</br>

---
### 5. Akka Cluster in Java
---

[Git repo for Akka cluster example] (https://github.com/jiayun/akka_samples)

</br>

---
### 6. 使用Java的Future/Promise API來撰寫非同步程式
---

前陣子也剛好在研究`CompletableFuture`，  
所以這是今天最有共鳴的一場session，

### Speaker
- koji
- work at Line (located at Japan)

#### 傳統做法開thread

thread缺點  

- 需要配合synchronized, wait, notify, join  
- 不同thread如何存取同變數
- 如何控管
- 不易組合和再利用
- 組合各種非同步方法，會變成callback hell

講者說：這個年代請不要再直接使用thread api

#### Future

- 一個等待結果的容器
- get會卡住！！


#### CompletableFuture

- 類似有callback概念，做完動作以後，通知我，我再繼續做想做的事情
- get會丟exception，可以用join 
- supplyAsync可以指定executorService，如果沒給，就會用forkjoin裡的tool。
- 盡量要給！
- `CF<CF<string>> c` 不好，要get 2次 ex: `c.get().get()`
- 請用.thenCompose, 回傳也是completeablefutre的時候，就不用get 2次，像是scala的flatmap，
- 優點
	- event driven
	- 容易組合 (easy to compose)
	- 控制權還給呼叫者
	- 減少thread的浪費，沿用thread繼續做之類的
- 缺點
	- future/promise的混合，不少語言都分開坐 
	- 爆多的方法數量 60+ 	
- 注意
	- 雖然有future，但cancel跟future不一樣，completablefutre不能interrupt。
	- 不能取消正在執行的工作
	- 盡量使用Async語尾的API


</br>



---
### 7. Other
---

#### 攤位sudo
- yvette lin 曾經做過創投
- 就專業的人力資源公司，但focus on RD

#### 攤位mysql
- speaker很專業
- 詢問了master與master的架構時，當replication發生error時，通常得取捨一台，必定有資料會遺失，該怎麼辦？
	- 回答：透過讓`master`等待的機制，然後取回binlog回來至`slave`，partially updated
- 還介紹了MySQL Enterprise Monitor
	- 有圖形化介面讓admin看究竟哪些sql command花最久
- 還賺到了一隻MySQL的海豚玩偶
	

![Ticket]({{site.url}}/assets/2015-12-04-mysql-dophin.jpg)

#### 攤位微軟
- 介紹Azure
	
#### 攤位Kxxxx banking
- 在做財務管理系統
- 在徵人，要`java`，前端用`extJS`
	 
#### 攤位ruckus
- 主要做無線網路設備
- 也是Java
	 

#### 攤位safari book online
- 線上遊覽書籍之類的網站 
- [Link] (https://www.safaribooksonline.com/)
	
	 
#### 晚上的lightning show
- ruckus的人出來送價值6000元的ap，很幸運地被我同事拿到
- 中鋼的人來分享他們內部系統心路歷程
- linkedin員工來分享在那工作的第一個月 （其實是fliptop的員工，但被linkedin收購）


#### Foods and Drinks
- 大概是我參加過所有conf吃最好的吧，晚上還有烤山豬肉可以吃
- 飲料是金色三麥贊助
- 茶也是知名廠商










