---
layout: post
title: VMware产品常见问题大全
categories: VMware
description: 
keywords: VMware,VMware Workstation
---



**目录**

* TOC
{:toc}
## 0x00 VMware产品常见问题大全

### 0x00.0 VMware Wrokstation

#### 0x00.0 VMware该虚拟机似乎正在使用中。如果该虚拟机未在使用，请按“获取所有权(T)”按钮获取它的所有权

- 今天虚拟机ESXI没正常关机，物理机突然断电了，打开VMware却开不了ESXI了
- 该虚拟机似乎正在使用中。如果该虚拟机未在使用，请按“获取所有权(T)”按钮获取它的所有权。否则，请按“取消(C)”按钮以防损坏。配置文件xxxxx

![/images/posts/image-20201221132304948](2020-12-21-VMware-FAQs.assets/image-20201221132304948.png)

- 解决办法：到你的安装目录找文件夹后缀为.vmx.lck的文件夹，将其改名或者删除掉。如下所示：

![/images/posts/image-20201221132329831](2020-12-21-VMware-FAQs.assets/image-20201221132329831.png)

![/images/posts/image-20201221132812308](2020-12-21-VMware-FAQs.assets/image-20201221132812308.png)

- 再次打开就好了

<img src="/images/posts/2020-12-21-VMware-FAQs.assets/image-20201221133040637.png" alt="image-20201221133040637" style="zoom:67%;" />