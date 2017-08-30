---
layout: post
title: "Firebase Realtime Database (Web)"
description: ""
category: 
tags: [firebase, realtime, database, web, javascript]
---
{% include JB/setup %}

{: style="font-size: 16px;"}

最近要替我們的服務加一個類似公告的服務，想了好幾個solution，譬如說在我們既有的API Server上加一隻API，讓前端獲取目前最新的公告，
但這麼做的缺點就是前端得用pulling or long pulling的方式，不管哪個方式對Server來說都是一種負擔，再加上把公告的獲取放在API Server的話，又得處理zero downtime的問題，基於以上的原因，所以打算使用[Firebase realtime database](https://firebase.google.cn/docs/database/)。 <br />
[Firebase realtime database](https://firebase.google.cn/docs/database/)是一個即時的資料庫，當你更改任一個在database的值時，會立即的同步到各個已經連結到資料庫上的client上面，傳統作法是得讓client自己定期去DB query，而Firebase會主動的讓各個client知道，然後目前已經支援非常多種client了，像是普遍的iOS, Android，而我這篇是選擇使用[Web](https://firebase.google.com/docs/database/web/start)，我可以在前端頁面上，使用JavaScript與Firebase realtime database做一個連結。

<br />

<!--more-->

---
### 1. Add Firebase to your JavaScript Project
---

{: style="font-size: 16px;"}

要將Firebase加入至我們的App之前，要先完成下面的步驟

{: style="font-size: 16px;"}

1. 先去[Firebase後台](https://console.firebase.google.com/)建立一個project.
2. 點擊 `Add Firebase to your web app`.
3. 畫面會出現類似下方的code snippet，不過由於我會使用webpack，所以下方的只有config對這篇是有意義的.



```java
<script src="https://www.gstatic.com/firebasejs/4.3.0/firebase.js"></script>
<script>
  // Initialize Firebase
  var config = {
    apiKey: "<API_KEY>",
    authDomain: "<PROJECT_ID>.firebaseapp.com",
    databaseURL: "https://<DATABASE_NAME>.firebaseio.com",
    projectId: "xxxxxxxxxx",
    storageBucket: "<BUCKET>.appspot.com",
    messagingSenderId: "<SENDER_ID>"
  };
  firebase.initializeApp(config);
</script>
```

<br />


---
### 2. Initialize the Realtime Database JavaScript SDK
---

{: style="font-size: 16px;"}

initial我們的project以後，接著就要在我們的project中加入JavaScript SDK，由於我的project中有使用webpack，所以可以使用npm的方式進行install SDK。

```bash
$ npm install --save firebase
```

{: style="font-size: 16px;"}

安裝完以後，就可以在project中import Firebase SDK，記得將下面的參數替換成自己project中的參數。

```javascript
import firebase from 'firebase/app';
import 'firebase/database';
import 'firebase/auth';
const config = {
    apiKey: "<API_KEY>",
    authDomain: "<PROJECT_ID>.firebaseapp.com",
    databaseURL: "https://<DATABASE_NAME>.firebaseio.com",
    projectId: "xxxxxxxxxx",
};
firebase.initializeApp(config);

```

<br />

---
### 3. Setup Database authentication rule
---

{: style="font-size: 16px;"}

接著要去後台設定Database的權限，意思是說讀/寫時，是否需要authentication，又或者authentication的方式是什麼。這篇會將讀的權限設為全部都可以讀，不需要authentication;但寫就必須要有權限。

```json
{
  "rules": {
    ".read": true,
    ".write": "auth != null"
  }
}
```

<br />

{: style="font-size: 16px;"}

有了rule以後，就要去`Authentication`中設定`誰`可以去寫database，在`Authentication`中可以使用很多種方式進行身份的驗證，像是Google Login的等等之類的，我這裡選擇比較簡單的`Email`的方式，如下圖中，我建立了二個Email待會用來做身份驗證所使用的。

![Ticket]({{site.url}}/assets/2017-08-28-1.png)


<br />

---
### 4. Sign In With Email
---

{: style="font-size: 16px;"}

有了Email以後，接著我們就可以用來登入，如果登入


```javascript
let isLogin = true;
firebase.auth().signInWithEmailAndPassword('<EMAIL>', '<PWD>').catch(function(error) {
    if (error) {
        isLogin = false;
    }
});
```


<br />

---
### 5. Write and Read Data
---

{: style="font-size: 16px;"}

完成了上面的身份驗證，就可以對Firebase Realtime Database進行讀寫資料了，如果沒有進行上面的身份驗證，你在write的時候，console會吐`PERMISSION_DENIED`，下面的例子就是在announcement底下寫一個title是`Hello World`的公告。我用的是`set`這方法，要注意一點，用`set`可是會去overwrite掉底下node所有的資料，所以如果你想更新部分資料的話，就要用`update`。

```javascript
const database = firebase.database();
firebase.database().ref('announcement/latest').set({
    'title': 'Hello World'
});
```

<br />

{: style="font-size: 16px;"}

成功以後，你就可以去後台看，會看到下面的JSON結構。

```json
{
  "announcement" : {
    "latest" : {
      "title" : "Hello World"
    }
  }
}
```

<br />

{: style="font-size: 16px;"}

要去Read Firebase Realtime Database上的資料有兩種方式，

{: style="font-size: 16px;"}

1. `on`：用on的話，一旦只要data變更了，client都會馬上收到通知
2. `once`：once則相反，只會拿到一次，之後有變更都不會收到通知


```javascript
const announcement = database.ref('announcement/latest');
announcement.on('value', function(snapshot) {
    console.log(snapshot.val().title);
});
```

{: style="font-size: 16px;"}

你就會在console看到`Hello World`，如果直接在Realtime Database後台更改value，console又會即時印出更改過後的value。





<br />
<br />
<br />












