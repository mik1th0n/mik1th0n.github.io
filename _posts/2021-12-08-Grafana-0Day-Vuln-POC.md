---
layout: post
title: 【原创】Grafana V8.0+版本存在未授权任意文件读取 0Day漏洞 - POC
categories: [POC, Grafana, Vuln]
description: some word here
keywords: POC, Grafana, 渗透测试, Vuln, 0Day
flow: true
sequence: true
mermaid: true
mathjax: true
---



## Grafana V8.0+版本存在未授权任意文件读取 0Day漏洞 - POC

- POC：[https://github.com/mik1th0n/Grafana-0Day-Vuln-POC](https://github.com/mik1th0n/Grafana-0Day-Vuln-POC)

### 1 漏洞信息

#### 1.1 基本信息

- 漏洞厂商：Grafana
- 厂商官网：[https://grafana.com/](https://grafana.com/)

#### 1.2 漏洞描述

- Grafana是一个跨平台、开源的数据可视化网络应用程序平台。用户配置连接的数据源之后，Grafana可以在网络浏览器里显示数据图表和警告。目前Grafana 存在未授权任意文件读取漏洞，攻击者在未经身份验证的情况下可通过该漏洞读取主机上的任意文件。
- 漏洞评级：高危

#### 1.3 受影响版本

- 受影响版本：V8.0+

### 1.4 修复建议

- 建议及时通过网络/安全设备设置黑白名单策略，非必要不暴露在外网。
- 等待官方补丁进行升级

![image-20211208220325089](https://s4.ax1x.com/2021/12/08/oWuNpn.png)

### 2 FOFA查询

- 使用fofa可以查询到grafana的站点，查询语法如下

```
"grafana" && country="CN" && status_code="200"
```

![image-20211208220359711](https://s4.ax1x.com/2021/12/08/oWuGkQ.png)

### 3 POC使用方法

1. 将FOFA获取到的URL导入 `url_list_file.txt` 中，每行一条URL

2. 运行脚本

```
python3 Grafana-0Day-Vuln-POC.py
```

![image-20211208220442378](https://s4.ax1x.com/2021/12/08/oWuYfs.png)

![image-20211208220531940](https://s4.ax1x.com/2021/12/08/oWu3Tg.png)

### 4 参考链接

- https://twitter.com/naglinagli/status/1468155313182416899
- https://github.com/jas502n/Grafana-VulnTips
