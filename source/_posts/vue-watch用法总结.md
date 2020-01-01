---
title: vue的watch用法总结
date: 2019-11-03 19:27:47
tags: 
  - vue
  - watch
---
watch可用于监听vue实例数据的变化，下面详细总结一下watch的用法。
#### deep属性
当变异对象发生变化的时候，vue并检查不到这种变化，所以不会触发watch函数来执行相应操作，这个时候我们使用deep属性，进行深入监听（注意：数组的变化不需要使用该属性）。
```js
  data() {
    return {
      obj: {
        name: 'Amber',
        age: 18
      }
    }
  },
  watch: {
    obj: {
      handler() {
        console.log('obj changed');
      },
      deep: true
    }
  }
```
<!-- more -->
当obj的属性发生变化的时候，就会打印“obj changed”。但这种写法只要obj发生改变，就会触发，容易造成内容溢出，如果只是想单纯的监听某一个属性的变化，可以改写成路径监听的方式。
#### 路径监听
改写上述代码：
```js
  data() {
    return {
      obj: {
        name: 'Amber',
        age: 18
      }
    }
  },
  watch: {
    'obj.a': {
      handler() {
        console.log('obj changed');
      },
    }
  }
```
这样只有在obj.a属性发生变化的时候，才会触发watch函数。注意，路径得以字符串的形式表示。
#### immediate属性
如果希望watch在最初绑定的时候就执行，可以使用immediate属性。
```js
  data() {
    return {
      obj: {
        name: 'Amber',
        age: 18
      }
    }
  },
  watch: {
    obj: {
      handler() {
        console.log('obj changed');
      },
      immediate: true,
      deep: true
    }
  }
```

参考文献：
* https://cn.vuejs.org/v2/api/#vm-watch
* https://www.jianshu.com/p/fd68e178f71e