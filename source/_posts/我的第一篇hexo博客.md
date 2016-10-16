---
title: 我的第一篇hexo博客
date: 2016-05-23 19:41:00
tags: hexo
categories: 博客
keywords: hexo博客搭建
description: hexo博客搭建
---
hexo博客搭建
-------
>1.首先配置环境，需要用到` git`，`nodejs`，这里google上都有教程
>2.安装hexo
    执行 *npm install hexo-cli -g*
    可能报错如下：

```shell
    npm ERR! Please try running this command again root/Administratot
```
需要root权限
执行
    ● *sudo npm install hexo-cli -g*
    ● *hexo init blog*
    ● *cd blog*
    ● *npm install*
    ● *hexo server*
然后访问`http://localhost:4000`即可访问本地博客，哎呦还不错哦

git迁移
------
现在我们的博客都只是可以在本地访问，但做好了博客肯定要放出去装下逼啊
>1.创建git仓库什么的就不说了，先来看看配置,打开根目录下的_config.yml

```javascript
    deploy:
      type: git
      repository: git@github.com:1364137942/1364137942.github.io.git
      branch: master
```
>2.然后执行

   ● *hexo clean*
   ● *hexo generate*
   ● *hexo deploy*
然后问题又来了
```shell
    ERROR Deployer not found: git
```
google了很久，终于找到了解决方法
执行下面一条代码后再执行
   ● *npm install hexo-deployer-git --save*

>3.大功告成，基本的博客总算做好了，然后还改了一下主题，把code的样式改了一下，然后写这篇博文的时候发现了 **代码不能高亮** 了,心累啊

找了一下原因，把我之前改的关于`code`的`css`还原了，还是不行，于是又找了强大的google
解决方法如下
```shell
    vi /themes/pacman/source/css/_base/variable.styl

    // 注释最后一行
    // highlight = hexo-config("highlight.enable")

    hexo clean
    hexo g
    hexo s
```
神奇的代码高亮回来了




