---
title: git关联远程仓库、上传本地文件
date: 2019-09-22 21:15:25
tags:
---
git remote add <远程仓库名> <远程仓库地址>
比如：git remote add origin https://github.com/AmberStory/AmberStory.github.io.git 
<远程仓库名>可以任意起，不过一般默认远程仓库名是origin

一个项目可以有多个远程仓库，只需要通过`git remote add [shortname] [url]`命令添加即可。

拉取远程仓库代码：
git pull <远程仓库名> <远程分支名>
例如：git pull dp-mp release_1.0.0

向远程提交代码：
git push <远程仓库名> <需要push的分支名>

## 上传本地文件
第一步：在需要上传至git的文件夹中执行git init，使这个项目变成git管理的仓库；
第二步：通过`git add . `和`git commit -m '提交'`命令，把文件提交至git临时仓库；
第三步：通过`git remote add origin <远程仓库地址>`将远程仓库与本地关联（如果管理错了，可以通过git remote rm origin删除）；
第四步：执行`git push --set-upstream origin master`将本地文件上次至git仓库。