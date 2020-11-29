---
layout: post
title: 	KeePass+坚果云+KeePass2Android 实现跨平台进行密码管理与同步
categories: KeePass
description: 昨天偶然看到一篇关于密码管理的文章，于是乎找到了使用KeePass + 坚果云 +  KeePass2Android 实现跨平台同步内容（主要为密码）的方法。
keywords: KeePass,密码管理
---

> 今年国家开始实施《中华人民共和国密码法》，想试着开始使用密码管理软件来管理各个应用、网站的账号和密码信息。需求很简单：密码管理，Windows 与 Android 间的密码同步。看了一圈，人气最高的的无非是三款：1Password、LastPass 以及本文的主角 KeePass。而 1Password 和 LastPass 的订阅式付费，就不想选择了。最终我决定采用 KeePass+WebDAV 的方式满足我的需求，晚上花了一点时间研究了一下，结果并没有让我失望。
>

### 0x00  准备工作

- **[注册坚果云账号](https://www.jianguoyun.com/d/signup)**

- 便捷下载地址：**[下载KeePass和KeePass2Android-百度云(hbbf)](https://pan.baidu.com/s/1wFdWZFRsCszhZhrVNkRyXw)**

- 官方下载地址

  - **[下载keePass](https://keepass.info/download.html)**
- **[下载keePass2Android](https://github.com/PhilippC/keepass2android/releases/tag/1.08c-r0)**

### 0x01 开启坚果云第三方应用服务

#### 0x01.0 登录坚果云账号之后，打开右上角的账户信息：

![image-20201129001002770](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129001002770.png)

#### 0x01.1 在【安全选项】中添加第三方应用（此处密码是自动生成）

![image-20201129001217600](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129001217600.png)

- 这里添加两个应用，分别给Windows客户端和安卓APP使用

![image-20201129113519682](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129113519682.png)



### 0x02 配置keePass（Windows版）

#### 0x02.0 安装完成keePass后如图

![image-20201129001358043](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129001358043.png)

#### 0x02.1 在本地新建一个Database.kdbx并同步到云端

![image-20201129001632238](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129001632238.png)

- 选择本地文件存放位置

  ![image-20201129002041218](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129002041218.png)

- 设置访问数据库密码：

  ![image-20201129002126287](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129002126287.png)

- 设置数据库详细信息：

  ![image-20201129002154346](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129002154346.png)

- 完成：

  ![image-20201129002213225](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129002213225.png)

- 安装**[坚果云应用](https://www.jianguoyun.com/s/downloads)**，将数据库移动到坚果云的同步目录下。



![image-20201129002831508](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129002831508.png)

- 安装完成之后自动同步数据库



![image-20201129002843096](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129002843096.png)

- 完成同步



![image-20201129002909298](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129002909298.png)



#### 0x02.2 使用URL打开数据库

![image-20201129003013646](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129003013646.png)



- 下图中一些内容的解释：
  - URL：坚果云个人页面安全选项中的WebDAV服务器地址 + 文件夹名 + 文件名，如：https://dav.jianguoyun.com/dav/keepass/Database.kdbx
  - User name：坚果云账号（注册邮箱）
  - Password：第三方应用的密码（自动生成的）

![image-20201129003121645](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129003121645.png)



- 然后填写数据库的密码就ok了 **（记得在更新了数据库内容之后要及时Ctrl + S）**

![image-20201129111911104](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129111911104.png)

### 0x03 配置KeePass2Android（安卓版）

#### 0x03.0 通过URL打开数据库即可

- 打开KeePass2Android  APP应用，选择【打开文件】

![image-20201129112132708](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129112132708.png)

- 选择存储类型为【HTTPS(WebDav)】

![image-20201129113811658](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129113811658.png)



- 下图中一些内容的解释：
  - 文件URL：坚果云个人页面安全选项中的WebDAV服务器地址 + 文件夹名 + 文件名，如：https://dav.jianguoyun.com/dav/keepass/Database.kdbx
  - 用户名：坚果云账号（注册的邮箱）
  - 密码：第三方应用的密码（自动生成的）

> PS：此处的第三方应用的密码与之前配置keePass时使用的密码是不同应用的密码）

![image-20201129112212769](/images/posts/2020-11-29-KeePass-jianguoyun-Keepass2Android-realizes-passwd-mgt-sync-across-platforms.assets/image-20201129112212769.png)

#### 0x03.1 登录成功之后即可看到数据库信息

- 此应用无法截屏，此处就不放图片了

### 0x04  补充

#### 0x04.0 数据库信息同步原理

- Windows客户端修改了数据，坚果云会自动同步到云端，在手机上需要手动点右上角三个点的符号，进去点击【同步数据库】
- 手机端修改了数据，会自动同步到坚果云端，坚果云也会同步到本地数据库，但是keepass应用没有刷新数据，需要重启应用进行才能看到修改的数据信息。

#### 0x04.1 密码生成

- 为了方便创建密码，KeePass和KeePass2Android都提供了自动创建密码的功能
- 但是为了给大家提供更广泛的使用需求，推荐一个在线随机密码生成网站
  - **[随机密码在线生成](https://suijimimashengcheng.51240.com/)**





