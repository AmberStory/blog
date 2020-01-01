---
title: Linux命令
date: 2019-07-30 18:58:31
tags: Linux
---
--------------------------
## curl
### -v
`-v`参数可以显示一次http通信的整个过程，包括端口连接和http request头信息。
`$ curl -v http://example`
### -b
`-b`参数可以用来指定cookie，也可以用过--cookie来指定cookie的值。
`$ curl -b "token=PS-1YW8oJ65LIc" http://example.com`

<!-- more -->

### 参考文档
1. https://itbilu.com/linux/man/4yZ9qH_7X.html
2. http://www.ruanyifeng.com/blog/2011/09/curl.html

## gzip
gzip file 可以用来压缩文件 
gzip -dv file 用来解压压缩后的文件 

### 参考文献
1. http://man.linuxde.net/gzip

## 编写一个sh文件test.sh并执行
```js
  #!/bin/bash
  echo "这是一个shell脚本"
```
`#!`是一个约定的标记，告诉系统这个脚本需要什么解释器来执行，即使用哪一种shell，可以省略不写。
执行test.sh的时候可以使用：
`sh test.sh` (注意当前目录必须是test.sh的根目录才可以直接使用文件名来执行，否则必须加上路径，比如：sh ./desktop/test.sh)

在执行shell命令的时候可以给.sh文件传参，比如`sh test.sh 这是个传参`。shell需要接收，第一个参数就用$1表示，以此类推。
```js
  #!/bin/bash
  echo "这是一个shell脚本 $1 $2"  # echo打印变量的话必须使用双引号
```
执行`sh test.sh 参数1 参数2`，执行结果`这是一个shell脚本 参数1 参数2`

