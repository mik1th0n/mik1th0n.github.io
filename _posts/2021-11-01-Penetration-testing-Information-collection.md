---
layout: post
title: 渗透测试-信息搜集
categories: 渗透测试
description: 本文章主要是针对学习渗透测试的信息搜集而记录的笔记，内容包括CDN绕过及识别、站点搭建及WAF、APP及其他资产、github监控等。
keywords: 渗透测试,信息搜集
flow: true
sequence: true
mermaid: true
mathjax: true
---



>  信息收集对于渗透测试前期来说是非常重要的，因为只有我们掌握了目标网站或目标主机足够多的信息之后，我们才能更好地对其进行漏洞检测。在安全测试中，信息收集是非常重要的一个环节，此环节的信息将影响到后续的成功几率，掌握信息的多少将决定发现漏洞机会大小，换言之决定着是否能完成目标的测试任务。

![信息收集](/images/posts/2021-11-01-Penetration-testing-Information-collection.assets/web2_1.png)

## 0x00 CDN绕过技术

### 0x00.0 CDN介绍

- CDN的全称是Content Delivery Network，即内容分发网络。
- CDN是构建在现有网络基础之上的智能虚拟网络，依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等模块，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率。但在安全测试过程中，若存在CDN服务，将会影响到后续的安全测试过程。

![image.png](/images/posts/2021-11-01-Penetration-testing-Information-collection.assets/1637338803900-25872279-1779-4128-97c5-bd8afbc0dd58.png)

### 0x00.1 判断目标是否存在CDN

