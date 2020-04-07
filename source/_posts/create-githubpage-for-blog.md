---
title: 用githubpage搭建自己的博客（一）
date: 2020-04-07 14:16:53
tags: [githubpage]
categories: 后端
---

需要准备的工具如下：
=================
- [git](https://git-scm.com/)
- [npm](https://www.npmjs.cn/getting-started/installing-node/)
- [hexo](https://hexo.io/zh-cn/docs/)
- 一个自己的域名（可以去 godaddy 买，或者去阿里云买，但是阿里云买就需要备案，手尾比较长，本文用godaddy 示例）
- github 账号
- 你机子ssh的public key

下面开始我们的搭建：
=================
1. 在 github 部署ssh。
    - 打开命令行终端（windows 用户打开git-bash）
    - 输入ssh-keygen，一直按enter
    - 生成完毕后，我们`cd ~/.ssh`（windows 用户在你本机用户的根目录下，有个.ssh 文件夹），打开之后我们会看到里面有两个文件，一个是`id_rsa`，一个是`id_rsa.pub`，这个`id_rsa.pub`，就是生成的公钥了
    - 打开我们的github的[SSH and GPG keys](https://github.com/settings/keys)
    - 点击New SSH Key，在下面的框里面填上`id_rsa.pub`的内容，这样我们就部署完ssh了
1. 创建一个仓库，名字是你的用户名开头，github.io 为结尾。比如我的用户名是JKair，那么就创建`jkair.github.io`。 
1. 用hexo 初始化我们的博客，并且上传到github。
    - 在你的工作目录下，`hexo init blog`
    - 打开创建好的 blog，然后再打开`_config.yml`，拉到最下面
    - 我们再修改里面的部署配置：
      ```
      deploy:
      type: git
      repo: 你的仓库地址，记得是ssh地址，这样不用老是输入密码
      #branch: 这个可填也不填，如果不指定的话，hexo 将会自己创建一个master 分支上传
      ```
    - 打开命令行，在 blog 的目录下输入 `hexo clean && hexo d`，等hexo 编译完就部署上去了了
    - 打开你的项目地址比如我的就是http://jkair.github.io
1. 绑定自己的域名（这里以godaddy 示例，因为阿里云买的话还要备案啥的，手尾很长）
    - 首先我们去[godaddy](https://sg.godaddy.com/zh)注册一个账号，注册完毕之后登陆，主页就能直接搜索域名了，花几块钱（土豪除外）买一个你自己想要的域名，一般 xyz 结尾这些或者是长名称的域名比较便宜
    - 接下来`点dns->管理区域->搜索自己刚买的域名（比如我买的是kair.xyz）`
    - 我们进入了一个设置dns 的界面，那么接下来根据 github 的[文档](https://help.github.com/cn/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site)我们要添加四个 ip 进去

    | 类型 | 名称 | 记录值 |
    | - | :-: | :-: |
    | A | @ | 185.199.108.153 |
    | A | @ | 185.199.109.153 |
    | A | @ | 185.199.110.153 |
    | A | @ | 185.199.111.153 |
    - 打开我们的github.io 的项目setting，在`Github Page`那一栏，`customer domain`填上你买的域名，比如我填的就是`kair.xyz`，回到项目，我们会发现，你的文件目录下，多了一个`CNAME`文件，里面写的就是你刚刚填的域名。
    - 打开[kair.xyz](https://kair.xyz)，看到我们刚刚打开的github.io 域名下的页面，我们就部署成功了。

整个部署过程就是这样，接下来就是如何维护自己的博客，并且修改主题样式了，我会额外再写一篇文章。