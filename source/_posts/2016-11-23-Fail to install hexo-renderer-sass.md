---
title: 安装hexo-renderer-sass失败
date: 2016-11-23 15:39:25
categories:
- 技术
tags:
- hexo
---
在使用hexo做github博客的时候，找到一些很简洁的主题想要使用，其中有些主题需要安装hexo-render-sass来渲染页面布局，此时遇到了错误：
``` bash
build error! Stack Error: ‘c:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe failed with exit code:1......
```
<!-- more -->
在Google上找了一下，有人说是国内网络造成的问题，可以使用淘宝npm镜像解决。淘宝镜像的使用方法：通过config命令指定镜像
``` bash
$ npm install -g cnpm --registry=http://registry.npm.taobao.org
```
成为指定镜像后，使用cnpm install --save hexo-renderer-sass命令重新安装，即可成功。