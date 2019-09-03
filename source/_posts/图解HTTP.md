---
title: 图解HTTP
date: 2019-07-30 18:30:10
tags: HTTP
---
<p>《图解HTTP》学习笔记</p>
---------------------------
### 请求状态码
301：永久重定向；<br>
302：临时重定向；<br>
304：未修改，表示客户机缓存的版本是最新的，客户机应该继续使用它。<br>
如果客户端缓存的文件有 Last Modified ，那么会在请求中会包含 If Modified Since，这个时间就是缓存文件的 Last Modified，服务端只要判断这个时间和当前请求的文件的修改时间就可以确定是返回 304 还是 200 。<br>
307：临时重定向；<br>
308：永久重定向。<br>
**注意：301和302这两种方法进行跳转的时候，都会将post请求改为get请求；307和308这两种方法进行重定向，禁止改变请求方法。**<br>

### location
该字段配合3xx的响应，将提供重定向的URI。
### catch-control
no-catch：强制进行服务端校验；<br>
no-store：不对资源进行缓存；<br>
max-age：失效前的最大期限；
## cookie和session
### Cookie
Cookie 技术通过在请求和响应报文中写入Cookie信息来控制客户端的状态。<br>
Cookie 会根据从服务器端发送的响应报文内的一个叫做Set-Cookie的首部字段信息，通知客户端保存Cookie。当下次客户端再往该服务器发送请求时，客户端会自动在请求报文中加入Cookie 值后发送出去。<br>

**Set-Cookie字段的属性**
* name
Cookie中属性的名称；
* value
属性对应的值；

* expires
Cookie的expires 属性指定浏览器可发送Cookie 的有效期。当省略expires 属性时，Cookie仅在浏览器关闭之前有效。

* path
Cookie 的path 属性可用于限制指定Cookie 的发送范围的文件目录。若不指定，则默认为文档所在文件目录。

* domain
通过Cookie的domain属性指定的域名可做到与结尾匹配一致。比如， 当指定http://example.com 后， 除http://example.com 以外，Example Domain或www2.example.com等都可以发送Cookie。因此，除了针对具体指定的多个域名发送Cookie 之外，不指定domain 属性显得更安全。

* secure
Cookie 的secure 属性用于限制Web 页面仅在HTTPS 安全连接时，才可以发送Cookie。当省略secure 属性时，不论HTTP 还是HTTPS，都会对Cookie 进行回收。

* HttpOnly
添加改属性之后，通常从Web页面内还可以对Cookie 进行读取操作。但使用JavaScript 的document.cookie 就无法读取附加HttpOnly 属性后的Cookie 的内容了。因此，也就无法在XSS 中利用JavaScript 劫持Cookie 了。

接下来，以client指代客户端，server指代服务端，说明一个 cookie 的整个作用机制：<br>
产生 cookie：client 第一次访问 server，server 在响应头中设置一个 cookie 返回给 client，cookie 的内容为要保存的数据；<br>
保存 cookie：client 在接收到 server 返回的 cookie 后，将 cookie 保存下来,并给cookie一个有效期，过了有效期，cookie 就会失效。<br>
传递 cookie：client 再次访问 server 将会在请求头中带上保存的 cookie，将 cookie 传递到 server；<br>
解析 cookie：server 得到 client 传递的 cookie 之后，会解析 cookie，然后将相应的信息返回给 client；<br>
在 cookie 没有失效之前，cookie 的使用都是围绕2,3,4三部分来进行的，第1步一般只需要进行一次。

### 通过Cookie来管理Session
Session 是存储在服务器端的，避免了在客户端Cookie中存储敏感数据。 Session 可以存储在HTTP服务器的内存中，也可以存在内存数据库（如redis）中， 对于重量级的应用甚至可以存储在数据库中。<br>

我们以存储在redis中的Session为例，还是考察如何验证用户登录状态的问题。

