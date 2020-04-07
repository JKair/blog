---
title: PHP7缺少ssl库的问题
date: 2016.05.11 17:51:27
tags: [php7]
categories: 后端
---

遇到问题的起因是因为我需要发送邮件，但是邮件需要通过ssl。
当我运行的时候，laravel提示
```
 [Swift_TransportException]                                                   
  Connection could not be established with host smtp.exmail.qq.com [Unable to  
   find the socket transport "ssl" - did you forget to enable it when you con  
  figured PHP? #0]   
```
这个问题就是php缺少的ssl库没办法使用，解决的方法其实也不难。
最简单的就是直接重新编译php7，加上--enable-openssl参数，但是不想重新编译怎么办，很简单，我们只要拿到openssl.so的扩展就好了。
1. 首先从php7的官网下载[php7](http://php.net/ChangeLog-7.php#7.0.6)
1. 解压并进入文件夹`cd php7.0/ext/openssl`
1. 运行`phpize`，如果命令无效，请用`whereis phpize `找到phpize的地址，用绝对地址运行即可
1. 运行`./configure --with-openssl --with-php-config=/path/to/php-config` 这里请注意后面的--with-php-config参数，你需要找到自己环境的php-config，同理，用`whereis php-config`查找一下。
1. 运行`make && make install`，会显示编译出来后so文件放置的地方，直接进入这个目录。
1. 运行`cp openssl.so /path/to/phpext`。这里你需要把文件复制到php的扩展目录里面，扩展目录在php.ini里面的`extension_dir`这个设置对应的地址，假设你甚至不知道php.ini在哪里，你可以直接运行`php -i`，在load ini那一项，你就可以看到php.ini的地址。

至此，此问题完全解决。`注意，其他版本的php也是同理，没有必要只为了一个库就重新编译整个php。`