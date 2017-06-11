---
layout: post
title: 如何把Google Blogger搬到Github pages
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
3. 用jekyll-import把blogger資料匯出與匯入
4. 安裝jekyll
5. 安裝jekyll-paginate
6. 整合Google Adsense
7. 整合comment system
8. 整合Google Analytics

<!--more-->

## 1. 建立github repo

首先要先去[github](https://github.com/new)的網站建立一個新的repo，  
repo名稱為`your_name.github.io`，  
所以我的就是`ken-yang.github.io`。

<br /> 
  
## 2. jekyll-bootstrap

[jekyll-bootstrap](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)是一個線上工具，  
可以讓你快速地建立一個template，  
然後這個template是以boostrap為基底。  
使用方式如下（記得把`USERNAME`換成你的`your_name`)：

```bash
$ git clone https://github.com/plusjade/jekyll-bootstrap.git USERNAME.github.io
$ cd USERNAME.github.io
$ git remote set-url origin git@github.com:USERNAME/USERNAME.github.ui.git
$ git push origin master
```

接著你就可以去`USERNAME.github.io`看看，應該會有default的index。

<br /> 

## 3. jekyll-import （匯出與匯入）

接著我們要把原本在google blogger上的資料轉出來，  
這裡使用[jekyll-import](http://import.jekyllrb.com/docs/installation/)這個tool。
安裝[jekyll-import](http://import.jekyllrb.com/docs/installation/)之前，   
如果你是用MAC，請先安裝xcode，  
如果是用linux，請先安裝make。

```bash
$ gem install -n jekyll-import
```
安裝成功以後，去google blogger的後台把文章通通export出來。  
然後再terminal執行下面的指令，  
記得把`/path/to/blog-MM-DD-YYYY.xml`換成對應的位置。


```bash
$ ruby -rubygems -e 'require "jekyll-import";
    JekyllImport::Importers::Blogger.run({
      "source"                => "/path/to/blog-MM-DD-YYYY.xml",
      "no-blogger-info"       => false, # not to leave blogger-URL info (id and old URL) in the front matter
      "replace-internal-link" => false, # replace internal links using the post_url liquid tag.
    })'
```

接著會產生二個資料夾，分別為

* `_posts` 放文章的地方
* `_draft` 放草稿的地方

接著就把_posts底下的資料搬移至剛剛的git project底下，

```bash
cp _posts/* USERNAME.github.ui.git/_posts/
```

<br />
## 4. 安裝jekyll

接著安裝[jekyll](https://jekyllrb.com/)，  
jekyll主要功用為:

* 在local preview你的post
* render post

安裝方式一樣是透過gem，

```bash
$ gem install -n /usr/local/bin jekyll
```
安裝成功就可以用jekyll去preview你的posts，

```bash
$ cd USERNAME.github.io
$ jekyll serve
```
然後會在你的local起一個web server，  
網址是`http://localhost:4000`，  
在執行`serve`的當下就會幫我們render post了，  
接著就可以把所有的changed push回去github上。  

```bash
$ git add . 
$ git cm -m 'add posts'
$ git push origin master
```
然後一樣去`USERNAME.github.io`看看，會有原本在blogger上的posts了！


<br />
## 5. 安裝jekyll-paginate

jekyll-paginate功用就是分頁，如果你不想要你的首頁就把所有的post通通列出來，  
那麼就可以安裝這個plugin，  

```bash
$ gem install -n /usr/local/bin jekyll-paginate
```

接著打開`_config.yml`，然後加入下面三行，

```
gems: [jekyll-paginate]
paginate: 3
paginate_path: "/blog/page:num"
```



然後有一個比較tricky的地方，  
如果你的首頁是index.md，pagenate是無法幫你gen出來，  
必須是index.html，但很簡單，你只要把`mv index.md  index.html`即可，  
mv完成以後，接著打開`index.html`，  
然後把下面的code貼入。

{%raw%}
```html
{% for post in paginator.posts %}
  <h1><a style="color: #F22430" href="{{ post.url }}">{{ post.title }}</a></h1>
  <div style="width: 100%; border-bottom: 1px solid #E7E7E7;"></div>
  <p class="author">
    <span class="date">{{ post.date | date_to_long_string }}</span>
  </p>
  <div class="content">
    {{ post.excerpt }}
  </div>
  <footer>
    <p><a class="myButton" href="{{ post.url | prepend: site.baseurl }}">{{ site.more }} &raquo;</a></p>
  </footer>
  <hr>
{% endfor %}
<!-- Pagination links -->
<div class="pagination">
  {% if paginator.previous_page %}
    <a href="{{ paginator.previous_page_path }}" class="previous">Previous</a>
  {% else %}
    <span class="previous">Previous</span>
  {% endif %}
    <span class="page_number ">Page: {{ paginator.page }} of {{ paginator.total_pages }}</span>
  {% if paginator.next_page %}
    <a href="{{ paginator.next_page_path }}" class="next">Next</a>
  {% else %}
    <span class="next ">Next</span>
  {% endif %}
</div>
```
{%endraw%}

接著就在重新render一次。

```bash
$ jekyll serve
```
完成以後，在`_site/blog/`底下，就會有很多`pageX`的資料夾，  
接著就可以通通在push回去github上。


<br />
## 6. 整合Adsense

這裡用adsense當作廣告source，  
首先先去adsense網站上申請，這裡就不教怎麼申請了，  
申請完以後，會給你一串javascript，  
把它複製起來，然後建立一個新的html檔案。

```bash
vim _includes/adsense.html
```
然後把剛剛adsense給的那串javascript貼進去，如下，

```html
<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- id10t_top -->
<ins class="adsbygoogle"
style="display:block"
data-ad-client="ca-pub-xxxxxxxxxxxx"
data-ad-slot="1161208442"
data-ad-format="auto"></ins>
<script>
    (adsbygoogle = window.adsbygoogle || []).push({});
</script>
```

接著我們就可以在想要放廣告的page裡面，include `adsense.html`這個file，  
假設我們想要在首頁放廣告，那麼就是打開 `index.html` ，  
然後在適當的地方貼入下面的code，  

{%raw%}
```html
{% include adsense.html %}
```
{%endraw%}

接著一樣重新render一次就完成了。

```bash
$ jekyll serve
```

<br />
## 7. 整合Comment system

jekyll裡面內建了很多的comment system，有disqus、facebook...etc.  
這裡我選擇了[disgus](https://disqus.com/home/explore/)，因為預設也是使用它。  
那首先要先去[disgus](https://disqus.com/home/explore/)上註冊，  
註冊非常簡單，也不需要認證。  
註冊完成以後就會取得一個`short name`，  
有了`short name`以後，就打開`_config.yml`，  
然後找到`comments`這個key，然後把你的`short name`填入即可。

```
comments :
  provider : disqus
  disqus :
    short_name : your_short_name
```

完成以後，一樣重新render就完成了。

<br />
## 8. 整合Google Analytics

jekyll也有內建analytics的功能，其實跟上面的comment system差不多，  
一樣填入analytics id即可。  
但我發現jekyll的analytics 好像比較舊。  
所以我就不用它內建的，我就用與adsense的方式一樣，  
先create一個`analytics.html`，接著把下面的code貼入，

```html
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-xxxxxx-1', 'auto');
  ga('send', 'pageview');

</script>
```

然後在我想要track的地方，貼入下面的code，

{%raw%}
```html
{% include analytics.html %}
```
{%endraw%}

然後在重新render就完成了！  
最後只要把你的cname指向username.github.io，  
以及在目錄底下建立一個名稱為`CNAME`的file，填入你的domain就好了。






