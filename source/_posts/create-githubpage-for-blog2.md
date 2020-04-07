---
title: 用githubpage搭建自己的博客（二）
date: 2020-04-07 14:16:53
tags: [githubpage]
categories: 后端
---


[上一篇文章](https://kair.xyz/2020/04/07/create-githubpage-for-blog/)我们已经搭建好了自己的博客，并且绑定好了自己的域名，这一篇我们来看看如何维护自己的博客

## 给自己的博客设置主题

由于我不是设计师，我真的不会设计，估计自己来设计也是歪瓜裂枣，所以我都选择别人写的。

首先我们可以上[主题推荐](https://hexo.io/themes/)选择自己喜欢的主题，当然你也可以在 github 上面搜索，这里以我的使用的主题[fexo](http://forsigner.com/fexo-doc-zh-cn/)为例

### 克隆主题到自己的项目

```
$ cd my-blog
$ git clone git@github.com:forsigner/fexo.git themes/fexo
```

### 修改配置文件

修改自己博客根目录下的`_config.yml`，设置`theme:fexo`

## 设置CNAME

前面我们绑定了自己的域名，所以这里我们要将CNAME，放到自己的`hexo`项目进来，不然每次我们更新博客，CNAME 就不见了。

在`your-blog/source`下，创建CNAME文件（注意，没有文件类型的后缀），CNAME 的内容就是你的域名。

## 上传项目

在你博客的根目录下，`hexo clean && hexo d`，上传成功后，刷新自己的博客（可能需要一分钟），就发现主题换上去了，但是都是主题默认的设置

## 博客的设置和更新

### 修改样式

我们可以根据你选择的主题的文档去进行修改设置你喜欢的样式等等，比如我使用的[fexo](http://forsigner.com/fexo-doc-zh-cn/)，如果我要修改头像，我就修改`/themes/images/avatar.jpg`就可以了。其他的设置每个主题都不一样，可以自己参考文档，这部分并不难

### 写文章

我们可以在自己的 blog 下，`hexo new your-post-name`，之后会在`source/_posts/`创建一个叫做`your-post-name`的文件，文章的表头

```
---
title: 用githubpage搭建自己的博客（二） //名字
date: 2020-04-07 14:16:53			//创建日期
tags: [githubpage]  				//标签
categories: 后端						//类别
---
```

然后空一行开始写就可以了，写完之后，还是`hexo clean && hexo d`。

### blog源

因为github.io的这个仓库上传的是打包后的，所以，我们自己维护的这个blog 项目，可以直接上传到其他的仓库保存，比如我就上传到一个 [blog](https://github.com/JKair/blog) 的项目。看你个人想不想上传这个项目。

本文到此就结束了，希望你看完有收获