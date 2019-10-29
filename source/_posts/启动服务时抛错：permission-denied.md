---
title: 启动服务时抛错：permission denied
date: 2019-10-28 10:06:42
tags: node
---
**遇到的问题：**启动项目时总是报错：某个路径下文件没有访问权限，permission denied。
<img src="./启动服务时抛错：permission-denied/permissionDenied.png" />
**解决方案：**
1、在启动服务的命令前加上sudo命令，比如：sudo npm start；
2、将没有权限的这个目录权限改为777，即chmod 777；