---
layout: post
title: "Setup HTTPS and HTTP/2 on Nginx"
description: ""
category: 
tags: [SSL, HTTPS, Nginx]
---
{% include JB/setup %}

{: style="font-size: 16px;line-height: 2;"}

最近發現某台提供HTTPS的server，在Android手機瀏覽器上訪問，常常被提示說不安全的憑證，<br />於是就去[這個檢測網站 https://www.ssllabs.com/ssltest/analyze.html](https://www.ssllabs.com/ssltest/analyze.html)做檢測，發現結果是分數是B，且有chain issue。
之後去server上檢查Nginx confg，發現我忘記放intermediate cert了，於是就把intermediate cert和cert合併再一起就解決。因此這篇筆記會紀錄：

- 為了避免忘記所以決定紀錄一下設定HTTPS的步驟，
- 如何讓HTTPS評分為A+。
- 設置HTTP2

<!--more-->


---
### 1. HTTP 1.1和HTTP/2的差別
---

{: style="font-size: 16px;line-height: 1.6;"}

HTTP 1.1是在1999年出現的，至今已經19年都沒有任何變化，但隨著前端技術的發展，使得HTTP 1.1的缺點也慢慢變放大，因此有了HTTP/2的產生，為了解決以下的問題：

{: style="font-size: 16px;line-height: 2;"}

- HTTP 1.1限制同時最多只能對一個域名發出6個請求（看瀏覽器的實作），因此有了些奇淫技巧，像是domain sharding，但HTTP/2的request是平行下載，不用依序下載，且也不需要建立多個TCP connection
- HTTP Header被壓縮了
- 資料傳輸上更有效率，因為是用binary形式，而不是文字檔
- Server可以主動的push資料給client，舉例有個index.html，server會知道裡面有哪些js/css，他就會主動一併推送給client，而不需要等client解析index.html才發請求給server


{: style="font-size: 16px;line-height: 1.6;"}

簡單說，就是用戶等待頁面的時間變短了，可以更快地看到網站內容。



<br />

---
### 2. 確認Nginx版本在1.9.5以上
---

{: style="font-size: 16px;line-height: 1.6;"}

因為Nginx在1.9.5以上的版本才有支援HTTP/2，因此請確認Nginx版本。

```bash	
$ nginx -v
```


<br />

---
### 3. 啟用HTTP/2
---

{: style="font-size: 16px;line-height: 1.6;"}

編輯檔案`/etc/nginx/site-enabled/default`，然後找到listen那行，在後面加上`http2`就可以。

```bash
listen 443 ssl http2 default_server;
listen [::]:443 ssl http2 default_server;
```

<br />

---
### 4. 產生Key+CSR以及下載Cert
---

{: style="font-size: 16px;line-height: 1.6;"}

接著我們用下面的指令產生一把private key以及CSR，記得key千萬別搞丟，然後再把CSR貼入至購買憑證的網站後台上，接著就可以下載intermediate cert和cert了。

```bash
» $ openssl req -nodes -newkey rsa:2048 -sha256 -keyout your_domain.key -out your_domain.csr
Generating a 2048 bit RSA private key
..............................................................................................................+++
..+++
writing new private key to 'your_domain.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) []:TW
State or Province Name (full name) []:Taiwan
Locality Name (eg, city) []:Taipei
Organization Name (eg, company) []:None
Organizational Unit Name (eg, section) []:None
Common Name (eg, fully qualified host name) []:*.your_domain
Email Address []:xxx@hi.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
```

<br />

---
### 5. 合併intermediate cert和cert
---

{: style="font-size: 16px;line-height: 1.6;"}

憑證下載以後，把這二張cert以及key上傳至server上，由於我們是使用Nginx，如同前面說的，Nginx不能設置intermediate cert，因此需要把intermediate cert和cert合併在一起，但如果你是Apache的話，就不需要合併，Apache有一個額外的設定。

```bash
$ cat cxxxxxxxxx.crt gd_bundle-g2-g1.crt > chained.crt
```

Note: 我有修改gd_bundle-g2-g1.crt，因為多了一個節點，所以我把最下面的節點刪除，這樣做檢測時，才不會有`Chain issues Contains anchor`


<br />

---
### 6. 設定SSL
---

{: style="font-size: 16px;line-height: 1.6;"}

接著編輯檔案`/etc/nginx/site-enabled/default`，然後修改你的憑證位置。

```bash
ssl_certificate /var/ssl/chained.crt;
ssl_certificate_key /var/ssl/your_key.key;
```

<br />

---
### 7. 避免使用老舊且不安全的Cipher
---

{: style="font-size: 16px;line-height: 1.6;"}

編輯檔案`/etc/nginx/site-enabled/default`，然後貼入(修改成)下方內容

```
ssl_prefer_server_ciphers on;
ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
```


<br />

---
### 8. 加強key交換的安全性
---

{: style="font-size: 16px;line-height: 1.6;"}


在建立連線之前，第一步是server和client之間先交換key，但在這交換的過程中是沒有任何加密的，所以任何人都可以看得到，
因此我們需要用`DHE` key去保護。

```
$ openssl dhparam -out dhparam.pem 4096
```

在檔案`/etc/nginx/site-enabled/default`，加入下面這行。

```
ssl_dhparam /var/ssl/dhparam.pem;
```



<br />

---
### 9. Cache SSL
---

{: style="font-size: 16px;line-height: 1.6;"}

跟HTTP相比，HTTPS通常會花比較久在建立連線，為了降低page load的速度，所以要啟用cache機制，也就是說每個請求上時，不會重新建立一個session，而是會沿用。

在檔案`/etc/nginx/site-enabled/default`，加入下面這二行。

```
sl_session_cache shared:SSL:10m;
ssl_session_timeout 1h;
```


<br />

---
### 10. 啟用HTTP Strict Transport Security (HSTS)
---

{: style="font-size: 16px;line-height: 1.6;"}

在這我們加入一個HSTS的header，目的是為了讓瀏覽器發現有此header時，他就不會用HTTP嘗試去connect server，

在檔案`/etc/nginx/site-enabled/default`，加入下面這行。

```
add_header Strict-Transport-Security "max-age=63072000; includeSubdomains; ";
```

<br />

---
### 11. 重啟及測試
---

{: style="font-size: 16px;line-height: 1.6;"}

```
$ service nginx restart
```

{: style="font-size: 16px;line-height: 1.6;"}

然後就可以再去這個網站[https://www.ssllabs.com/ssltest/analyze.html](https://www.ssllabs.com/ssltest/analyze.html)做檢測，分數應該就會是A+了。









<br />
<br />
<br />














