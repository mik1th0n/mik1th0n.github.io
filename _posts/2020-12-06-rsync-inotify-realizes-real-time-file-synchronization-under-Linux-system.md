---
layout: post
title: 	使用rsync+sersync(inotify)实现Linux系统下文件实时同步
categories: FileSync
description: 
keywords: rsync,inotify,Linux,文件同步,文件实时同步
---







### 0x00  服务介绍

#### 0x00.0 为什么要用rsync+sersync架构？

- sersync是基于inotify开发的，类似于inotify-tools的工具
- sersync可以记录下被监听目录中发生变化的（包括增加、删除、修改）具体某一个文件或者某一个目录的名字，然后使用rsync同步的时候，只同步发生变化的文件或者目录
- 因为服务异常导致的同步失败有记录，便于恢复，确保高可用！

#### 0x00.1 rsync+inotify-tools与rsync+sersync架构的区别？

- **rsync+inotify-tools**
  - inotify只能记录下被监听的目录发生了变化（增，删，改）并没有把具体是哪个文件或者哪个目录发生了变化记录下来；
  - rsync在同步的时候，并不知道具体是哪个文件或目录发生了变化，每次都是对整个目录进行同步，当数据量很大时，整个目录同步非常耗时（rsync要对整个目录遍历查找对比文件），因此效率很低

- **rsync+sersync**
  - sersync可以记录被监听目录中发生变化的（增，删，改）具体某个文件或目录的名字；
  - rsync在同步时，只同步发生变化的文件或目录（每次发生变化的数据相对整个同步目录数据来说很小，rsync在遍历查找对比文件时，速度很快），因此效率很高。

#### 0x00.2 同步过程

- 在同步服务器上开启sersync服务，sersync负责监控配置路径中的文件系统事件变化；
- 调用rsync命令把更新的文件同步到目标服务器；
- 需要在主服务器配置sersync，在同步目标服务器配置rsync server（注意：是rsync服务）

#### 0x00.3 同步过程和原理

- 用户实时的往sersync服务器上写入更新文件数据；
- 此时需要在同步主服务器上配置sersync服务；
- 在另一台服务器开启rsync守护进程服务，以同步拉取来自sersync服务器上的数据。

> 通过rsync的守护进程服务后可以发现，实际上sersync就是监控本地的数据写入或更新事件；然后，在调用rsync客户端的命令，将写入或更新事件对应的文件通过rsync推送到目标服务器。

### 0x01 环境准备

- 现有业务上的服务器列表
  - server1（主服务器）IP：192.168.31.94
  - server2（从服务器 / 备份服务器）IP：192.168.31.95



- 业务的要求就是在server1上操作一个文件（增删改），实时同步到server2中。也就是保证server2要与server1上面的文件要时刻保持一致。
- 为了达到上述业务要求，按照正常测试逻辑我们还是先对server2进行相关配置，因为server2是作为备份服务器所以在server2上我们只需要安装rsync软件即可。

### 0x02 环境配置

> 本教程是基于CentOS 7.6系统配置的，为了适合不同环境配置的节点安装，本教程将采用编译安装的方式

- [gcc_rpm.tar.gz]()

#### 0x02.0 环境配置检查

```bash
[root@server2 ~]# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 
# 环境检查

#查看gcc【编译器】版本
gcc -v 
#查看perl【解释器】版本
perl -v

# 提示无该命令则需要安装，如果有版本信息则直接执行下步操作
#安装编译器gcc：
```

#### 0x02.1 安装gcc

```bash
#第一步：上传gcc_rpm.tar.gz到/usr/src 下
[root@server2 ~]# cd /usr/src/
[root@server2 src]# rz
rz waiting to receive.
Starting zmodem transfer.  Press Ctrl+C to cancel.
Transferring gcc_rpm.tar.gz...
  100%   24686 KB    2468 KB/sec    00:00:10       0 Errors  

[root@server2 src]# ll
total 24688
drwxr-xr-x. 2 root root        6 Apr 11  2018 debug
-rw-r--r--. 1 root root 25279215 Dec  6 23:07 gcc_rpm.tar.gz
drwxr-xr-x. 2 root root        6 Apr 11  2018 kernels

#第二步：解压缩到/usr/local
[root@server2 src]# tar -zxvf gcc_rpm.tar.gz -C /usr/local

#第三步：rpm安装
[root@server2 src]# cd /usr/local/gcc_rpm
[root@server2 gcc_rpm]# 
[root@server2 gcc_rpm]#  rpm -Uvh  *.rpm  --nodeps  --force 

#第四步：版本查看，确认是否安装成功
[root@server2 gcc_rpm]# gcc -v
Using built-in specs.
Target: x86_64-redhat-linux
Configured with: ../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-languages=c,c++,objc,obj-c++,java,fortran,ada --enable-java-awt=gtk --disable-dssi --with-java-home=/usr/lib/jvm/java-1.5.0-gcj-1.5.0.0/jre --enable-libgcj-multifile --enable-java-maintainer-mode --with-ecj-jar=/usr/share/java/eclipse-ecj.jar --disable-libjava-multilib --with-ppl --with-cloog --with-tune=generic --with-arch_32=i686 --build=x86_64-redhat-linux
Thread model: posix
gcc version 4.4.7 20120313 (Red Hat 4.4.7-4) (GCC) 
[root@server2 gcc_rpm]# g++ -v 
Using built-in specs.
Target: x86_64-redhat-linux
Configured with: ../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-languages=c,c++,objc,obj-c++,java,fortran,ada --enable-java-awt=gtk --disable-dssi --with-java-home=/usr/lib/jvm/java-1.5.0-gcj-1.5.0.0/jre --enable-libgcj-multifile --enable-java-maintainer-mode --with-ecj-jar=/usr/share/java/eclipse-ecj.jar --disable-libjava-multilib --with-ppl --with-cloog --with-tune=generic --with-arch_32=i686 --build=x86_64-redhat-linux
Thread model: posix
gcc version 4.4.7 20120313 (Red Hat 4.4.7-4) (GCC) 
```

