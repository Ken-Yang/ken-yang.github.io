---
layout: post
title: "Getting started with webpack"
description: ""
category: 
tags: [webpack,loader]
---
{% include JB/setup %}



最近在看Vue.js，很多文章都是搭配著Webpack來說明，所以也順便研究了一下webpack，
這篇會說明[webpack](https://webpack.js.org/)的優點、以及用途。

<br />

---
### 1. What's webpack?
---

什麼是[webpack](https://webpack.js.org/)呢？其實官網用一張圖片說明的很清楚，圖片上有幾個title，分別為:

1. bundle your assests
1. bundle your scripts
1. bundle your images
1. bundle your styles

一句話簡單說，可以想像成`打包工具` （但當然不僅於此，否則不會這麼紅）。


<br />

<!--more-->

---
### 2. Install webpack
---

官網提供了三種的安裝方式，分別為：

1. Local Installation
2. Global Installation (我使用這種）
3. Bleeding Edge

<br />

#### Local Installation

```bash
npm install webpack --save-dev

npm install webpack@<version> --save-dev
```

官方是最建議用第一種的Local Installation，
但因為是install在local node\_modules底下，即便你在此project folder底下，打`webpack`指令，也是會找不到，但如果你是使用npm script在你的project中的話，就不用太擔心，因為npm會去你的node_modules底下找webpack。

```javascript
"scripts" : {
    "start": "webpack --config mywebpack.config.js"
}
```

<br />


#### Global Installation

第二種是官方不推薦使用的Global方式，原因是你會被鎖死在某一個webpack版本，然後假設你有多個project，那有可能就會造成一些error。 

```bash
npm install webpack -g
```

<br />

#### Bleeding Edge

第三種就是geek在用的，如果想體驗新版本的話就使用此方式。

```bash
npm install webpack/webpack#<tagname/branchname>
```

<br />

---
### 3. How to bundle?
---

安裝完成以後，要來使用`webpack`來bundle一個project，首先要先建立三個檔案，分別為index.html、app.js以及util.js。

**utils.js**

```javascript
var div = document.createElement("div")
div.innerHTML="Hello worl Ken!";
document.body.appendChild(div);
```

util.js會去建立一個div element，然後把它塞入body之中。<br />

**app.js**

```javascript
document.getElementById('app').innerHTML="Your first webpack pacakge";
require("./util.js");
```

app.js會去找到id為app的element，然後把它塞入字串。

**index.html**

```html
<!DOCTYPE html>
<html lang="en">
    <body>
        <div id="app"></div>
        <script src="bundle.js"></script>
    </body>
</html>
```

你會發現index.html中，使用的js是bundle.js，卻不是先前建立的任何一隻js！因為bundle.js就是透過webpack bundle出來的js file。<br />
有了檔案以後，接下來就可以透過`webpack`來bundle，如果成功的話，就會看到bundle的過程，以及有哪幾隻檔案。

```bash
$ webpack app.js bundle.js
Hash: dc7cbcef87369e1a8e24
Version: webpack 2.2.1
Time: 79ms
    Asset     Size  Chunks             Chunk Names
bundle.js  2.78 kB       0  [emitted]  main
   [0] ./util.js 104 bytes {0} [built]
   [1] ./app.js 87 bytes {0} [built]
```

<br />

webpack會從app.js中去resolve dependency，把相關的js都bundle至bundle.js當中，而每個js都會被視為一個module，webpack會給每個module一個id，然後透過`__webpack_require__`去呼叫其他module，

完成以後，就可以打開`index.html`，就會看到對應的文字了。

<br />


---
### 4. Configuration (webpack.config.js)
---

如果我每次改一次檔案就要打一次build指令，然後又都那麼長，是不是很煩？不用擔心，哪一種工具會沒提供configuration？不管是json or yaml，所以webpack也有提供。根據上面的指令，所對應的configuration會長的像下面一樣，

**webpack.config.js**

```javascript
var Webpack = require("webpack");
module.exports = {
    entry: ["./app.js"],
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [{
            test: /\.css$/,
            loader: "style-loader!css-loader"
        }]
    }
}
```

先說明一下上面的每一個參數用意：

1. `entry` : 就是我們javascript的進入點
2. `output` : bundle出來後的檔案位置和檔案名稱
3. `module` : 定義如何處理module的邏輯，以上面例子來說，`test: /\.css$/`意思是，如果檔案為.css結尾的，就使用style-loader和css-loader來去轉換它

了解各個參數的意思以後，就可以直接在資料夾下，輸入`webpack`就可以build了，不需要再打很長串的指令。

```bash
webpack -p
```

<br />

---
### 5. What's loader?
---

`webpack`本身只能處理javascript的文件，其他靜態的檔案是沒有能力處理的，如果想要處理其他類型的檔案，譬如說css，那就要透過`loader`去將css轉換成js module，這樣我們就能用`require`的方式去使用css。



<br />

---
### 6. How to use loader?
---

loader在npm上，有很多種，有html template、css、i18n、url-loader、json等等之類的。<br />
我會以css loader和url-loader來做說明，
<br />

#### css-loader style-loader


首先先安裝`css-loader`和`style-loader`，

```bash
$ npm install css-loader style-loader --save-dev
```

<br />
然後來撰寫我們的style.css，

```css
div {
    color: red;
}
```

然後在`app.js`中，引用這個css，

```javascript
document.getElementById('app').innerHTML="first webpack pacakge";
require("./util.js");
require("./style.css");
```

接著把loader的設定加入至`webpack.config.js`，然後重新build一次，


**webpack.config.js**

```javascript
var Webpack = require("webpack");
module.exports = {
    entry: ["./app.js"],
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [{
            test: /\.css$/,
            loader: "style-loader!css-loader"
        }]
    }
}
```

```bash
webpack -p
```

再打開index.html，就會發現你的字體顏色變紅色了

<br />

#### url-loader

我覺得這是一個超強大的loader，他可以根據你的需求，把一些圖片轉成base64，這樣就可以減輕一些loading！首先先來安裝`url-loader`，


```bash
npm install url-loader --save-dev
```

安裝完之後，來編輯`style.css`，在裡面設定image，

```css
#content {
    background-image: url(img/bg.png);
}
```

然後在`index.html`中，加入對應的`id`，

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
</head>
<body>
    <div id="app"></div>
    <div id="content"></div> 
    <script src="bundle.js"></script>
</body>
</html>
```

然後一樣要在`webpack.config.js`中加入url-loader的設定，

```javascript
var Webpack = require("webpack");
module.exports = {
    entry: ["./app.js"],
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.css$/, loader: "style-loader!css-loader" },
            { test: /\.(png|jpg)$/, loader: 'url-loader?limit=40000' }
        ]
    }
}
```

最後再來build一次，

```bash
webpack -p
```

build完以後，打開`index.html`以及browser developer mode，會發現該張圖片已經變成base64編碼。


<br />

---
### 7. webpack-dev-server
---

這應該是我覺得webpack這屌的東西了，`webpack-dev-server`是一個小型的Node.js Express server，一旦你啟動了`webpack-dev-server`，<br />
它會根據你的`config`去bundle，然後host一個HTTP Server，你可以透過此server看到你的html。<br />
最強的是！它會去幫你monitor你所有檔案，一旦有變更，都會是live reload，譬如說你改了css某個值，<br />
然後馬上跑去browser看，會發現它已經變更了！完全不用自己手動refresh！<br />

使用方式很簡單，首先先安裝，

```bash
npm install webpack-dev-server -g
```

安裝完以後，在放置`webpack.config.js`的folder底下，輸入指令，

```bash
webpack-dev-server
```

如果成功應該會看到一長串的內容，包含跟你說server的port等等相關資訊。
webpack就先講到這了，有進階應用的話再來補充。

<br />