- 利用多节点技术进行请求返回判断
  - 超级ping网址：[http://ping.chinaz.com/](http://ping.chinaz.com/)
  - 无CDN情况：发现返回的IP都一样
  - 有CDN情况：发现有多个IP返回
- Windows命令查询：nslookup
  - nslookup命令用于查询DNS的记录，查看域名解析是否正常，在网络故障的时候用来诊断网络问题。
  - 若目标存在多个IP的话，就很有可能有CDN服务

### 0x00.2 常见的CDN绕过技术

- 子域名查询
  - gobuster进行查询
    - gobuster dns -d 域名 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -i -t 30 -r 114.114.114.114
    - dns 指定使用DNS探测模式
    - -d 指定主域名
    - -w 指定字典文件
    - -i 显示域名对应的IP地址
    - -t 指定线程数
    - -r 指定DNS服务器
  - 在线工具进行扫描
  - 为什么要进行子域名查询？
    - 因为搭建CDN要花钱，所以管理员会对主站，访问量比较大的做cdn服务，但是不会对子站做cdn，这时候就可以通过查找子域名来查找网站的真实IP，一般情况下，子站跟主站在同一个C段内
  - 子域名查询工具
    - DNS查询：https://dnsdb.io/zh-cn/
    - 微步在线：[https://x.threatbook.cn/](https://x.threatbook.cn/)
    - 在线域名信息查询：[http://toolbar.netcraft.com/site_report?url=](http://toolbar.netcraft.com/site_report?url=)
    - DNS、IP等查询：[http://viewdns.info/](http://viewdns.info/)
    - CDN查询IP：[https://tools.ipip.net/cdn.php](https://tools.ipip.net/cdn.php)
    - SecurityTrails平台： [https://securitytrails.com/domain/www.baidu.com/history/a](https://securitytrails.com/domain/www.baidu.com/history/a)
    - 在线子域名二级域名查询：[http://tools.bugscaner.com/subdomain/](http://tools.bugscaner.com/subdomain/)
  - 子域名小技巧
    - 一般情况下，“[www.XXX.com](http://www.xxx.com/) ”和XXX.com指向的是同一个DNS服务器，进入XXX.com会自动跳转到“[www.XXX.com](http://www.xxx.com/) ”，所以XXX.com不需要大流量，不用做CDN。如果加www检测不出来，可以试着去掉，或许就可以得到真实IP了
- 邮件服务查询很多公司内部都会有一个邮箱服务器，这种邮箱服务器大部分是不会做CDN的。因为邮箱服务器一般都是公司内部人去访问，所以大部分不做CDN。因此，我们就可以根据对方邮件服务器给我们发送的邮件，来判断对方的IP地址
- 国外地址查询
  - 有些网站为了节省成本，不会把CDN部署在国外。假设现在你自己的网络公司有一个网站，但你的客户群体主要是在国内，因为国外用户不多，所以就不值得在国外搭建CDN，因此这样从国外访问国内的网站就很可能直接访问的就是主站的真实ip地址。
- 遗留文件，扫描全网
  - 一些站点在搭建之初，会用一些文件测试站点，例如“phpinfo()”文件，此类文件里就有可能包含了真实的IP地址。可以利用Google搜索引擎搜索关键字“site:xxx.com inurl:phpinfo.php”，搜索站点是否有遗留文件
  - 扫描工具：fuckcdn,zmap等
- 黑暗引擎搜索特定文件

- google，shodan，zoomeye，fofa等
  - 这里的特定文件，指的是站点的icon文件，也就是网站的图标，一般查看网页源代码可以找到，格式大致“http://www.xx.com/favicon.ico ”。在shodan搜索网站icon图标的语法为：http.favicon.hash:hash值，hash是一个未知的随机数，我们可以通过shodan语法来查看一个已经被shodan收录的网站的hash值，来进一步获取到所有带有某icon的网站。
  - 获取icon的hash值
- DNS历史记录，以量打量
  - 站点在使用CDN服务之前，它的真实IP地址可能被DNS服务器所记录到，此时我们就可以通过DNS历史记录找到目标真实IP。而“以量打量”就是常说的ddos攻击或者说是流量耗尽攻击，在网上开CDN的时候，都会分地区流量，就比如这个节点有100M流量，当这流量用完后，用户再访问就会访问网站真实的ip地址。
  - 情报社区：
    - [https://x.threatbook.cnDNSdb](https://x.threatbook.cnDNSdb)
    - [https://dnsdb.io/zh-cn/](https://dnsdb.io/zh-cn/)

### 0x00.3 涉及资源

- SHODAN搜索引擎：[https://www.shodan.io](https://www.shodan.io)
- 微步在线情报社区：[https://x.threatbook.cn](https://x.threatbook.cn)
- 站长工具-ping：[http://ping.chinaz.com](http://ping.chinaz.com)
- Check ip for any site on the web：[https://www.get-site-ip.com/](https://www.get-site-ip.com/)
- App Synthetic Monitor：[https://asm.ca.com/en/ping.php](https://asm.ca.com/en/ping.php)
- CDN真实IP扫描：[https://github.com/Tai7sy/fuckcdn](https://github.com/Tai7sy/fuckcdn)
- 通过扫描全网绕过CDN获取网站IP地址：[https://github.com/boy-hack/w8fuckcdn](https://github.com/boy-hack/w8fuckcdn)
- 实战秒杀BC下的CDN节点：[https://mp.weixin.qq.com/s?__biz=MzA5MzQ3MDE1NQ==&mid=2653939118&idx=1&sn=945b81344d9c89431a8c413ff633fc3a&chksm=8b86290abcf1a01cdc00711339884602b5bb474111d3aff2d465182702715087e22c852c158f&token=268417143&lang=zh_CN#rd](https://mp.weixin.qq.com/s?__biz=MzA5MzQ3MDE1NQ==&mid=2653939118&idx=1&sn=945b81344d9c89431a8c413ff633fc3a&chksm=8b86290abcf1a01cdc00711339884602b5bb474111d3aff2d465182702715087e22c852c158f&token=268417143&lang=zh_CN#rd)

## 0x01 架构、搭建及WAF

### 0x01.0 站点搭建分析

- **目录类站点**
  - 原则是一个网站，但区别在于目录下的差异，两个网站两者可能仅仅是目录路径不同，如phpstudy搭建
  - 两个网站是两套程序，一个网站源码出现漏洞，另一个网站也会遭殃，就等于给与了两套漏洞方案
  - 可通过目录扫描工具可以查询到
- **端口类站点**
  - 在同一台服务器上，以端口来把网站进行分开，一个网站出现安全问题，也会导致另一个网站出现安全问题，如vulhub靶场
    对于这类站点，进行端口扫描，或者在网上搜索信息。
- **子域名站点**
  - 两个网站有可能不在同一个服务器上
  - 同一网段：内网横向渗透
  - 不同网段：收集分站敏感信息，如密码，套用到主站尝试
- **类似域名站点**
  - 出自于同一个公司的网站可能有多个域名，可写脚本尝试，多是基于域名后缀的更改
- **旁注，C段站点**
  - 旁注
    - 旁注是一种入侵方法，在字面上解释就是－”从旁注入”，利用同一主机上面不同网站的漏洞得到webshell，从而利用主机上的程序或者是服务所暴露的用户所在的物理路径进行入侵。同服务器不同站点。两个网站或者多个网站放在同一个服务器上，其中一个网站是目标。
      前提条件：有多个站点服务器
      192.168.1.1
      [www.a.com（目标）](http://www.a.xn--com()-qm0tm52k/)
      [www.b.com](http://www.b.com/)
      ……..
  - C段
    - 同网段不同服务器不同站点。网站有一个或多个站点，通过服务器IP地址的网段来进行测试。
      192.168.1.1
      [www.a.com（目标）](http://www.a.xn--com()-qm0tm52k/)
      [www.b.com](http://www.b.com/)
      ……..
      192.168.1.101
      [www.a.com](http://www.a.com/)
      [www.b.com](http://www.b.com/)
      ……..
      通过查询网段1-254，去获取101网段服务器权限，在通过服务器同一个网段目标主机来实施内网安全的测试方法，来获取指定网服务器的权限。
- 搭建软件特征站点
  - 一体化搭建软件：宝塔、PHPstudy、WMAP、INMAP（Nginx）
  - 常规的搭建软件都有常规的数据库的默认账号密码，如果搭建者不去更改的话，就能成为突破思路。
  - 例：
    - Apache/2.4.41(win32)OpenSSL/1.1.1c mod_fcgid/2.3.9a 宝塔 (信息很全基本上是搭建软件)
    - Apache/2.4.41(win32)OpenSSL/1.0.2j PHP/5.4.45 默认的安全设置(/phpmyadmin root/root)

### 0x01.1 WAF防护分析

- 什么是WAF应用
  - Web应用防护系统（也称为：网站应用级入侵防御系统。英文：Web Application Firewall，简称： WAF）。利用国际上公认的一种说法：Web应用防火墙是通过执行一系列针对HTTP/HTTPS的安全策略来专门为Web应用提供保护的一款产品。
  - 分为：硬件形式和软件形式。
- 如何快速识别WAF
  - wafw00f Github地址：[https://github.com/EnableSecurity/wafw00f](https://github.com/EnableSecurity/wafw00f)
  - 官方使用文档：[https://github.com/enablesecurity/wafw00f/wiki](https://github.com/enablesecurity/wafw00f/wiki)
  - 发现waf后尽量别用扫描工具，ip可能会被拉入黑名单

### 0x01.2 涉及资源

- SHODAN搜索引擎：[https://www.shodan.io/](https://www.shodan.io/)
- WebScan：[https://www.webscan.cc/](https://www.webscan.cc/)
- WAF识别工具：[https://github.com/EnableSecurity/wafw00f](https://github.com/EnableSecurity/wafw00f)

## 0x02 APP及其他资产等

- 在安全测试中，若WEB无法取得进展或无WEB的情况下，我们需要借助APP或其他资产在进行信息收集，从而开展后续渗透。

![APP及其他资产](/images/posts/2021-11-01-Penetration-testing-Information-collection.assets/web2_2.png)

### 0x02.0 APP

- APP提取一键反编译提取
  - apkAnalyser
- APP抓数据包进行工具配合
  - burpsuite
- 各种第三方应用相关探针技术
- 各种服务接口信息相关探针技术
- APP提取及抓包及后续配合
  - 某APK一键提取反编译
  - 利用burp历史抓更多URL

### 0x02.1 其他

- 某IP无web框架下的第三方测试

- 各种端口乱扫
  - nmap
    - nmap -sV -O 目标IP
  - 搜索引擎
    - shodan
    - 钟馗之眼（ZoomEye）
    - Fofa
- 各种接口乱扫
- 接口部分进行测试
- 旁站查询
- C段查询
- 类似域名站点-备案查询：找网址，再ping出ip地址，再用黑暗引擎搜索内容
- 网站标题：用百度、谷歌搜索相关键网站
- 检查-Network， 抓包看内容，搜索域名关键字

### 0x02.2 实战渗透小结

- 查看网站信息
  - 判断有无常见漏洞（是否静态）
- 使用黑暗引擎进行扫描（shodan、Fofa、钟馗之眼）
  - 站点打不开，可能是被云服务商防止攻击给屏蔽掉了，可以打开代理尝试一下
  - 收集IP、端口、开启的服务
  - 搜索服务常见的漏洞，渗透测试
- 更多信息收集参见文章开头思维导图

### 0x02.3 涉及资源

- FOFA搜索引擎：[https://fofa.so/](https://fofa.so/)
- 站长工具：[http://tool.chinaz.com](http://tool.chinaz.com)
- SHODAN搜索引擎：[https://www.shodan.io/](https://www.shodan.io/)
- 钟馗之眼：[https://www.zoomeye.org/](https://www.zoomeye.org/)
- Nmap：[https://nmap.org/download.html](https://nmap.org/download.html)

## 0x03 资产监控拓展

### 0x03.0 POC、EXP、Payload与Shellcode

- 概念
  - POC：全称 ‘ Proof of Concept ‘，中文 ‘ 概念验证 ‘ ，常指一段漏洞证明的代码。
  - EXP：全称 ‘ Exploit ‘，中文 ‘ 利用 ‘，指利用系统漏洞进行攻击的动作。
  - Payload：中文 ‘ 有效载荷 ‘，指成功exploit之后，真正在目标系统执行的代码或指令。
  - Shellcode：简单翻译 ‘ shell代码 ‘，是Payload的一种，由于其建立正向/反向shell而得名。
- 几点注意
  - POC是用来证明漏洞存在的，EXP是用来利用漏洞的，两者通常不是一类，或者说，PoC通常是无害的，Exp通常是有害的，有了POC，才有EXP。
  - Payload有很多种，它可以是Shellcode，也可以直接是一段系统命令。同一个Payload可以用于多个漏洞，但每个漏洞都有其自己的EXP，也就是说不存在通用的EXP。
  - Shellcode也有很多种，包括正向的，反向的，甚至meterpreter。
  - Shellcode与Shellshock不是一个，Shellshock特指14年发现的Shellshock漏洞。
- Payload模块
  - 在Metasploit Framework 6大模块中有一个Payload模块，在该模块下有Single、Stager、Stages这三种类型，Single是一个all-in-one的Payload，不依赖其他的文件，所以它的体积会比较大，Stager主要用于当目标计算机的内存有限时，可以先传输一个较小的Stager用于建立连接，Stages指利用Stager建立的连接下载后续的Payload。Stager和Stages都有多种类型，适用于不同场景。
- 总结
  - 想象自己是一个特工，你的目标是监控一个重要的人，有一天你怀疑目标家里的窗子可能没有关，于是你上前推了推，结果推开了，这是一个POC。之后你回去了，开始准备第二天的渗透计划，第二天你通过同样的漏洞渗透进了它家，仔细查看了所有的重要文件，离开时还安装了一个隐蔽的窃听器，这一天你所做的就是一个EXP，你在他家所做的就是不同的Payload，就把窃听器当作Shellcode吧！

### 0x03.1 Github 监控

- 便于收集整理最新 exp 或 poc 便于发现相关测试目标的资产
  - GitHub监控推送微信：[https://github.com/weixiao9188/wechat_push](https://github.com/weixiao9188/wechat_push)

### 0x03.2 各种子域名查询

![子域名查询](/images/posts/2021-11-01-Penetration-testing-Information-collection.assets/web2_3.png)

- 工具：
  - 子域名查询工具1：[https://github.com/euphrat1ca/LayerDomainFinder](https://github.com/euphrat1ca/LayerDomainFinder)
  - 子域名查询工具2：[https://github.com/bit4woo/teemo](https://github.com/bit4woo/teemo)

### 0x03.3 监控github上最新的CVE、RCE发布及其他

```python
# Title: wechat push CVE-2020
# Date: 2020-5-9
# Exploit Author: weixiao9188
# Version: 4.0
# Tested on: Linux,windows
# coding:UTF-8

import requests
import json
import time
import os
import pandas as pd

time_sleep = 20 #每隔20秒爬取一次

while(True):
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.25 Safari/537.36 Core/1.70.3741.400 QQBrowser/10.5.3863.400"}
    #判断文件是否存在
    datas = []
    response1=None
    response2=None
    if os.path.exists("olddata.csv"):
        #如果文件存在则每次爬取10个
        df = pd.read_csv("olddata.csv", header=None)
        datas = df.where(df.notnull(),None).values.tolist()#将提取出来的数据中的nan转化为None
        response1 = requests.get(url="https://api.github.com/search/repositories?q=CVE-2020&sort=updated&per_page=10",
                                 headers=headers)
        response2 = requests.get(url="https://api.github.com/search/repositories?q=RCE&ssort=updated&per_page=10",
                                 headers=headers)

    else:
        #不存在爬取全部
        datas = []
        response1 = requests.get(url="https://api.github.com/search/repositories?q=CVE-2020&sort=updated&order=desc",headers=headers)
        response2 = requests.get(url="https://api.github.com/search/repositories?q=RCE&ssort=updated&order=desc",headers=headers)

    data1 = json.loads(response1.text)
    data2 = json.loads(response2.text)
    for j in [data1["items"],data2["items"]]:
        for i in j:
            s = {"name":i['name'],"html":i['html_url'],"description":i['description']}
            s1 =[i['name'],i['html_url'],i['description']]
            if s1 not in datas:
                #print(s1)
                #print(datas)
                params = {
                     "text":s["name"],
                    "desp":" 链接:"+str(s["html"])+"\n简介"+str(s["description"])
                }
                print("当前推送为"+str(s)+"\n")
                print(params)
                requests.get("https://sc.ftqq.com/XXXX.send",params=params,timeout=10)
                #time.sleep(1)#以防推送太猛
                print("推送完成!")
                datas.append(s1)
            else:
                pass
                #print("数据已处在!")
    pd.DataFrame(datas).to_csv("olddata.csv",header=None,index=None)
    time.sleep(time_sleep)
```

### 0x03.4 涉及资源

- 证书查询：[https://crt.sh](https://crt.sh)
- DNS查询：[https://dnsdb.io/zh-cn/](https://dnsdb.io/zh-cn/)
- server酱（推送）：[https://sct.ftqq.com/3.version](https://sct.ftqq.com/3.version) 
- CDN查询：[https://tools.ipip.net/cdn.php](https://tools.ipip.net/cdn.php) 
- IP历史查询：[https://securitytrails.com/domain/baidu.com/history/a](https://securitytrails.com/domain/baidu.com/history/a)
- IP定位查询：[https://www.opengps.cn/](https://www.opengps.cn/)
