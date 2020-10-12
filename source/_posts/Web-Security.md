---
title: 'Web Security '
date: 2020-10-12 10:21:52
tags:
      - 网络
---

### 1. DHCP协议
动态主机配置协议（Dynamic Host Configuration Protocol,DHCP）是一个用于局域网的网络通讯协议，位于OSI模型的应用层，使用UDP协议工作，主要用于自动分配地址给用户，方便管理员进行统一管理。
报文格式如下：
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/security/DHCP.jpg)

### 2.路由算法

  在因特网中，路由协议可以分为内部网关协议IGP和外部网关协议EGP
### 3.域名系统

  DNS是一个简单的请求-响应协议，是将域名和ip地址相互映射的一个分布式数据库，能够使人更方便的访问互联网，DNS使用TCP和UDP协议的53端口
域名系统工作原理

  **DNS解析过程是递归查询的，具体过程如下：**
  
  * 用户要访问域名www.example.com时，先查看本机hosts是否有记录或者本机是否有DNS缓存，如果有，直接返回结果，否则向递归服务器查询该域名的IP地址
  * 递归缓存为空时，首先向根服务器查询com顶级域的IP地址
  * 根服务器告知递归服务器com顶级域名服务器的IP地址
  * 递归向com顶级域名服务器查询负责example.com的权威服务器的IP
  * com顶级域名服务器返回相应的IP地址
  * 递归向example.com的权威服务器查询www.example.com的地址记录
  * 权威服务器告知www.example.com的地址记录
  * 递归服务器将查询结果返回客户端
  
### 4.搜索引擎利用

* site:www.hao123.com

  返回此目标站点被搜索引擎抓取收录的所有内容
* site:www.hao123.com keyword

  返回此目标站点被搜索引擎抓取收录的包含此关键词的所有页面
  
  此处可以将关键词设定为网站后台，管理后台，密码修改，密码找回等
* site:www.hao123.com inurl:admin.php

  返回目标站点的地址中包含admin.php的所有页面，可以使用admin.php/manage.php或者其他关键词来寻找关键功能页面
* link:www.hao123.com

  返回所有包含目标站点链接的页面，其中包括其开发人员的个人博客，开发日志，或者开放这个站点的第三方公司，合作伙伴等
* related:www.hao123.com

  返回所有与目标站点”相似”的页面，可能会包含一些通用程序的信息等
  
* intitle:"500 Internal Server Error" "server at"

  搜索出错的页面
  
* inurl:"nph-proxy.cgi" "Start browsing"

## 运营商网络架构
  
### 链路层
 
1. LAN:以太网、802.3
2. WAN: HDLC/PPP

### 网络层
 
1. 是否对传输数据具有绝对寻址能力
2. 局域网私有ip
3. 广域网公有ip
  
### 运营商网络总体架构
 
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/security/operator.png)
 
### 运营商接入网
 
  宽带端到端是用户终端至各网站服务器的连接，目前主流接入方式包括ADSL、LAN、PON三种
  
### DCN与网游加速器