title: 'grub2修复windows引导'
date: 2014-07-12 12:39:43
tags:
---
可参考网址[http://blog.csdn.net/hengyunabc/article/details/6071683](http://blog.csdn.net/hengyunabc/article/details/6071683)

今天装了个centos 7，进去之后发现没有windows引导了，centos 7开始用grub 2引导了。
首先可以试试:
``` sh
$ gurb2-mkconfig -o /boot/grub/grub.cfg
```
它非常智能，能自动检测电脑上安装的操作系统，将它们加入到引导中去。
不过，有时候它也不好使，就像我今天这样。

<!-- more -->

可以采用第二种方式，手动修改/boot/grub/grub.cfg

首先查看分区的UUID
``` sh
$ blkid
```
找到windows所在分区的UUID。一定要记住UUID，不要记住分区号，因为分区号有可能改变，但是UUID一定不会变。
然后修改grub.cfg
``` sh
$ vim /boot/grub/grub.cfg
```
在该文件中加入如下内容:
``` sh
menuentry "windows 8" {
    insmod chain
    insmod ntfs
    search --fs-uuid --set 10ac99d8ac99b8a4
    chainloader +1
}
```
其中的“10ac99d8ac99b8a4”要替换为上一步记住的UUID。

最后，重启电脑就可以看到windows 8的引导了，选择就可以进入系统了。