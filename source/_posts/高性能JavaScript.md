---
title: 高性能JavaScript
date: 2019-07-04 23:09:46
tags: JavaScript Web
---
<p>《高性能JavaScript》学习笔记</p>
------
## JavaScript加载与执行
> JavaScript代码的下载和执行经常会阻塞浏览器的其他进程，从而带来严重的用户体验问题，管理好JavaScript代码是提高web性能的关键。

**减少JavaScript对性能影响的方法：**
* <b>`</body>`闭合标签之前，将所有`<script>`标签放在页面底部。</b>
由于浏览器在解析到<body>标签之前，不会渲染页面的任何内容，如果把script脚本放在页面顶部加载，将会导致页面明显延迟。
<br>
* <b>合并脚本</b>
页面中`<script>`标签越少，加载越快。比如下载单个100KB的文件，比下载4个25KB的文件更快，因为每一次HTTP请求都会带来额外的性能开销。虽然很多浏览器已经支持并行下载，但并行下载的文件数量有限。
<!-- more -->
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
* querySelectorAll() 方法返回文档中匹配指定 CSS 选择器的所有元素，返回 NodeList 对象。此时的NodeList对象是静态的，所以返回的节点不会对应实时的文档结构。（注意：childNodes返回的NodeList对象是动态的,会随着文档结构的变化而变化）但是如果只是单纯的根据tag name来查找元素，建设使用getElementsByTagName()方法，因为getElementsByTagName()方法返回的HTML集合是动态的，动态集合比静态更快能够更快的被创建和返回。（参考：https://blog.csdn.net/renfufei/article/details/41088521）

思考题：找出页面中class="warning"或class="notice"的div元素。
考虑一下代码：
```js
let errs = document.querySelectAll('div.warning, div.notice');
```
如果不使用querySelectAll，要获得同样的结果，会更加复杂。

## 重绘与重排
> 当DOM的变化影响了元素的几何属性（宽和高）的时候，就会引起页面重绘和重排。浏览器会使渲染树中受到影响的部分失效，并重新构造渲染树，这个过程称为“重排”；完成重排之后，浏览器会重新绘制受影响的部分到屏幕中，这个过程称为“重绘”。重绘和重排是代价昂贵的操作，它们会导致web应用程序UI反应迟钝。

下面几种情况会引起页面重排：
* 添加或删除可见的DOM元素。
* 改变普通文档流中元素的位置。
* 改变元素尺寸（包括：外边距、内边距、边框宽度、厚度、高度等属性）。
* 内容改变，例如：文本改变、图片改变（尺寸不一样的图片）。
* 页面渲染器初始。
* 浏览器窗口改变。
* 出现滚动条（会引起整个页面的重排）。

为了提高web性能，大多数浏览器会通过队列化修改并批量执行来优化重排过程，但是修改样式的过程中如果用户获取了布局信息，就会导致浏览器立即执行队列中的任务，即无法等到批量执行，然后触发重排以返回正确值，例如以下代码：
```js
let computed = window.getComputedStyle('document.body', '');
let style = document.body.style;
let tem = '';
style.color = 'red';
tem = computed.height;
style.color = 'green';
tem = computed.width;
style.color = 'yellow';
tem = computed.background;
```
示例中，body元素改变了三次color值，且每改变一次都读取一个computed样式属性，虽然读取的属性都与改变的样式无关，但是浏览器却需要刷新渲染队列和重排，因为computed样式属性被请求了。
一个更高效的方法是，不要在布局信息改变的时候查询它，以上代码改写如下：
```js
let computed = window.getComputedStyle('document.body', '');
let style = document.body.style;
let tem = '';
style.color = 'red';
style.color = 'green';
style.color = 'yellow';
tem = computed.height;
tem = computed.width;
tem = computed.background;
```
#### 批量处理样式
* 通过cssText批量设置属性
```js
let el = document.getElementById('app');
app.style.cssText = 'height: 100px; width: 200px; color: green;';
```
  cssText属性可以合并所有的样式改变一次处理，这样只会修改一次DOM。但是cssText属性会覆盖已有的样式信息，如果想要保留已有样式，可以把需要变更的样式附加在cssText字符串后面，上面的代码改写如下：
```js
  let el = document.getElementById('app');
  app.style.cssText += '; height: 100px; width: 200px; color: green;';
```
  参考文档：https://cloud.tencent.com/developer/article/1057545
  <br>
