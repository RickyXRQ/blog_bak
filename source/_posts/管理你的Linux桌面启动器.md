title: 管理你的Linux桌面启动器
date: 2015-09-10 16:28:44
tags: Linux
---
如果想在Linux上方便地打开一个频繁使用的程序，可能最常见的手段就是把它放在桌面左侧的快速启动栏了。
例如，最近在我的Ubuntu系统下安装了Sublime Text 2后，因为是从官网下载压缩包安装的，因此采用的启动方法是将sublime text 2与/usr/bin/sublime进行链接：
``` bash
$ ln -s /usr/lib/Sublime\ Text\ 2/sublime_text /usr/bin/sublime
```
然后在终端执行sublime后即可以启动sublime。
在启动sublime后，左侧的快速启动栏会出现相应的图标，右击并且点选“锁定到启动器”后即可比较快速的启动sublime。
但是，往往锁定到启动器后的效果是这样的：
![](/img/2015091001.jpg)
突然间好奇启动器是由Linux中哪个文件来管理的，于是就又详细的了解了下：
<!-- more -->
在当前用户的~/.local/share/applications中，储存着启动器的各项信息：
``` bash
ricky@XPS:~/.local/share/applications$ ls
chrome.desktop google-chrome-stable.desktop mimeapps.list sublime.desktop
```
打开sublime.desktop后可以看到如下内容：
``` bash
[Desktop Entry]
Encoding=UTF-8
Version=1.0
Type=Application
Name=Sublime Text 2
Icon=sublime_text.png
Path=/home/ricky
Exec=sublime
StartupNotify=false
StartupWMClass=Sublime
OnlyShowIn=Unity;
X-UnityGenerated=true
```
可以看到这个配置文件包含了启动器一个项目的版本，名称，类型，图标等配置信息。而从SublimeText官网上下载的压缩包里已经包含各个分辨率的icon，做相应替换即可：
将Icon=sublime_text.png改为Icon=/usr/lib/Sublime Text 2/Icon/48x48/sublime_text.png后即可发现启动器中SublimeText的图标已经达到了我们想要的效果。
