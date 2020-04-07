---
title: Mac 下搭建laravel的坑
date: 2015.04.17 20:07:15
tags: [laravel]
categories: 后端
---

最近在搞laravel，由于入手了MacBook，所以打算在MacBook下搭建环境开发，结果遇到了各种大坑，神坑。如何使用laravel去开发就不赘述了。
1. 首先我跪在composer的安装下，后来参考了[此篇文章](http://jingyan.baidu.com/article/7f41ecec16842d593d095c9c.html)，还有composer中国的，具体自己百度吧，不难。
然后有一个问题，它居然提示
`
Missing PHP53 or PHP54 from homebrew-php. Please install one or the other before continuing
Error: An unsatisfied requirement failed this build.
`
但是我觉得不可能啊，不是Mac会自带php的吗？于是
`php -v`
php 5.5无误啊！为何！！！
百度了好久，全部是废话，于是Google了，瞬间得到[答案](http://stackoverflow.com/questions/15185152/missing-php53-or-php54-from-homebrew-php)参考二楼说的话

1. 第二个坑
`
Composer could not find a composer.json file in /root
To initialize a project, please create a composer.json file as described in the http://getcomposer.org/ "Getting Started" section
`
天呐，我我怎么运气那么不好，于是我又搜搜搜搜搜，终于找到答案了，原因就是composer的版本低，只要升级一下版本就好了
`composer self-update`

1. 第三个坑
nginx的问题，个人比较喜欢nginx，brew安装完nginx之后访问localhost:8080，可以正常访问，welcome nginx。于是配了80端口。
这里有一个权限问题请注意，启动nginx，要
`sudo nginx`
百度一下就知道了，1024以下的端口都需要root访问权限
然后就又来一个坑了，配到80之后，一直没办法使用，Nginx一直提示502。于是又配了日志，日志提示如下
`2794#0: *1 kevent() reported that connect() failed (61: Connection refused) while connecting to upstream, client: 127.0.0.1,
 server: localhost, request: "GET /favicon.ico HTTP/1.1", upstream: "fastcgi://127.0.0.1:9000", host: "localhost"`
联想到有可能是php-fpm的问题，于是尝试启动php-fpm，居然不行！干，于是又搜搜搜，原来是没有配好php-fpm.conf导致。于是
`
$cd /private/etc
$ls
`
你可以看到存在`php-fpm.conf.default`文件，于是你把这份文件复制一份出来就好了
`sudo cp php-fpm.conf.default php-fpm.conf`
本以为到此爬坑之旅就结束了
又来了一个错误
`
[17-Apr-2015 17:56:15] ERROR: failed to open error_log (/usr/var/-1): No such file or directory (2)
[17-Apr-2015 17:56:15] ERROR: failed to post process the configuration
[17-Apr-2015 17:56:15] ERROR: FPM initialization failed
`
从字面上的意思，肯定是日志配置除了问题，于是又看了看文件的位置，配了如下两项
`
[global]
error_log = log/php-fpm.log#这里写你要存放日志的路径，你可以使用默认路径
[www]
catch_workers_output = yes
`
php-fpm终于正常启动了。

1. 接下来打开localhost，发现页面全部空白。。。。。。。
百度之，发现要给storage 777权限，于是给之。终于！
又来一个问题！！！
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/121543-852f1551cf29005d.png)
这个问题的原因，是因为laravel需要依赖PHP的mcrypt扩展，只要开启了就可以了
参考此篇[文章](http://benohead.com/mac-os-x-php-notice-use-undefined-constant-mcrypt_rijndael_128/)

1. 再来一个坑，提示
`
SQLSTATE[HY000] [2002] No such file or directory
`
解决方案如下
```
bogon:mylaravel lvweichong$ sudo find / -name mysql.sock
Password:
find: /dev/fd/3: Not a directory
find: /dev/fd/4: Not a directory
/private/tmp/mysql.sock
bogon:mylaravel lvweichong$ ll /var/mysql
-bash: ll: command not found
bogon:mylaravel lvweichong$ ls /var/mysql
ls: /var/mysql: No such file or directory
bogon:mylaravel lvweichong$ sudo mkdir /var/mysql
bogon:mylaravel lvweichong$ sudo ln -s /private/tmp/mysql.sock /var/mysql/mysql.sock
bogon:mylaravel lvweichong$ ls /var/mysql/
mysql.sock
```
爬坑之旅结束。

伤心：逗比的自己，修改php.ini你要重启的是php-fpm，不是nginx！
附上mac下php-fpm重启的命令`pgrep php-fpm |xargs sudo kill -USR2`

备注：php.ini，mac自带的也是php.ini.default，所以你也要搞一份出来。╮(╯▽╰)╭，问题解决了就是舒畅