---
layout: post
title: 【原创】基于CentOS 7.6搭建图形化Syslog服务器（Rsyslog+MySQL+LogAnalyzer）
categories: [Syslog]
description: some word here
keywords: CentOS 7, Syslog, Rsyslog, MySQL, LogAnalyzer
flow: true
sequence: true
mermaid: true
mathjax: true
---

## 0x00 安装CentOS 7.6系统

- **CentOS 7.6镜像下载：**[https://vault.centos.org/7.6.1810/isos/x86_64/](https://vault.centos.org/7.6.1810/isos/x86_64/)

- **Centos7.6操作系统安装及优化全纪录**：[https://blog.51cto.com/loong576/2398136](https://blog.51cto.com/loong576/2398136)



## 0x01 环境准备

- 确认系统与外网访问，确认与 Client 互通

- **关闭 Firewall**

```shell
#关闭防火墙
[root@syslog ~]# firewall-cmd --state
running
[root@syslog ~]# systemctl stop firewalld.service
#禁止启动防火墙
[root@syslog ~]# systemctl disable firewalld.service
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
```

- **关闭 SELinux**

```shell
/etc/selinux/config
把 SELINUX=enforcing 改成 SELINUX=disabled
[root@syslog ~]# getenforce
Enforcing
[root@syslog ~]# setenforce 0
[root@syslog ~]# sed -i 's/^ *SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
[root@syslog ~]# getenforce
Permissive
[root@syslog ~]# cat /etc/selinux/config

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

# 重启系统
[root@syslog ~]# reboot
```

## 0x02 配置 LAMP 环境

### 0x02.0 安装 MySQL

- 创建下载文件存放目录

```shell
[root@syslog ~]# mkdir /home/rsyslog_server/tools -p
```

- 安装 MySQL 官方 yum 仓库

```shell
[root@syslog ~]# cd /home/rsyslog_server/tools
[root@syslog tools]# yum install wget -y
[root@syslog tools]# wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm

......

100%[===================================================================================>] 9,116       --.-K/s   in 0s

2021-11-25 02:25:34 (248 MB/s) - ‘mysql57-community-release-el7-8.noarch.rpm’ saved [9116/9116]

[root@syslog tools]# ls
mysql57-community-release-el7-8.noarch.rpm
[root@syslog tools]#
[root@syslog tools]# rpm -Uvh mysql57-community-release-el7-8.noarch.rpm
warning: mysql57-community-release-el7-8.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql57-community-release-el7-8  ################################# [100%]
```

- 安装并启动 MySQL

```shell
[root@syslog tools]# yum install mysql-community-server -y
[root@syslog tools]# systemctl start mysqld.service
[root@syslog tools]# systemctl status mysqld.service
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2021-11-25 02:28:07 CST; 1s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 22604 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 22441 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 22607 (mysqld)
    Tasks: 27
   CGroup: /system.slice/mysqld.service
           └─22607 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

Nov 25 02:27:59 syslog systemd[1]: Starting MySQL Server...
Nov 25 02:28:07 syslog systemd[1]: Started MySQL Server.
```

- 查看初始密码

```shell
[root@syslog tools]# grep 'temporary password' /var/log/mysqld.log
2021-11-24T18:28:06.137917Z 1 [Note] A temporary password is generated for root@localhost: ,:<<xboSx1V*
[root@syslog tools]#
```

- 初始密码为：`,:<<xboSx1V*`

- 使用初始密码登录

```shell
[root@syslog tools]# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.36

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
mysql>
```

- 更改密码，密码需要符合以下规则：至少一个大写字母，一个小写字母，一个数字，一个特殊字符，而且密码长度需要超过 8 位，完成后登陆确认

```shell
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'Syslog@2021';
Query OK, 0 rows affected (0.00 sec)

mysql>
```

### 0x02.1 安装 Apache 及 PHP

```shell
[root@syslog tools]# yum install httpd -y
[root@syslog tools]# yum install php php-gd php-xml php-mysql -y
```

### 0x02.2 启动服务并加入开机自启动

```shell
[root@syslog tools]# systemctl start httpd.service
[root@syslog tools]# systemctl enable httpd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
[root@syslog tools]# systemctl start mysqld.service
[root@syslog tools]# systemctl enable mysqld.service
```

### 0x02.3 测试 PHP 环境

```shell
[root@syslog tools]# cd /var/www/html/
[root@syslog html]# vim index.php
输入内容：<?php phpinfo() ?>
```

- 在浏览器中输入 http://ip，若显示以下内容，则配置成功。

![image-20211125023340988](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125023340988.png)

## 0x03 配置 MySQL 环境

### 0x03.0 检查 rsyslog 安装

```shell
[root@syslog html]# rpm -qa rsyslog
rsyslog-8.24.0-34.el7.x86_64
```

### 0x03.1 与 MySQL 对接

- rsyslog 使用此模块将数据传入 MySQL 数据库，必须安装

```
[root@syslog html]# yum install rsyslog-mysql -y
```

## 0x04  配置 Rsyslog

- 导入 rsyslog-mysql

```shell
[root@syslog html]# cd /usr/share/doc/rsyslog-8.24.0/
[root@syslog rsyslog-8.24.0]# ll
total 640
-rw-r--r-- 1 root root    501 Apr 15  2016 AUTHORS
-rw-r--r-- 1 root root 588023 Jan 10  2017 ChangeLog
-rw-r--r-- 1 root root  35146 Apr 15  2016 COPYING
-rw-r--r-- 1 root root   9137 Apr 15  2016 COPYING.ASL20
-rw-r--r-- 1 root root   7639 Apr 15  2016 COPYING.LESSER
-rw-r--r-- 1 root root   1046 Apr 15  2016 mysql-createDB.sql

[root@syslog rsyslog-8.24.0]# mysql -uroot -p < mysql-createDB.sql
Enter password:
[root@syslog rsyslog-8.24.0]#
```

- 登录数据库检查

```shell
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| Syslog             |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use Syslog;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------------+
| Tables_in_Syslog       |
+------------------------+
| SystemEvents           |
| SystemEventsProperties |
+------------------------+
2 rows in set (0.00 sec)
```

- 创建 rsyslog 新用户

```shell
mysql> grant all on Syslog.* to 'rsyslog'@'localhost' identified by 'Syslog@2021';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
```

- 配置 rsyslog 模块

```shell
[root@syslog rsyslog-8.24.0]# vi /etc/rsyslog.conf

#按如下进行更改

#### MODULES ####
# The imjournal module bellow is now used as a message source instead of imuxsock.
$Modload ommysql
*.* :ommysql:localhost,Syslog,rsyslog,Nsfocus@123
$ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
$ModLoad imklog # provides kernel logging support (previously done by rklogd)
$ModLoad immark # provides --MARK-- message capability

# Provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514

# Provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 514

# Use default timestamp format
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
$template DynamicFile,"/var/log/ttlogs/%$YEAR%/%$MONTH%/%$DAY%/%fromhost-ip%-test.log"
*.* ?DynamicFile
```

![image-20211125033014433](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125033014433.png)

- 重启服务

```
[root@syslog rsyslog-8.24.0]# systemctl restart rsyslog.service
```

## 0x05 安装 LogAnalyzer

```shell
[root@syslog rsyslog-8.24.0]# cd /home/rsyslog_server/tools/
[root@syslog tools]# wget http://download.adiscon.com/loganalyzer/loganalyzer-3.6.5.tar.gz --no-check-certificate
[root@syslog tools]# tar zxf loganalyzer-3.6.5.tar.gz
[root@syslog tools]# cd loganalyzer-3.6.5
[root@syslog loganalyzer-3.6.5]# cp -a src/* /var/www/html/
cp: overwrite ‘/var/www/html/index.php’? yes
[root@syslog loganalyzer-3.6.5]#
```

## 0x06 配置 LogAnalyzer

### 0x06.0 初始化配置

- 浏览器访问 http://ip

![image-20211125025735662](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125025735662.png)

- 测试异常，提示没有 config.php 文件，使用 contrib 中的 configure.sh 脚本可生成

![image-20211125025744860](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125025744860.png)

```shell
[root@syslog loganalyzer-3.6.5]# cd contrib/
[root@syslog contrib]# cp configure.sh /var/www/html/
[root@syslog contrib]# cd /var/www/html/
[root@syslog html]# sh configure.sh
```

- 此部分操作在 `/var/www/html/` 目录下创建 `config.php` 文件并配置权限为 666，也可以使用 mkdir 及 chmod 命令执行。

![image-20211125025916413](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125025916413.png)

- 重新访问web界面并进行配置

![image-20211125030100730](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125030100730.png)

- 填写数据库信息

![image-20211125030129174](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125030129174.png)

![image-20211125030359261](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125030359261.png)

- 生成表并覆盖原表

![image-20211125030429722](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125030429722.png)

![image-20211125030450694](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125030450694.png)

- 设置管理员

![image-20211125030537283](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125030537283.png)

- `syslogadmin`/`Syslog@2021`

- 设置源和数据库

![image-20211125030745871](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125030745871.png)

- 完成配置

![image-20211125030807454](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125030807454.png)

### 0x06.1 使用python脚本发送syslog数据

- 修改syslog的IP地址

```
import logging
import logging.handlers  # handlers要单独import

logger = logging.getLogger()
fh = logging.handlers.SysLogHandler(('192.168.1.4', 514), logging.handlers.SysLogHandler.LOG_AUTH)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
fh.setFormatter(formatter)
logger.addHandler(fh)
logger.warning("msg")
logger.error("msg")
```

![image-20211125033541460](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125033541460.png)

- 以下所有的log已经存入syslog服务器mysql 中，但是host全部为ZXHN。

![image-20211125033214495](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125033214495.png)

- 修改hosts文件,添加IP与名称对应表

![image-20211125033434838](/images/posts/2021-11-17-CentOS7-Graphic-Syslog-Rsyslog-MySQL-LogAnalyzer.assets/image-20211125033434838.png)

- 重启服务器即可。

```
[root@syslog ~]# reboot
```









