---
layout: post
title: "Docker Container Summit 2015   Day 1"
description: ""
category: 
tags: []
published: false
---
{% include JB/setup %}

<font size=3>

---
### 1. Opening: iThome副總編輯
---

* Google用了20億個Container
* google覺得docker的設計更好
* docker還在測試，redhat就要內建
* 競爭對手VMware搶著喊合作
* openstack與container:magnum
* Contiainer OS
	* photon
	* coreos
	* atomic
	* rancher os
* 一片Raspberry Pi能跑2334個container
* 超級電腦cray也要支援docker
* 單主機跨入多主機架構
* **特色**： 精簡架構，提高資源利用率
* **目標**：將大量workload整併到


</br>
---
### 2. 以Mesos搭建大規模Container平台
---

**Speaker**

- Timothy Chen
- 美國Mesosphere分散式系統首席工程師
- Apache Mesos, Drill PMC
- Maintain Apache Spark on Mesos, Docker Swarm on Mesos

**Applications don't fit on a single machine anymore**

**Mesos**

- resource mgmt
- programming abstractions
- security
- monitoring, debugging, logging
- 提供一個API讓你更快速的rescale
- 會把沒在用的resource給其他task用
	- 假設給nodejs這個task 8G ram，但只用到2G，剩下的會給其他task用

**who use**

- siri (從open stack搬到mesos）

**Scale最好，最stable**

**Ref**

- [Swarm v. Fleet v. Kubernetes v. Mesos](http://radar.oreilly.com/2015/10/swarm-v-fleet-v-kubernetes-v-mesos.html)
- [What's the difference between Apache Mesos and Google's kubernetes?](https://kismatic.com/community/apaches-mesos-vs-googles-kubernetes/)

**Cronos**

- Distributed Cron
- REST interface


</br>
---
### 3. Docker in CI：Cacoo與Backlog導入Docker的經驗分享
---

**Speaker**

- 染田貴志
- NuLab Growth Hacker
- [nulab-inc.com](http://nulab-inc.com/)
	- Backlog (recruiting)
	- Cacoo
	- [Typetalk] (http://www.typetalk.in/)


**Use case of Docker**

- Using Docker for Job Execution
- Using Docker for CI cluster

**Nulab development overview**



**Use Case in CI**
**Summary**

</br>
---
### 4. 談Docker對傳統DevOps工具鏈的衝擊
---

**Speaker**
- 葉秉哲

**DevOps是什麼？**

**真正嚴肅的DevOps問題只有三個**

1. How to **recreate** your system?
2. How to safely **change** your system?
3. When something has gone **wrong** 問題發生，可以很快知道


**1. How to recreate your system?**

分三點來談，

1. 機器
2. 組態
3. 角色

**作業系統有必要這麼肥嗎？**

所以有了給container用的os出來

1. coreos
2. rancheros
3. red hat atomic
4. VMware photon
5. snappy ubuntu core
6. windows nano server


**docker的隔離性沒有傳統的vm好**

雖然docker很快，但隔離性不夠好，有些人想到一個怪招，每個container就跑在每一個vm裡面，這類型的container per vm概念，裡面的os不會這麼肥，很輕薄短小。  

有些人覺得還不夠，所以有了unikernel概念，

docker -H tcp://0.0.0.0:2375 info

**2. How to safely change your system?**

1. in-place update(就地改變)
2. [immutable structure](http://radar.oreilly.com/2015/06/an-introduction-to-immutable-infrastructure.html)

**roling upgrade**

```
kubectl rolling-update my-nginx --image=nginx:1.9.1
```



把電腦當成牲畜看待，不要當作寵物一樣，壞了就很擔心，
要像對待乳牛一樣！壞了在抓別隻來擠就好！


</br>
---
### 5. 如何在Kubernetes上持續部署微服務型應用
---

**Speaker**

- Evan Brown / Google 雲端解決方案架構師

**talk**

- simple microservice
- kubernetes
	 -  manage a cluster of linux containers as a single system
	 -  **cluster**: VMs that kubernetes manage
	 -  at least 1 master, * Nodes.
	 -  **pod: pod.yaml**
		-  configuration of the container
	 	-  co-located docker containers. 一群container。
	 	-  share volumes and localhost
	 	-  `kubectl create -f pod.yaml`
	 	- `kubectl exec -it pod-demo sh` (the same with docker cmd)
	 - **rc.yaml**
		- replication controller
	 - **service: svc.yaml**
		- abstraction to communicate to pods
		- 像是proxy一樣
	 - **namespace**
- continuous deployment



</br>
---
### 6. 如何搭建企業級容器服務
---

**Speaker**

- 梁胜 / Rancher Labs 共同創辦人暨執行長
- started cloud.com in 2008
- cloud.com built cloudstack, and sold to many large enterprises and service providers
- Citrix acquird cloud.com in 2011
- Left Citrix and started Rancher Labs in 2014


**Rancher**

- open source container management platform for building a private container service


</br>
---
7. 	深入瞭解Docker Container Networks
---

**Speaker**

- Willy Kuo

**--link可能將來會被砍掉**



**docker network modes (1.9之前就有)**

* bridge (default)
* none
* host
* container
* used-defined network

**docker netowrk commands**

* docker network create
	* docker network create -d birdge bride-ken
	* docker run -it --net=bridge2 ubuntu
	* ifconfig (in different subnet)
* docker network connect
* docker network ls
* docker network rm
* docker network disconnect
* docker network inspect

**overlay network**

* 把不同host的container串起來


**weave**

為何一定要連起來？  
不能透過expose port的方式，  
然後aws上的連去azure上的機器嗎？  
連起來有什麼特別意義？


</br>
---
8. 以Docker swarm打造多主機cluster環境
---

**Speaker**

- linking container together
- linking docker engine
- multi-host networking
	- docker swarm on multiple VM
	- docker swarm on multiple cloud instance

github.



</br>
---
1. Clone cobalt2
---


git clone https://github.com/wesbos/Cobalt2-iterm.git  
git clone https://github.com/powerline/fonts.git

```
./install.sh
```

 curl -L https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh | sh
 
 ```
 chsh -s /bin/zsh
 ```
 
 ```
 vim ~/.zshrc
ZSH_THEME="agnoster"
 ```
 





</font>