---
title: 两个action依次执行，第一个赋值，第二个立即取值
categories:
  - vue
  - vuex
tags:
  - vue
  - vuex
date: 2019-05-30 14:00:00
---

# 在mounted中的两个actions依次执行，第一个给state中的属性赋值，另外一个获取刚刚赋值的属性。commmit()是异步的，所以赋值操作其实是发生在第二个action获取属性之后。
<!-- more -->
```javascript
// vue 组件中： 
mounted() {
  this.$store.dispatch('setTest', 'test已经被赋值')
  this.$store.dispatch('getTest')
}
```
```javascript
// state.js中
export default {
  test: null // test初值为null
}
```
```javascript
// actions.js中
export default {
  setTest ({commit}, val) {
    commit('setTest', {val}) // 1
  },
  getTest ({commit, state}) {
    console.log('test:', state.test) // 2
  }
}
```
```javascript
// mutations.js中
export default {
  setTest (state, {val}) {
    state.test = val // 3
  }
}
```
控制台打印：`test: null`

执行顺序其实是： 1  2  3 ， 也就是说赋值操作在取值操作之后。


# 利用setTimeout验证commit是异步执行的
```javascript
// vue组件中
mounted() {
  this.$store.dispatch('setTest', 'test的赋值操作')
  setTimeout(() => {
    this.$store.dispatch('getTest')
  }, 2000) // 延迟两秒，在执行
}
```
控制台打印：`test: test已经被赋值`

此时的执行顺序为：1  3  2 ， 利用setTimeout使得`this.$store.dispatch('getTest')`异步且在 步骤2赋值操作 之后执行。

# 使用监听，让取值操作在所取值在变化之后进行
```javascript
// vue组件中
mounted() {
  this.$store.diapatch('setTest', 'test已经被赋值')
},
watch: {
  '$store.state.test' () {
    this.$store.dispatch('getTest')
  }
}
```
控制台打印：`test: test已经被赋值

当然，此时的执行顺序为：1  3  2