---
title: nestjs连接mysql数据库遇到的问题
date: 2020-02-17 14:54:36
tags: nestjs typeorm mysql
---
> 最近在学习使用nestjs搭配typeorm连接mysql数据库，学习期间遇到过的问题记录下来和大家分享，希望能够对一些同学有所帮助。（基于mac笔记本）。
<!-- more -->
# 连接mysql过程出现错误
nestjs根模块（app.mudule.ts）配置的数据库连接参数如下：
```js
  TypeOrmModule.forRoot({
    "type": "mysql",  // 数据库类型
    "host": "localhost",
    "port": 3306, // mysql默认端口
    "username": "root", // 数据库用户名，默认是root
    "password": "password", // 数据库密码
    "database": "NestTest", // 你的数据库库名
    "entities": [Photo],  // 访问的数据库中的表名
    "synchronize": true // 确保每次运行应用程序时实体都将与数据库同步
  }),
```
我安装的mysql是v8.0.19版本，启动连接时出现错误 `Client does not support authentication protocol requested by server;`。
错误原因：mysql8 之前的版本中加密规则是 mysql_native_password，而在 mysql8 之后,加密规则是 caching_sha2_password。
解决方法：把 mysql 用户登录密码加密规则还原成 mysql_native_password。

# 修改mysql用户登录密码加密规则
1、查找mysql的安装位置进入bin文件夹，mac系统一般在`/usr/local/mysql/bin`目录下。

2、在`~/.bash_profile`文件中配置`export PATH=${PATH}:/usr/local/mysql/bin`，可以执行`open ~/.bash_profile`打开`.bash_profile`文件然后输入，如果没有这个文件，可以自己创建。这样配置是为了使终端认识`mysql`这条命令，配置完成之后关闭文件，在终端执行`source ~/.bash_profile`，使配置的命令生效。（如果使用的是zsh命令，可以将`source ~/.bash_profile`命令放在`~/.zshrc`文件中保存）。

3、在终端输入`mysql -u root -p`命令，然后输入mysql密码获取权限。

4、获取权限之后就可以进行密码规则配置了。
```js
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER; #修改加密规则
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的新密码'; #更新一下用户的密码
FLUSH PRIVILEGES; #刷新权限
```

# mysql查询命令（命令行后面的分号不能省）
* select user from mysql.user;  // 查询数据库用户名

* show databases;   // 查看所有数据库库名

* update user set user ='Amber' where user =’root’;   // 修改数据库用户名



# 参考文档
* https://www.cnblogs.com/mybilibili/p/11604558.html
