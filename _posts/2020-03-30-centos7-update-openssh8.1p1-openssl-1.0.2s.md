---
layout: post
title: CentOS 7升级OpenSSH 8.1p1和OpenSSL 1.0.2s
categories: OpenSSH
description: 由于默认安装的CentOS系统的OpenSSH和OpenSSL版本较低，存在较多漏洞，容易被攻击，因此进行升级修复漏洞。
keywords: CentOS7,OpenSSH,OpenSSL
---

> 注意：此升级方案适用于CentOS 7的系统，且OpenSSH版本低于等于 7.4p1，才能升级到OpenSSH 8.1p1。

相关链接：[CentOS 7升级OpenSSH 8.4p1和OpenSSL 1.1.1h](https://mik1th0n.github.io/2020/11/24/centos7-update-openssh8.4p1-openssl-1.1.1h/)

## 0x0 CentOS 7升级OpenSSH 8.1p1和OpenSSL 1.0.2s

### 0x00 升级准备

#### 0x00.0  安装包下载

- [安装包下载地址(qg1r)](https://pan.baidu.com/s/1_1XjBkVL5ykcbuhKC2CoEg)
- 将安装包下载到桌面

#### 0x00.1  上传升级包

- 使用**SecureCRT**登录系统，在菜单栏**[File]->[Connect SFTP Session]**链接FTP服务器。

```shell
sftp:/root> put C:\Users\1\Desktop\openssh_openssl_update.tar
sftp:/root> put C:\Users\1\Desktop\perl-5.28.1.tar.gz

[root@001 ~]# ll
total 59364
-rw-r--r-- 1 root root 60788506 Mar 30 12:27 openssh_openssl_update.tar
```

#### 0x00.2  解压升级包

```shell
[root@001 ~]# tar -xvf openssh_openssl_update.tar
.
.
.
[root@001~]#ls
openssh_openssl_update openssh_openssl_update.tar
[root@001~]#cd openssh_openssl_update
[root@001~]#ls
1.telent 2.gcc-gcc+ 3.other 4.ssh_ssl
```

- 解压之后出现如下几个文件夹
  - 1.telnet
  - 2.gcc-gcc+
  - 3.other
  - 4.ssh_ssl

### 0x01  安装和配置telnet-server及xinetd

#### 0x01.0  进入1.telnet文件夹

```SHELL
[root@001 ~]# cd openssh_openssl_update/1.telent/
[root@001 ~]# ll
total 244
-rwxrwxrwx 1 root root 65632 Nov 28 10:46 telnet-0.17-64.el7.x86_64.rpm
-rwxrwxrwx 1 root root 41804 Nov 28 11:10 telnet-server-0.17-64.e17.x86_64.rpm
-rwxrwxrwx 1 root root 131304 Nov 28 10:46 xinetd-2.3.15-13.e17.x86_64.rpm
```

#### 0x01.1  运行安装命令

```shell
[root@001 1.telent]# rpm -Uvh xinetd-2.3.15-13.el7.x86_64.rpm
Preparing...                         ######################[100%]
Updating / installing...
   1：xinetd-2：2.3.15-13.el7         ######################[100%]
[root@001 1.telent]# rpm -Uvh telnet-server-0.17-64.el7.x86_64.rpm
Preparing...                         ######################[100%]
Updating / installing...
   1：telnet-server-0.17-64.el7       ######################[100%]
[root@001 1.telent]# rpm -Uvh telnet-0.17-64.el7.x86_64.rpm
Preparing...                         ######################[100%]
Updating / installing...
   1：telnet-0.17-64.el7              ######################[100%]
```

#### 0x01.2  现在很多centos7版本安装telnet-server以及xinetd之后没有一个叫telnet的配置文件了；如果下面telnet文件不存在的话，可以跳过这部分的更改；如果下面文件存在，请更改配置telnet可以root登录，把disable = no改成disable = yes。

```SHELL
[root@001 1.telent]# vi /etc/xinetd.d/telnet
# default: on
# description: The telnet server serves telnet sessions; it uses \
#   unencrypted username/password pairs for authentication.
service telnet
{
    disable = yes
    flags       = REUSE
    socket_type = stream       
    wait        = no
    user        = root
    server      = /usr/sbin/in.telnetd
    log_on_failure  += USERID
}
```

#### 0x01.3  配置telnet登录的终端类型，在/etc/securetty文件末尾增加一些pts终端，如下

```shell
[root@001 1.telent]# vi /etc/securetty
.
.
.
xvc0
pts/0
pts/1
pts/2
pts/3
[root@001 1.telent]#
```

#### 0x01.4  启动telnet服务，并设置开机自动启动

```shell
[root@001 1.telent]# systemctl enable xinetd
[root@001 1.telent]# systemctl enable telnet.socket
Created symlink from /etc/systemd/system/sockets.target.wants/telnet.socket to /usr/lib/systemd/system/telnet.socket.
[root@001 1.telent]# systemctl start telnet.socket
[root@001 1.telent]# systemctl start xinetd
[root@001 1.telent]# netstat -lntp|grep 23
tcp6     0       0  :::23         :::*           LISTEN        1/systemd 
```

#### 0x01.5  默认情况下，telnet是不允许root登录的。

```shell
[root@001 1.telent]# vi /etc/pam.d/login
```

![0X01.5-1](/images/posts/2020-03-30-centos7-update-openssh8.1p1-openssl-1.0.2s.assets/0X01.5-1.png)

#### 0x01.6  编辑配置文件

```shell
[root@001 1.telent]# vi /etc/pam.d/remote
```

![0X01.6-1](/images/posts/2020-03-30-centos7-update-openssh8.1p1-openssl-1.0.2s.assets/0X01.6-1.png)

#### 0x01.7 测试

![0X01.7-1](/images/posts/2020-03-30-centos7-update-openssh8.1p1-openssl-1.0.2s.assets/0X01.7-1.png)

### 0x02  安装gcc-g++编译器

- **安装gcc-g++**

  ```shell
  [root@001 1.telent]# cd ../2.gcc-gcc+/
  [root@001 1.telent]# ls
  total 42756
  -rwxrwxrwx 1 root root 6215884 Nov 28 18:16 cpp-4.8.2-16.el7.x86_64.rpm
  -rwxrwxrwx 1 root root 16823176 Nov 28 18:24 gcc-4.8.2-16.e17.x86_64.rpm
  -rwxrwxrwx 1 root root 3741952 Nov 28 18:16 glibc-2.17-55.el7.x86_64.rpm 
  -rwxrwxrwx 1 root root 11730016 Nov 28 18:21 glibc-common-2.17-55.el7.x86_64.rpm
  -rwxrwxrwx 1 root root 1063840 Nov 28 18:15 glibc-devel-2.17-55.e17.x86_64.rpm
  -rwxrwxrwx 1 root root 665640 Nov 28 18:15 glibc-headers-2.17-55.el7.x86_64.rpm
  -rwxrwxrwx 1 root root 1574948 Nov 28 18:16 glibc-static-2.17-55.el7.x86_64.rpm
  -rwxrwxrwx 1 root root 197596 Nov 28 18:15 glibc-utils-2.17-55.el7.x86_64.rpm
  -rwxrwxrwx 1 root root 1489100 Nov 28 18:19 kernel-headers-3.10.0-123.el7.x86_64.rpm
  -rwxrwxrwx 1 root root 51732 Nov 28 18:15 Libmpc-1.0.1-3.el7.x86_64.rpm
  -rwxrwxrwx 1 root root 208316 Nov 28 18:15 mpfr-3.1.1-4.e17.x86_64.rpm
  [root@001 2.gcc-gcc+]# rpm -Uvh *.rpm --nodeps --force
  .
  .
  .
     13：qlibc-2.17-260.el7_6.3         #########################[100%]
  ```

- **查看版本**

  ```shell
  [root@001 2.gcc-gcc+]# gcc -v
  Using built-in specs.
  COLLECT_GCC=gcC COLLECTILTO_WRAPPER=/usr/Libexec/gcc/x86_64-redhat-1inux/4.8.2/Lto-wrapper Target:×86 64-redhat-linux Configured with:../configure--prefix=/usr --mandir=/usr/share/man--infodir=/usr/share/in fo--with-bugurl=http://bugzilla. redhat. com/bugzilla--enable-bootstrap--enable-shared--e nable-threads=posix--enable-checking=release--with-system-zlib--enable-cxa atexit--di sable-libunwind-exceptions --enable-gnu-unique-object--enable-linker-build-id--with-linker-hash-style=gnu--enable-Languages=c,c++, objc, obj-c++, java, fortran, ada, go, Lto--enable-plu gin--enable-initfini-array--disable-libgcj--with-isl=/builddir/build/BUILD/gcc-4.8.2-20140120/obj-×8664-redhat-linux/isl-install--with-cloog=/builddir/build/BUILD/gcc-4.8.2-20140120/obj-×86_64-redhat-linux/cloog-install--enable-gnu-indirect-function--with-tune=gener ic--with-arch_32=x86-64--build=x86_64-redhat-linux Thread model: posix 
  gcc version 4.8.2 20140120(Red Hat 4.8.2-16)(GCC)
  ```

### 0x03  安装其他依赖

#### 0x03.0  切换目录

```shell
[root@001 2.gcc-gcc+]# cd ../3.other/
[root@001 2.gcc-gcc+]# ls
total 11632
-rwxrwxrwx 1 root root 7534972 Nov 12 2018 gcc-c++-4.8.5-36.el7.x86_64.rpm
-rwxrwxrwx 1 root root 104264 May 23 2019 libgcc-4.8.5-36.el7_6.2.x86_64.rpm
-rwxrwxrwx 1 root root 161352 May 23 2019 lihgomp-4.8.5-36.e17_6.2.×86_64.rpm
-rwxrwxrwx 1 root root 51732 May 27 2019 libmpc-1.0.1-3.e17.x86_64.rpm
-rwxrwxrwx 1 root root 312040 May 23 2019 libstdc++-4.8.5-36.e17_6.2.x86_64.rpm
-rwxrwxrwx 1 root root 1580364 May 23 2019 libstdc++-devel-4.8.5-36.e17_6.2.×86_64.rpm
-rwxrwxrwx 1 root root 734260 Apr 25 2018 pam-1.1.8-22.e17.i686.rpm 
-rwxrwxrwx 1 root root 736920 Apr 25 2018 pam-1.1.8-22.e17.x86_64.rpm
-rwxrwxrwx 1 root root 188824 Apr 25 2018 pam-devel-1.1.8-22.e17.i686.rpm
-rwxrwxrwx 1 root root 188792 Apr 25 2018 pam-devel-1.1.8-22.el7.×86_64.rpm 
-rwxrwxrwx 1 root root 92840 Nov 12 2018 zlib-1.2.7-18.e17.i686.rpm
-rwxrwxrwx 1 root root 91960 Nov 12 2018 zLib-1.2.7-18.el7.x86_64.rpm
-rwxrwxrwx 1 root root 51164 Nov 12 2018 zlib-devel-1.2.7-18.el7.i686.rpm
-rwxrwxrwx l root root 51128 Nov 12 2018 zlib-devel-1.2.7-18.e17.x86 64.rpm
```

#### 0x03.1  安装依赖文件：安装pam和zlib等（后面的升级操作可能没用到pam，安装上也没啥影响，如果不想安装pam请自行测试）

```shell
[root@001 3.other]# rpm -Uvh *.rpm --nodeps --force
.
.
.
19：1ibgomp-4.8.5-16.e17            #######################[100%]
```

### 0x04  安装openssl

#### 0x04.0  切换目录

```shell
[root@001 3.other]# cd ../4.ssh_ssl/
[root@001 3.other]# ll
total 6812
-rwxrwxrwx 1 root root 1625894 0ct 9 09:53 openssh-8.1pl.tar.gz
-rwxrwxrwx 1 root root 5349149 May 29 2019 openssl-1.0.2s.tar.gz
```

#### 0x04.1  解压openssl文件

```shell
[root@001 4.ssh_ssl]# tar -zxvf openssl-1.0.2s.tar.gz
.
.
.
openss1-1.0.2s/VMS/NISHLIST.TXT 
openss1-1.0.2s/VMS/VMSify-conf.pl
```

#### 0x04.2  进入openssl文件夹

```shell
[root@001 4.ssh_ssl]# cd openssl-1.0.2s/
```

#### 0x04.3  查看版本

```shell
[root@001 openssl-1.0.2s]# openssl version
OpenSSL 1.0.2k-fips 26 Jan 2017
```

#### 0x04.4  备份下面2个文件或目录（如果存在的话就执行）

```shell
[root@001 openssl-1.0.2s]# ll /usr/bin/openssl
[root@001 openssl-1.0.2s]# mv /usr/bin/openssl /usr/bin/openssl_bak
[root@001 openssl-1.0.2s]# ll /usr/include/openssl
[root@001 openssl-1.0.2s]# mv /usr/include/openssl /usr/include/openssl_bak
```

#### 0x04.5 配置、编译、安装3个命令一起执行，&&符号表示前面的执行成功才会执行后面的

```shell
[root@001 openssl-1.0.2s]# cd ~
[root@001 ~]# tar -zxvf perl-5.28.1.tar.gz
[root@001 ~]# mkdir /usr/local/perl
[root@001 ~]# cd perl-5.28.1
[root@001 perl-5.28.1]# ./Configure -des -Dprefix=/usr/local/perl -Dusethreads -Uversiononly

[root@xz-jgyw-001-003 perl-5.28.1]# make && make install

[root@001 perl-5.28.1]# cd /usr/bin
[root@001 bin]# mv perl perl.old
[root@001 bin]# ln -s /usr/local/perl/bin/perl /usr/bin/perl

[root@001 bin]# cd ~/openssh_openssl_update/4.ssh_ssl/openssl-1.0.2s
[root@001 openssl-1.0.2s]# ./config shared && make && make install
```

#### 0x04.6  以上命令执行完毕，echo $?查看下最后的make install是否有报错，0表示没有问题

```shell
[root@001 openssl-1.0.2s]# echo $?
0
```

#### 0x04.7  下面2个文件或者目录做软链接 （刚才前面的步骤mv备份过原来的）

```shell
[root@001 openssl-1.0.2s]# ln -s /usr/local/ssl/bin/openssl /usr/bin/openssl
[root@001 openssl-1.0.2s]# ln -s /usr/local/ssl/include/openssl /usr/include/openssl
[root@001 openssl-1.0.2s]# ll /usr/bin/openssl
lrwxrwxrwx 1 root root 26 Mar 30 13:56 /usr/bin/openssl -> /usr/local/ssl/bin/openssl
[root@001 openssl-1.0.2s]# ll /usr/include/openssl -ld
lrwxrwxrwx 1 root root 30 Mar 30 13:56 /usr/include/openssl -> /usr/local/ssl/include/openssl
```

#### 0x04.8  命令行执行下面2个命令加载新配置

```shell
[root@001 openssl-1.0.2s]# echo "/usr/local/ssl/lib" >> /etc/ld.so.conf
[root@001 openssl-1.0.2s]# /sbin/ldconfig
```

#### 0x04.9  查看确认版本，没问题

```shell
[root@001 openssl-1.0.2s]# openssl version
OpenSSL 1.0.2s 28 May 2019
```

### 0x05  安装openssh

#### 0x05.0  切换目录

```shell
[root@001 openssl-1.0.2s]# cd ..
[root@001 4.ssh_ssl]#
```

#### 0x05.1  解压openssh文件

```shell
[root@001 4.ssh_ssl]# tar -zxvf openssh-8.1p1.tar.gz
.
.
.
openssh-8.1p1/conf igure openssh-8.1p1/ssh config.0openssh-8.1p1/config.h. in
[root@001 4.ssh ssl]# ls 
openssh-8.1p1 openssh-8.1p1.tar.gz openssl-1.0.2s openssL-1.0.2s.tar.gz
[root@001 4.ssh ssl]#
```

#### 0x05.2  可能文件默认显示uid和gid数组都是1000，这里重新授权下。不授权可能也不影响安装（请自行测试）

```shell
[root@001 4.ssh_ssl]# chown -R root.root openssh-8.1p1
```

#### 0x05.3  切换目录&查看版本

```shell
[root@001 4.ssh_ssl]# cd openssh-8.1p1/
[root@001 openssh-8.1p1]# ssh -V
0penSSH_7.4p1, OpenSSL 1.0.2k-fips 26 Jan 2017
```

#### 0x05.4 备份&删除原文件

```shell
[root@001 openssh-8.1p1]# cp -r  /etc/ssh /root/openssh_openssl_update/4.ssh_ssl/
[root@001 openssh-8.1p1]# rm -rf /etc/ssh
```

#### 0x05.5  编译安装openssh8.1

```shell
[root@001 openssh-8.1p1]# ./configure --prefix=/usr/ --sysconfdir=/etc/ssh --with-openssl-includes=/usr/local/ssl/include --with-ssl-dir=/usr/local/ssl --with-zlib --with-md5-passwords --with-pam && make && make install
```

#### 0x05.6  测试是否有报错

```shell
[root@001 openssh-8.1p1]# echo $?
0
```

#### 0x05.7  修改/etc/ssh/sshd_config如下

```shell
[root@001 openssh-8.1p1]# vi /etc/ssh/sshd_config
```

![0x05.7-1](/images/posts/2020-03-30-centos7-update-openssh8.1p1-openssl-1.0.2s.assets/0x05.7-1.png)

![0x05.7-2](/images/posts/2020-03-30-centos7-update-openssh8.1p1-openssl-1.0.2s.assets/0x05.7-2.png)

- **最终为如下内容，其他的不要动**

```shell
[root@001 openssh-8.1p1]# grep "^PermitRootLogin"  /etc/ssh/sshd_config
PermitRootLogin yes
[root@001 openssh-8.1p1]#  grep  "UseDNS"  /etc/ssh/sshd_config
UseDNS no
```

#### 0x05.8  设置开机启动，这里确保service 和systemctl都设置开机启动

```shell
[root@001 openssh-8.1p1]# cp -a contrib/redhat/sshd.init /etc/init.d/sshd
[root@001 openssh-8.1p1]# cp -a contrib/redhat/sshd.pam /etc/pam.d/sshd.pam
[root@001 openssh-8.1p1]# chmod +x /etc/init.d/sshd
[root@001 openssh-8.1p1]# chkconfig --add sshd
[root@001 openssh-8.1p1]# mv  /usr/lib/systemd/system/sshd.service  /root/openssh_openssl_update/4.ssh_ssl/
[root@001 openssh-8.1p1]# mv  /usr/lib/systemd/system/sshd.socket  /root/openssh_openssl_update/4.ssh_ssl/
[root@001 openssh-8.1p1]# chkconfig sshd on
```

#### 0x05.9  查看进程运行情况

```shell
[root@001 openssh-8.1p1]# netstat -lntp | grep 22
```

#### 0x05.10  查看升级后的版本

```shell
[root@001 openssh-8.1p1]# ssh -V
OpenSSH_8.1p1, OpenSSL 1.0.2s 28 May 2019
```

#### 0x05.11  进行端口测试

```shell
[root@001 openssh-8.1p1]# telnet 127.0.0.1 22
```

### 0x06  关闭telnet

```shell
[root@001 openssh-8.1p1]# systemctl disable xinetd.service
Removed symlink /etc/systemd/system/multi-user.target.wants/xinetd.service.
[root@001 openssh-8.1p1]# systemctl stop xinetd.service
[root@001 openssh-8.1p1]# systemctl disable telnet.socket
Removed symlink /etc/systemd/system/sockets.target.wants/telnet.socket.
[root@001 openssh-8.1p1]# systemctl stop telnet.socket
```



