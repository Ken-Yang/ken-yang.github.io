---
layout: post
title: "JCConf Taiwan 2015   Day 2"
description: ""
category: 
tags: [JCConf, java]
---
{% include JB/setup %}



這篇只是紀錄Java Community Conference 2015的第二天筆記，  
今天聽了以下幾個session，

1. 阿里 JVM 的工作方向
2. Universal Scala	
3. Deploy your own Spark cluster in 4 minutes using sbt
4. Immutable Infrastructure：觀念與實作 
5. JDK8 JIT 行為和效能分析
6. 自己的JVM自己救 - 解救 OOM 實務經驗談 

<!--more-->

</br>

---
### 1. 阿里 JVM 的工作方向
---

主要在說他為什麼從facebook回中國阿里雲，  
在fb調校php，在阿里雲調校jvm，  
在講在阿里雲裡的JVM心路歷程。

**Speaker**

- 趙海平
- work @阿里雲
- previous work @FB


**Coroutine**

 - like wait,notify in Java

**Java young generation, old generation**

</br>

---
### 2. Universal Scala
---
在講怎麼用scala在frontend與backend上。

**Speaker**

- Walter Chang

**What is universal?**

- isomorphic
- run the same code on both client and server


**Functional language**

- Pure functions [Wiki] (https://en.wikipedia.org/wiki/Pure_function)

**immutable 優點**

- no need to handle synchronization


**val is immutable (like final in Java)**

- var i: Int = 123
- var i = 123

**var is mutable**


**case class**


**What is Scala.js**

- compile `Scala` to `javascript`

**Example code**
- [github] (https://github.com/weihsiu/universal.git)

</br>

---
### 3. Deploy your own Spark cluster in 4 minutes using sbt.
---

介紹他們的tool（spark-deployer）

**Speaker**

- Pishen Tsai
- Works @KKBox

**Tool: spark-deployer**

- [Github] (https://github.com/KKBOX/spark-deployer)
- SBT plugin
- productively used in KKBOX
- 100% scala

**Current Solutions to deploy Spark cluster**

- **spark-ec2**
	- 缺點 
  		* command 太多
  		* 需要裝`sbt`
  		* 需要裝`ec2`
  		* slow startup time (~20mins)
  		* 一小時算一次錢，三分之一錢花在deploy上
	- 要用spark-ec2的flow
		* ![flow]({{site.url}}/assets/2015-12-05-spark-ec2-flow.jpg)
	- spark-ec2的command
		* ![command]({{site.url}}/assets/2015-12-05-spark-ec2-cmd.jpg)
	
- **amazon emr (elastic map reduce)**
	- 缺點
  		* command 太多
  		* 需要裝`sbt`
  		* 需要裝`emr`
  		* spark version is old
	- 要用Amazon emr的flow
		* ![flow]({{site.url}}/assets/2015-12-05-aws-emr-flow.jpg)
	- emr的cmd
		* ![cmd]({{site.url}}/assets/2015-12-05-aws-emr-cmd.jpg)
- **spark-deployer**
	- 優點
		* 只需要安裝`sbt`
		* fast and parallel startup (~4mins)
		* Dynamic scale out
		* Flexible design 
	- 用Spark deployer的flow
		* ![flow]({{site.url}}/assets/2015-12-05-spark-deployer-flow.jpg)
	- 用Spark deployer的cmd
		* ![cmd]({{site.url}}/assets/2015-12-05-spark-deployer-cmd.jpg)


**Prerequisite**

- java
- sbt
- AWS\_ACCESS\_KEY\_ID

</br>

---
### 4. Immutable Infrastructure：觀念與實作 
---

**Speaker**

- 葉秉哲
- Works @gogolook


**改題目** -> immutable infrastructure觀念與實作（建議）

因為talk是在下午第一場，怕大家看code太想睡覺，所以就不講detail實作

**演算法的領域也說，immutable 的東西會比 mutable 的東西來得好設計**

**why immutable objects** 

- simpler	to understand
- iherently thread-safe
- offer higher security than mutable objects


**immutable object**

```java
String s = "ABC";
s.toLowerCase();
```

**java hotswap**

- jdk 1.4

**Christian Posta said:**
>don't hotdeploy/redeploy/migrate your java service in production at runtime.  
>do have a very trong focus on your delivery pipeline/automation/testing to quickly make change to your app


**What's an immutable infrastructure**

- quote from Docker 大神

>re-create images each time you change a line of code.  
>prevent modifications of running images.

**Why immutable infrastructure?**

- Simplify change management
	- hard to keep or restore "desired state" in place
- Enforce `dev/prod` parity
	- configure & test `infra` **before** deployed to production environment
- Reason about apps at a higher level
	- ... than just `deployable pacakages` containing the code (jar/war/zip/msi)

**Why `NOT` immutable infrastructure?**

- Cost may be too high
- DevOps maturity level

**講者把image視為一種`母體`**  
主要分為以下三種，  

* VM image
* Container image
* Unikernel image
	- 對現有container技術有一個比較激進的改良
	- 傳統的vm，架構在hypervisor上，也打包一個os在裡面
	- docker不打包，但會共用底下os的東西，有安全性考量
	- 所以有了`container per VM`，但還是太肥了。
	- 把作業系統也看成lib，不會把所有os通通包進來，只抽取它要的。
	- [boxfuse] (https://boxfuse.com/)

**母體 => 增生 => 替換 => 自動化**



</br>

---
### 5. JDK8 JIT 行為和效能分析
---

這個session是Jserv在講，  
聽過Jserv演講過幾次，  
也曾經在去旁聽過，他在台大講的Android Open Source，  
給我的印象就是高手、講話很快、講話很有梗的。  
其實我原本要去聽別場的`MySQL Connector/J`，但一方面覺得實在太累了，  
懶得換room，又想輕鬆一點，聽一下Jserv講講笑話，  
Jserv一樣講得很專業，但真的是不同領域，幾乎都聽不懂。

**Speaker**

- Jserv

**topic**

- How JIT works?
- How to monitor the JIT?
- How to figure out performance problem?

**jserv自己準備的共筆**

- [url] (https://jcconf.hackpad.com/R0-13PeQD5hpDx#:h=JDK8-JIT-行為和效能分析)

**hotspot**

-  template based interpreter

**Jserv Said:**
>
>1. 作業我自己改，門我自己開，助教負責笑  
>2. 台灣什麼東西都可以靠北
>3. 靠北JVM
>4. 怕沒人寫hackpad共筆，所以我自己寫
>5. 在聯發科工作的時候，大家都是看安兔兔的benchmark買手機
>

**JVM如何支援動態語言**

- invokedynamic
	- lambda
- java裡的method都是`ivnokevirtual`

**匿名class本質上會生出另一個class，inner class，和lambda不一樣**

</br>

---
### 6. 自己的JVM自己救 - 解救 OOM 實務經驗談 
---

這場我聽到放空了，不過前半段內容，大致上我都是先前就知道的。  
例如：stack, heap差別...etc.

**Shallow heap**


**Retained heap**


</br>

---
### 感想
---

參加完，不能說在知識上收穫很多，  
但反而是一種開眼界的感覺，會覺得人外有人，天外有天，  
覺得自己還得更加努力才行。

還想推薦一下MySQL的speaker (Ivan Tu)，雖然沒有去聽他的talk，  
但在休息時間時，去到MySQL攤位，都會聽到他很熱情地講解。




