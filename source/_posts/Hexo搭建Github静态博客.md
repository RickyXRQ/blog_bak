title: Hexo搭建Github静态博客
date: 2015-08-24 19:43:25
tags: www
---
刚开始尝试搭建自己的个人博客大概是在大二那会儿，那时候基本上个人博客偷懒的方法都用wordpress，于是一开始我也只是在本机搭建了wordpress环境，初步接触了下wordpress和个人博客的搭建。大四保研后除了一份儿简单的实习外基本没什么操心的地方，才决定购买域名并且搭建自己的个人博客。因为之前对wordpress比较熟悉，所以基本上花了一天就搭好了自己简单的个人博客。
渐渐地，发现身边很多人都抛弃了wordpress，包括带我走进个人博客世界的几位大神，身边也陆续有同学用Hexo开通了自己的个人博客。而我因为懒惰迟迟没有去了解Hexo，最近实验室有一位小伙伴也要开通自己的wordpress个人博客，趁着给他讲域名购买和虚拟主机购买的工夫，我去了解了一下Hexo，并且搭建了自己的Hexo个人博客。本来想分分钟搞定的事儿，没想到竟然让我忙了三个多小时。其间碰到了一些比较奇怪的错误，经过一番搜索才找到解决的办法，在这里一一列举，希望能帮大家少走些弯路。
<!-- more -->
## 准备工作
在搭建的过程中我用的环境是64位windows 7，因为担心半路倒是查岗得随时打开某些windows下的东西，所以并没有在Linux平台上完成。首先，需要安装Node JS与Git Bash，比较坑爹的是Git官方网站提供的下载链接会跳转至amazonaws.com，而在兲朝并不能顺利访问，最后还是在一个国内的网站下载的一个极老的Git。另外，须在Github上注册自己的账户~因为这儿为了简单，就按照[http://username.github.io]()的格式来搭建，并没有想解析到个人购买的网址上去。
## 搭建过程
首先，需要在本机上配置和使用Github，包括配置SSH keys、添加SSH Key到Github和设置用户信息。相信大部分人都已经提前配好了这些东西，就不再赘述。
其次，在Github上建立前缀与自己Github用户名相同的Repository，如我的Github账户名是RickyXRQ，因此我简历的Repository名字是RickyXRQ.github.io。
接着，在创建好Github Repository后，需要在本机上配置Hexo：首先在本机上创建文件夹“Hexo”，然后在此文件夹中点击右键，选择Git Bash。然后输入以下指令来安装hexo：
``` bash
npm install -g hexo
```
另外，进行初始化：
``` bash
hexo init
```
同时，安装依赖包：
``` bash
npm install
```
此时，在本机的Hexo环境已经搭建完毕了，如果没有意外的话，访问[http://localhost:4000](http://localhost:4000)会跳转至本机的Hexo网页。
接着，在个人的Github页面中，复制创建好的Repository链接，然后编辑hexo文件下的_config.yml文件，在最后的deploy中做如下修改：
``` bash
# Deployment
## Docs: http://hexo.io/docs/deloyment.html
deploy
	type: git 
	repository:https://github.com/RickyXRQ/RickyXRQ.github.io.git
	branch: master 
```
注意：在这个地方极其容易出问题，我在这个地方耽搁了快一个小时。
首先，从Hexo某个版本其，deploy的type这儿不再是github，而变成了”git”，同时，在git后还得留一个空格。另外，还需要在git bash中输入以下命令：
``` bash
npm install hexo-deployer-git --save
```
另外，在第三行的branch也要求在master后面添加一个空格
如果还是配置过程有问题，可以在[这里](http://http//pan.baidu.com/s/1ntEJmdj)下载我配置好的_config.ym文件。
之后，在Git Bash中输入以下两条指令，即可将本地配置好的hexo push到Github上：
``` bash
hexo generate
hexo deploy
```
如果最后的结果如下图所示，则说明搭建成功：
![](/img/2015082401.jpg)
之后，在浏览器中访问[http://RickyXRQ.github.io](http://RickyXRQ.github.io)即可。
## 域名解析
因为购买的域名[http://RickyXu.me](http:RickyXu.me)已经解析到了用wordpress搭建个人网站的虚拟主机IP了，因此我打算把Hexo解析到[http://hexo.rickyxu.me](http://hexo.rickyxu.me)。
首先，在/Hexo/source中建立无后缀名的文件CNAME，内容为要解析的子域名(hexo.rickyxu.me)。
其次，在Godaddy的域名管理中，建立Record Type为“CNAME”，Host为“Hexo”且Points To为“hexo.rickyxu.me”的Zone Record。
最后，hexo generate与hexo deploy即可。















