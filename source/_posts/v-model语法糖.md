---
title: v-model语法糖
date: 2019-10-20 20:01:24
tags: Vue
---
> v-model 本质上是语法糖，它负责监听用户的输入事件以更新数据，并对一些极端场景进行一些特殊处理。
v-model 在内部为不同的输入元素使用不同的属性并抛出不同的事件：
text 和 textarea 元素使用 value 属性和 input 事件；
checkbox 和 radio 使用 checked 属性和 change 事件；
select 字段将 value 作为 prop 并将 change 作为事件。
<children v-model="msg"/>相当于在children组件内部，props中有一个名为value的属性，且children上有一个input事件，即
<children v-bind:value = "msg" v-on:input="msg=$event.target.value">

子组件：
<!-- more -->
```js
  <template>
    <input type="text" :value="value" @input="inputHandle($event.target.value)" />
  </template>
  <script>
    export default {
      props: { 
        value: '',
      }
      methods: {
        inputHandle(value) {
          return this.$emit("input", value);
        }
      }
    }
  </script>
```

### v-model修饰符
* .lazy
在默认情况下，v-model 在每次 input 事件触发后将输入框的值与数据进行同步 (除了输入法组合文字时)。你可以添加 lazy 修饰符，从而转变为使用 change 事件进行同步，而非input时进行更新，例如`<input v-model.lazy="msg">`。
* .number
如果想自动将用户的输入值转为数值类型，可以给 v-model 添加 number 修饰符，例如`<input v-model.number="age" type="number">`。
* .trim
如果要自动过滤用户输入的首尾空白字符，可以给 v-model 添加 trim 修饰符，例如`<input v-model.trim="msg">`