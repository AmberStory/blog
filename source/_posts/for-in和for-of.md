---
title: for in和for of
date: 2019-10-15 22:58:21
tags: 
  - JavaScript
  - es5
  - es6
---
### for...in 
for...in是es5标准，以任意顺序遍历一个对象的除Symbol以外的可枚举属性（循环将遍历对象本身的所有可枚举属性，以及对象从其构造函数原型中继承的属性（更接近原型链中对象的属性覆盖原型属性）)。如果只要考虑对象本身的属性，而不是它的原型，那么使用 getOwnPropertyNames() 或执行 hasOwnProperty() 来确定某属性是否是对象本身的属性（也能使用propertyIsEnumerable）。
<!-- more -->
#### 示例
```js
  var car = {a: 1, b: 2, c:3};
  function ColorCar() {
    this.color = 'red';
  }
  ColorCar.prototype = car;
  var obj = new ColorCar();
  for(var prop in obj) {
    if(obj.hasOwnProperty(prop)) {
      console.log('own prop:', prop); // own prop: color
      continue;
    }
    console.log('extend prop:', prop); // extend prop:a extend prop:b extend prop:c
  }
```
提示：for...in不应该用于迭代一个 Array，因为迭代的顺序是依赖于执行环境的，所以数组遍历不一定按次序访问元素。

### for...of
for...of 是es6标准，遍历可迭代对象（包括 Array，Map，Set，String，TypedArray，arguments 对象等等）要迭代的数据。
#### 示例
```js
  arr = [3,5,7];
  for (item of arr) {
    console.log('item:', item); // item: 3 item:5 item:7
  }
```