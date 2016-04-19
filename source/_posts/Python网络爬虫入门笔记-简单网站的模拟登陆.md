title: Python网络爬虫入门笔记--简单网站的模拟登陆
date: 2015-10-16 10:05:20
tags: Python
---
有了之前几篇文章的基础，我们已经可以对一般网站进行简单的爬取。但是个别网站，如[coursera](https://www.coursera.org/)，如果想对其进行爬取，只有在登陆后才能实现。因此，利用Python进行模拟登陆就显得非常重要了。这篇文章主要是调用Requests标准库来实现对简单网站[coursera](https://www.coursera.org/)和[北邮人论坛](http://bbs.byr.cn)进行模拟登陆。
<!-- more -->
对网站的模拟登陆背后的原理，我感觉网上的参考资料都整理的比较混乱，我主要是参考[如何用Python，C#等语言去实现抓取静态网页+抓取动态网页+模拟登陆网站](http://www.crifan.com/how_to_use_some_language_python_csharp_to_implement_crawl_website_extract_dynamic_webpage_content_emulate_login_website/)来了解它背后的原理的。在阅读了网上大量的参考资料后，我对模拟登陆的认识如下：（主要针对POST方法）

- 我们向目标url提供必要的信息如headers、cookies和需要post的其他数据。
- 获得返回的内容。

在了解了基本原理后，我又学习了用Chrome浏览器来观察登陆网站的过程中数据的收发情况。在接下来的例子中会涉及到。
# 模拟登陆Coursera
首先，虽然知道了基本的原理，但是还得学习范例来了解具体的执行流程。我参考了[一步步爬取Coursera课程资源](http://zhaofei.tk/2014/09/03/how_to_crawl_coursera/)这篇日志来学习python模拟登陆的细节。参考日志模拟了对[coursera](https://www.coursera.org/)的模拟登陆。我照着了做了一遍后，打算用自己的语言来组织出来。学习过程中使用的浏览器为Chrome
首先，想要模拟登陆，我们需要找到该网站的登陆页面，先访问[coursera](https://www.coursera.org/)的[登录页面](https://accounts.coursera.org/signin)。然后打开Chrome的"开发者工具"，并且切换到Network标签中：
![](/img/Scrape&Login/2015101601.jpg)
为了观察在登陆过程中内容的发送情况，我们需要排除掉其他的干扰因素，因此可以先有意地随便输入一串用户名和密码，点击登陆，观察"开发者工具"里的变化。可以看到，在点击"登录"后，'Network'中的新出现了两条条目，其中的'login'是我们比较感兴趣的，内容如下：
``` bash
-------General
Remote Address:52.4.1.216:443
Request URL:https://accounts.coursera.org/api/v1/login
Request Method:POST
Status Code:401 Unauthorized
-------Response Headers
view source
Cache-Control:private, no-cache, no-store, must-revalidate, max-age=0
Connection:keep-alive
Content-Encoding:gzip
Content-Type:text/plain; charset=utf-8
Date:Fri, 16 Oct 2015 01:53:41 GMT
Strict-Transport-Security:max-age=31536000
transfer-encoding:chunked
Vary:Accept-Encoding
X-Coursera-Request-Id:ugZAl3OoEeWDxwp-vHqt0w
X-Frame-Options:SAMEORIGIN
-------Request Headers
view source
Accept:*/*
Accept-Encoding:gzip, deflate
Accept-Language:zh-CN,zh;q=0.8
Connection:keep-alive
Content-Length:66
Content-Type:application/x-www-form-urlencoded
Cookie:__204u=4548148073-1444827998618; CSRF3-Token=1445691998.gVt0il3sbvvf22TZ; __204r=http%3A%2F%2Fzhaofei.tk%2F2014%2F09%2F03%2Fhow_to_crawl_coursera%2F; maestro_login_flag=1; CAUTH=DpfaQq0xGTZYlyGFAZMYlqGvnJ_TExbtudI2fvWj2dMKxT4sghvzbWtwWzIHUMWBOSNi5-c_JAkkkC9dWqHJaw.vkJMJZZ7QufWAY185RHH1w.oX9C5dTHySgK_UHrl5d8d-1qFGO6HGHhPsRlVc1bqb3IbeTf2of3dXdpseWcnLzqBbp7QNULkzYV7w6E-43WspfnQ0SGDt6vXesF5gUrKQHku_qgcwIJKXv-Yl2oG9PzhlBzrdhRbvfu99QWPOfxHmmevJExfPCHBXQSOr5oJGE; __utmt=1; __utma=158142248.1017025.1444828066.1444875509.1444960389.3; __utmb=158142248.2.10.1444960389; __utmc=158142248; __utmz=158142248.1444960389.3.2.utmcsr=zhaofei.tk|utmccn=(referral)|utmcmd=referral|utmcct=/2014/09/03/how_to_crawl_coursera/; csrftoken=N7GnCHWnaBhUcctkz22g72k6; csrf2_token_vpdL26qG=6tEX6cTT4EnmhPBR220kLNsc
Host:accounts.coursera.org
Origin:https://accounts.coursera.org
Referer:https://accounts.coursera.org/signin
User-Agent:Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.101 Safari/537.36
X-CSRF2-Cookie:csrf2_token_vpdL26qG
X-CSRF2-Token:6tEX6cTT4EnmhPBR220kLNsc
X-CSRFToken:N7GnCHWnaBhUcctkz22g72k6
X-Requested-With:XMLHttpRequest
-------Form Data
view source
view URL encoded
email:xuruiqiang@bupt.edu.cn
password:xuruiqiang
webrequest:true
```
从以上内容中我们就可以找到在登录过程中发生的内容交换。通过对以上几行日志的分析，我们可以看到post的页面地址为[https://accounts.coursera.org/api/v1/login](https://accounts.coursera.org/api/v1/login)。至于向服务器提交的内容，通过以上日志的分析也可以轻松得到。其中，最明显的要数最后几行的日志，从那里面可以看到我们刚刚输入错误的信息，这正是向服务器发送的数据，包括登录邮箱、登录密码和一个叫做'webrequest'的附加字段。同时，为了利用Requests标准库来编写模拟登录过程，我们还需要user_agent等其他内容去构建post_headers。一般情况下，指的是"Request Header"中'Referer'及其以后的部分。关于user_agent的介绍之前我们的一篇文章中已经提到过，这里不再赘述。而*Referer是浏览器向web服务器发送请求的时候附加的一部分内容，这部分内容会告诉当前访问者是从哪个页面链接过来的，服务器通过Referer可以获得一些信息用于处理，如统计访问自己页面的来源情况。*另外，可以观察到还有几项内容，包括X-CSRF2-Cookie、X-CSRF2-Token、X-CSRFToken和X-Requested-With。在参考链接中，作者提到他一开始也并不清楚这些内容的具体意思，通过参考Coursera的[批量下载脚本](https://github.com/coursera-dl/coursera)，发现以上这几项可能是服务器端用来做限制使用的。具体来讲，XCSRF2Token，XCSRFToken是长度为24的随机字符串，XCSRF2Cookie为”csrf2token“加上长度为8的随机字符串，Cookie是”csrftoken”和其他三个的组合。有了以上的分析后，我们就可以比较轻松地构造出模拟登陆Coursera的Python脚本：
``` python
import requests
import random
import string

def randomString(length):
    return ''.join(random.choice(string.letters + string.digits) for i in xrange(length))

XCSRF2Cookie = 'csrf2_token_%s' % ''.join(randomString(8))
XCSRF2Token = ''.join(randomString(24))
XCSRFToken = ''.join(randomString(24))
cookie = "csrftoken=%s; %s=%s" % (XCSRFToken, XCSRF2Cookie, XCSRF2Token)

signin_url = "https://accounts.coursera.org/api/v1/login"

logininfo = {"email": "SteveJobs@apple.com",
             "password": "fuckandroid&windows",
             "webrequest": "true"
             }

user_agent = ("Mozilla/5.0 (Windows NT 6.1; WOW64)"
              "AppleWebKit/537.36 (KHTML, like Gecko)"
              "Chrome/45.0.2454.101 Safari/537.36")

post_headers = {"User-Agent": user_agent,
                "Referer": "https://accounts.coursera.org/signin",
                "X-Requested-With": "XMLHttpRequest",
                "X-CSRF2-Cookie": XCSRF2Cookie,
                "X-CSRF2-Token": XCSRF2Token,
                "X-CSRFToken": XCSRFToken,
                "Cookie": cookie
                }

coursera_session = requests.Session()

login_res = coursera_session.post(signin_url,
                                  data=logininfo,
                                  headers=post_headers,
                                  )

if login_res.status_code == 200:
    print "Login Successfully!"

else:
    print login_res.text
```
分析以上代码：
首先是对库文件的导入，包括模拟登录最基础的库requests和生成随机字符串的库random以及字符串处理的库string。
其次是一个函数的定义，randomString返回一个指定长度的随机字符串，取值范围包括字母和数。
接着是按照说明构造的一系列post_headers中的变量、目标url、logininfo、user_agent和最红的post_headers的构建。
可以看到，loginifo就是从'开发者工具'里观察到的Form Data，而user_agent就是'开发者工具'中的User-Agent，post_headers就是'开发者工具’中Request Headers中的Referer、User-Agent及其以下的东西。
最后新定义一个Session类，然后调用该类的post方法去发送。然后根据返回的状态值来打印输出登陆成功与否的信息。
最后执行该脚本，可以看到打印输出了登陆成功的信息：
![](/img/Scrape&Login/2015101602.jpg)

# 模拟登陆北邮人论坛
在有了以上例子的基础上，我自己有尝试着模拟登陆北邮人论坛，非常容易地就成功了。
同样地，可以随意地在登陆页面输错密码，得到内容的发送情况：
``` bash
---------General
Remote Address:114.255.40.86:80
Request URL:http://bbs.byr.cn/user/ajax_login.json
Request Method:POST
Status Code:200 OK
---------Response Headers
view source
Cache-Control:no-store, no-cache, must-revalidate
Connection:Keep-Alive
Content-Encoding:gzip
Content-Length:93
Content-Type:application/json;charset=GBK
Date:Fri, 16 Oct 2015 01:55:13 GMT
Expires:Thu, 18 Feb 1988 01:00:00 GMT
Keep-Alive:timeout=5, max=100
Pragma:no-cache
Server:Apache/2.2.21 (Unix) PHP/5.3.10
Vary:Accept-Encoding
X-Powered-By:PHP/5.3.10
---------Request Headers
view source
Accept:application/json, text/javascript, */*; q=0.01
Accept-Encoding:gzip, deflate
Accept-Language:zh-CN,zh;q=0.8
Connection:keep-alive
Content-Length:31
Content-Type:application/x-www-form-urlencoded; charset=UTF-8
Cookie:nforum[UTMPUSERID]=guest; nforum[UTMPKEY]=11893242; nforum[UTMPNUM]=24226; Hm_lvt_38b0e830a659ea9a05888b924f641842=1444790293,1444829147,1444917981,1444960490; Hm_lpvt_38b0e830a659ea9a05888b924f641842=1444960495; left-index=0100000000; login-user=xuruiqiang
Host:bbs.byr.cn
Origin:http://bbs.byr.cn
Referer:http://bbs.byr.cn/
User-Agent:Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.101 Safari/537.36
X-Requested-With:XMLHttpRequest
---------Form Data
view source
view URL encoded
id:SteveJobs
passwd:TimCook
```
根据以上内容很容易编写Python模拟登录脚本，应该比Coursera的还方便：
``` python
import requests

signin_url = "http://bbs.byr.cn/user/ajax_login.json"

logininfo = {"id": "SteveJobs",
             "passwd": "TimCook"
             }

user_agent = ("Mozilla/5.0 (Windows NT 6.1; WOW64)"
              "AppleWebKit/537.36 (KHTML, like Gecko)"
              "Chrome/45.0.2454.101 Safari/537.36")

post_headers = {"User-Agent": user_agent,
                "Referer": "https://accounts.coursera.org/signin",
                "X-Requested-With": "XMLHttpRequest"
                }

coursera_session = requests.Session()

login_res = coursera_session.post(signin_url,
                                  data=logininfo,
                                  headers=post_headers,
                                  )

if login_res.status_code == 200:
    print "Login Successfully!"

else:
    print login_res.text
```
执行该脚本，即可打印出登陆成功。
但是，目前还存在一个问题：Coursera的那个脚本如果你输错密码的话会提示用户名和密码不匹配的信息"Username/password didn't match":
![](/img/Scrape&Login/2015101603.jpg)
但是北邮人论坛的脚本即使输错用户名和密码还是会显示登陆成功，看来我写的脚本还是有问题啊。
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-这里是分割线
中午的时候仔细看了Cousera和北邮人论坛登录时候的开发者工具，发现是Status Code出了问题。在Coursera中，如果用户名和密码错误则Status Code会返回401，不是输入正确的200。而在北邮人论坛的模拟登录的过程中，即使密码输入错误，仍然会返回200的Status Code。因此，仅仅靠status code来判断北邮人论坛是否模拟登录成功是不现实的。在故意输错密码的情况下，如果把login_res.text全部打印出来，则会发现是以下内容：
``` bash
{"ajax_st":0,"ajax_code":"0101","ajax_msg":"您的用户名并不存在，或者您的密码错误"}
```
而如果登陆成功，相应的返回的login_res.text为
``` bash
{"id":"SteveJobs","user_name":"SteveJobs","face_url":"http://static.byr.cn/uploadFace/P/Pod.6283.jpg","face_width":120,"face_height":120,"gender":"m","astro":"天秤座","life":666,"qq":"","msn":"","home_page":"","level":"用户","is_online":true,"post_count":743,"last_login_time":1445001729,"last_login_ip":"114.255.40.34","is_hide":false,"is_register":true,"score":17426,"is_admin":false,"first_login_time":1286449413,"login_count":12779,"stay_count":10404016,"role":"在校生用户","is_login":true,"forum_total_count":4018,"forum_user_count":1330,"forum_guest_count":2688,"new_mail":false,"full_mail":false,"new_reply":0,"new_at":0,"ajax_st":1,"ajax_code":"0005","ajax_msg":"操作成功"}
```
因此，为了提高模拟登陆北邮人论坛的健壮性，应当修改判断的机制。