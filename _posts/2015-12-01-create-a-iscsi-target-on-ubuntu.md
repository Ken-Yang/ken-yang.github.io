---
layout: post
title: "Create a iSCSI target on Ubuntu"
description: "Configure iSCSI Target"
category: 
tags: [iSCSI, ubuntu]
---
{% include JB/setup %}

<font size=3>
這篇會介紹如何在Ubuntu上，建立一個iSCSI Target Server，  
提供給別人測試使用。

</br>
#### Install tgt
---
首先要先安裝tgt這個package。  

```bash
$ apt-get install tgt
```

</br>
#### Create virtual disks
---

這裡會用`dd`來建立幾個假的block device，  
待會就把devices加入到iSCSI中。

```bash
$ dd if=/dev/zero of=/var/tmp/iscsi-disk1 bs=1M count=100
$ dd if=/dev/zero of=/var/tmp/iscsi-disk2 bs=1M count=100
```

</br>
#### Create targets
---

這裡建立了2個target，分別為，

* iqn.2015-07.net.kenyang:ken.iscsi.1
* iqn.2015-07.net.kenyang:ken.iscsi.2


```bash
$ tgtadm --lld iscsi --op new --mode target --tid 1 -T iqn.2015-07.net.kenyang:ken.iscsi.1
$ tgtadm --lld iscsi --op new --mode target --tid 2 -T iqn.2015-07.net.kenyang:ken.iscsi.2
```

<!--more-->

</br>
#### Create LUN
---

然後替上面的2個target各加入一個LUN。

```bash
$ tgtadm --lld iscsi --mode logicalunit --op new --tid 1 --lun 1 -b /var/tmp/iscsi-disk1
$ tgtadm --lld iscsi --mode logicalunit --op new --tid 2 --lun 1 -b /var/tmp/iscsi-disk2
```

</br>
#### Enable the target to accept any initiators
---

如果你想要讓你的target，  
可以讓任何人連，不需要密碼，  
就可以用下面的指令。

```bash
$ tgtadm --lld iscsi --op bind --mode target --tid 1 -I ALL
$ tgtadm --lld iscsi --op bind --mode target --tid 2 -I ALL
```



</br>
#### Create user
---

如果你想要讓你的target，有credential才能連，  
那就要create user，以及把user bind在某個target上。

```bash
$ tgtadm --mode account --op new --user safesync --password safesync
$ tgtadm --mode account --op bind --tid 1 --user safesync
```

</br>
#### Show configuration
---

```
$ tgtadm --lld iscsi --mode target --op show
```

</font>

