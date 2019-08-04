---
title: SVG实现倒计时动效
date: 2019-08-04 23:32:00
tags: SVG
---
---------------------
利用strke-dasharray创建虚线，结合stroke-dashoffset控制虚线开始的位置，实现倒计时条滑动的效果。
strke-dasharray接收两个参数，第一个表示虚线的长度，第二个表示虚线之间的间隔，当两个值相等的时候，只需要给一个参数。
stroke-dashoffset指定线段的起始位置偏移量。整数从起始位置向前偏移，负数向后偏移。

利用requestAnimationFrame计算剩余时间：
```js
window.requestAnimationFrame(this.timeFlow);
timeFlow(time) {
  if (!time) return;
  if (!this.record) this.record = time;

  const flowTime = time - this.record;

  this.record = time;
  const remain = this.remainTime - Math.floor(flowTime);
  if (remain > 0) {
    this.remainTime = remain;
    this.raf = window.requestAnimationFrame(this.timeFlow);
  } else {
    this.remainTime = 0;
  }
}
```