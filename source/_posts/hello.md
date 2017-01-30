---
title: 开博第一篇纪念
date: 2017-01-28 19:41:19
tags: [Hexo,博客]
---

# 前言
今天鸡年的第一天，说了好久想搭建自己的博客记录开发的日常，结果一拖再拖Orz（拖延症伤不起）。今天花了一下午终于搭建完成。

## 使用技术
1. Hexo
前端建站神器，这里不多说了。[Hexo文档][1]

2. 主题
当时在发现这个主题的时候是比较惊艳的，个人比较喜欢谷歌的这种风格，作者是位国人。很感谢无私的付出
[Material Theme][2]

3. Coding Pages
本来打算使用Github Pages，有时候墙的厉害。使用[Coding Pages][3]做为承载网站。
主要使用「用户 Pages」类型，在个人项目里新建一个名为 {user_name}.coding.me 的项目，然后将Hexo中生成的Public文件夹作为Git的主目录即可。


## 踩坑
1. _config.yml 是全站的控制文件，在配置的时候，有时候稍不注意将下面代码前的空格删掉了运行就会报错。
``` js
//source前面有一个空格
source_dir: source
```
2. 如果安装了新的主题，就需要到该主题下的_config.yml去配置。



## 结尾
新的一年，以此篇开始。Fighting~


  [1]: https://hexo.io/zh-cn/docs/themes.html
  [2]: https://material.viosey.com/
  [3]: https://coding.net/help/doc/pages/index.html