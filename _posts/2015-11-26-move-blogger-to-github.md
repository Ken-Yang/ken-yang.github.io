---
layout: post
title: 如何把blogger搬到github pages
date: '2015-11-26'
author: Ken Yang
tags:
- blogger
- github
- jekyll
---

最近一直很想從Google Blogger中離家出走，因為在blogger寫筆記實在太麻煩了，  
每一個筆記的format每次都調得好累，  
且我很常在local寫一份markdown，  
這樣等於我要寫二次..  
所以就動起了念頭要搬到一個支援markdown格式的地方。  
那我選的是github pages，然後搭配著jekyll這個tool去做publish。  
所以這篇會圍繞在下面的主題，

1. 建立github repo
2. 用jekyll-bootstrap建立一個template
3. 用jekyll-import把blogger資料轉出來
4. 安裝jekyll
5. 安裝paginator
6. 整合Adsense
7. 整合comment system


## 1. 建立github repo

首先要先去[github](https://github.com/new)的網站建立一個新的repo，  
repo名稱為`your_name.github.io`，  
所以我的就是`ken-yang.github.io`。

</br> 
  
## 2. jekyll-bootstrap


然後使用[jekyll-bootstrap](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)來快速地建立一個template，  
這個template是以boostrap為基底。  
使用方式如下（記得把`ken-yang`換成你的`your_name`)：


```mysql
mysql -uroot -psafesync osdp -e "select * from users";
select * from users;
ifconfig eth0  
echo hi
```

```javascript
var s = "JavaScript syntax highlighting";
alert(s);
```











