---
title: 网盘seafile搭建过程
date: 2014.12.03 15:22:03
tags: [centos]
categories: 后端
---

#网盘seafile搭建过程(centos 6.5，python2.7)
-------------
##注意事项
- ###`请注意，安装seafile要2.7，我也不知道为什么`
- ###`2.7以下的版本貌似无法识别一样`
- ###`安装请自己寻找安装包，编译安装`[地址](https://pypi.python.org/pypi)
----------
##安装步骤
1. 切记，第一点绝对是先安装zlib
    `yum install zlib`
    检查无误了才可以开始安装Python，否则之后的安装肯定会失败，这个坑我踩了三次，最后才学乖了
2. [安装python2.7](http://blog.csdn.net/laiahu/article/details/6903100)，这里特别注意，一定要把yum默认使用的python2.4改回来，否则也同样深陷大坑一直百度。
 
    关键命令如下：
    `mv /usr/bin/python /usr/bin/python.bak`
    `ln -s /usr/local/Python2.7/bin/python2.7 /usr/bin/python`
    `vim /usr/bin/yum`
3. 之后我们就需要来运行seafile的mysql安装程序，
    `./setup-seafile-mysql.sh`
    如果你缺少什么模块，它会进行提示，这时候神奇的事情发生了，比如它会提示缺少setuptools，但是你使用yum安装的时候，却提示不缺少，问题就在这里了，因为你已经安装了python2.7，而yum默认使用的是其他版本的python，导致seafile运行脚本的时候，检查不到模块的存在，所以我们需要手动安装模块，编译。这些只要按照顺序去编译安装就没问题了
4. 需要的模块如下：
    - setuptools
    - imaging
    - mysql-devel
    - mysqldb
5. 这里有一个细节，就是mysql-devel不需要自己去找包也可以的，直接`yum installl mysql-devel`
6. 至此，seafile就可以正常安装了，其他详细的可以参照官方文档