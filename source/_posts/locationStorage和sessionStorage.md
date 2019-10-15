---
title: locationStorage和sessionStorage
date: 2019-10-15 22:38:53
tags: 
  - JavaScript
  - storage
---
locationStorage和sessionStorage都属于本地存储，存储的数据都保存在浏览器会话中，二者的区别在于：
locationStorage存储的数据可以长期保留，除非强制清除；而sessionStorage存储的数据仅存在于页面会话期间，即当页面关闭之后，sessionStorage中的数据就会被清除。
<!-- more -->
#### 用法
// 保存数据到 localStorage
localStorage.setItem('key', 'value');

// 从 localStorage 删除保存的数据
localStorage.removeItem('key', 'value');

// 保存数据到 sessionStorage
sessionStorage.setItem('key', 'value');

// 从 sessionStorage 获取数据
let data = sessionStorage.getItem('key');

// 从 sessionStorage 删除保存的数据
sessionStorage.removeItem('key');

// 从 sessionStorage 删除所有保存的数据
sessionStorage.clear();