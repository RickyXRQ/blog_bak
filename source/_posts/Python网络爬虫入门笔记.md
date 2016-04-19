title: Python网络爬虫入门笔记--爬虫框架Scrapy的简单应用
date: 2015-09-21 21:49:15
tags: Python
---
最近一段时间学习了下Python网络爬虫的相关内容，对爬虫有了一点感性的认识。为了督促自己学习防止眼高手低，特意写了这篇文章来加深学习。参考文章主要有[Scrapy at a glance](http://doc.scrapy.org/en/latest/index.html)与[专栏：Python爬虫入门教程](http://blog.csdn.net/column/details/why-bug.html)与[专栏：Scrapy](http://blog.csdn.net/column/details/younghz-scrapy.html).
<!-- more -->
# 一点认识
结合这几天看得参考资料，在前几天一位同学去吃饭的路上聊天的时候给出了自己对网络爬虫原理的一个认识：
同我们有目的的浏览网页一样，进行网络爬虫的时候也是通过一个地址来获得网页的相关内容，只不过是我们在上网的时候获取的网页内容是浏览器对HTML语言解析后的结果，而网络爬虫获取的网页内容是HTML语言的。在获取到HTML语言描述的网页内容后，利用一定的匹配规则从中筛选出感兴趣的内容，并且存放到一定的容器中。同时，也可以从获取的网页内容中获取到向其他网页的跳转信息，直至达到我们的要求。最后，将存放摘取的爬虫内容的容器通过一定方法存放到文件或数据库中，从而完成了一次爬虫过程。
# Python中的几个相关的基本函数
有了以上的对爬虫的感性认识后，将自己的学习经历记录在下。
首先，我显示了解了抓取网页的含义和URL的基本构成。一般有过上网经验的人都会有这方面的基础，因此不再赘述。
接着，我利用最朴素的urllib2库中的一些函数来对网页内容进行了最简单的抓取。正如如下代码：
``` bash
import urllib2
response = urllib2.urlopen('http://www.baidu.com')
html = response.read()
print html
```
执行后会有如下结果：
![](/img/2015092101.jpg)
同时，如果我们打开百度的主页，右击“查看源代码”会发现和截图同样的内容。也就是说，上面的几行代码将我们访问百度时候浏览器收到的代码全部打印了出来。事实上，通过对代码的阅读也可以发现正是完成了这样的工作。其实，urllib2一般用一个Request对象来映射我们提出的http请求。比如说，我们可以通过调用urlopen来传入Request对象，得到返回的Response对象，正如C里面的文件对象，之后调用Response的.read()即可。例如：
``` bash
import urllib2
req = urllib2.Request('http://www.baidu.com')
response=urllib2.urlopen(req)
page = response.read()
print page
```
可以发现得到了和刚才一样的效果。同时，在向创建请求时，我们还可以创建ftp请求。另外，在HTTP请求时，允许我们做两件额外的事情：
1.使用POST请求发送一些数据到URL.
2.使用GET方式请求。
同时，参考文章的作者还介绍了urllib2中的两种方法：info和geturl。
geturl()用来返回获取的真实的url，因为现在很多社交网站都提供了短域名服务，因此我们获取到的URL很有可能跟请求的URL不同：
``` bash
from urllib2 import Request, urlopen, URLError, HTTPError

old_url = 'http://t.cn/RyvONYY'
req = Request(old_url)
response = urlopen(req)
print 'Old URL :' + old_url
print 'Real URL :' + response.geturl()
```
可以得到如下输出，通过输出可以看到链接指向的真正的地址：
![](/img/2015092102.jpg)
info返回对象的字典对象，描述了获取的页面情况。通常是服务器发送的特定头headers：
``` bash
from urllib2 import Request, urlopen, URLError, HTTPError

old_url = 'http://www.rickyxu.me'
req = Request(old_url)
response = urlopen(req)
print 'Info(): ' + str(response.info())
```
运行后可以得到页面的相关信息：
![](/img/2015092103.jpg)
其他还有一些零碎的东西，我只是大概的看了一下，并没有去详细了解。接着，就着手去学习Scrapy：
# Scrapy
Scrapy的官方地址为[Scrapy_Latest](http://doc.scrapy.org/en/latest/intro/overview.html)，有人翻译了一份中文的文档[Scrapy入门教程](http://scrapy-chs.readthedocs.org/zh_CN/latest/intro/tutorial.html)，但是版本较老，而且部分语句可能读起来比较晦涩，因此建议阅读英文版本。
首先，Scrapy平台的搭建在官方文档中都给出了详细的说明。我在Linux与Windows下分别搭建了一次，并且在两种平台下都使用Scrapy进行了简单的爬取，感觉还是在Linux下使用起来比较顺手（如直接在终端内使用vim指令来编辑.py文件等，但是在windows下cmd使用起来就比较困难了）。
简单地搭建完成后，就可以尝试进行最简单的爬取了。这里仍然参照官方文档与参考文档中的例子，对著名的开放式分类目录网站[DMOZ](http://www.dmoz.org/)进行爬取。
## 新建项目
在命令提示符中Scrapy目录下输入以下命令：
``` bash
scrapy startproject tutorial
```
其中的tutorial为项目的名称。在创建新项目后会得到一个以项目名称为名字的文件夹，目录结构如下：
``` bash
tutorial/  
    scrapy.cfg  
    tutorial/  
        __init__.py  
        items.py  
        pipelines.py  
        settings.py  
        spiders/  
            __init__.py  
            ... 
```
其中scrapy.cfg为项目的配置文件，文件夹tutorial为项目的Python模块，包含了实现的Python代码。tutorial下的items.py为项目的item文件，item可以理解为项目中对获取对象的数据结构的定义。pipelines.py为项目的pipelines文件，与对爬取的内容的存储有关。settings.py为项目的设置文件。而spiders文件夹存储了爬虫的实现。
## 简单爬虫的实现
为了实现对DMOZ部分内容的爬取，我们首先需要对items进行重定义。编辑items.py，加入我们的Items的定义：
``` bash
import scrapy

from scrapy.item import Item, Field

class TutorialItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    pass

class DmozItem(scrapy.Item):
    title = Field()
    link = Field()
    desc = Field()
```
其实我看了几天后也没太明白这一部分的用处，但是发现对之后的影响不大，只是我们实现了对items模型的定义，包含名称（title）、链接（link）与描述（desc）。
另外，在spiders文件夹下编写我们的爬虫：
新建dmoz_spider.py文件，完成对爬虫的制作。即建立一个Spider，确定几个必须的属性：爬虫的名称、爬取的URL列表、解析的方法。一个简单的例子如下：
``` bash
from scrapy.spiders import Spider

class DmozSpider(Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",  
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
        ]

    def parse(self, response):
        filename = response.url.split("/")[-2]
        open(filename,"wb").write(response.body)
```
从中很容易看到相关的一些东西：
类名即我们定义的爬虫名，这个比较自由。名称我们定义为"dmoz"，允许搜索的域名范围即爬虫的约束区域，规定了爬虫只能爬取这个域名下的网页。而parse函数则定义了爬取的具体规则。结合我这几天对Scrapy的了解，先有必要解释一下response的含义，response是parse函数通过url获取的对象，类似于这篇博文前面的通过调用urllib2库中的urlopen()函数获取的对象一样。因此，可以很容易看到，该爬虫完成的工作是从start_urls中获取链接并得到response，然后调用切分函数split把用“/”切分的链接的倒数第二个字符作为文件名进行保存，保存的内容为response的body。
之后，在tutorial的目录下运行该爬虫即可：
``` bash
scrapy crawl dmoz
```
执行结果如下：
![](/img/2015092201.jpg)
出现的“Closing spider (finished)”表明爬虫已经成功运行并且自行关闭。
在爬虫执行后，可以在目录中看到两个新的文件：Books与Resources，分别记录从这两个URL上爬取下的内容。
但是，打开Books或Resources后发现爬取到的内容为整个页面的内容。为了针对性地获取到我们想要的内容，Scrapy中采用了XPath表达式。关于XPath的表达式可以参考[XPath教程](http://www.w3school.com.cn/xpath/index.asp)。
以http://www.dmoz.org/Computers/Programming/Languages/Python/Books/为例，执行
``` bash
scrapy shell http://www.dmoz.org/Computers/Programming/Languages/Python/Books/
```
会得到如下的结果：
![](/img/2015092202.jpg)
输入response.body后可以看到刚才抓取到的内容。修改爬虫的实现可以抓取到我们想要的内容：
``` bash
from scrapy.spiders import Spider
from scrapy.selector import Selector

class DmozSpider(Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",  
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
        ]

    def parse(self, response):  
        sel = Selector(response)  
        sites = sel.xpath('//ul[@class="directory-url"]/li')  
        for site in sites:  
            title = site.xpath('a/text()').extract()  
            link = site.xpath('a/@href').extract()  
            desc = site.xpath('text()').extract()  
            print title, link, title
```
注意我们这里导入了Selecotr类，并且实例化了一个新的Selector对象sel。
爬虫执行的结果为：
![](/img/2015092203.jpg)
即抓取的title是源文件中在ul标签中的属性class=“directory-url”下li标签中的a标签中的文本。抓取的link为在ul标签中的属性class=“directory-url”下li标签中的href属性。抓取的desc为在ul标签中的属性class=“directory-url”下li标签中的文本。调用pirnt语句把抓取的结果在控制台上打印显示。
### 使用Item
为了将爬虫爬到的数据保存到Item对象中，我们可以对代码进行以下修改：
``` bash
from scrapy.spiders import Spider
from scrapy.selector import Selector

from tutorial.items import DmozItem

class DmozSpider(Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",  
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
        ]

    def parse(self, response):  
        sel = Selector(response)  
        sites = sel.xpath('//ul[@class="directory-url"]/li')
        items = []
        for site in sites:
            item = DmozItem()
            item['title'] = site.xpath('a/text()').extract()  
            item['link'] = site.xpath('a/@href').extract()  
            item['desc'] = site.xpath('text()').extract()  
            items.append(item)
        return items
```
以上代码先是定义了一个空的items，然后在每次爬取到新的item后append到items中，逐渐保存到页面中的信息。同时，在运行该爬虫的时候，通过Feed exports来保存信息，执行以下指令即可将结果用JSON导出：
``` bash
scrapy crawl dmoz -o items.json -t json 
```
即可以得到爬出的最终结果：
![](/img/2015092204.jpg)
### 使用pipelines
根据这几天我对Scrapy粗略的了解，目前自己对Pipelines的认识是pipelines是一种连接爬取到的Items与最终存储文件中的管道。之前我们的例子中要么是在Parse()函数中不返回Item而是去打印输出爬取到的Item的值，要么是使函数返回Item然后在执行特定爬虫的时候指定输出文件。而利用Pipelins则可以在Parse()执行返回item后直接进行数据的储存。在上述爬虫的基础上，我们可以修改工程的pipeline来实现我们的目标。
首先，编辑工程中的pipelines.py文件：
``` bash
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: http://doc.scrapy.org/en/latest/topics/item-pipeline.html

import json
import codecs


class TutorialPipeline(object):
    def __init__(self):
        self.file = codecs.open('dmoz_data_utf8.json', 'wb', encoding = 'utf-8')
        
    def process_item(self, item, spider):
        line = json.dumps(dict(item)) + '\n'
        self.file.write(line.decode("unicode_escape"))
        return item
```
根据pipeline文件中的代码很容易看出pipeline具体的实现步骤：
首先，导入了两个头文件，json与codecs，如果对这两个文件不了解，可以暂时将其理解为与字符的编码解码有关；
其次，类TutorialPipeline的构造函数中，调用codecs库中的open函数，用utf-8编码的二进制写方式打开了本地的一个.json文件dmoz_data_utf8.json
然后，类TutorialPipeline的另一个成员函数process_item()实现的是将item转换为dict类型后再调用json库中的编码函数dumps，与一个换行符拼接后组成一个新的对象。
之后，将json编码的line对象再译码成unicode_escape对象，并且写入到文件中。
最后再返回。
在修改完pipelines.py文件后，还需要对setting.py进行修改，编辑setting.py文件，加入以下行：
``` bash
ITEM_PIPELINES = {
    'tutorial.pipelines.TutorialPipeline': 300,
}
```
我理解的是将我们刚才编写的pipelines添加到整个项目的配置中。
然后，在项目的根目录用scrapy scrawl执行爬虫dmoz即可。在执行完毕后可以看到本地生成了json文件“dmoz_data_utf8.json”，内容如下：
``` bash
{"title": ["Core Python Programming"], "link": ["http://www.pearsonhighered.com/educator/academic/product/0,,0130260363,00%2Ben-USS_01DBC.html"], "desc": ["
    
                                ", " 
            
                                - By Wesley J. Chun; Prentice Hall PTR, 2001, ISBN 0130260363. For experienced developers to improve extant skills; professional level examples. Starts by introducing syntax, objects, error handling, functions, classes, built-ins. [Prentice Hall]
                                
                                ", "
                                "]}
{"title": ["Data Structures and Algorithms with Object-Oriented Design Patterns in Python"], "link": ["http://www.brpreiss.com/books/opus7/html/book.html"], "desc": ["
    
                                ", " 
            
                                - The primary goal of this book is to promote object-oriented design using Python and to illustrate the use of the emerging object-oriented design patterns.
A secondary goal of the book is to present mathematical tools just in time. Analysis techniques and proofs are presented as needed and in the proper context.
                                
                                ", "
                                "]}
{"title": ["Dive Into Python 3"], "link": ["http://www.diveintopython.net/"], "desc": ["
    
                                ", " 
            
                                - By Mark Pilgrim, Guide to Python 3  and its differences from Python 2. Each chapter starts with a real code sample and explains it fully. Has a comprehensive appendix of all the syntactic and semantic changes in Python 3


                                
                                ", "
                                "]}
{"title": ["Foundations of Python Network Programming"], "link": ["http://rhodesmill.org/brandon/2011/foundations-of-python-network-programming/"], "desc": ["
    
                                ", " 
            
                                - This book covers a wide range of topics. From raw TCP and UDP to encryption with TSL, and then to HTTP, SMTP, POP, IMAP, and ssh. It gives you a good understanding of each field and how to do everything on the network with Python.
                                
                                ", "
                                "]}
```
可以看到json文件中已经自动将爬取的结果按照一定的格式在本地进行了保存。
## 爬取自己的Blog!
在有了上述基础的前提下，我打算把我[个人博客](http://hexo.rickyxu.me)所有的文章名和链接进行爬取。基本上利用上面提到的几项就可以实现这个功能，需要新学的东西非常少。
### 创建project
``` bash
scrapy startproject HexoBlog
```
### items.py的编写
因为只需要爬取博客中文章的文章名和链接，因此仅需要两项就足够：
``` bash
# -*- coding: utf-8 -*-

# Define here the models for your scraped items
#
# See documentation in:
# http://doc.scrapy.org/en/latest/topics/items.html

import scrapy
from scrapy.item import Item, Field

class HexoblogItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    article_name = Field()
    article_url = Field()
```
在这里，article_name用来存放博客文章的题目，article_url用来存放博客文章的链接。
### pipelines.py的编写
理想中的实现是把爬取的内容逐项分行保存在一个本地文件中，因此可以编写如下pipelines.py：
``` bash
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: http://doc.scrapy.org/en/latest/topics/item-pipeline.html

import json
import codecs

class HexoblogPipeline(object):

    def __init__(self):
        self.file = codecs.open('HexoBlogList.json', mode = 'wb', encoding = 'utf-8')
        
    def process_item(self, item, spider):
        line = json.dumps(dict(item)) + '\n'
        self.file.write(line.decode("unicode_escape"))
        return item
```
这一部分的具体细节与“简单爬虫的实现”中"使用pipelines"部分没区别，因此不再赘述。
### settings.py的编写
这一部分主要是设置好pipelines，另外有一处参数需要设置：COOKIES_ENABLED设置为False来防止被ban
``` bash
ITEM_PIPELINES = {
    'HexoBlog.pipelines.HexoblogPipeline': 300,
}
```
### 爬虫的编写
之前的"简单爬虫的实现"中我们只是对单个页面进行了爬取，但是在博客中文章不可能仅有一篇，因此就需要想办法从开始爬取的页面一次获得其他页面的地址。而在一般的博客中，每篇日志的末尾或其他位置都有其他日志的跳转链接。例如，在我的博客中，每篇文章的结尾处会有如下标示：
![](/img/2015092301.jpg)
我们就可以利用这个来从当前页面获取到其他文章的链接。因此可以编写出如下爬虫：
``` bash
# -*- coding: utf-8 -*-
from scrapy.spider import Spider
from scrapy.http import Request
from scrapy.selector import Selector
from HexoBlog.items import HexoblogItem


class HexoBlogSpider(Spider):

    name = "HexoBlog"

    download_delay = 1
    allowed_domains = ["hexo.rickyxu.me"]
    start_urls = [
        "http://hexo.rickyxu.me/2015/09/21/Python%E7%BD%91%E7%BB%9C%E7%88%AC%E8%99%AB%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B0/"
        ]

    def parse(self, response):
        sel = Selector(response)

        item = HexoblogItem()

        article_url = str(response.url)
        article_name = sel.xpath("//h1[@class='article-title']/text()").extract()

        item['article_name'] = [n.encode('utf-8') for n in article_name]
        item['article_url'] = article_url.encode('utf-8')

        yield item
        ##获取下一篇文章的链接
        urls = sel.xpath("//a[@id='article-nav-older']//../@href").extract()
        for url in urls:
            url = "http://hexo.rickyxu.me" + url
            yield Request(url, callback = self.parse)
```
分析源码：
首先开头的是一堆导入的类。
接着，如同之前"简单爬虫的实现"中一样，先是爬虫的名称；和以往不同的是很有益处download_delay的参数设置，这个参数将下载器下载下一个页面的等待时间设置为1s，是防止被ban的一种策略，这样做可以减轻服务器端的负载；之后是允许爬取的链接范围，当然是限定在我的博客中了；然后起始链接选为博客中最新的一篇文章的链接，这样就可以按照时间顺序倒着往回爬。
之后就是最重要的爬取方法函数的设计。因为我们会用到XPath语句，因此开始就定义了选择器；之后又声明了HexoblogItem类，用于存放爬取的结果；爬取的文章链接可以直接由response.url转换为字符串得到，而文章题目需要用XPath语句来寻找。利用Firefox下的XPath Checker插件可以帮助我们快速准确地获得Xpath语句。我们观察源码的时候可以发现如下标签中包含了文章的标题：
```
    <h1 class="article-title" itemprop="name">
      Python网络爬虫入门笔记
    </h1>
```
因此很容易写出article_name的获得方式：h1标签下class属性为'article'的文本内容。而因为可能文章的标题中含有汉语，因此在往item中存储的时候要进行utf-8的编码。
之后就是yield对迭代器的使用。
接着，我们还需要获取下一篇文章的链接。同样利用Firefox下的XPath Checker插件，观察页面的源代码，我们可以找到此处有下一篇文章的链接信息：
``` bash
    <a href="/2015/09/10/管理你的Linux桌面启动器/" id="article-nav-older" class="article-nav-link-wrap">
      <div class="article-nav-title">管理你的Linux桌面启动器</div>
      <strong class="article-nav-caption">></strong>
    </a>
```
进而得出XPath语句：在a标签下id属性为‘article-nav-older’的父类的href属性的值。但是可以看到链接缺少根域名部分，所以需要拼接上"http://hexo.rickyxu.me"部分。
最后，获取到下一篇日志的链接后，我们需要把新的request返回给爬取函数，实现循环爬取，因此调用了callback来实现。
### 执行
``` bash
scarpy crawl HexoBlog
```
结果如下：
![](/img/2015092302.jpg)
同时本地生成的爬取结果如下：
``` bash
{"article_name": ["
      Python网络爬虫入门笔记
    "], "article_url": "http://hexo.rickyxu.me/2015/09/21/Python%E7%BD%91%E7%BB%9C%E7%88%AC%E8%99%AB%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B0/"}
{"article_name": ["
      管理你的Linux桌面启动器
    "], "article_url": "http://hexo.rickyxu.me/2015/09/10/%E7%AE%A1%E7%90%86%E4%BD%A0%E7%9A%84Linux%E6%A1%8C%E9%9D%A2%E5%90%AF%E5%8A%A8%E5%99%A8/"}
{"article_name": ["
      DIABLO 3 Patch 2.3.0 will be released today!
    "], "article_url": "http://hexo.rickyxu.me/2015/08/28/DIABLO-3-Patch-2-3-0-will-be-released-today/"}
{"article_name": ["
      将hexo搭建的博客绑定至子域名
    "], "article_url": "http://hexo.rickyxu.me/2015/08/27/%E5%B0%86hexo%E6%90%AD%E5%BB%BA%E7%9A%84%E5%8D%9A%E5%AE%A2%E7%BB%91%E5%AE%9A%E8%87%B3%E5%AD%90%E5%9F%9F%E5%90%8D/"}
{"article_name": ["
      My first Hexo article
    "], "article_url": "http://hexo.rickyxu.me/2015/08/25/My-first-Hexo-article/"}
{"article_name": ["
      Hexo搭建Github静态博客
    "], "article_url": "http://hexo.rickyxu.me/2015/08/24/Hexo%E6%90%AD%E5%BB%BAGithub%E9%9D%99%E6%80%81%E5%8D%9A%E5%AE%A2/"}
```
至此，对自己的博客内容的爬取结束。在爬取的过程中，犯了一个低级的错误，没有意识到/spiders/文件夹下的.pyc文件才是最终真正执行的文件，有一次对文件重命名后，进行了保存，但是每次执行爬虫的时候仍然调用原来生成的.pyc文件，折腾了好久才反应过来是这个原因。
## 进阶工具--CrawlSpider
之前的工作的思想是从爬取的网页中获取链接并继续爬取，完成这个工作有一个更适合的工具：CrawlSpider。
CrawlSpider是Spider类的派生类，与Spider类相比，它多了一个rules参数，用来定义提取动作，rules参数包含一个或者多个Rule对象。Rule类与CrawlSpider类均位于scrapy.contrib.spiders模块中。
以爬取自己的blog为例，应用CrawlSpider进行爬取的工程如下：
### items.py pipelines.py settings.py
这三个文件的修改与先前的工程类似，这里不再赘述
### items.py
``` bash
# -*- coding: utf-8 -*-

# Define here the models for your scraped items
#
# See documentation in:
# http://doc.scrapy.org/en/latest/topics/items.html

import scrapy
from scrapy.item import Item, Field

class BlogscrapyItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    article_name = Field()
    article_url = Field()
```
### pipelines.py
``` bash
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: http://doc.scrapy.org/en/latest/topics/item-pipeline.html
import json
import codecs

class BlogscrapyPipeline(object):
    
    def __init__(self):
        self.file = codecs.open('HexoBlogList.json', mode = 'wb', encoding = 'utf-8')
        
    def process_item(self, item, spider):
        line = json.dumps(dict(item)) + '\n'
        self.file.write(line.decode("unicode_escape"))
        return item
```
### 爬虫的编写
``` bash
# -*- coding:utf-8 -*-

from scrapy.contrib.spiders import CrawlSpider, Rule
from scrapy.contrib.linkextractors.sgml import SgmlLinkExtractor
from scrapy.selector import Selector
from BlogScrapy.items import BlogscrapyItem

class BlogScrapy(CrawlSpider):

    name = "BlogScrapyXRQ"
    #download_delay = 2
    allowed_domains = ['hexo.rickyxu.me']
    start_urls = ['http://hexo.rickyxu.me/2015/09/21/Python%E7%BD%91%E7%BB%9C%E7%88%AC%E8%99%AB%E5%85%A5%E9%97%A8%E7%AC%94%E8%AE%B0/']

    rules = [
        Rule(SgmlLinkExtractor(allow=('/2015'),restrict_xpaths=("//a[@id='article-nav-older']")),
             callback='parse_item',
             follow = True)
        ]
    #print "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!"

    def parse_item(self, response):
        #print "??????????????????????????????????????"
        item = BlogscrapyItem()
        sel = Selector(response)
        blog_url = str(response.url)
        blog_name = sel.xpath("//h1[@class='article-title']/text()").extract()
        print str(blog_name)
        item['article_name'] = [n.encode('utf-8') for n in blog_name]
        item['article_url'] = blog_url.encode('utf-8')

        yield item
```
结合本代码可以更好地了解rules的使用方法：在该爬虫中，首先获取到start_url，然后定义了rules，rules的具体使用参数情况如下：
```
class scrapy.contrib.spiders.Rule (  
link_extractor, callback=None,cb_kwargs=None,follow=None,process_links=None,process_request=None )  
```
其中第一个参数link_extractor用于定义要提取的链接。第二个参数callback用来指示从link_extractor获取链接后选取的回调函数。**其他几个参数的意思还暂时不清楚，希望以后有机会能够补充。**因此，示例代码中定义的rules是指定了从第一个参数获取到链接后立刻调用parse_item函数。
进一步分析第一个参数link_extractor。顾名思义，即链接提取器，它的作用是从response对象中获取链接，并且该链接接下来会被爬取。其参数结构如下：
``` bash
classscrapy.contrib.linkextractors.sgml.SgmlLinkExtractor(  
allow=(),deny=(),allow_domains=(),deny_domains=(),deny_extensions=None,restrict_xpaths=(),tags=('a','area'),attrs=('href'),canonicalize=True,unique=True,process_value=None) 
```
满足第一个参数allow内的正则表达式的值会被提取，如果置空则全部匹配；第二个参数deny的作用则同allow正好相反，与这个正则表达式不匹配的URL一定不提取；第三个参数allow_domains是会被提取链接的domains；第四个参数deny_domains是一定不会被提取链接的domains；另外restrict_xpaths这个参数中用XPath表达式，同allow共同作用过滤链接。
因此，我们在爬取过程中就是寻找所有符合/2015且满足XPath表达式的链接。在提取到这些链接后调用parse_item()函数来进行后面的操作。后续操作的方法同上一个例子“爬取自己的blog”基本相同，此处不再赘述。
编写完成后执行该爬虫，可以看到如下的结果：
![](/img/2015092303.jpg)
最后本地保存的文件如下：
``` bash
{"article_name": ["
      管理你的Linux桌面启动器
    "], "article_url": "http://hexo.rickyxu.me/2015/09/10/%E7%AE%A1%E7%90%86%E4%BD%A0%E7%9A%84Linux%E6%A1%8C%E9%9D%A2%E5%90%AF%E5%8A%A8%E5%99%A8/"}
{"article_name": ["
      DIABLO 3 Patch 2.3.0 will be released today!
    "], "article_url": "http://hexo.rickyxu.me/2015/08/28/DIABLO-3-Patch-2-3-0-will-be-released-today/"}
{"article_name": ["
      将hexo搭建的博客绑定至子域名
    "], "article_url": "http://hexo.rickyxu.me/2015/08/27/%E5%B0%86hexo%E6%90%AD%E5%BB%BA%E7%9A%84%E5%8D%9A%E5%AE%A2%E7%BB%91%E5%AE%9A%E8%87%B3%E5%AD%90%E5%9F%9F%E5%90%8D/"}
{"article_name": ["
      My first Hexo article
    "], "article_url": "http://hexo.rickyxu.me/2015/08/25/My-first-Hexo-article/"}
{"article_name": ["
      Hexo搭建Github静态博客
    "], "article_url": "http://hexo.rickyxu.me/2015/08/24/Hexo%E6%90%AD%E5%BB%BAGithub%E9%9D%99%E6%80%81%E5%8D%9A%E5%AE%A2/"}
{"article_name": ["
      为文件夹建立.txt的树形目录
    "], "article_url": "http://hexo.rickyxu.me/2015/05/12/%E4%B8%BA%E6%96%87%E4%BB%B6%E5%A4%B9%E5%BB%BA%E7%AB%8B-txt%E7%9A%84%E6%A0%91%E5%BD%A2%E7%9B%AE%E5%BD%95/"}
{"article_name": ["
      Extmail for Linux CentOS 5
    "], "article_url": "http://hexo.rickyxu.me/2015/05/11/Extmail-for-Linux-CentOS-5/"}
{"article_name": ["
      Extmail中删除服务器中用户若干天前的邮件
    "], "article_url": "http://hexo.rickyxu.me/2015/05/11/Extmail%E4%B8%AD%E5%88%A0%E9%99%A4%E7%94%A8%E6%88%B7%E8%8B%A5%E5%B9%B2%E5%A4%A9%E5%89%8D%E7%9A%84%E6%96%87%E4%BB%B6/"}
{"article_name": ["
      .wav声音文件的二三事
    "], "article_url": "http://hexo.rickyxu.me/2015/04/21/wav%E5%A3%B0%E9%9F%B3%E6%96%87%E4%BB%B6%E7%9A%84%E4%BA%8C%E4%B8%89%E4%BA%8B/"}
```
基本达到了我们预期的目标。**但是，如果仔细观察输出结果和代码，还会发现一些问题：**
- 起始的链接即start_url的内容并没有被爬取(**经过和一个同学讨论可能是因为其实链接的request是调用的默认的parse函数而非parse_item函数，有待验证**）；
- 如果在rules中不添加allow中2015的限制，仅仅添加XPath的限制，爬取的时候会很卡，这也是比较奇怪的现象；
- 同上一个范例“爬取自己的blog”XPath限制下一页链接的语句不同。上一个范例中的语句为
``` bash
"//a[@id='article-nav-older']//../@href"
```
用Xpath Checker获取的内容为：
![](/img/2015092304.jpg)
符合我们的想法，获取到下一篇文章的链接。
但是本例子中如果采用和上个例子中一样的XPath语句，则执行后爬不到任何语句。显示Crawl 0。而采用下列Xpath语句则会取得正确的结果：
``` bash
//a[@id='article-nav-older']
```
用Xpath Checker获取的内容为：
![](/img/2015092305.jpg)
**不清楚为什么非得用这样匪夷所思的链接去筛选**
- 如果把XPath语句删掉，仅仅用allow中的正则表达式去限制，竟然还可以得到不删XPath语句的结果。我猜想是符合正则表达式的链接正好仅仅只有下一篇文章的链接，然后给碰巧了。

同时，在编写这个爬虫的时候发现了一个很致命的问题，导致我折腾了一晚上，最后还是在[这里](http://stackoverflow.com/questions/10570635/scrapy-importerror-no-module-named-items)找到了答案，那就是在编写工程的时候，**工程名、爬虫名、编写爬虫的文件名能不一样尽量做到不一样，否则会有很麻烦的问题产生。**

## 防止被禁止访问的策略--User Agent
User Agent指的是一串包含浏览器信息、操作系统信息等的字符串。服务器主要是通过它来判断当前访问对象是浏览器、客户端还是网络爬虫。例如，当调用scrapy shell后，访问request.headers即可查看user agent。
``` bash
scarpy shell http://hexo.rickyxu.me
```
之后通过request.headers就可以得到user agent的信息：
``` bash
>>> request.headers
{'Accept-Language': ['en'], 'Accept-Encoding': ['gzip,deflate'], 'Accept': ['text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8'], 'User-Agent': ['Scrapy/1.0.3 (+http://scrapy.org)']}
```
从中可以看出我们的User-Agent信息为scrapy/1.0.3。
很多网站都会对爬虫进行禁止，我们在之前的例子中采用过通过设置download_delay的方法与禁止cookies的方法来防止被禁止。这里还可以使用user agent池。
使用user agent池需要编写自己的UserAgentMiddle中间件，新建rotate_useragent.py：
``` bash
# -*-coding:utf-8-*-  
  
from scrapy import log  
  
"""避免被ban策略之一：使用useragent池。 
 
使用注意：需在settings.py中进行相应的设置。 
"""  
  
import random  
from scrapy.contrib.downloadermiddleware.useragent import UserAgentMiddleware  
  
class RotateUserAgentMiddleware(UserAgentMiddleware):  
  
    def __init__(self, user_agent=''):  
        self.user_agent = user_agent  
  
    def process_request(self, request, spider):  
        ua = random.choice(self.user_agent_list)  
        if ua:  
            #显示当前使用的useragent  
            print "********Current UserAgent:%s************" %ua  
  
            #记录  
            log.msg('Current UserAgent: '+ua, level='INFO')  
            request.headers.setdefault('User-Agent', ua)  
  
    #the default user_agent_list composes chrome,I E,firefox,Mozilla,opera,netscape  
    #for more user agent strings,you can find it in http://www.useragentstring.com/pages/useragentstring.php  
    user_agent_list = [\  
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.1 "  
        "(KHTML, like Gecko) Chrome/22.0.1207.1 Safari/537.1",  
        "Mozilla/5.0 (X11; CrOS i686 2268.111.0) AppleWebKit/536.11 "  
        "(KHTML, like Gecko) Chrome/20.0.1132.57 Safari/536.11",  
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.6 "  
        "(KHTML, like Gecko) Chrome/20.0.1092.0 Safari/536.6",  
        "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.6 "  
        "(KHTML, like Gecko) Chrome/20.0.1090.0 Safari/536.6",  
        "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.1 "  
        "(KHTML, like Gecko) Chrome/19.77.34.5 Safari/537.1",  
        "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/536.5 "  
        "(KHTML, like Gecko) Chrome/19.0.1084.9 Safari/536.5",  
        "Mozilla/5.0 (Windows NT 6.0) AppleWebKit/536.5 "  
        "(KHTML, like Gecko) Chrome/19.0.1084.36 Safari/536.5",  
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "  
        "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3",  
        "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/536.3 "  
        "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3",  
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_0) AppleWebKit/536.3 "  
        "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3",  
        "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "  
        "(KHTML, like Gecko) Chrome/19.0.1062.0 Safari/536.3",  
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "  
        "(KHTML, like Gecko) Chrome/19.0.1062.0 Safari/536.3",  
        "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "  
        "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3",  
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "  
        "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3",  
        "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/536.3 "  
        "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3",  
        "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "  
        "(KHTML, like Gecko) Chrome/19.0.1061.0 Safari/536.3",  
        "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/535.24 "  
        "(KHTML, like Gecko) Chrome/19.0.1055.1 Safari/535.24",  
        "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/535.24 "  
        "(KHTML, like Gecko) Chrome/19.0.1055.1 Safari/535.24"  
       ]  
```
分析以上代码：
首先是库文件的导入，包括log库文件、随机库文件以及UserAgent相关的库文件。
接着定义了继承自UserAgentMiddleware类的子类RotateUserAgentMiddleware，包含了一个构造函数与一个普通的行为函数，同时还有定义的user agent池。
构造函数这里不再赘述，主要来分析process_request()这个函数。本函数首先从user agent池中随机选择出一个user agent来，(random.choice()从序列中获取一个随机元素)，然后赋给ua，之后调用request.headers.setdefault来把获取的随机user agent值赋给'User-Agent'。最后的user agent池可以看到是一些产生的user agent，包含Chrome、IE、Safari等信息。
在编写完后，还需要修改设置部分的内容，即工程的settings.py：
``` bash
#取消默认的useragent,使用新的useragent  
DOWNLOADER_MIDDLEWARES = {  
        'scrapy.contrib.downloadermiddleware.useragent.UserAgentMiddleware' : None,  
        'HexoBlog.spiders.rotate_useragent.RotateUserAgentMiddleware' :400  
    } 
```