#### 0x02.2 安装perl

```bash
#第一步：上传perl-5.26.1.tar.gz到/usr/src 下
[root@server2 gcc_rpm]# cd /usr/src
[root@server2 src]# rz
rz waiting to receive.
Starting zmodem transfer.  Press Ctrl+C to cancel.
Transferring perl-5.26.1.tar.gz...
  100%   16542 KB    16542 KB/sec    00:00:01       0 Errors  

#第二步：解压缩到 /usr/local/perl
[root@server2 src]# tar -zxvf perl-5.26.1.tar.gz  -C /usr/local
#第三步：进入文件目录,设置config
[root@server2 src]# cd /usr/local/perl-5.26.1/
[root@server2 perl-5.26.1]# ./Configure -des -Dprefix=/usr/local/perl
#第四步：编译并检测
[root@server2 perl-5.26.1]# make && make test
#第五步：安装
[root@server2 perl-5.26.1]# make install
#第六步：验证
[root@server2 perl-5.26.1]# perl -v
```





### 0x03 server2安装及配置rsync

#### 0x03.0 server2下载及安装rsync

```bash
# 将rsync.tar包上传至安装节点/opt/rsync目录下
[root@server2 perl-5.26.1]# mkdir /opt/rsync
[root@server2 perl-5.26.1]# cd /opt/rsync/
[root@server2 rsync]# rz
rz waiting to receive.
Starting zmodem transfer.  Press Ctrl+C to cancel.
Transferring rsync-3.0.9.tar.gz...
  100%     774 KB     774 KB/sec    00:00:01       0 Errors  

[root@server2 rsync]# ll
total 776
-rw-r--r--. 1 root root 792725 Dec  5 23:32 rsync-3.0.9.tar.gz

#然后解压
[root@server2 rsync]# tar -zxvf rsync-3.0.9.tar.gz

# 编译
[root@server2 rsync]# cd rsync-3.0.9
[root@server2 rsync-3.0.9]# ./configure --prefix=/opt/cosmo/rsync
...
config.status: creating popt/dummy
config.status: creating shconfig
config.status: creating config.h

    rsync 3.0.9 configuration successful
# 安装
[root@server2 rsync-3.0.9]# make  && make install
# 验证
[root@server2 rsync-3.0.9]# echo $?
0
```

#### 0x03.1 server2配置rsync配置文件

```bash
[root@server2 rsync]# vim /etc/rsyncd.conf
uid = root                         #运行进程的身份                     
gid = root                         #运行进程的组                
address =192.168.31.4              #监听IP          
port =873                          #监听端口              
hosts allow =192.168.31.0/24       #允许同步客户端的IP地址，可以是网段，或者用*表示所有 192.168.1.0/24或192.168.1.0/255.255.255.0  
use chroot = yes                   #是否囚牢，锁定家目录，rsync被黑之后，黑客无法再rsync运行的家目录之外创建文件，选项设置为yes            
max connections =10                 #最大连接数           
pid file =/var/run/rsyncd.pid       #进程PID，自动生成  
lock file =/var/run/rsync.lock      #指max connectios参数的锁文件
log file =/var/log/rsyncd.log       #日志文件位置

 
[wwwroot]                           #同步模块名称           
path =/opt/cosmo/test/              #同步文件路径     
comment = used for web-data root    #模块描述
read only = false                   #设置服务端文件读写权限   
list = yes                          #是否允许查看模块信息
auth users = rsyncuser              #备份的用户，和系统用户无关
secrets file =/etc/rsync.passwd     #存放用户的密码文件，格式是  用户名：密码
```

#### 0x03.2 server2配置密码文件

```bash
# 创建 rsync 认证文件  可以设置多个，每行一个用户名: 密码，注意中间以“:” 分割
[root@server2 ~]# echo "rsync:rsync" > /etc/rsync.pass

# 设置文件所有者读取、写入权限
[root@server2 ~]# chmod 600 /etc/rsyncd.conf
[root@server2 ~]# chmod 600 /etc/rsync.pass 

# 启动服务器 B 上的 rsync 服务
#rsync --daemon -v
[root@server2 ~]# rsync --daemon

# 监听端口 873

```







#### 0x02.2 server2启动rsync服务



















