---
title: windows下使用git bash的alias
date: 2020.04.04 23:52:31
tags: [git]
categories: 后端
---

虽然我的主要开发环境不是`windows`，但是有时候会使用到，所以也会在windows安装一些环境。
cmd的界面实在太差了，而且如果使用习惯Linux下的命令，在cmd就极其不适应了。
举个例子就是`ls --> dir`，因为我们就算再windows下开发，也一定会安装git，然而这时候，git有自带一个命令行`git bash`，Linux下的命令基本上都支持。
然而我们有时候会想自定义一些缩写，比如使用docker的时候，我会习惯性不打`docker-compose`这么长的命令，而是直接缩写成`dc`，所以如果没有`alias`我会很不习惯，然而windows并没有像linux那样，将配置放在`~/.bash_profile`，所以使用`git bash`的时候，如果要修改就不一样。
解决步骤如下：
- 找到git的安装目录下的`etc\profile.d`，如果你是默认安装的就是在`C:\Program Files\Git\etc\profile.d`
- 像在linux下一样，修改`bash_profile.sh`，重启，搞定。
![image.png](https://upload-images.jianshu.io/upload_images/121543-4c8c0e3f4a2e6c3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
