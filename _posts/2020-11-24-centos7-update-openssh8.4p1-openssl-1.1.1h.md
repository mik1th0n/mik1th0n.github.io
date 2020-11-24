---
layout: post
title: CentOS 7升级OpenSSH 8.4p1和OpenSSL 1.1.1h
categories: OpenSSH
description: 由于升级后的OpenSSH和OpenSSL版本，过了较长时间，仍会有漏洞，因此进行升级修复漏洞。
keywords: CentOS7,OpenSSH,OpenSSL
---

> 注意：此升级方案适用于CentOS 7的系统，且OpenSSH版本等于 8.1p1，才能升级到OpenSSH 8.4p1。

相关链接：[CentOS 7升级OpenSSH 8.1p1和OpenSSL 1.0.2s](https://mik1th0n.github.io/2020/03/30/centos7-update-openssh8.1p1-openssl-1.0.2s/)

## 0x0 CentOS 7升级OpenSSH 8.4p1和OpenSSL 1.1.1h

### 0x00 升级准备

#### 0x00.0  安装包下载

- [安装包下载地址(p2zp)](https://pan.baidu.com/s/1RvzaJ4VcWJJSthrkIQBNKw)
- 将安装包下载到桌面

#### 0x00.1  启动telnet

```shell
[root@001 ~]# systemctl enable xinetd
Created symlink from /etc/systemd/system/multi-user.target.wants/xinetd.service to /usr/lib/systemd/system/xinetd.service.
[root@001 ~]# systemctl enable telnet.socket
Created symlink from /etc/systemd/system/sockets.target.wants/telnet.socket to /usr/lib/systemd/system/telnet.socket.
[root@001 ~]# systemctl start telnet.socket
[root@001 ~]# systemctl start xinetd
[root@001 ~]# netstat -lntp|grep 23
```

#### 0x00.2  测试telnet

- 打开Windows命令行，telnet测试。

  ```shell
  C:\Users\1>telnet  ip
  ```
  

#### 0x00.1 上传升级包

- 使用**SecureCRT**登录系统，在菜单栏**[File]->[Connect SFTP Session]**链接FTP服务器。
- 上传openssh-8.4p1.tar.gz、openssl-1.1.1h.tar.gz、zlib-1.2.11.tar.gz包

```shell
sftp:/root> put C:\Users\1\Desktop\openssh-8.4p1.tar.gz
sftp:/root> put C:\Users\1\Desktop\openssl-1.1.1h.tar.gz
sftp:/root> put C:\Users\1\Desktop\zlib-1.2.11.tar.gz

[root@001 ~]# ls
openssh-8.4p1.tar.gz  openssl-1.1.1h.tar.gz  zlib-1.2.11.tar.gz
```

#### 0x00.2  解压升级包

```shell
[root@001 ~]# tar xf openssh-8.4p1.tar.gz -C /usr/local/src/
[root@001 ~]# tar xf openssl-1.1.1h.tar.gz -C /usr/local/src/
[root@001 ~]# tar xf zlib-1.2.11.tar.gz -C /usr/local/src/
[root@001 ~]# ll /usr/local/src/
total 24
drwxr-xr-x.5 root root 68 Nov 23 16:42.
drwxr-xr-x.20 root root 267 Jul 20 16:04..
drwxr-xr-x 7 ctcss ctcss 12288 Sep 27 15:42 openssh-8.4p1
drwxrwxr-x 18 root root 4096 Sep 22 20:55 openssl-1.1.1h 
drwxr-xr-x 14 501 qames 4096 Jan 16 2017 zlib-1.2.11
```

- 解压之后出现如下几个文件夹
  - openssh-8.4p1
  - openssl-1.1.1h
  - zlib-1.2.11

### 0x01  安装

#### 0x01.0  安装zlib-1.2.11.tar.gz

```shell
[root@001 ~]# cd /usr/local/src/zlib-1.2.11/
[root@001 zlib-1.2.11]# ./configure --prefix=/usr/local/zlib && make -j 4 && make install
.
.
.
chmod 644/usr/Local/zlib/include/zlib.h /usr/local/zlib/include/zconf.h
[root@001 zlib-1.2.11]# echo $?
0
```

#### 0x01.1  安装 openssl-1.1.1h.tar.gz

```shell
[root@001 zlib-1.2.11]# cd /usr/local/src/openssl-1.1.1h/
[root@001 openssl-1.1.1h]# ./config --prefix=/usr/local/ssl -d shared
.
.
.

**********************************************************************
***    OpenSSL has been successfully configured                    ***
***                                                                ***
***    If you encounter a problem while building,please open an    ***
***    issue on Gitlub <https://github.com/openssl/openssl/issues> ***
***    and include the output from the following command:          ***
***                                                                ***
***    (If you are new to OpenSSL,you might want to consult the    ***
***    Troubleshootingsection in the INSTALL file first)           ***
***                                                                ***
**********************************************************************
[root@001 openssl-1.1.1h]# make -j 4 && make install
.
.
.
/usr/Local/ssl/share/doc/openssl/html/man7/ssl.html
/usr/local/ssl/share/doc/openssl/html/man7/x509.html
[root@001 openssl-1.1.1h]# echo $?
0
[root@001 openssl-1.1.1h]# echo '/usr/local/ssl/lib' >> /etc/ld.so.conf
[root@001 openssl-1.1.1h]# ldconfig -v
```

#### 0x01.2  安装openssh-8.4p1.tar.gz

```shell
[root@001 openssl-1.1.1h]# mv /etc/ssh /etc/ssh.bak
[root@001 openssl-1.1.1h]# cd /usr/local/src/openssh-8.4p1/
[root@001 openssh-8.4p1]# ./configure --prefix=/usr/local/openssh --sysconfdir=/etc/ssh --with-ssl-dir=/usr/local/ssl --with-zlib=/usr/local/zlib
[root@001 openssh-8.4p1]# make -j 4 && make install
.
.
.
/usr/bin/mkdir -p /etc/ssh
ssh-keygen: generating new host keys:RSA DSA ECDSA ED25519
/usr/Local/openssh/sbin/sshd-t-f /etc/ssh/sshd_config
[root@001 openssh-8.4p1]# echo $?
0

[root@001 openssh-8.4p1]# echo "X11Forwarding yes" >> /etc/ssh/sshd_config
[root@001 openssh-8.4p1]# echo "X11UseLocalhost no" >> /etc/ssh/sshd_config
[root@001 openssh-8.4p1]# echo "XAuthLocation /usr/bin/xauth" >> /etc/ssh/sshd_config
[root@001 openssh-8.4p1]# echo "UseDNS no" >> /etc/ssh/sshd_config
[root@001 openssh-8.4p1]# echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config
[root@001 openssh-8.4p1]# echo 'PubkeyAuthentication yes' >> /etc/ssh/sshd_config
[root@001 openssh-8.4p1]# echo 'PasswordAuthentication yes' >> /etc/ssh/sshd_config

[root@001 openssh-8.4p1]# mv /usr/sbin/sshd /usr/sbin/sshd.bak
[root@001 openssh-8.4p1]# cp -rf /usr/local/openssh/sbin/sshd /usr/sbin/sshd
[root@001 openssh-8.4p1]# mv /usr/bin/ssh /usr/bin/ssh.bak
[root@001 openssh-8.4p1]# cp -rf /usr/local/openssh/bin/ssh /usr/bin/ssh
[root@001 openssh-8.4p1]# mv /usr/bin/ssh-keygen /usr/bin/ssh-keygen.bak
[root@001 openssh-8.4p1]# cp -rf /usr/local/openssh/bin/ssh-keygen /usr/bin/ssh-keygen

[root@001 openssh-8.4p1]# ssh -V
0penSSH 8.4p1,0penSSL 1.1.1h 22 Sep 2020
```

#### 0x01.3  启动sshd

```shell
[root@001 openssh-8.4p1]# systemctl start sshd
[root@001 openssh-8.4p1]# systemctl stop sshd.service
[root@001 openssh-8.4p1]# rm -rf /lib/systemd/system/sshd.service
[root@001 openssh-8.4p1]# systemctl daemon-reload
[root@001 openssh-8.4p1]# cp /usr/local/src/openssh-8.4p1/contrib/redhat/sshd.init /etc/init.d/sshd
y
[root@001 openssh-8.4p1]# /etc/init.d/sshd restart
[ root@ 001 openssh-8.4p1]#/etc/init.d/sshd restart 
Restarting sshd (via systemct1): Warning: sshd.service changed on disk. Run "systemctl daemon-reload' to reload units.
                                                      [  0K  ]
[ root@ 001 openssh-8.4p1]# systemctl daemon-reload
[root@001 openssh-8.4p1]# systemctl status sshd
sshd. service - SYSV: OpenSSH server daemon 
  Loaded: loaded(/etc/rc.d/init.d/sshd; bad; vendor preset: enabled)
  Active: active (running) since Mon 2020-11-23 17:13:31 CST;39s ago 
    Docs: man: systemd-sysv-generator(8)
Main PID:23169(sshd)
  CGroup:/system. slice/sshd. service
          |- 5478 sshd: root@ pts/05489-bash
          |- 23169 sshd:/usr/sbin/sshd [ listener] 0 of 10-100 startups
          |- 23940 systemctl status sshd
          
[root@001 openssh-8.4p1]# /etc/init.d/sshd restart
[root@001 openssh-8.4p1]# systemctl status sshd

[root@001 openssh-8.4p1]# chkconfig --add  sshd
[root@001 openssh-8.4p1]# chkconfig --list sshd
```

#### 0x01.4  关闭telnet

```shell
[root@001 ~]# systemctl disable xinetd.service
Removed symlink /etc/systemd/system/multi-user.target.wants/xinetd.service.
[root@001 ~]# systemctl stop xinetd.service
[root@001 ~]# systemctl disable telnet.socket
Removed symlink /etc/systemd/system/sockets.target.wants/telnet.socket.
[root@001 ~]# systemctl stop telnet.socket
```























