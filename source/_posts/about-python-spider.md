---
title: 关于python爬虫
date: 2015.07.28 16:19:22
tags: [python]
categories: 后端
---

最近在弄一个需求，需要写到爬虫，本来是使用php的，但是php没有一款可以满足需要的爬虫框架，于是转而使用python。
目前比较流利的python爬虫框架有好多，这次最主要接触了`ghost.py`、`spynner`
解析就使用了`beautiful`
这篇文章最主要还是记录一下一些坑。
先说一下这两款的共性，就是都会用到pyqt，所以安装之前一定先要安装pyqt，至于原因，则是因为两者都是基于webkit浏览器内核的爬虫框架，所以会使用到pyqt，总之你懂的。

关于ghost.py
===================
安装这款框架过程曲折离奇，我首先是在osx系统下面安装的，首先我
`pip install ghost.py`
写好了例子，之后提示没有qt库
然后则安装pyqt
`sudo brew install pyqt`
安装完毕之后，你还会发现错误，因为你还需要安装`pyside`
`pip instll pyside`
再之后，你才可以使用ghost.py，这过程究竟有多少辛酸，由于没有及时记录，忘记了，如果你发现了什么其他的恶心的错误可以联系我，我们来讨论。
当你安装好，照着网上的教程乱敲一阵代码之后，你会发现，python始终提示你
```
ghost no attritube open
```
当你看到这几个触目惊心的单词，又完全没有头绪的时候！你就能体会我当时的痛苦，一个代码都没错，但是就是报错，就是不能用！我Google了一阵，才发现！是版本问题，网上的教程太老了，全部都是0.1的，现在都0.2了，你会发现，0.2的使用方法改变了，在0.2的里面，你不可以直接open，你需要这样写
```
from ghost import Ghost

ghost = Ghost()
session = ghost.start()
session.open("Your Url")
```
顺便附上[文档](http://ghost-py.readthedocs.org/en/latest/#ghost.Session)

关于spynner
==================
安装过程略过，假设你安装成功ghost.py，那这款框架，你可以直接pip搞定。
然而我想要说的坑并不在这里！
1. `spynner`在解析中文网站的时候，全部都是乱码，意思就是解析不了，然而如何改变这种情况呢？我所搜到的方法就是在python的`Lib/site-packages/`下，找到spynner的源码包，然后在`/spynner`下修改`browser.py`
把477行的`def _get_html(self):`函数下的return改成`return unicode(self.webframe.toHtml().toUtf8(), 'utf-8', 'ignore')`
1. 还有一个问题，就是windows用户可能会遇到，`.egg文件`结尾的文件并不能以文件夹形式打开，那这时候，你只要用压缩软件打开，然后解压出来，就可以了，再之后你只要把文件替换回去，就可以解决这个问题

关于beautifulsoup
================
你在使用beautifulsoup的时候可能会遇到一个问题，也是编码问题，这时候，你记得在实例化beautifulsoup的时候加上`from_encoding="utf-8"`参数，就可以解决这个问题。

关于这两天爬虫遇到的坑就记录到此。


后记
=============
最近我又开始研究爬虫的东西了，然后我发现，其实有一种很好的解决方案，来自于chrome的扩展插件。其实如果利用插件功能，配合你服务器上的接口用ajax来接收数据，能够很简单的实现爬虫功能，并且纯模拟浏览器操作，成本非常低，能够省去好多繁复的研究各种框架的时间