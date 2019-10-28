---
title: 微信sdk签名错误：invalid signature
date: 2019-10-25 19:23:39
tags: js-sdk
---
> 最近一段时间遇到个十分头疼的问题，微信小程序环境下进行微信签名的时候一直报错：invalid signature。在严格按照官方文档提供的签名的方法进行后，还是给出这个提示。另外获取js-sdk配置信息时，传入的url必须和当前页面的url保持一致，但不能保留url上携带的hash值。检查这条规则，也是符合的，url是通过location.href获取的，没有问题。后来又经过反复排查，反复试验，才找到问题真相：传入的url并不是进入页面时拿到的url，即使是通过location.href获取的。
#### location.href取到的不是页面真实url
<!-- more -->
在小程序中，如果传递的url地址中包含参数，但是这些参数又没有经过encodeURIComponent编码，那么通过location.href获取的url实际上是被浏览器自动encode之后的，拿着encode后的url获取js-sdk配置信息的时候，在ios下没有问题，安卓下就会出现invalid signature的错误提示。
<img src="./微信sdk签名错误：invalid signature/signature.png" />
如果你的微信签名一直提示“invalid signature”错误，其他配置又完全正常的情况下，请仔细检查url，80%是url的问题。
（我直接在浏览器端测试了一下，拿到的url参数是没有encode过的）。

* 附上一个整理的超级详细的文档：https://www.fengerzh.com/jssdk-invalid-signature/
* 微信开发文档：https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html