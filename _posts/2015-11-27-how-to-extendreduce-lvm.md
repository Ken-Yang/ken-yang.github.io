---
layout: post
title: "How to resize a LVM partition?"
description: "Extend/Shrink a LVM partition"
category: 
tags: [linux, LVM, PV, VG, PE, lvm, physical volume, volume group, physical extent]
---
{% include JB/setup %}


這篇算是延續[上一篇](http://blog.kenyang.net/2015/11/27/do-not-allocate-all-the-lvm-space-with-preseed/)的主題，假設VG裡面還有free space尚未用到，  
那麼過一段時間以後，該怎麼把它拿出來用?  
所以這篇主要會講到下面2個主題：

1. Extend
2. Shrink

### 1. Extend
---
Extend會講二種方法，分別為，  

   * 1-1. Extend from `VG`
   * 1-2. Extend from `new disk`

</br>
#### 1-1. Extend from PV
---
第一種是延續[上一篇](http://blog.kenyang.net/2015/11/27/do-not-allocate-all-the-lvm-space-with-preseed/)，  
從既有的VG中，把Free PE加入到LV中，  
以[上一篇](http://blog.kenyang.net/2015/11/27/do-not-allocate-all-the-lvm-space-with-preseed/)來看，你的VG應該還會剩下60GB。  
可以用下面的指令`vgs`去看VG中剩下多少free PE，  
最後一欄的`VFree`就是指剩下的free PE，  
以我的機器為範例，裡面還有829.99G，  
以及我有5個LV。


```
$ vgs
  VG        #PV #LV #SN Attr   VSize    VFree
  ubuntu-vg   1   5   0 wz--n- 1019.76g 829.99g
  
$ lvs
  LV     VG        Attr   LSize   Origin Snap%  Move Log Copy%  Convert
  db     ubuntu-vg -wi-ao  83.92g
  log    ubuntu-vg -wi-ao  19.07g
  root   ubuntu-vg -wi-ao  66.75g
  swap_1 ubuntu-vg -wi-ao 976.00m
  tmp    ubuntu-vg -wi-ao  19.07g
```

<!--more-->

假設我要讓`tmp`這個LV加大100G，我們可以透過以下二個指令完成，

1. lvresize
2. resize2fs

</br>
使用方式如下，

```bash
$ lvresize -L +100G /dev/mapper/ubuntu--vg-tmp
```

</br>
完成上述指令以後，可以再透過`lvs`來看一下tmp這個LV是否真的有加大100G。  

```bash
$ lvs
  LV     VG        Attr   LSize   Origin Snap%  Move Log Copy%  Convert
  db     ubuntu-vg -wi-ao  83.92g
  log    ubuntu-vg -wi-ao  19.07g
  root   ubuntu-vg -wi-ao  66.75g
  swap_1 ubuntu-vg -wi-ao 976.00m
  tmp    ubuntu-vg -wi-ao 119.07g
  
$ df -h /tmp/
Filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-tmp   19G   44M   18G   1% /tmp
```
</br>
根據上面的output，你會發現`LV`真的有加大100G，  
可是`df`中的filesystem size並沒有變動，  
因為少用了`resize2fs`。  
`resize2fs`作用就是把filesystem變大。  

```bash
$ resize2fs /dev/mapper/ubuntu--vg-tmp [119G]
```

最後一個是size的參數，其實可以不用加，  
如果不加，就是用整個LV可以用的空間。

</br>
#### 1-2. Extend from new disk
---
第二種Extend的方法是把新disk來加入至lv中。  
其實步驟跟上述差不多，只是多了一些準備作業，  
準備作業如下：

1. 新增 `PV`
2. 把`新PV` 加入至 `VG`

首先你要有一顆硬碟，  
然後用`lsblk`找到該block device的名稱，  
我機器上的新硬碟就是sdb。

```bash
$ lsblk -d
NAME MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
fd0    2:0    1     4K  0 disk
sda    8:0    0  1020G  0 disk
sdb    8:16   0   200G  0 disk
```
</br>

在新增一個`PV`前，  
必須先format disk成LVM的格式。

```bash
$ fdsik /dev/sdb
```

然後請按照下面的步驟去create，

1. 按下`n`以建立新的partition
2. 按下`p`選擇primary partition
3. 按下`1`選擇partition number
4. 按下`enter`選擇Default first sector
5. 按下`enter`選擇Default last sector
6. 按下`t`改變partition type
7. 輸入`8e`, 8e為LVM的代碼
8. 按下`w`儲存寫入


完成以後你就有一個新的partition了，名稱應該為`sdb1`，

```bash
$ fdisk -l /dev/sdb
```
</br>
這時候就可以用`pvcreate `來新增一個PV，  
然後再用`pvs`看看是否有成功。

```bash
$  pvcreate /dev/sdb1

$ pvs
  PV         VG        Fmt  Attr PSize    PFree
  /dev/sda5  ubuntu-vg lvm2 a-   1019.76g 629.99g
  /dev/sdb1            lvm2 a-    200.00g 200.00g
```
</br>
接著就可以用`vgextend`這個指令，  
把剛剛新增的`PV`加入至原有的`ubuntu-vg`當中，

```bash
$ vgextend ubuntu-vg /dev/sdb1
```
</br>
然後一樣用`vgs`去看看`Free PE`，應該會多了200g。  
然後就可以用上面的`lvresize`以及`resize2fs`去Extend某個LV了。




</br></br></br>


### 2. Shrink
---
接著要來講Shrink的部分，  
其實Shrink也是透過`lvresize`以及`resize2fs`來完成。  
但Shrink和Extend有以下幾點的差別：

1. 不能online shrinking
2. 要先縮filesytem，再縮LVM
3. Data有可能會遺失

那因為不能online shrinking，所以我們要先umount，

```bash
$ umount /tmp
```
</br>
接著才可以shrink filesystem，  
透過`resize2fs`去把`tmp`縮到19G。

```bash
$ resize2fs /dev/mapper/ubuntu--vg-tmp 19G
```
</br>

最後才是去shrink LV，一樣是用`lvresize`去做，  
過程中會show warning，告訴你資料可能會遺失，是否真的要做shrink？  
如果沒有什麼疑慮，就按下`y`吧！

```bash
$ lvresize -L -100G /dev/mapper/ubuntu--vg-tmp
  WARNING: Reducing active logical volume to 19.07 GiB
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce tmp? [y/n]: y
  Reducing logical volume tmp to 19.07 GiB
  Logical volume tmp successfully resized
```
</br>
最後在mount起來，以及用`df`去看，  
應該會發現`tmp`只剩下19G了！

```bash
$ mount /tmp

$ df -h /tmp
Filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-tmp   19G   44M   18G   1% /tmp
```




