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
