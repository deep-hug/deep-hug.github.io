---
title: yarn 安装node-sass 失败处理方案
author: DeepHug
index_img: /img/wallhaven-e7zogk.jpg
category: 技术
tags:
  - Fluid
excerpt: 不要再去改yarn的源了！！！不要再去改yarn的源了！！！不要再去改yarn的源了！！！
date: 2023-01-03 22:08:08
---

[原文档链接](https://valor.blog.csdn.net/article/details/104044201?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2~default~CTRLIST~Rate-1-104044201-blog-102843257.pc_relevant_3mothn_strategy_recovery&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2~default~CTRLIST~Rate-1-104044201-blog-102843257.pc_relevant_3mothn_strategy_recovery&utm_relevant_index=1)

不要再去改yarn的源了！！！不要再去改yarn的源了！！！不要再去改yarn的源了！！！

重要的事情说三遍。

解决方案：

修改node-sass 源。

1. 修改源

> yarn config set sass_binary_site https://npm.taobao.org/mirrors/node-sass/ -g 

2. 安装

yarn install 
 

最后 如果需要的话，修改yarn的源 在这里：

> yarn config set registry https://registry.npm.taobao.org -g