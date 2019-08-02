---
title: 高性能JavaScript
date: 2019-07-04 23:09:46
tags: JavaScript Web
---
------
## JavaScript加载与执行
> JavaScript代码的下载和执行经常会阻塞浏览器的其他进程，从而带来严重的用户体验问题，管理好JavaScript代码是提高web性能的关键。

**减少JavaScript对性能影响的方法：**
* <b>`</body>`闭合标签之前，将所有`<script>`标签放在页面底部。</b>
由于浏览器在解析到<body>标签之前，不会渲染页面的任何内容，如果把script脚本放在页面顶部加载，将会导致页面明显延迟。
<br>
* <b>合并脚本</b>
页面中`<script>`标签越少，加载越快。比如下载单个100KB的文件，比下载4个25KB的文件更快，因为每一次HTTP请求都会带来额外的性能开销。虽然很多浏览器已经支持并行下载，但并行下载的文件数量有限。
<br>
* <b>使用无阻塞方式下载JavaScript</b>
    1. 带有defer、async属性的script，允许js资源与页面中的其他资源并行下载，不会阻塞浏览器其他进程。但是带有async属性的script，会在js脚本下载完成之后马上执行，此时脚本的执行会阻塞页面进程；而带有defer属性的script下载完js脚本后，会等到页面所有元素解析完成之后，onload事件处理器执行之前执行js脚本，不会阻塞页面进程。
    （注意：defer属性仅当src属性声明时才生效-------------测一测）
    2. 动态创建脚本元素
    动态创建`<script>`标签的方法，无论何时启动下载，文件的下载和执行都不会阻塞页面其他进程，但是动态加载的文件会按照从服务器返回的顺序下载和执行脚本，也就是说不一定能按照指定的顺序执行。因为浏览器对动态插入的script默认设置了async（各个浏览器可能不同），而async的执行是没有顺序的，所以我们把script标签的async属性改成false就可以了。
    （参考：http://echizen.github.io/tech/2017/04-22-script-exec）
    3. 使用XML对象下载JavaScript代码。
    
## 数据存储
> JavaScript中，数据存储位置会对代码整体性能产生重大影响。数据存储共有4种方式：字面量、变量、数组、对象。

**存储方式解析：**
* <b>访问字面量和局部变量的速度比访问数组和对象的速度快。</b>
<br>
* <b>访问局部变量比访问全局变量更快</b><br>
局部变量存在作用域链的起始位置，代码执行过程中，会先查找局部变量，没有找到，就会根据作用域链依次往上查找，因此变量在作用域链中的位置越深，访问的时间越长。全部变量存在执行环境作用域的最末端，因此最远。
如果某个跨作用域的值或全局变量被引用一次以上，我们可以将这个值存储在当前执行环境的局部变量里，比如：
```js
function setStyle() {
    let doc = document; // 将document对象存在局部变量doc里
    let div = doc.getElementById('app');
    let span = doc.getElementsByTagName('text');
    let len = span.length;

    div.onClick = function() {
        //
    }
    for(let i=0; i<len; i++) {
        update(span[i]);
    }
}
```

* <b>访问局部变量比访问全局变量更快</b><br>
尽量减少使用嵌套成员，嵌套的越少，越影响性能，比如执行window.location.href总比location.href要慢。我们可以将对象成员、数组元素等保存在局部变量中使用，例如：
```js
function getAttribute() {
    let doc = document;
    let name = doc.getElementById('name');
    let age = doc.getElementById('age');
    let sex = doc.getElementById('sex');
}
```
    改写如下：
    ```js
    function getAttribute() {
        let doc = document;
        let getId = doc.getElementById; // 将doc.getElementById对象存储在局部变量中，减少读取对象成员的次数
        let name = getId('name');
        let age = getId('age');
        let sex = getId('sex');
    }
    ```
## DOM编程
> 有一个比喻，ECMAScript和DOM好比两座岛屿，他们之间用收费桥梁连接，ECMAScript每次访问DOM，都要途径这座桥，并缴纳过桥费，访问DOM次数越多，费用就越高。所以减少DOM操作次数，能有效提高页面的响应速度。

#### 操作DOM修改
思考以下代码：
```js
function innerHTMLLoop() {
    for(let count=0; count<1000; count++) {
        document.getElementById('app').innerHTML = document.getElementById('app').innerHTML + '1';
    }
}
```
每循环一次，该DOM元素都会被访问两次，十分影响性能，换一下方式，用局部变量存储的方式改下，如下：
```js
function innerHTMLLoop() {
    let content = '';
    for(let count=0; count<1000; count++) {
        content += '1';
    }
    document.getElementById('app').innerHTML = document.getElementById('app').innerHTML + content;
}
```
改写后代码的运行速度能明显提升。
#### HTML集合
HTML集合是包含DOM节点引用的类数组对象，它不是真正的数值，但是含有数组中的length属性，还能以数字索引的方式访问列表中的元素。
思考以下代码：
```js
let allDivs = document.getElementsByTagName('div');
for (let i=0; i<allDivs.length; i++) {
  document.body.appendChild(document.createElement('div'));
}
```
分析：
根据前面“数据存储”章节的讲解，上面这段代码首先访问了三次`document`这个全局对象，可以优化为一个局部变量。
然后我们来看循环中的`allDivs.length`。<b>实际上HTML集合一直与文档保持着连接，即能实时获取到文档的更新，每次文档一更新，HTML集合也会跟随改变。</b>这段代码看上去是把div的数量翻倍，每循环一次，body里添加一个新的div，但实际上这是一个死循环。因为`allDivs`这个集合事实与文档保持着连接，当文档中div的数量增加时，`allDivs.length`实际上也会一起增加，循环永远无法退出。

上面的代码可以改写如下：
```js
let doc = document;     // 将全部对象保存在局部变量里
let allDivs = doc.getElementsByTagName('div');
let len = allDivs.length;   // 把集合的长度保存在局部变量里
for (let i=0; i<len; i++) {
    doc.body.appendChild(doc.createElement('div'));
}
```

另外，变量数组比遍历集合快，可以将集合元素拷贝到数组中再进行遍历。

#### 快速选取DOM
DOM提供了多种方法来读取文档的特定结构，我们最好为特定操作选择高效的api。比如选择子节点，childNodes不会区分元素节点和其他类型的节点，比如注释和文本节点等（HTML中的空白、换行其实是文本节点），如果我们只想访问元素节点，那么用children会更高效，不需要做额外的过滤处理。

另一方面，使用CSS选择器也是定位节点的一种方式。推荐两个高效的原生DOM方法：querySelect()和querySelecterAll()。
* querySelect() 方法返回文档中匹配指定 CSS 选择器的第一个元素。
* querySelectorAll() 方法返回文档中匹配指定 CSS 选择器的所有元素，返回 NodeList 对象。此时的NodeList对象是静态的，所以返回的节点不会对应实时的文档结构。（注意：childNodes返回的NodeList对象是动态的,会随着文档结构的变化而变化）

思考题：找出页面中class="warning"或class="notice"的div元素。
考虑一下代码：
```js
let errs = document.querySelectAll('div.warning, div.notice');
```
如果不使用querySelectAll，要获得同样的结果，会更加复杂。

## 重绘与重排