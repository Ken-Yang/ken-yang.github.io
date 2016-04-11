---
layout: post
title: "Getting started with Kubernetes"
description: ""
category: 
tags: [gce,gke, docker, container, kubernetes]
---
{% include JB/setup %}

<font size=3>

Docker玩了一陣子，總覺得沒有玩一些container management的service好像少了點什麼，</br>
剛好team裡用到[Kubernetes](http://kubernetes.io/docs/)，所以這裡就記錄一下怎麼使用[Kubernetes](http://kubernetes.io/docs/)。</br>
然後這篇會把Kubernetes架在Google Cloud Platform上面，所以還得去安裝Google Cloud SDK。


</br>

---
### 1. Downloading Kubernetes
---

第一步驟要先安裝Kubernetes，安裝方式有好幾種，

1. tarball解開
2. Build from source
3. Homebrew
4. Remote shell script 
5. 用Google Cloud SDK安裝

各個的詳細步驟可以去[Kubernetes安裝頁面](http://kubernetes.io/docs/getting-started-guides/binary_release/)看，我自己是選擇透過Google Cloud SDK來安裝，所以首先要先安裝Google Cloud SDK。


</br>
**安裝Google Cloud SDK**

Google Cloud SDK這個tool讓你可以對Google Cloud Platform進行操作。安裝指令如下，Default installation path會在你的home目錄底下，

```bash
$ curl https://sdk.cloud.google.com | bash
```
</br>
接著要restart shell和設定gcloud environment，`gcloud init`這個指令會彈出browser要你login，以及要你輸入default zone，我是選asia-east1-a。

```bash
$ exec -l $SHELL
$ gcloud init
```
</br>
**安裝kubectl**

```bash
$ gcloud components install kubectl
```


</br>

<!--more-->

---
### 2. Setup Google Cloud
---

在這步驟要做幾件事情，

1. 註冊[Google Cloud](https://console.cloud.google.com/)
2. Create project
3. 打開billing
4. 開啟Container Engine API.




</br>
**Step 1. 註冊[Google Cloud](https://console.cloud.google.com/)**

點了上面的連結至console以後，就發現其實是以前的Google App Engine+Google APIs的後台，</br>
只不過好久沒用了，發現改版改好多..
然後又發現新註冊的user有免費300美金的quota可以使用，</br>
舊有的用戶還沒有這300美金可以用，所以建議可以乾脆註冊新帳號使用比較好。

</br>
**Step 2: Create project**

有了帳號以後，就去Create project，Create時的project名稱，</br>
如果project名稱沒有重複的話，應該就會是你的`PROJECT_ID`。

</br>
**Step 3: 打開billing**

這一步驟就是[點這個連結](https://console.developers.google.com/billing)去打開Billing，進去以後要填寫信用卡資料，目的是將來要收費，</br>不過如果你是新註冊的話，一開始先不用擔心這個問題，因為有300美金可以使用。

</br>
**Step 4: 開啟Container Engine API**

接著要去Enable Container Engine API，[點這個連結](https://console.cloud.google.com/project/_/kubernetes/list)去enable，進去以後就選取剛剛create的project，然後按下Continue。(如下圖)

![cmd]({{site.url}}/assets/2016-04-08-enable-container-api.png)



</br>

---
### 3. Create your Node.js application
---

接著來寫一個簡單的Node.js application，等等會把這application包成docker image，</br>
先create一個folder，然後建立一個server.js，

```bash
$ mkdir hellonode-app
$ cd hellonode-app
$ vim server.js
```
</br>
再把下面的內容貼入到server.js當中，下面的內容就是建立一個http server，且listen在8080 port上，

```javascript
var http = require('http');
var handleRequest = function(request, response) {
  response.writeHead(200);
  response.end("Hello World!");
}
var www = http.createServer(handleRequest);
www.listen(8080);
```

</br>

---
### 4. Create and Push a Docker container image
---

接著要把上面的application變成一個docker image以及把image push上去，所以首先要先建立一個`Dockerfile`，Dockerfile作用在之前的[文章](http://blog.kenyang.net/2015/11/30/how-to-use-nodejs-mongodb-with-docker)就有解釋過了，
</br>
簡單說就是用來定義這個image的資訊。

```bash
FROM node:0.12
EXPOSE 8080
COPY server.js .
CMD node server.js
```
</br>
有了`Dockerfile`以後，就可以來build image了，記得把下面的`ken-kubernetes-lab`換成你們自己的project id，

```bash
$ docker build -t gcr.io/ken-kubernetes-lab/hello-node:v1 .
```

</br>
build完以後，可以在自己local測試看看，

```bash
$ docker run -d -p 8080:8080 gcr.io/ken-kubernetes-lab/hello-node:v1
$ curl http://localhost:8080
```

</br>
接著就可以把image push上去Google Container Registry，

```bash
$ gcloud docker push gcr.io/ken-kubernetes-lab/hello-node:v1
```

</br>
---
### 5. Create a cluster
---

接著去console，建立一個cluster，如下圖。

![cmd]({{site.url}}/assets/2016-04-08-create-cluster.png)


</br>
建完以後，就把cluster的info餵給kubectl，記得把`cluster-1`改成你的cluster名稱。

```bash
$ gcloud container clusters get-credentials cluster-1
``` 

</br>
---
### 6. Create a pod
---

`pod`在kubernetes的定義是`pods are the smallest deployable units of computing that can be created and managed in Kubernetes.`，
</br>
簡單說，如果你要在kubernetes的cluster裡面run起來`一組`container，就得有一個pod，</br>
`一組`container的意思是，這個pod裡面由一個以上的container所組成。
</br></br>
用kubectl run來建立一個pod:

```bash
$ kubectl run hello-node --image=gcr.io/ken-kubernetes-lab/hello-node:v1 --port=8080
deployment "hello-node" created
```

如上面的output所示，`kubectl run`建立了一個`deployment`，`deployment`主要是用來建立或者scale一個pod的一種方式，以這個例子來說，deployment管理了1個pod replica，想看deployment的資訊，就打：

```bash
$ kubectl get deployments
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   1         1         1            1           3d
```



</br>
---
### 7. Allow external traffic 
---

pod預設只能被內部access，如果想要讓外部存取，必須把`pod` expose成一個`service`:

```bash 
$ kubectl expose deployment hello-node --type="LoadBalancer"
```

你會發現上面的指令，我們是expose deployment這個object出來，而不是pod，</br>
如上面所說，deployment可能會管理多個pods，所以可以透過deployment來當作load balancer。
取得service ip的指令，請打:

```bash
$ kubectl get services hello-node
NAME         CLUSTER-IP       EXTERNAL-IP       PORT(S)    AGE
hello-node   10.123.244.242   130.211.247.198   8080/TCP   3d
```

`EXTERNAL_IP`可能需要一點點時間才會顯示出來，所以如果`EXTERNAL_IP`一開始是空白的，請等一下再試一次。


</br>
---
### 8. Scale up your website
---

Kubernetes其中最強大的一點就是可以很輕鬆的scale你的application，
</br>
假設你的application突然需要更多的capacity，你可以簡單地叫`deployment`去建立新的replica:

```bash
$ kubectl scale deployment hello-node --replicas=4
```

現在你的application就有4個replicas，每一個獨立地在cluster中運作，且load balancer去serve traffic。

```bash
$ kubectl get deployments
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   4         4         4            4           3d
```

```bash
$ kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
hello-node-3344141985-j2zun   1/1       Running   0          2m
hello-node-3344141985-j70fh   1/1       Running   0          2m
hello-node-3344141985-lzynx   1/1       Running   0          3d
hello-node-3344141985-x3ycw   1/1       Running   0          2m
```

</br>
---
### 9. Roll out an upgrade to your website
---

假設你的application有bug fixes，Kubernetes也可以輕易地幫助你deploy新版本上去。</br>
首先，先來改剛剛的`server.js`，修改response message:

```javascript
  response.end("Hello Kubernetes World!");
```

接著可以build及publish新版本至registry:

```bash
$ docker build -t gcr.io/ken-kubernetes-lab/hello-node:v2 . 
$ gcloud docker push gcr.io/ken-kubernetes-lab/hello-node:v2
```

push完成以後，我們就有一個v2版本的image可以使用，接下來只要透過`kubectl edit`指令就可以進行update，
</br>
這個指令會打開一個text editor，內容是一個deployment的yaml config，</br>
我們只需要把**`gcr.io/ken-kubernetes-lab/hello-node:v1`**改成**`gcr.io/ken-kubernetes-lab/hello-node:v2`**就好。

```bash
$ kubectl edit deployment hello-node
```

編輯完成以後，可以去看看pods有什麼變化:

```bash
$ kubectl get pods
NAME                          READY     STATUS        RESTARTS   AGE
hello-node-3344141985-j2zun   1/1       Terminating   0          19m
hello-node-3344141985-j70fh   1/1       Terminating   0          19m
hello-node-3344141985-lzynx   1/1       Terminating   0          3d
hello-node-3344141985-x3ycw   1/1       Terminating   0          19m
hello-node-3422850722-b0niu   1/1       Running       0          13s
hello-node-3422850722-gu7c3   1/1       Running       0          19s
hello-node-3422850722-oaqw7   1/1       Running       0          19s
hello-node-3422850722-ria1t   1/1       Running       0          13s
```

從output中會發現，deployment建立新的pod，把舊版的pod關掉了。

</br>
---
### 10. Delete
---

由於只有300美金啊，雖然我開了三天也才花費五塊美金，不過沒在用還是關掉吧！</br>
刪除Deployment時，也會一併刪除pod；刪除service會刪除load balancer。

```bash
$ kubectl delete service,deployment hello-node
```

最後記得cluster也要刪除:

```bash
$ gcloud container clusters delete cluster-1
```

</br>
</br>
</br>
</br>
</br>


 
</font>