---
title: Vue父子组件通信方式总结
date: 2019-12-22 16:21:50
tags: Vue
---
## 方式一：自定义事件emit
** 官方文档：https://cn.vuejs.org/v2/guide/components-custom-events.html? **
```js
  // parent.vue
  <template>
    <div v-on:handleClick="alertAction($event, 'parentParams')">
      这是父组件
    </div>
    <child />
  </template>
  <script>
    methods: {
      // 第一个参数是子组件通过emit传递的内容，第二个参数是父组件自己需要传递的内容
      alertAction(e, parentParams) {
        alert(e); // handleClick
        alert(parentParams); // parentParams
      }
    }
  </script>

  // child.vue
  <template>
    <div v-on:click="clickAction">
      这是子组件
    </div>
  </template>
  <script>
    methods: {
      clickAction() {
        this.$emit('handleClick', '子组件');
      }
    }
  </scrip>
```
<!-- more -->

## 方式二：自定义事件v-model
```js
  // parent.vue
  <template>
    <div>这是父组件</div>
    <child v-model="showChild" />
  </template>
  <script>
    data() {
      return {
        showChild: true
      }
    }
  </script>

  // child.vue
  <template>
    <div v-on:click="close">
      这是子组件
    </div>
  </template>
  <script>
    props: {
      value: Boolean
    },
    methods: {
      close() {
        this.$emit('input', false);
      }
    }
  </scrip>
```
* v-model会自动绑定一个value属性和input事件，一般用来控制一个组件是否展示。

## 方式三：事件发射器this.\$emit和this.\$on
** 官方文档：https://cn.vuejs.org/v2/guide/migration.html#dispatch-%E5%92%8C-broadcast-%E6%9B%BF%E6%8D%A2 **
在事件中心管理组件间的通信，将在各处使用该事件中心，可以使用 \$emit, \$on, \$off 分别来分发、监听、取消监听事件。
```js
  // 事件管理中心
  let eventHub = new Vue();
```
```js
  // todos.vue
  <script>
    eventHub.$on('addItem', item);
  </script>

  // item.vue
  <script>
    eventHub.$emit('addItem', {text: '整理vue父子组件通信博客'});
  </script>
```

## 方式四：通过refs访问子组件实例
** 官方文档：https://cn.vuejs.org/v2/guide/components-edge-cases.html#%E8%AE%BF%E9%97%AE%E5%AD%90%E7%BB%84%E4%BB%B6%E5%AE%9E%E4%BE%8B%E6%88%96%E5%AD%90%E5%85%83%E7%B4%A0 **
```js
  // parent.vue
  <template>
    <child ref="childProp"></child>
  </template>
  <script>
    methods: {
      focus() {
        this.$refs.childProp.execFocus();
      }
    }
  </script>

  // child.vue
  <template>
    <input ref="input">
  </template>
  <script>
    methods: {
      execFocus() {
        this.$refs.input.focus();
      }
    }
  </scipt>
```
* 注意：\$refs 只会在组件渲染完成之后生效，并且它们不是响应式的。这仅作为一个用于直接操作子组件的“逃生舱”——你应该避免在模板或计算属性中访问 \$refs。

## 通过.sync实现prop的“双向绑定”
** 官方文档：https://cn.vuejs.org/v2/guide/components-custom-events.html#sync-%E4%BF%AE%E9%A5%B0%E7%AC%A6 **
```js
  // parent.vue
  <template>
    <child
    v-bind="doc.title"
    v-on:update:title="doc.title=$event"
    />
    // 等价于：<child v-bind:title.sync="doc.title" />
  </template>

  // child.vue
  <script>
    methods: {
      changeTitle() {
        this.$emit('update:title', newTitle);
      }
    }
  </script>
```
* 一般用于需要修改父组件传递给子组件参数的情况，也可以通过对象的方式给子组件传值，此时可以在子组件内直接修改父组件传递的对象属性值，并且父组件中的值也会相应发生变化。