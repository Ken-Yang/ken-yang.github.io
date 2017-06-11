---
layout: post
title: "Install zsh and oh-my-zsh on Mac"
description: ""
category: 
tags: []
---
{% include JB/setup %}



今天把mac上面的default shell換成`zsh`，其實主要是看上了`oh-my-zsh`，所以才想換成zsh。  
過程有點複雜，所以還是紀錄一下怎麼做。



</br>

---
### 1. Install oh-my-zsh
---

由於mac預設就有zsh了，所以不需要安裝，只需要把default shell改成zsh即可。

```bash
$ chsh -s /bin/zsh
```
</br>
接著就安裝`oh-my-zsh`，

```bash
$ curl -L https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh | sh
```

<!--more-->

</br>

---
### 2. Clone cobalt2
---

首先先去clone `cobalt2`下來，`cobalt2`是人家寫的theme，裡面主要有二個theme，

1. **cobalt2.itermcolors** : 給iTerm用的
2. **cobalt2.zsh-theme**   : 給zsh用的

```bash
$ git clone https://github.com/wesbos/Cobalt2-iterm.git  
$ cd Cobalt2-iterm
$ mv cobalt2.zsh-theme ~/.oh-my-zsh/themes/
```

</br>
然後打開iterm，

1. 點選`Preferences`
2. 點`Profiles`
3. 點`+`按鈕新增一個Profile
4. 點`Colors`
5. 點`Load Presets`
6. 選擇`import`
7. 找到剛剛clone下來的`cobalt2.itermcolors`
8. 點左下角的`Other Actions`
9. 選擇`Set as default`


</br>

---
### 3. Install font
---

由於oh-my-zsh中的theme，用了一些特殊符號，所以必須安裝額外的font，

```bash
$ git clone https://github.com/powerline/fonts.git
$ cd fonts
$ ./install.sh
```
</br>
接著一樣在打開iterm，

1. 點選`Preferences`
2. 點`Profiles`
3. 點`Text`
4. 點`Change Font` （**Regular**和**Non-ASCII**都要改)
5. 選擇`inconsolata for powerline`


</br>

---
### 4. 變更Theme
---

接著就可以把theme改成剛剛下載的cobalt2

```bash
$ vim ~/.zshrc
```

</br>
然後把ZSH_THEME換成cobalt2

```
ZSH_THEME="cobalt2"
```
 
然後source一下，就完成了。
```bash
$ source ~/.zshrc
```

 
</br>

---
### 5. 客製化
---

最後一步是客製化，如果你還是不喜歡cobalt2的theme，  
你還是可以更改，只要去編輯`~/.oh-my-zsh/themes/cobalt2.zsh-theme`就好，  
打開以後，看到最下面，應該會有下面幾個function。

{%raw%}

```bash
## Main prompt
build_prompt() {
  RETVAL=$?
  #prompt_status
  prompt_context
  prompt_dir
  prompt_git
  prompt_end
}

PROMPT='%{%f%b%k%}$(build_prompt) '
```
{%endraw%}

可以看到我把`prompt_status`就註解掉，因為我不想要prompt上有icon出現。


