---
title: Mac Docker无法启动
date: 2018.11.09 15:07:17
tags: [docker]
categories: 后端
---

问题的具体症状如下：
- 当你启动Docker 之后，像往常一样，Docker 会问你输入密码
- 你点了ok，接下来系统就要你输入管理员密码
- 然后就没反应了，直到你自己点击退出

我是更新到最新版本的Mac OS 10.14.1 Beta (18B45d)之后出现的
在github 上面查了，也有人出现过这个问题，但是并没有很好的解决方法。
后面我在[github docker issue](https://github.com/docker/for-mac/issues/1903)看到也有人遇到这个问题

按照**mrichman**的方法并不能成功启动。

最后我只得@其他哥们看看解决的那个人是怎么说的，然后那个大胡子兄弟跟我说，具体不大记得，就大概是重装了一下。

我尝试了重装，但是并不起作用，后面我自己推测大概是卸载不干净导致，于是我就上网上搜了一些教程，按照这篇[卸载](https://therealmarv.com/how-to-fully-uninstall-the-offical-docker-os-x-installation/)文章操作之后重装，Docker 终于可以重新启动了。
