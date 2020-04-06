---
title: gitlab搭建过程中遇到的坑
date: 2014.12.02 14:25:53
tags: [gitlab]
categories: 后端
---

近期尝试过搭建一个gitlab，搭建参考的是这篇[文章](http://www.xuebuyuan.com/150435.html)，搭建过程中也着实遇到好多坑，这篇文章中没有做过多说明。以下是遇坑的记录。
1. 关于ruby的坑，安装ruby之前一定要先安装好yaml，否则执行有关gem的命令时，会一直报如下错误
    ```
    It seems your ruby installation is missing psych (for YAML output). 
    To eliminate this warning, please install libyaml and reinstall your ruby.
    ```
    解决这个问题的办法参考[此文章](http://www.hiceon.com/topic/Ruby-on-rail-yaml/)，意思就是在安装ruby之前要先安装好libyaml，此文章的作者说只要`make clean`就可以解决，我实验过后发现不行，不知道是不是我理错了作者的意思。
1. 关于gitolite的，在安装gitolite之前，记得先安装好perl-Time-HiRes。
否则会报错如下
    ```
    Can't locate Time/HiRes.pm in @INC (@INC contains: /home/git/gitolite/src/lib /usr/local/lib/perl5 /usr/local/share/perl5 /usr/lib/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib/perl5 /usr/share/perl5 .) at /home/git/gitolite/src/lib/Gitolite/Common.pm line 76.
    BEGIN failed--compilation aborted at /home/git/gitolite/src/lib/Gitolite/Common.pm line 76.
    Compilation failed in require at gitolite/install line 15.
    BEGIN failed--compilation aborted at gitolite/install line 15.
    ```
    具体的解决方案是查看了[此文章](http://segmentfault.com/q/1010000000380149)解决的。
    意思就是先`yum install perl-Time-HiRes`之后再去安装gitolite。