用户提交包含用户名和密码的表单，发送HTTP请求。<br>
服务器验证用户发来的用户名密码。<br>
如果正确则把当前用户名（通常是用户对象）存储到redis中，并生成它在redis中的ID。<br>

这个ID称为Session ID，通过Session ID可以从Redis中取出对应的用户对象， 敏感数据（比如authed=true）都存储在这个用户对象中。<br>

向客户端返回响应时，会在首部字段Set-Cookie内写入Session ID。<br>
客户端接收到从服务器端发来的Session ID 后，会将其作为Cookie 保存在本地。下次向服务器发送请求时，浏览器会自动发送Cookie，所以Session ID 也随之发送到服务器。<br>
服务器收到此后的HTTP请求后，发现Cookie中有SessionID，进行篡改验证。如果Session ID 被第三方盗走，对方就可以伪装成你的身份进行恶意操作了。另外，为减轻跨站脚本攻击（XSS）造成的损失，建议事先在Cookie内加上httponly 属性。<br>
如果通过了验证，根据该ID从Redis中取出对应的用户对象， 查看该对象的状态并继续执行业务逻辑。<br>

Session 代表着服务器和客户端一次会话的过程，当客户端关闭会话，或者 Session 超时失效时会话结束。

### 使用Cookie和Session
一般情况下，Cookie数据存在本地，Session数据存在服务器端。以添加购物车为例。<br>
未登录情况下：购物车数据存在Cookie中；<br>
登录情况下：购物车数据存在Session中。<br>
cookie：<br>
优点：数据保存在用户浏览器中，不占用服务端内存；用户体检效果好；代码实现简单；<br>
缺点：cookie的存储空间只有4k；更换设备时，购物车信息不能同步；cookie禁用，不提供保存。<br>
session：<br>
优点：数据能够持久化；实现了购物车同步；<br>
缺点：增加了数据库的压力，速度慢。<br>

## 拓展
### Cookie和token的区别<br>
Cookie：<br>
1. 验证有状态，验证记录或者会话需要一直在服务端和客户端保持；<br>
1. 登录验证之后session存在服务器端，session ID 存在客户端的Cookie中。一旦用户登出，则 session 在客户端和服务器端都被销毁。<br>
1. 能够处理单域和子域，但是遇到跨域的问题就会变得难以处理。<br>
1. cookie无法抵御csrf（跨站点请求伪造）攻击，因为在post请求的瞬间，cookie会被浏览器自动添加到请求头中，[参考文献1](https://zhuanlan.zhihu.com/p/27669892 "Title")。<br>

Token：<br>
1. 验证无状态，因为用户的状态在服务端的内存中是不存储的，所以这是一种无状态的认证机制。服务器不记录哪些用户登录了或者哪些 JWT 被发布了，而是每个请求都带上了服务器需要验证的token，token 存储在客户端，大多数通常在 local storage，也可以存在Cookie中。<br>
1. 一旦用户登出，token 在客户端被销毁，不需要经过服务器端。<br>
1. token 的 CORS 可以很好的处理跨域的问题。由于每次发送请求到后端，都需要检查 JWT，只要它们被验证通过就可以处理请求，这使得用从myapp.com获取的授权向myservice1.com和myservice2.com获取服务成为可能。<br>
1. token可以防止csrf攻击，客户端发送网络请求(一般不是登录请求)的时候，就会将token的值附带到参数中发送给服务器。如果是跨站点访问的网站，则无法拿到该token。<br>
1. token比较大，如果 token 中包含很多声明，那问题就会变得比较严重。


【参考文献1】：https://zhuanlan.zhihu.com/p/27669892
【参考文献2】：https://juejin.im/post/5a7c6c415188257a780da590
【参考文献3】：https://cloud.tencent.com/developer/article/1130355
【参考文献4】：https://www.jianshu.com/p/af8360b83a9f?spm=a2c4e.11153940.blogcont654868.10.7c1e280fzh7Wm5