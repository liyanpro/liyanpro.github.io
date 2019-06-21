---
title: 初窥Scrapy
date: 2018-02-27 15:35:34
header-img: https://bucketyy.oss-cn-beijing.aliyuncs.com/image/post-bg-os-metro.jpg
tags:
---

Scrapy是一个为了爬取网站数据，提取结构性数据而编写的应用框架。可以应用在包括数据挖掘，信息处理或存储历史数据等一系列的程序中。

**安装**
使用python的包管理器pip进行安装

	pip install scrapy

**创建项目**

这里以爬取打开shell，进入需要创建项目的路径，执行命令：

	scrapy startproject myproject
myproject为项目名称，执行完生成一个完整的爬虫项目，目录结构如下图。
spiders包中的wechat_spider.py是自己新建的，主要写爬取逻辑；middlewares.py为中间件，里面可以定义下载器中间件、spider中间件等等；item为一组数据，经过数据流在pipline中进行处理，或存储在文本，或存储在数据库中；settings是整个scrapy项目的配置
![图片.png](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/scrapy.png)

>注：项目中的main.py文件是我添加的方便在pycharm中调试，使用cmdline来执行shell命令，内容如下：
from scrapy import cmdline
cmdline.execute("scrapy crawl wechat".split())

**编写spider**
    
     import scrapy 
     from ..items import WechatItem
     class WechatSpider(scrapy.Spider):
     name = "wechat"
     allowed_domains = ["weixin.sogou.com"]
     start_urls = [
        "http://weixin.sogou.com"
     ]
     def parse(self,response):
         item = WechatItem()
         #item['user'] = response.xpath('//p[@title]/text()').extract()
         item['content'] = response.xpath('//p[@class="txt-info"]/text()').extract()
         item['text'] = response.xpath('//h3/a/text()').extract()
         item['title'] = response.xpath('//div[@class="fieed-box"]/a/text()').extract()
         return item

allowed_domains是请求的域，允许请求的范围，start_urls是请求地址，parse函数是对返回的结果进行处理，定义一个item用来接收解析的数据，使用[xpath]("http://www.w3school.com.cn/xpath/index.asp")来解析返回的html页面，最后将item返回给pipline

**编写pipline**
  以下pipeline将所有(从所有spider中)爬取到的item，存储到一个独立地 wechat.json 文件，每行包含一个序列化为JSON格式的item:

    import json
    class WechatPipeline(object):
    def __init__(self):  
        self.file = open('wechat.json','wb',encoding='utf-8')
    def process_item(self, item, spider):
        line = json.dumps(dict(item)) + '\n'
        self.file.write(line.decode("unicode_escape"))
        return item
**Scrapy shell**
Scrapy shell是一个交互式的终端，在还没有启动scrapy项目的情况下尝试爬取及调试你的代码，在爬虫开发中很方便，绝对是利器。例如测试豆瓣图书的api能否返回图书信息

    scrapy shell  https://api.douban.com/v2/book
执行后将通过scrapy的下载器返回整个response，输入response.status，查看请求的状态，状态码为200说明请求成功，数据体随响应返回给客户端，输入response.text查看返回的内容，也可以使用xpath选择器对返回的页面进行信息筛选。
![图片.png](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/scrapy2.png)
Scrapy shell 也可以添加请求头,例如添加header中的User-Agent

       scrapy shell -s USER_AGENT="Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, likeGecko) Chrome/59.0.3071.86 Safari/537.36"  url　　#url指你所需要爬的网址
*框架概览*
[scrapy官方文档](http://scrapy-chs.readthedocs.io/zh_CN/0.24/topics/architecture.html)
