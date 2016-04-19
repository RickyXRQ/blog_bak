title: 将hexo搭建的博客绑定至子域名
date: 2015-08-27 22:45:06
tags: www
---
首先，在/Hexo/source中建立无后缀名的文件CNAME，内容为要解析的子域名(即hexo.rickyxu.me)。
其次，在Godaddy的域名管理中，建立Record Type为“CNAME”，Host为“Hexo”且Points To为“hexo.rickyxu.me”的Zone Record，如图所示：
<!-- more -->
![](/img/2015082701.jpg)
最后，执行hexo generate与hexo deploy即可。