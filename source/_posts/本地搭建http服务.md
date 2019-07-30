---
title: 本地搭建http服务遇到的问题
date: 2019-07-29 08:59:43
tags: node server http
---
1. 本地打开的html文件无法访问server层接口

<b>问题描述：</b>在本地发起一个node server层服务，并监听3000端口号，然后直接打开本地的html文件，访问server层接口的时候报错；

<b>原因分析：</b>浏览器无法支持通过file协议执行http请求。

<b>解决方案：</b>npm安装http-server包，将本地html文件通过http server的形式打开。
执行完`npm install http-server -g`后，在html文件目录下执行`hp -o -p 8888`，其中`-o`表示启动服务后打开浏览器，`-p`表示设置端口号。（注意：http服务和server层启动的服务不能使用同一个端口号）
npm包地址：https://www.npmjs.com/package/http-server

<br>
2. http-server无法跨域请求node-server层接口

<b>问题描述：</b>访问接口的时候浏览器报错。

<b>原因分析：</b>由于http server和node server不能使用同一个端口号，所以存在跨域请求的的问题，AJAX有同源使用的限制。（协议、域名、端口号三者都相同属于同源，否则属于跨域请求）。

<b>解决方案：</b>
1. 通过CORS（Cross-origin resource sharing）方式解决
在node层发起服务的头部信息里加上“Access-Control-Allow-Origin”: "*"或者“Access-Control-Allow-Origin”: “http://127.0.0.1:8080（html文件打开地址）”。
参考：http://www.ruanyifeng.com/blog/2016/04/cors.html
2. JSONP的方式解决
3. 阻止浏览器对跨域请求的拦截
在终端输入 open -n /Applications/Google\ Chrome.app/ --args --disable-web-security  --user-data-dir=/Users/hzr/MyChromeDevUserData/，其中--user-data-dir表示文件存放的根目录，只需要替换“MyChromeDevUserData/”前面的内容即可（注意这里使用的是mac，window可能不一样）。
