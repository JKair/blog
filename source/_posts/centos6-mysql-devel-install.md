---
title: 解决centos6没办法安装mysql-devel的问题
date: 2014.12.03 15:17:02
tags: [centos]
categories: 后端
---

#解决centos6没办法安装mysql-devel的问题

最近打算在服务器上安装seafile这款网盘，seafile是基于python开发的，需要许多python的扩展库，最重点的就是mysqldb。
在放上服务器之前已经在虚拟机先测试过，测试结果是没问题，但是转移到服务器的时候，出现了mysql-devel无法安装的问题。问题如下
```
[root@iamca yum.repos.d]# yum install mysql-devel
Loaded plugins: fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
 * epel: ftp.riken.jp
 * ius: ftp.neowiz.com
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package mysql-devel.x86_64 0:5.1.73-3.el6_5 will be installed
--> Processing Dependency: mysql = 5.1.73-3.el6_5 for package: mysql-devel-5.1.73-3.el6_5.x86_64
--> Processing Dependency: libmysqlclient_r.so.16()(64bit) for package: mysql-devel-5.1.73-3.el6_5.x86_64
--> Processing Dependency: libmysqlclient.so.16()(64bit) for package: mysql-devel-5.1.73-3.el6_5.x86_64
--> Running transaction check
---> Package mysql-devel.x86_64 0:5.1.73-3.el6_5 will be installed
--> Processing Dependency: mysql = 5.1.73-3.el6_5 for package: mysql-devel-5.1.73-3.el6_5.x86_64
---> Package mysqlclient16.x86_64 0:5.1.61-4.ius.centos6 will be installed
--> Finished Dependency Resolution
Error: Package: mysql-devel-5.1.73-3.el6_5.x86_64 (updates)
           Requires: mysql = 5.1.73-3.el6_5
           Installed: mysql-5.5.29-1.el6.x86_64 (@CentALT)
               mysql = 5.5.29-1.el6
           Available: mysql-5.1.71-1.el6.x86_64 (base)
               mysql = 5.1.71-1.el6
           Available: mysql-5.1.73-3.el6_5.x86_64 (updates)
               mysql = 5.1.73-3.el6_5
           Available: mysql55-5.5.39-1.ius.centos6.x86_64 (ius)
               mysql = 5.5.39-1.ius.centos6
           Available: mysql56u-5.6.20-2.centos6.x86_64 (ius)
               mysql = 5.6.20-2.centos6

 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest
```

之后用百度和google搜索了多种方法

- 找rpm源包，找到相对应的版本也不能安装。十分之郁闷。
- 换源，解决不了
- 更新yum，解决不了
- 更新mysql,由于是服务器上的，不敢随便更新，此方法没试过
- 刷新源，解决不了
- 卸载mysql重新安装，同上，不敢试

想到用源码或许可以直接编译出来，于是在google搜索mysql-devel-5.5.29-1.el6.x86_64.tar.gz

没找到源码包，但是却有一个东西引起我的注意：`mysql-devel-5.5.29-1.el6.x86_64.rpm`,于是重新搜索`mysql-devel-5.5.29-1.el6.x86_64.rpm`,终于被我找到正确版本的rpm包
网站：`http://mirrors.vpsmate.org/CentALT/repository/centos/6/x86_64/`

但是随之而来又出现一个问题，安装centos rpm包的时候提示：`Header V3 DSA signature: NOKEY`

百度搜索之，看了[这个网站](http://www.jb51.net/os/RedHat/32482.html)

这是由于yum安装了旧版本的GPG keys造成的，解决办法就是
引用
`rpm --import /etc/pki/rpm-gpg/RPM*`


之后得到解决，现在seafile依赖库已经全部安装完毕。问题就此得到解决