* 通过改变DOM中class的值来改变样式

#### 批量修改DOM
当需要对DOM元素进行一系列操作时，可以通过以下步骤减少重排和重绘的次数：
1. 使元素脱离文档流；
2. 对其应用多重改变；
3. 把元素带回文档中。
<br>

下面有三种基本方法可以使用。
1. 隐藏元素，应用修改，然后重新显示；
```js
let el = document.getElememtById('app');
el.style.display = 'none';
···（执行修改DOM的操作，比如追加元素）
el.style.display = 'block';
```
2. 使用文档片段；
```js
let fragment = document.createDocumentFragment();
···（执行修改DOM的操作，比如追加元素）
document.getElementById('app').appendChild(fragment);
```
  因为文档片段存在于内存中，并不在DOM树中，所以将子元素插入到文档片段时不会引起页面回流（对元素位置和几何上的计算）。因此，使用文档片段通常会带来更好的性能。
  <br>
3. 将原始元素拷贝到一个脱离文档的节点中，修改副本，完成后在替换成原始元素。

## 算法和流程控制
> 改善循环性能的最佳方式是减少每次迭代的运算量和减少循环迭代次数。另外需要注意，避免使用for-in循环，因为该循环会同时搜索实例和原型属性，产生更多开销。除非需要遍历一个属性数量未知的对象。
#### 减少迭代工作量
* 比如遍历数组的时候，不要将数组的长度计算放在for循环中，可以将数组的长度作为局部变量存储起来，这样只用对数组长度进行一次属性查找。
<br>
* 通过颠倒数组的顺序来提高循环性能，思路以下代码：
```js
for(let i=0, len = arr.length; i<len; i++) {
  // 其他操作
}
```
  这样每次循环的时候需比较：1、len的值是否小于数组长度；2、查看控制条件的结果是否为true（i<len === true）。如果将以上代码改写如下：
  ```js
  for(len = arr.length; i--;) {
    // 其他操作
  }
```
  这样每次循环的时候只需判断控制条件的结果是否为true。随着迭代次数的增多，性能提升的趋势会更趋明显。

#### 减少迭代次数
即使在循环体内执行最简洁的代码，累计迭代上千次，运行速度也会慢下来，减少迭代次数能获得显著的性能提升。“达夫设备（Duff's Device）”是一种广为人知的限制循环迭代次数的模式。
```js
function useDuffDevice(arr){
  var len=arr.length, j=len%8;
  while(j){
      process(arr[j--]);
  }

  j=Math.floor(len/8);
  while(j--){
    process(arr[j--]);
    process(arr[j--]);
    process(arr[j--]);
    process(arr[j--]);
    process(arr[j--]);
    process(arr[j--]);
    process(arr[j--]);
    process(arr[j--]);
  }
}
```

#### 基于函数的迭代
比如forEach是一个便利的迭代方法，但它比基于循环的迭代要慢，对每个数组调用外部方法所带来的开销是速度慢的主要原因，因此在运行速度严格时，尽量避免使用基于函数的迭代方法。

#### 优化if-else
* 大概率的条件放前面判断。
* 使用二分法的方式判断条件。

#### Memoization优化技术
Memoization是一种避免重复工作的方法，它缓存前一个计算的结果供后续使用，避免了重复计算，在递归算法中十分有用。
下面是一段利用Memoization技术实现阶乘算法的示例：
```js
function memfactorial(n) {
    if(!memfactorial.catch) {
        memfactorial.catch = {
            "0": 1,
            "1": 1
        }
    }
    if(!memfactorial.catch.hasOwnProperty(n)) {
        memfactorial.catch[n] = n * memfactorial(n-1);
    }
    return memfactorial.catch[n];
}
```
解析：在计算一个阶乘之前先检查这个缓存对象是否已经存在相应的计算结果，没有的话则认为是第一次计算，计算完成之后，结果被存储在缓存中供以后使用。比如计算完`memfactorial(5)`之后，再计算`memfactorial(4)`就可以直接从缓存中取，不需要重新计算。

## 字符串和正则表达式
#### 字符串
需要大量循环叠加字符串的时候，可以利用数组项合并的方法，比如：
```js
let str = 'string';
let arr = [];
let num = 5000;
while(num--) {
    arr[arr.length] = str;
}
let newStr = arr.join('');
```

#### 正则表达式