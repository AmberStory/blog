---
title: promise踩坑记
date: 2019-09-02 08:58:57
tags: ES6 promise
---


### 记得return
采用箭头函数的写法：
```js
let p = new Promise((res)=>{ res() });
p.then((res) => success(res));
```
then里面的回调函数此时没有加花括号，其实是以下写法的简写：
```js
let p = new Promise((res)=>{ res() });
p.then((res) => { return success(res); });
```
加上花括号之后一定要加return，否则如果success函数返回的是一个promise，外面的函数是感知不到的，比如以下示例：
```js
let p = new Promise((res)=>{ res() });
p.then((res) => { Promise.reject('fail') })
.then(success, fail);
function success() {
  console.log('success');
}
function fail() {
  console.log('fail');
}
```
此时第一个then已经抛reject了，但是由于没有return，第二个then感知不到，所以把第一个then的返回值当普通函数处理，此时会进入第二个then的success函数里，但是我们预期的是希望进到fail函数里。


### promise的return
思考以下两组代码：
```js
new Promise((resolve, reject) => {
  return resolve(1);
})
.then(r => {
  console.log(r);
	return Promise.resolve(3);
})
.then(r => {
	console.log(r);
});
```

```js
new Promise((resolve, reject) => {
  return resolve(1);
})
.then(r => {
  console.log(r);
	return 3;
})
.then(r => {
	console.log(r);
});
```
以上两组代码都打印出：1 3


### resolve和reject等价
实例：
```js
let p = new Promise((res)=>{ res() });
p.then(success, fail).catch(err => console.error(err));
function success() {
  console.log('success');
  throw new Error('出错了');
}
function fail() {
  console.log('fail');
}
```
此时在success函数里抛错之后，并不会进入fail函数，而是会被catch捕获，但是，如果第一个promise出错了呢
```js
let p = new Promise((res)=>{ throw new Error('promise就出错了') });
p.then(success, fail).catch(err => console.error(err));
function success() {
  console.log('success');
}
function fail() {
  console.log('fail');
}
```
