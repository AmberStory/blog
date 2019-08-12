---
title: ES6语法
date: 2019-07-30 18:44:10
tags: ES6 JavaScript
---

-------------------------

## async await与forEach的爱恨情仇
### async函数和await关键字
async和await是由ES7提供的函数和关键字，async用于声明一个异步函数，await强制其他异步代码等待promise请求结果返回（相当于then的用法），但不能放在普通函数中使用，只能用在async函数里面使用。<br>
`async function myFunction() {
try {
  await requestAPromise();
} catch(err) {
  console.log(err);}
}`
await最好放在try中，如果请求出错可以就进行catch捕获。

`async function(db) dbFun{
  docs = [{},{},{}];
  docs.forEach(function(doc){
    await db.post(doc);
  })
}`
报错，因为await用在了普通函数中。

### for循环中使用await
将上述例子修改如下：
`async function(db) dbFun{
  docs = [{},{},{}];
  docs.forEach(async function(doc){
    await db.post(doc);
  })
}`

此时三个db.post(doc)是并发完成的，而不是继发执行。
分享原因：
forEach内部实现方法：
`var arr = [1, 2, 3, 4, 5]
Array.prototype.forEach = function(fn){
  var len = this.length;
  for(var i = 0; i < len; i ++){
    //将元素传给回调函数
    fn(this[i],i);
  }
}
arr.forEach(function (ele, index){
  console.log(ele, index);
})`

从forEach的内部实现原理来看，实际上是在for循环中调用forEach中传入的回调函数，上述代码经过forEach的转换之后，实际上变成了
`async function(db) dbFun{
  docs = [{},{},{}];
  async function(doc){
    await db.post(doc);
  }
  async function(doc){
    await db.post(doc);
  }
  async function(doc){
    await db.post(doc);
  }
}`

如果希望上述代码继发执行，写法如下：
`async function(db) dbFun{
  docs = [{},{},{}];
  for(let doc in docs){
    await db.post(doc);
  })
}`

如果希望多个请求并发执行，可以使用ES6的Promise.all方法。
`async function(db) dbFun{
  docs = [{},{},{}];
  return Promise.all(docs.map(function(doc){
    return db.post(doc)
  })).then(function(result){
  console.log(result);
})`

## 对象拷贝
#### 数组拷贝
* 方法一
```js
arr1 = [1, 2];
arr2 = [3, 4];
arr3 = [...arr1, ...arr2];  //1,2,3,4
```
arr3改变不会影响arr1，arr2

* 方法二
```js
arr1 = [1, 2];
arr2 = [3, 4];
arr3 = arr1.concat(arr2);  //1,2,3,4
```
arr3改变不会影响arr1，arr2

#### 对象拷贝
* 方法一
```js
obj1 = {a:1, b:2};
obj2 = {c:3, d:4};
obj3 = {...obj1, ...obj2};  //1,2,3,4
```

* 方法二
```js
obj1 = {a:1, b:2};
obj2 = {c:3, d:4};
obj3 = Object.assign({}, obj1, obj2);
```
注意assign实现的是浅拷贝，深拷贝、浅拷贝参考文档：
https://juejin.im/post/59ac1c4ef265da248e75892b#heading-2