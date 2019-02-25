---
layout: post
title: "Upgrade NGINX to latest version on CentOS"
description: ""
category: 
tags: [nginx]
---
{% include JB/setup %}

{: style="font-size: 16px;line-height: 2;"}

這篇主要是要說明如何在centos7把NGINX升級至latest stable version。


---
### 1. Add NGINX yum repository
---

{: style="font-size: 16px;line-height: 1.6;"}

首先，先建立一個名稱為`/etc/yum.repos.d/nginx.repo`的檔案，並且把下列的設置貼上:

```bash
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```

接著就執行

```bash
$ yum update
```

<br />

---
### 2. Install NGINX 
---

完成上述的設置，就可以安裝最新版本NGINX，安裝前可以檢查下版本。

```bash
$ yum list nginx
Loaded plugins: fastestmirror, post-transaction-actions, rpm-warm-cache, versionlock
Loading mirror speeds from cached hostfile
 * fasttrack: mirrors.aliyun.com
Available Packages
nginx.x86_64                                       1:1.14.2-1.el7_4.ngx                                       nginx
```

如果確認版本為當時最新的，那麼就可以直接安裝了。

```bash
$ yum install nginx
```

<br />




<br />
<br />

