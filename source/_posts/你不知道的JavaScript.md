---
title: 你不知道的JavaScript
date: 2019-07-04 23:15:51
tags:
 - JavaScript
---
<p>《你不知道的JavaScript》学习笔记</p>
---------

### 立即执行函数
`(function(){···})()` 或者`(function(){···}())`
立即执行函数只能包含···所代表的作用域，不会污染外部作用域。
### 匿名和具名函数
匿名函数写起来简洁，比如setTime()里我们常使用匿名函数，但是有一下几个缺点：
1. 匿名函数在站追踪中不会显示出有意义的函数名，使调试变得困难；
1. 如果没有函数名，匿名函数引用自身时只能使用过期的arguement.callee（递归中使用arguement.callee会获取到一个不同的this值，所以不推荐使用）；
1. 匿名函数省略了对代码的可读性。
所以，给函数表达式一个名称是最佳实践。
<!-- more -->
### 函数作用域
>属于这个函数的全部变量都可以在整个函数范围内使用。
```js
function foo(a){
  var b=1;  
    function bra() {
    // 这里面也可以使用b
    }
}
```

### 块级作用域
通常情况下JavaScript不存在块级作用域，像下面这行代码：  
```js
if(){
  var bar = 2;
  console.log(bra);  //2
}
  console.log(bra);  //2
```
在if的内部通过var定义的变量，在外边仍然可以使用。  

es6语法下的块级作用域：  
let关键字将变量绑定在任意作用域中（通常是{···}内部）
```js
if(){
  let bar = 2;
  console.log(bra);  //2
}
  console.log(bra);  //ReferenceError
```
### 变量提升
>引擎会在解释JavaScript代码之前首先对其进行编译，编译阶段中的一部分工作就是找到所有的声明，并用合适的作用域将他们关联起来。所以，包括变量在内的所有声明都会在代码被执行前首先被处理。  

思考以下代码：
```js
a = 2;  
var a;  
console.log(a);
```
打印：2
分析原因：
上述代码的处理顺序实际上是：
```js
var a;  
a = 2;  
console.log(a);
```
因为引擎会先对变量做预解析，然后再赋值。

再来看一组例子:
`foo(); function foo() { console.log(a); var a=1; }`
打印：undefined 
分析原因：
上述代码的处理顺序实际上是：
`function foo() { var a; console.log(a); a=1; } foo();`

foo函数声明被提升了，因此可以先调用，再声明。函数里变量a的声明虽然也被提升，但打印之前并没有被赋值，所以回报undefined。另外注意，函数表达式并不会被提升。

`foo(); var foo = function bar() { ··· }`
打印：TypeError 
分析原因：
上述代码的处理顺序实际上是：
`var foo; foo(); foo = function () {}` 
虽然foo已经被定义了，但是被没有赋值，foo的值相当于是undefined，当对undefined进行函数调用的时候，会导致非法操作，因此抛出TypeError。

> 注意：函数声明和变量声明都会被提升，但是函数会先被提升，然后才是变量。如果是重复声明的var，则会被忽略，但是如果是重复声明的函数，则后面的声明会覆盖前面的声明。

### undefined、ReferenceError、TypeError
**undefined:** 定义了某个变量，但是却没有赋值，会报undefined，它是JavaScript原始数据类型。
`var a; console.log(a);  //undefined`
**ReferenceError:** 当尝试引用一个未定义的变量时，会报改错。
`console.log(b);  //Uncaught ReferenceError: b is not defined`
**TypeError:** 当传入函数的操作数或参数的类型并非操作符或函数所预期的类型时，将抛出一个 TypeError 类型错误。
`var fn; fn(); //Uncaught TypeError: fn is not a function`

### 作用域闭包
什么是闭包?
函数发生在本身的词法作用域以外执行，就产生了闭包。

```js
function foo() { 
  var a=2;
  function bar() { 
    console.log(a);
  } 
  return bar;
} 
var baz = foo();
baz(); //2
```

## 关于this
> this是在运行时进行绑定的，不是在编写时绑定的，它的上下文取决于函数调用时的各种条件。this的绑定和函数的声明位置没有任何关系，只取决于函数的调用方式。

