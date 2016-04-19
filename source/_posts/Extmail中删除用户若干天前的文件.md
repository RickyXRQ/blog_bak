title: Extmail中删除服务器中用户若干天前的邮件
date: 2015-05-11 21:38:41
tags: Linux
---
首先，在root权限下，可以采用命令来对服务器上的用户根据邮件大小进行排序。例如，使用如下指令可以取出占用空间最大的10个用户：
``` bash
$ du -s /home/domains/wspn.com/ *|sort -rn|head
```
会得到如下列表：
``` bash
5020 /home/domains/wspn.com/pengtao
3612 /home/domains/wspn.com/zhupengbo
1768 /home/domains/wspn.com/liulibin
216  /home/domains/wspn.com/xuruiqiang
168  /home/domains/wspn.com/xiaoshan
168  /home/domains/wspn.com/gejiajun
132  /home/domains/wspn.com/rickyxrq
132  /home/domains/wspn.com/fengyudong
128  /home/domains/wspn.com/zhangjiayue
124  /home/domains/wspn.com/wangzhe
```
<!-- more --> 
之后，进入相应用户目录，执行下列命令即可：
``` bash
$ find -ctime +30 | xargs rm
```
