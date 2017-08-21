---
layout: post
title: "Nginx: Block traffic by country and bypass traffic for whitelist"
description: ""
category: 
tags: []
---
{% include JB/setup %}

由於我們的服務都是由n台API Server + 各個client (Android, iOS, HTML5)所組成，然後我們的服務在某些國家是不開放的，所以勢必得做到block by country，然後又要有白名單機制；做法有很多種，但都大同小異，譬如說用iptables, nginx，然後搭配一些IP to country module（[ip2location](https://www.ip2location.com/), [GeoIP](http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz)）去阻擋，這篇就會使用Nginx + [GeoIP](http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz)來阻擋。


<br />

---
### 1. Check Nginx version
---

首先先看看你安裝的Nginx是否有支援`geoip`這功能，如果有`grep`東西出來就代表有支援。

```bash
$ nginx -V 2>&1 | tr ' ' '\n' | grep geoip
--with-http_geoip_module
```

<br />

<!--more-->

---
### 2. Download GeoIP
---

如果Nginx有支援`geoip`，那就去下載GeoIP的gz檔下來，然後再解壓縮至`/etc/nginx`底下。

```bash
wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz | gzip -d - > /etc/nginx/GeoIP2.dat
```

<br />

---
### 3. Configure nginx.conf & default configuration file
---

這步驟要來設定`nginx.conf`和`default`這二個檔案，首先先設定nginx.conf，在http的block中加入下面的設定，
一個是geoip的地址，另一個是白名單的IP，以下面的example來說，`8.8.8.8`是白名單，所以就把value設成1。
然後如果你nginx前面是有掛proxy或者CDN的，記得不要使用`remote_addr`，要拿`XFF`或者拿CDN提供的Client IP（每個CDN提供的header不同）。

#### `1. /etc/nginx/nginx.conf`

```
http {
    geoip_country /etc/nginx/GeoIP.dat;
    geo $remote_addr $ip_whitelist {
        default 0;
        8.8.8.8 1;
    }
    ....
}
```
<br />

接著就來設定default這個檔案，在這檔案中的location block加入下面的判斷，因為nginx的config沒辦法吃if-else和nested if，他只能吃單一if，所以變成寫法很醜，下面的判斷先去看我們自己的IP是不是在白名單裡面，如果是就直接break；如果不是的話就會繼續往下看來源的國家是不是在`HK, TW, PH, US`這幾個地區，如果是這幾個地區就直接回404給用戶。

#### `2. /etc/nginx/sites-available/default`

```
location / {
    if ($ip_whitelist = 1) {
        break;
    }

    if ($geoip_country_code ~ "(HK|TW|PH|US)") {
        return 404;
    }
    ...
}
```
<br />

最後就restart nginx，就可以進行verify。

<br />

---
### 4. Advanced topic: CDN
---

剛好因為我們前面有掛CloudFlare CDN，其實我們不用整合`geoip`就可以達到上述的效果，因為CloudFlare會帶上`HTTP_CF_IPCOUNTRY`這個header給我們的source(nginx)；因此如果前面有CDN的話，就不需要上面的第二步驟（下載GeoIP）以及第三步驟的    `geoip_country /etc/nginx/GeoIP.dat`，只需要把`default`配置改成下面的方式就可以了。

```

location / {
    if ($ip_whitelist = 1) {
        break;
    }

    if ($http_cf_ipcountry ~ "(HK|TW|PH|US)") {
        return 404;
    }
    ...
}
```






<br />
<br />
<br />


