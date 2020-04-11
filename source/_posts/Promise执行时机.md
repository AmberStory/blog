---
title: Promise执行时机
date: 2020-04-11 17:03:33
tags: Promise JavaScript
---
我们分析一下下面这段promise代码的执行结果。
```js
p = new Promise((res, rej) => {
 console.log(1);
 setTimeout(()=>{console.log('5')},10);
 console.log(2);
 res('7');
});

p.then((res)=>{ console.log(res); console.log(6); }).then(()=>{console.log(8);});
console.log(3);

for(let i=0;i<90000000;i++){};  // 会执行50ms以上
console.log(4);

setTimeout(()=>{console.log('0')},0);
```
<!-- more -->
<b>执行结果打印：</b>1 2 3 4 7 6 8 5 0

#### 结果分析：
首先需要明确一个概念，异步操作中微任务一定在宏任务之前执行。这里promise属于微任务，setTimeout属于宏任务。

第一步：创建了一个promise实例p，promise实例化的时候会执行传入的这个函数，此时的执行顺序是同步的，所以会马上打印出“1 2”，由于setTimeout是异步的宏任务函数，所以暂时放到任务队列中不执行；

第二步：执行then函数，由于then函数是异步的微任务函数，所以会放到任务队列中去暂不执行；

第三步：继续执行主流程代码，打印“3 4”。接下来又是一个setTimeout函数，继续放到任务队列中不执行；

第四步：主流程的全部代码已经执行完毕，接下来我们需要看看任务队列中有哪些待执行的任务。由于微任务优先于宏任务执行，所以会先执行then函数，此时打印promise实例返回的结果“7”，接着再打印“6”；

第五步：接下来又是一个then的微任务函数，我们继续优先执行微任务，打印“8”；

第六步：此时微任务已经全部执行完毕，接下来执行宏任务。由于for循环执行的时间在50ms以上，而第一个setTimeout等待的时间是10ms，此时已经超过了第一个setTimeout的等待时间，所以会优先打印“5”，然后才打印第二个setTimeout函数的“0”。至此，所有的代码已经全部执行完毕。


## 参考文献
* http://www.ruanyifeng.com/blog/2014/10/event-loop.html

* https://juejin.im/post/5b73d7a6518825610072b42b