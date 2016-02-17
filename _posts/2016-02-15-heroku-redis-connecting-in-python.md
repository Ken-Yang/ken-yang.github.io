---
layout: post
title: "Heroku Redis: Connecting in Python"
description: ""
category: 
tags: [heroku, redis]
---
{% include JB/setup %}

<font size=3>

這篇要講怎麼在Heroku上，使用它的[Heroku Redis](https://devcenter.heroku.com/articles/heroku-redis)，  
大致上分為三個步驟，分別為安裝，配置，寫code。


</br>

---
### 1. Installing CLI
---

第一步驟是要先安裝heroku-redis的command line tools，
但如果你不想要用command line，你也可以至heroku的後台裡面的Add-ons加一台redis。

```bash
$ heroku plugins:install heroku-redis
```

</br>

---
### 2. Provisioning the add-on
---

接著要配置一台Redis server，有二個比較特別的參數，

1. hobby-dev: 這個是這台server的[Plan & Pricing](http://elements.heroku.com/addons/heroku-redis)，hobby-dev是免費的。
2. sushi: 你的heroku app name

```bash
$ heroku addons:create heroku-redis:hobby-dev -a sushi
```


</br>
當你成功配置一台redis以後，你可以透過下面的指令找到該台redis URL，
<!--more-->
```bash
$ heroku config | grep REDIS
```

</br>

---
### 3. Connecting in Python
---

要在Python中使用Redis，你必須安裝`redis`這個package，

```bash
$ pip install redis
$ pip freeze > requirements.txt
```
</br>
接著就可以用這個`redis`這個package連結至redis server，  
下面的example是用Flask這個framework來做示範，分別有`get`和`set`這二個URL。

```python
import os
import redis

r = redis.from_url(os.environ.get("REDIS_URL"))

@app.route('/get')
def get():
    if r.get("name") is None:
        return "no such key in Redis"
    else:
        return r.get("name")

@app.route('/set')
def set():
    r.setex('name','Ken Yang',10)
    return "set done"
```

</br>
---
### 4. Testing in local environment
---

如果你想要在local測試，需要把REDIS_URL的參數寫在.env裡面才行。  
否則會找不到Redis在哪。

```bash
$ heroku config:get REDIS_URL -s  >> .env
$ heroku local web
```
</font>