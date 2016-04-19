title: Python网络爬虫入门笔记--Scrapy与MongoDB数据库
date: 2015-10-08 16:03:09
tags: Python
---
很多时候，爬虫爬取的结果并不是直接保存在本地的文件中，而是通过调用数据库来进行保存。以下简单介绍下Scrapy利用MongoDB数据库来保存爬取的数据。
这里以对[stackoverflow.com](http://www.stackoverflow.)的新问题爬取为例，介绍Scrapy与MongoDB数据库的使用。
<!-- more -->
## 安装与配置MongoDB数据库
这里仅介绍windows平台下的MongoDB数据库的安装和使用方法。
首先，在[MongoDB官网](http://mongodb.org)下载最新的安装包(v3.0.6)。但是由于网络问题，这个地址下载了好多次都没能成功。在[这个地址](http://xz.cr173.com/soft2/mongodb.zip)下载到了最新版本(v3.0.6)的压缩包。
下载到压缩包后，把压缩包解压并且将其中的内容（文件夹bin）放置在C:\mongodb中。然后在C:\mongodb中建立文件夹data，在data文件夹中建立子文件夹db与子文件夹log，然后在子文件夹log中建立MongoDB.log文件。至此，C:\mongodb文件夹中的PATH列表如下所示：
C:\mongodb.
│  
├─bin
│      bsondump.exe
│      mongo.exe
│      mongod
│      mongod.exe
│      mongod.pdb
│      mongodb
│      mongodump.exe
│      mongoexport.exe
│      mongofiles.exe
│      mongoimport.exe
│      mongooplog.exe
│      mongoperf.exe
│      mongorestore.exe
│      mongos.exe
│      mongos.pdb
│      mongostat.exe
│      mongotop.exe
│      
└─data
    ├─db
    └─log
            MongoDB.log
创建好文件夹后，在命令提示符中切换到目录C:\mongodb\bin，依次执行以下两条指令：
``` bash
C:\mongodb\bin>mongod.exe
```
与
``` bash
C:\mongodb\bin>mongod.exe --logpath=C:\mongodb\data\log\MongoDB.log --dbpath=C:\mongodb\data\db
```
会出现如下结果：
![](/img/Scrapy&Mongo/2015100801.jpg)
之后开启另一个命令提示符，在C:\mongodb\bin文件夹下键入mongo，出现如下图：
![](/img/Scrapy&Mongo/2015100802.jpg)
或者在浏览器中打开[http://localhost:27017](http://localhost:27017)，出现如下图所示则说明MongoDB安装成功：
![](/img/Scrapy&Mongo/2015100803.jpg)
mongodb默认连接端口为27017。
**注：通过以上方法安装后，每次在使用MongoDB数据库的时候需要重复输入指令的步骤**
关于MongoDB更详细的了解，我主要参考了[PyMongo--非关系型数据库mongodb入门（一步一步版）](http://www.cnblogs.com/2010Freeze/archive/2012/06/25/2560863.html)与[windows下mongodb安装与使用整理](http://www.cnblogs.com/lecaf/archive/2013/08/23/mongodb.html)
## 安装pymongo库
在[https://pypi.python.org/pypi/pymongo/#downloads](https://pypi.python.org/pypi/pymongo/#downloads)中下载对应平台与Python版本的pymongo库即可，我这儿下载的是MS Windows installer版的。
但是，在安装pymongo的时候发现系统找不到Python的路径，但是明明我已经安装Python 2.7了。随后参考了[安装第三方库出现 Python version 2.7 required, which was not found in the registry](http://blog.csdn.net/zklth/article/details/8117207)，只需要建立一个文件register.py，内容如下，然后执行该脚本即可：
``` bash
import sys  
    
from _winreg import *  
    
# tweak as necessary  
version = sys.version[:3]  
installpath = sys.prefix  
    
regpath = "SOFTWARE\\Python\\Pythoncore\\%s\\" % (version)  
installkey = "InstallPath"  
pythonkey = "PythonPath"  
pythonpath = "%s;%s\\Lib\\;%s\\DLLs\\" % (  
    installpath, installpath, installpath  
)  
    
def RegisterPy():  
    try:  
        reg = OpenKey(HKEY_CURRENT_USER, regpath)  
    except EnvironmentError as e:  
        try:  
            reg = CreateKey(HKEY_CURRENT_USER, regpath)  
            SetValue(reg, installkey, REG_SZ, installpath)  
            SetValue(reg, pythonkey, REG_SZ, pythonpath)  
            CloseKey(reg)  
        except:  
            print "*** Unable to register!"  
            return  
        print "--- Python", version, "is now registered!"  
        return  
    if (QueryValue(reg, installkey) == installpath and  
        QueryValue(reg, pythonkey) == pythonpath):  
        CloseKey(reg)  
        print "=== Python", version, "is already registered!"  
        return  
    CloseKey(reg)  
    print "*** Unable to register!"  
    print "*** You probably have another Python installation!"  
      
if __name__ == "__main__":  
    RegisterPy()  
```
## 试用MongoDB数据库来保存爬虫结果
### 爬虫的编写
爬取著名的[http://stackoverflow.com/](http://stackoverflow.com/)Newest Questions:
首先，建立Scrapy工程stack：
``` bash
scrapy startproject stack
```
定义如下的Item：
``` bash
# -*- coding: utf-8 -*-

# Define here the models for your scraped items
#
# See documentation in:
# http://doc.scrapy.org/en/latest/topics/items.html

import scrapy
from scrapy.item import Item, Field

class StackItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    title = Field()
    url = Field()
```
定义如下的Spiders：
``` bash
# -*- coding:utf-8 -*-

from scrapy import Spider
from scrapy.selector import Selector
from stack.items import StackItem
class StackSpider(Spider):
	name = "stack"
	allowed_domains = ["stackoverflow.com"]
	start_urls = ["http://stackoverflow.com/questions?pagesize=50&sort=newest"]

	def parse(self, response):
		questions = Selector(response).xpath('//div[@class="summary"]/h3')
		for question in questions:
			item = StackItem()
			item['title'] = question.xpath('a[@class="question-hyperlink"]/text()').extract()[0]
			item['url'] = question.xpath( 'a[@class="question-hyperlink"]/@href' ).extract()[0]
			yield item
```
此时，如果执行该爬虫，则会得到如下所示的爬取结果:
``` bash
2015-10-08 16:27:58 [scrapy] DEBUG: Scraped from <200 http://stackoverflow.com/questions?pagesize=50&sort=newest>
{'title': u'Python script that takes an array and return all elements [duplicate]',
 'url': u'/questions/33010364/python-script-that-takes-an-array-and-return-all-elements'}
```
以上内容在上一篇文章中都有过详细的介绍，不再赘述。
### 在MongoDB中存储数据
在使用MongoDB之前，有必要先了解下MongoDB与传统数据库MySQL的区别。参考了[Mongo db 与mysql 语法比较](http://www.cnblogs.com/xffy1028/archive/2011/12/03/2272837.html)
#### 修改pipelines.py文件
为了将数据保存至MongoDB数据库，应该对pipelines.py文件做如下修改：
``` bash
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: http://doc.scrapy.org/en/latest/topics/item-pipeline.html
import pymongo
from scrapy.conf import settings
from scrapy.exceptions import DropItem
from scrapy import log
from pymongo import MongoClient

#class StackPipeline(object):
#    def process_item(self, item, spider):
#        return item

class MongoDBPipeline( object ):
    def __init__( self ):
#    	connection = pymongo.Connection(
#        settings[ 'MONGODB_SERVER' ],
#        settings[ 'MONGODB_PORT' ]
#        )
		connection = MongoClient()
		db = connection[settings['MONGODB_DB']]
		self.collection=db[settings['MONGODB_COLLECTION']]
#        self.collection = db[settings['MONGODB_COLLECTION']]


    def process_item(self, item, spider):
        valid = True
        for data in item:
            if not data:
                valid = False
                raise DropItem("Missing {0}!".format(data))
        if valid:
            self.collection.insert(dict(item))
            log.msg("Question added to MongoDB database!",
                    level=log.DEBUG, spider=spider)
        return item
```
参考范例中应用了pymongo库中的Connection方法，但是在爬虫执行的过程中总提示该库文件中没有Connection方法。经过我了解后的确pymongo库中不存在Connection方法，参考[Python无法连接mongodb：'module' object has no attribute 'Connection'](http://zhidao.baidu.com/link?url=hSG7WDEmCYJEkAmElNwQLvnVZmenqhgtB_i853O_OIYTDbS1rE_aDuJKFOGD2nukP7vR9sBTOfJedbJqkaxatebQk8Xc_QK7C9WIEjFyQGK)。因此我把原来代码中的Connection方法都注释掉了，导入MongoClient库来使用。
#### 修改settings.py文件
在settings.py文件中加入以下内容：
``` bash
ITEM_PIPELINES = ['stack.pipelines.MongoDBPipeline', ] 
MONGODB_SERVER = "localhost" 
MONGODB_PORT = 27017 
MONGODB_DB = "stackoverflow" 
MONGODB_COLLECTION = "questions"
```
因为网上并没有特别详细的对Scrapy采用MongoDB的原理的介绍，因此我这儿简单写一下我自己的理解。
首先settings.py文件中定义了类似于C语言工程中的一些typedef变量：采用的pipelines为MongoDBPipeline，MongoDB数据库的服务器地址为本机localhost，端口号为27017（MongoDB的默认连接端口就是27017），连接的数据库名称为stackoverflow，集合collection（对应MySQL数据库中的表table一样的东西）为questions。
在pipelines.py文件中，先是导入了一些将要采用的库文件。接着就是对MongoDBPipeline这个类的定义。类中包含了两个函数，自身的构造函数和对item处理的process_item()函数。
构造函数主要完成的是对数据库的连接操作。调用MongoClient()方法连接到了本机localhost的27017端口，之后又调用settings中的‘MONGODB_DB’指定了具体的数据库名称，调用settings中的'MONGODB_COLLECTION'指定了连接数据库的进一步的层级（实在是不清楚collection对应的下一层在MongoDB中叫啥名字了。。。）。
另一个函数process_item()则是对爬取到的item进行处理：对于爬取到的item中的每一个数据，如果该数据为空，则引发异常DropItem（关于可能的异常，参考[Built-in Exceptions reference](http://doc.scrapy.org/en/0.24/topics/exceptions.html#scrapy.exceptions.DropItem)）。即遇到该异常后，停职处理item。否则，如果爬取到的item中存在数据，则将item转换为dict并且插入到MongoDB数据库的collection中，同时打印输出日志文件。
### 运行结果
爬虫执行后输出与原来简单的不采用MongoDB数据库存储的结果类似。但加入了把内容增添到MongoDB数据库的调试信息：
``` bash
2015-10-08 17:34:12 [scrapy] DEBUG: Question added to MongoDB database!
2015-10-08 17:34:12 [scrapy] DEBUG: Scraped from <200 http://stackoverflow.com/questions?pagesize=50&sort=newest>
{'title': u'Bootstrap columns same height inside ng-repeat','url': u'/questions/33011753/bootstrap-columns-same-height-inside-ng-repeat'}
```
在命令提示符中访问数据库
``` bash
db.questions.find()
```
可以看到如下内容：
``` bash
> db.questions.find()
{ "_id" : ObjectId("561638929b1f2620cc70d18a"), "url" : "/questions/33011837/oauth2-providerkey-externallogininfo-c-sharp-mvc-5", "title" : "OAUTH2 PROVIDERKEY EXTERNALLOGININFO C# MVC 5" }
{ "_id" : ObjectId("561638939b1f2620cc70d18b"), "url" : "/questions/33011832/python-fail-install-a-github-script-hls-server-loop", "title" : "Python fail install a github script (HLS Server loop)" }

{ "_id" : ObjectId("561638939b1f2620cc70d18c"), "url" : "/questions/33011831/share-on-facebook-using-mobile-is-not-working", "title" : "share on facebook using mobile is not working" }
{ "_id" : ObjectId("561638939b1f2620cc70d18d"), "url" : "/questions/33011828/typeerror-draggable-is-not-a-function-and-typeerror-browser-is-und", "title" : "“TypeError: $(…).draggable is not a function” and “TypeError: $.browser is undefined”" }
{ "_id" : ObjectId("561638939b1f2620cc70d18e"), "url" : "/questions/33011827/excess-record-in-cookies", "title" : "Excess record in cookies" }
{ "_id" : ObjectId("561638939b1f2620cc70d18f"), "url" : "/questions/33011825/learning-rate-of-a-q-learning-agent", "title" : "Learning rate of a Q learning agent" }
{ "_id" : ObjectId("561638939b1f2620cc70d190"), "url" : "/questions/33011823/clear-framgnetbackstack-when-particular-fragment-was-opened", "title" : "Clear framgnetBackStack when particular fragment was opened" }
{ "_id" : ObjectId("561638939b1f2620cc70d191"), "url" : "/questions/33011821/jquery-changing-img-src-of-html-image-doesnt-refresh", "title" : "jQuery changing img src of html image doesn't refresh" }
{ "_id" : ObjectId("561638939b1f2620cc70d192"), "url" : "/questions/33011819/ruby-application-with-user-input-that-calculates-food-order-with-tax-and-discoun", "title" : "Ruby application with user input that calculates food order with tax and discount" }
{ "_id" : ObjectId("561638939b1f2620cc70d193"), "url" : "/questions/33011817/how-to-make-tab-bar-icon-image-small", "title" : "How to make tab bar icon image small?" }
{ "_id" : ObjectId("561638939b1f2620cc70d194"), "url" : "/questions/33011816/why-travis-ci-org-always-show-build-failing-badge", "title" : "Why travis-ci.org always show [ build : failing ] badge?" }
{ "_id" : ObjectId("561638939b1f2620cc70d195"), "url" : "/questions/33011815/telerik-radcombobox-selectedvalue-not-working", "title" : "Telerik Radcombobox.SelectedValue not working" }
{ "_id" : ObjectId("561638939b1f2620cc70d196"), "url" : "/questions/33011814/chart-js-set-a-max-value", "title" : "Chart.js, set a max value" }
{ "_id" : ObjectId("561638939b1f2620cc70d197"), "url" : "/questions/33011813/ext-slider-multi-scale-change-color-between-thumbs", "title" : "Ext.slider.Multi : scale & change color between thumbs" }

{ "_id" : ObjectId("561638939b1f2620cc70d198"), "url" : "/questions/33011809/how-to-change-user-location-pin-image-in-mkmapview-type-satellite", "title" : "how to change user location pin image in mkmapview type=satellite?" }
{ "_id" : ObjectId("561638939b1f2620cc70d199"), "url" : "/questions/33011808/error-while-running-the-windows-emulator-in-appcelerator", "title" : "Error while running the windows emulator in appcelerator" }
{ "_id" : ObjectId("561638939b1f2620cc70d19a"), "url" : "/questions/33011807/create-a-little-2d-game-with-c-sharp-displayed-in-wpf", "title" : "create a little 2d game with c# displayed in wpf" }
{ "_id" : ObjectId("561638939b1f2620cc70d19b"), "url" : "/questions/33011804/broken-ssl-paths-after-build-python-app-with-py2app", "title" : "Broken ssl paths after build python app with py2app" }
{ "_id" : ObjectId("561638939b1f2620cc70d19c"), "url" : "/questions/33011800/creating-gene-coverage-plot-in-r", "title" : "Creating Gene Coverage Plot in R" }
{ "_id" : ObjectId("561638939b1f2620cc70d19d"), "url" : "/questions/33011796/how-to-add-a-pdf-file-to-a-visual-studio-project", "title" : "How to add a PDF file to a visual studio project?" }
Type "it" for more
```
另外，借用可视化数据库工具Robomongo也可以看到在数据库中存储的结果。
![](/img/Scrapy&Mongo/2015100804.jpg)