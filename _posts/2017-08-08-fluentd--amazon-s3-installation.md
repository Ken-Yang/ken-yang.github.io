---
layout: post
title: "fluentd + Amazon S3 installation"
description: ""
category: 
tags: [log, s3, fluentd]
---
{% include JB/setup %}



---
### 1. What's fluentd?
---

[fluentd](https://www.fluentd.org/)是一個用來搜集data的open source，以及可以對搜集來的data進行簡單的分析，
甚至可以把搜集來的data再foward出去mongodb, Elasticsearch, S3, GCP...etc.
<br />

因為我們的API有好多台，在管理log時非常不方便，必須ssh進去每一台看，或者scp出來，這方法實在太落後，
因此最近才把fluentd整進來，我們應用的情景如下：

- 安裝一台Log Server (儲存所有API上的log)
- 每一台API Server都裝有fluentd，並且把本地的log foward to Log Server
- 把本地的Log上傳至S3備份


<br />


<!--more-->


---
### 2. Install fluentd (Setup log server)
---

首先在一台linux server（我選用ubuntu 16.04）裡面進行安裝log server，
這台server是要收來自各地API的log。安裝很簡單，只要一行指令就可以：<br />

```bash
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-xenial-td-agent2.sh | sh
```

<br />
安裝完以後要更改config，cofig路徑為`/etc/td-agent/td-agent.conf`，default config裡面有很多設定，預設fluentd會起三個port，分別給不同的用途使用，不過我們不需要用到這麼複雜，所以可以把config改成下面的配置

```

# source是指資料從哪裡來的
<source>
  @type forward # log source會來自於foward的
</source>

# match是用來定義，資料來源為某個tag時，要做的處理
<match api-server-1> # 記得把這個tag api-server-1改成你要的，tag意思是會把source tag為api-server-1的歸類到這個配置裡
 @type file # 把source儲存為file
 path /var/log/td-agent/api-server-1 # 儲存路徑
 time_slice_format %Y%m%d%H # 當file超過buffer_chunk_limit時，會做切割，此配置為切割的檔名
 time_slice_wait 10m
 time_format %Y%m%dT%H%M%S%z # file裡面的時間格式
 compress gzip # 使用gzip壓縮被切割的log
 utc
</match>
```

<br />

完成配置以後，就重啟fluentd。

```bash
$ /etc/init.d/td-agent restart
```


<br />


---
### 3. Setup fluentd client on API Server
---

這步驟標題雖然是setup fluentd client，但其實fluentd沒有server與client之分，
所以安裝方式和第二步驟其實是一樣的，決定他是client還是server的關鍵在於config，
安裝指令，
```bash
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-xenial-td-agent2.sh | sh
```

<br />
安裝完以後，一樣要更改config，cofig路徑為`/etc/td-agent/td-agent.conf`，把config改成下面的配置

```
# source是指資料從哪裡來的
<source>
  @type tail # 資料會用tail的方式
  path /var/log/api-server-1/out.log # tail這個檔案
  pos_file /var/log/td-agent/api-server-1.log.pos # 紀錄out.log最後讀取的position
  tag api-server-1 # 送過去的tag，記得第二步驟的配置一致
  format none
</source>

# match是用來定義，資料來源為某個tag時，要做的處理
<match api-server-1>
  @type forward # 收到來自於api-server-1的log時，要進行foward
  heartbeat_interval 1s
  flush_interval 10s # 每10秒送一次
  
  # 送至哪個Server
  <server>
    name log-server
    host 123.123.123.123 # 記得改成自己的IP
    port 24224
  </server>

  # 如果失敗以後
  <secondary>
    @type file
    path /var/log/td-agent/forward-failed
  </secondary>
</match>
```

<br />

完成配置以後，就重啟fluentd，然後就可以去Log Server上的 `/var/log/td-agent/`底下看有沒有一個檔名為 `api-server-1.xxxxxxxx`的檔案。

```bash
$ /etc/init.d/td-agent restart
```


<br />


<br />


---
### 4. Forward log to S3
---

完成上面的前三步驟，基本上就沒問題了，這一部是進階的。fluentd有很多output plugin，像是S3, splunk, mongodb，甚至有很多人整合了Elasticsearch + Kibana，但我需求沒這麼複雜，所以先用S3就好。首先記得先去你的[S3](https://console.aws.amazon.com/s3/home?region=ap-northeast-1#)上面create a bucket。
<br />
接著要先安裝`fluent-plugin-s3`，然後可以使用fluentd提供的gem進行安裝，指令如下：

```
sudo /usr/sbin/td-agent-gem install fluent-plugin-s3
```

安裝完成之後，一樣去API-SERVER上改掉你的配置，改完以後應該每60秒會上傳一個part上去，記得在正式環境不要用60秒，配置如下：

```

# source是指資料從哪裡來的
<source>
  @type tail # 資料會用tail的方式
  path /var/log/api-server-1/out.log # tail這個檔案
  pos_file /var/log/td-agent/api-server-1.log.pos # 紀錄out.log最後讀取的position
  tag api-server-1 # 送過去的tag，記得第二步驟的配置一致
  format none
</source>

# match是用來定義，資料來源為某個tag時，要做的處理
<match api-server-1 >
  @type s3 # 上傳至S3
  aws_key_id xxxxxx # 記得換成你的key
  aws_sec_key # 記得換成你的key
  s3_bucket api-server-1 # 換成你的bucket name
  s3_region ap-northeast-1
  path logs/ # bucket name底下的folder
  buffer_path /var/log/td-agent/s3 # 在上傳上去前，緩存的地方

  time_slice_format %Y%m%d
  time_slice_wait 1m
  utc

  buffer_chunk_limit 256m
  flush_interval 60s  # 每60秒flush一次
</match>
```



<br />
<br />
<br />
<br />
<br />




