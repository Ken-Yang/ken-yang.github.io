---
layout: post
title: "Do not allocate all the LVM space with preseed"
description: "Resize LVM when using preseed file to do disk partition"
category:
tags: [linux,ubuntu,preseed,lvm,lvresize]
---
{% include JB/setup %}


使用Preseed file分割LVM時，  
並無法做到只使用某部分的VG就好，  
舉例來說，假設你有一個100G的硬碟，    
然後preseed file如下，

```bash
d-i partman-auto/expert_recipe string  \
      boot-root ::                                                  \
              20480 10 20480 ext4                                   \
                      $lvmok{} lv_name{ root } $defaultignore{ }    \
                      method{ format } format{ }                    \
                      use_filesystem{ } filesystem{ ext4 }          \
                      mountpoint{ / }                               \
              .                                                     \
              20480 10 20480 ext4                                   \
                      $lvmok{ } lv_name{ log } $defaultignore{ }    \
                      method{ format } format{ }                    \
                      use_filesystem{ } filesystem{ ext4 }          \
                      mountpoint{ /var/log }                        \
```

我們分別給每個lv的大小為：

* `root` : 20G
* `log`  : 20G

你可能會以為，剩下的60G不會被加到LV裡面，  
但其實剩下的60G會通通被放到log裡面去。  

不過有比較tricky的解法，可以參考下面的Preseed file，  
我們多增加了一個LV，叫做`to_be_free`，  
然後把所有的free space都指派給`to_be_free`，  
最後在`late_command`裡，用`lvremove`把`to_be_free`砍掉，  
這樣就可以做到把某部分space歸還了。  


```bash
d-i partman-auto/expert_recipe string  \
      boot-root ::                                                  \
              20480 10 20480 ext4                                   \
                      $lvmok{} lv_name{ root } $defaultignore{ }    \
                      method{ format } format{ }                    \
                      use_filesystem{ } filesystem{ ext4 }          \
                      mountpoint{ / }                               \
              .                                                     \
              20480 10 20480 ext4                                   \
                      $lvmok{ } lv_name{ log } $defaultignore{ }    \
                      method{ format } format{ }                    \
                      use_filesystem{ } filesystem{ ext4 }          \
                      mountpoint{ /var/log }                        \
              .                                                     \
              1 10 -1 ext4                                          \
                      $lvmok{ } lv_name{ to_be_free } $defaultignore{ }    \
                      method{ format } format{ }                    \
                      use_filesystem{ } filesystem{ ext4 }          \
                      mountpoint{ /to_be_free }                     \
                      
d-i preseed/late_command string \
umount /target/to_be_free/ ; \
lvremove -f /dev/*/*to_be_free > /dev/null 2>&1 ; \
sed -i '/to_be_free/d' /target/etc/fstab ; \
```  
  
  