更正：书中this一章中的1.2.2节中，有一段代码经过验证，和书中给的结果不太一致，代码如下：
```js
function foo() {
  var a =2; 
  this.bar();
} 
function bar() { 
  console.log(this.a); 
} 
foo();
```
打印结果：undefined 
书中给的结果是ReferenceError: a is not defined 
实际上这里的this指向的是window，如果window上面没有某个对象的话，都会输出undefined，所以这里的this.a也会输出undefined。

### this全面解析
> this的绑定规则分为四种：默认绑定、隐式绑定、显示绑定、new绑定 

**默认绑定** 
> 不带任何修饰的函数引用进行调用的。

例举1：
`var a = 1; function foo() {console.log(a);} foo()`
打印：1

例举2：
`var name = "The Window";`
　　`var object = {`
　　　　`name : "My Object",`
　　　　`getNameFunc : function(){`
　　　　　　`return function(){`
　　　　　　　　`return this.name;`
　　　　　　`};` 
　　　　`}`
　　`};`
`console.log(object.getNameFunc()());`
打印：The Window 
分析原因：
object.getNameFunc()执行之后实际上返回一个匿名函数，第二个括号表示执行这个匿名函数。返回的函数会先在自己的作用域内查找name变量，如果没有找到，则会到window对象上查找。 

**隐式绑定**
> 当函数引用有上下文对象时，或者说被某个对象拥有或者包含，这个时候会产生隐式引用。

例举1：
`var a=1; function foo() {console.log(this.a)} var obj = {a:2, foo: foo}; obj.foo();` 
打印：2 
分析原因：当调用foo函数的时候，它的前面加上了对obj的引用。

例举2：
```js
var arr = [1, 2, 3, 4, 5];
Array.prototype.myForEach = function(){
  var len = this.length;
  console.log(this);
}
arr.myForEach();
```
打印：[1, 2, 3, 4, 5];
分析原因：myForEach方法被挂载在arr这个数组上，所以其内部的this指向arr数组。

注意一种情况：隐式丢失。
`var a=1; function foo() {console.log(this.a)} var obj = {a:2, foo: foo}; var fn = obj.foo;`
`fn();`
打印：1 
分析原因：fn实际上是对foo函数的直接引用，此时fn实际上是一个不带任何修饰的函数调用，应用了默认绑定。
注意，参数传递其实也隐藏了是一种赋值，会进行默认绑定。

**显示绑定** 
> 如果想在某个对象上强制调用某个函数，那么称之为显示绑定。call，apply可以提供显示绑定方法。

例举1：
`function foo() {console.log(this.a)} obj={a:1}; foo.call(obj);` 
打印：1 

**new绑定** 
> 使用new来调用函数时，可以影响函数this的绑定行为。

例举1：
```js
function foo(a){
  this.a = a;
}
var bar = new foo(2);
console.log(bar.a);
```
打印：2  

**几种绑定关系的优先级** 
* 显示绑定 > 隐式绑定 
* new绑定 > 隐式绑定 
* new绑定 > 显示绑定 

**es6箭头函数中this的指向** 
> 箭头函数的this是在定义函数时绑定的，不是在执行过程中绑定的。简单的说，函数在定义时，this就继承了定义函数的对象。一旦确定了箭头函数的this指向，就无法进行修改。

例举1：
```js
function foo() { 
  return (a) => {
    console.log(this.a);
  }
}
var obj1 = { a:2 };
var obj2 = { a:3 };
var bar = foo.call(obj1);
bar.call(obj2);
```
打印：2 

例举2：
```js
function foo() {
  setTimeout(()=>{
    console.log(this.a)
  }, 1000)
}
var obj = {a:1};
var a = 2;
foo.call(obj);
```
打印：1 

如果将例举2中setTimeOut中的回调函数从箭头函数改为普通函数呢？
```js
function foo() {
  setTimeout(function(){
    console.log(this.a)
  }, 1000)
}
var obj = {a:1};
var a = 2;
foo.call(obj);
```
打印：2 
分析原因：异步函数的回调函数如果是普通函数（非箭头函数），则函数的this指向window。
