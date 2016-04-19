title: Python网络爬虫入门笔记--保存利用Scrapy爬取到的图片
date: 2015-10-10 16:08:04
tags: Python
---
有了前两篇文章的基础，现在我们已经可以利用Scrapy编写简单的爬虫，并且将爬取到的内容保存到数据库中。但是网上的资源并不限于仅仅有链接和文字。在这篇文章中，我参考一个爬取一个图片网站的工程的[范例](https://github.com/ZhangBohan/fun_crawler)，了解利用pipelines来保存爬取到的链接的图片。
<!-- more -->
相信有了前两篇文章中爬虫工程的经验，现在对爬虫的认识已经集中到了spiders、pipelines、settings和items这四个方面上。因此现在就逐一分析以上四个文件的实现。
## items
items的编写算是这几个对象编写中最容易的一个了，无非是导入相关的库文件然后定义一个我们要爬取的类。该工程的items.py的编写如下：
``` python
# -*- coding: utf-8 -*-

# Define here the models for your scraped items
#
# See documentation in:
# http://doc.scrapy.org/en/latest/topics/items.html

import scrapy
from scrapy.contrib.loader import ItemLoader
from scrapy.contrib.loader.processor import MapCompose, TakeFirst, Join


class MeizituItem(scrapy.Item):
    url = scrapy.Field()
    name = scrapy.Field()
    tags = scrapy.Field()
    image_urls = scrapy.Field()
    images = scrapy.Field()
```
从中可以看出，定义的items类包含了url、name、tags、image_urls和images这几个成员。
## settings
结合前几个爬虫工程，我对settings文件的认识是“对工程所采用的方法的声明”。如采用什么样的pipelines等。该工程的settings文件如下所示：
``` bash
# -*- coding: utf-8 -*-

# Scrapy settings for fun project
#
# For simplicity, this file contains only the most important settings by
# default. All the other settings are documented here:
#
#     http://doc.scrapy.org/en/latest/topics/settings.html
#

BOT_NAME = 'fun_crawler'

SPIDER_MODULES = ['fun_crawler.spiders']
NEWSPIDER_MODULE = 'fun_crawler.spiders'

ITEM_PIPELINES = {'fun_crawler.pipelines.ImageDownloadPipeline': 1}

IMAGES_STORE = '/tmp/images'


DOWNLOAD_DELAY = 0.25    # 250 ms of delay
```
从中可以看出，比较关键的几点是对使用的pipelines的声明、对下载图片链接的声明和下载延时的设置。在本工程的settings文件中声明了采用ImagDownloadPipelin，并且设置了保存下载图片的路径'/tmp/images'，同时将下载延时设置为0.25秒。
## pipelines
结合前几个简单的scrapy爬虫工程，我对pipelines的理解是“对爬取到的Item的再加工”。如保存为本地的json文件或调用数据库。在本例中，采用了一种之前从来没有用过的方法：将爬取到的链接的图片下载到本地硬盘中。本例中的pipelins文件的编写如下：
``` bash
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: http://doc.scrapy.org/en/latest/topics/item-pipeline.html
import requests
from fun_crawler import settings
import os


class ImageDownloadPipeline(object):
    def process_item(self, item, spider):
        if 'image_urls' in item:
            images = []
            dir_path = '%s/%s' % (settings.IMAGES_STORE, spider.name)

            if not os.path.exists(dir_path):
                os.makedirs(dir_path)
            for image_url in item['image_urls']:
                us = image_url.split('/')[3:]
                image_file_name = '_'.join(us)
                file_path = '%s/%s' % (dir_path, image_file_name)
                images.append(file_path)
                if os.path.exists(file_path):
                    continue

                with open(file_path, 'wb') as handle:
                    response = requests.get(image_url, stream=True)
                    for block in response.iter_content(1024):
                        if not block:
                            break

                        handle.write(block)

            item['images'] = images
        return item
```
分析以上源码，首先是导入requests、os标准库与settings中的设置。在我的windows7+Python2.7平台默认缺少requests库，我在命令提示符中采用'pip install requests'来安装了requests库。关于requests标准库的介绍可以参考[Requests:HTTP for Humans](http://cn.python-requests.org/en/latest/)和[python requests的安装与简单运用](http://www.zhidaow.com/post/python-requests-install-and-brief-introduction)。基本上可以认为requests标准库是更人性化的urllib2模块，它能够用更简单的代码实现同样的工作，参考范例中有很多基本的调用requests标准库进行的操作。
接着，定义了将要使用的Pipelines类ImageDownloadPipeline类，对爬取到的Items进行了再处理。该类中仅有一个方法：process_item()。该方法首先根据settings中定义的IMAGES_STORE和spider的名称来创建将要保存爬取到结果的路径，对于本例，即'/tmp/images/meizitu',在获取到路径后，调用os标准库中来检查该路径是否存在，如果不存在则创建该路径。接着，对于爬取到的Item中的每一个image_url，先调用split函数来对'/'进行切分，并且取出特定的一部分赋值给us，之后再用join函数将'_'加入到us中生成新的变量'image_file_name'。此时，file_path即包括dir_path与image_file_name，最后将file_path增添到images这个在一开始就定义的元组中。完成以上生成文件目录与文件名称的工作后，调用os标准库的.path.exists来判断对应的该文件是否存在，如果存在则本次循环终止，不再进行之后的保存操作，继续获取新的image_url。
当该文件不存在时，调用with...as来以二进制读来打开file_path指代的文件作为文件句柄。调用requests标准库中的get方法获取图片链接。这里，代码采用了"[响应体内容工作流](http://requests-docs-cn.readthedocs.org/zh_CN/latest/user/advanced.html)"的方法，即一般我们在进行网络请求的时候，响应体会立刻被下载。但是，通过stream参数覆盖这个行为，会推迟下载响应体直到我们访问response.content属性。而iter_content的参数可以参考[开发者接口-Requests 1.1.0文档](http://requests-docs-cn.readthedocs.org/zh_CN/latest/api.html#requests.Response.iter_content)。在本代码中，我们不立刻获取图片链接的内容到内存中，而是每次只获取1024 bytes的数据。当单次获取的数据为空的时候，说明链接对应的内容已经全部获取完毕，则终止获取图片链接数据的循环，如果不是空，则将获取的数据写入到handle文件句柄中，继续进行获取数据的循环。当链接的数据全部下载完毕后，为Item中的'images'赋值。
## 爬虫的编写
接下来就是重头戏爬虫的编写了。本爬虫想要完成的工作是爬取[一个网站](http://www.meizitu.com/)的美女图片~大晚上的在实验室反复打开这个网站，顶着同学的眼神的压力，真是压力山大啊。。。
回归正题，该爬虫实现的代码如下：
``` python
from scrapy.selector import Selector
import scrapy
from scrapy.contrib.loader import ItemLoader, Identity
from fun_crawler.items import MeizituItem


class MeizituSpider(scrapy.Spider):
    name = "meizitu"
    allowed_domains = ["meizitu.com"]
    start_urls = (
        'http://www.meizitu.com/',
    )

    def parse(self, response):
        sel = Selector(response)
        for link in sel.xpath('//h2/a/@href').extract():
            request = scrapy.Request(link, callback=self.parse_item)
            yield request

        pages = sel.xpath("//div[@class='navigation']/div[@id='wp_page_numbers']/ul/li/a/@href").extract()
        print('pages: %s' % pages)
        if len(pages) > 2:
            page_link = pages[-2]
            page_link = page_link.replace('/a/', '')
            request = scrapy.Request('http://www.meizitu.com/a/%s' % page_link, callback=self.parse)
            yield request

    def parse_item(self, response):
        l = ItemLoader(item=MeizituItem(), response=response)
        l.add_xpath('name', '//h2/a/text()')
        l.add_xpath('tags', "//div[@id='maincontent']/div[@class='postmeta  clearfix']/div[@class='metaRight']/p")
        l.add_xpath('image_urls', "//div[@id='picture']/p/img/@src", Identity())

        l.add_value('url', response.url)
        return l.load_item()
```
其实有了之前几个工程的了解，这个爬虫的算法并不是很复杂。以下做简单分析：
第一部分还是基本的标准库文件和我们之前定义的若干东西的导入。
第二部分就是爬虫类的编写：该爬虫类MeizituSpider继承自基类Spider，先定义了爬虫的名称'meizitu'，允许的爬虫域和起始链接。
第三部分是两个爬虫类方法的定义，分别是parse和parse_item。下面分别做分析：
首先，parse类获取了response对象后，利用xpath语句获取了起始页面中的链接集合，即每一篇美女图片集合的链接，对于每一个链接，再调用parse_item进行处理并增加到迭代器中。此后，获取起始页面中的导航页的链接，即网站最底下部分的页码1、2、3等等等等。并且将获取到的页面地址赋值为pages同时打印输出。当获取的导航链接的数量大于2时，得到这一系列链接的倒数第二个链接的地址，观察链接的组织结构可以知道，这里的倒数第二个地址即"下一页"的地址。
![](/img/Scrapy&Download/2015101201.jpg)
接着，利用得到的下一页地址的链接，组合成新的一个次起始页面的链接(之所以起这个名字是因为和一开始的start_url的功能类似)，然后再调用parse方法。最后同样增添到迭代器中。
之后是parse_item方法，该方法先采用了Item Loaders，具体的说明文档可参考[Item Loaders &mdash; Scrapy 1.0.3 documentation](http://doc.scrapy.org/en/latest/topics/loaders.html)，大致上可以理解为是抽取不同的东西去组合为一个Item。结合本例，即从当前页面中获取'name'、'tags'、'image_urls'和url。**这儿有一处不明白的地方，就是'image_urls'中的Identity()的用法，在[Item Loaders &mdash; Scrapy 1.0.3 documentation](http://doc.scrapy.org/en/latest/topics/loaders.html)中也并没有找到合理的解释，有待继续调试观察。**完成以上工作后调用load_item()来返回一个item。
以上就是该爬虫的基本实现分析。可能讲解有些混乱，但是基本把大体思路说了出来~
最后附上爬取成功后的一个截图：
![](/img/Scrapy&Download/2015101202.jpg)
**目前仍然存在一个问题，即爬取之后仅仅获得了起始url(即第一页)中文章中的图片，第二页及其之后的页面中文章的图片没法获得，还有待于调试。**