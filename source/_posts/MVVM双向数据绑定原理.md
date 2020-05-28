---
title: MVVM双向数据绑定原理
categories:
  - js
  - MVVM
tags:
  - js
  - MVVM
date: 2019-07-13 09:00:00
---
# MVVM双向数据绑定原理
+ angular
  脏值检测
+ vue
  数据劫持
<!-- more -->
### Object.defineProperty（ES5）
+ 定义方式作比较
```javascript
// 以往方式定义对象
var obj = {}
obj.school = 'lala'
// 使用Object.defineProperty定义对象
var obj2 = {}
Object.defineProperty(obj2, 'school', {
  value: 'lala'
})
```
1. 使用Object.defineProperty()定义的属性不能被删掉，当然也不允许使用“对象.属性=新值”的方式重新赋值
```javascript
{
  let obj = {
    school: 'lala'
  }
  let obj2 = {}
  let a = 'masia'
  Object.defineProperty(obj2, 'school', {
    value: a
  })
  delete obj.school
  delete obj2.school // 使用Object.defineProperty()定义的属性不能被删掉
  console.log(obj); // {}
  console.log(obj2); // {school: "masia"}
}
```
2. Object.defineProperty()中的configurable、writable、ennumberable属性
```javascript
{
  let a = {
    name: 'masia'
  }
  let obj2 = {}
  Object.defineProperty(obj2, 'school', {
    configurable: true, // 允许使用delete 对象.属性的方式去删除，默认为false
    writable: true, // 允许去写，默认为false
    enumerable: true, // 是否可枚举（使用for in遍历），默认为false
    value: a,
  })
  delete obj2.school;
  console.log(obj2); // {}
}
```
3. get和set不能与value、writable共存
如果你写了get和set，那就不要value writable属性了，不然会报错。当然，此时你写不写enumerable: true,当前属性都是可枚举的。

### 数据劫持
```javascript
class Vue {
  constructor (options = {}) {
    this.$options = options;
    this.$data = options.data;
    this.defineProperty(this.$data);
    for (let key in this.$data) {
      this[key] = this.$data[key];
    }
  }
  defineProperty (obj) {
    let seft = this;
    for (let property in obj) {
      let val = obj[property];
      Object.defineProperty(obj, property, {
        enumerable: true,
        get () { // 每次通过vue.$data.属性 获取值的时候，都会走这个方法
          return val; // 把val的值返回
        }, 
        set (newVal) { // 每次通过vue.$data.属性 = 'xxx'赋值的时候，都会走这个方法
          if (newVal !== val) {
            val = newVal; // 把新值保存在val中
          }
          
        }
      })
      if (typeof val == 'object') {
        this.defineProperty.call(seft, val);
      }
    }
  }
}
let vue = new Vue({
  el: '#app',
  data: {
    a: 'aaa',
    b: {
      bb: 'bb'
    },
    c: 'ccc'
  }
})
```
输出：
{% asset_img 数据劫持.png 数据劫持 %}

### 数据代理
+ 错误方式：浅复制方法
如果使用常规浅复制方式，发现并不适用this.$data中基本类型的属性。因为通过this.a='xx'的方式尝试去改变this.$data.a的值，发现并没有生效，因为this.a是string类型的，this.a = this.$data.a 也只是把后者的值赋给前者，所以并没有改变this.$data.a的值。而通过this.b.bb = 'xx'的方式去改变this.$data.b.bb的值，则生效了。因为this.b是引用类型啊。
```javascript
class Vue {
  constructor (options = {}) {
    this.$options = options;
    this.$data = options.data;
    this.defineProperty(this.$data);
    for (let key in this.$data) {
      this[key] = this.$data[key];
    }
  }
  defineProperty (obj) {
    let seft = this;
    for (let property in obj) {
      let val = obj[property];
      Object.defineProperty(obj, property, {
        enumerable: true, // 这里加不加可枚举属性,vue.$data的属性都是可枚举的
        get () {
          document.getElementById('div').innerHTML = val;
          return val;
        }, 
        set (newVal) {
          if (newVal !== val) {
            val = newVal;
          }
          
        }
      })
      if (typeof val == 'object') {
        this.defineProperty.call(seft, val);
      }
    }
  }
}
let vue = new Vue({
  el: '#app',
  data: {
    a: 'aaa',
    b: {
      bb: 'bb'
    },
    c: 'ccc'
  }
})
console.log(vue.$data.a); // aaa
console.log(vue.$data.b.bb); // bb
vue.a = '修改后的a'
vue.b.bb = '修改后的bb';
console.log(vue.$data.a); // aaa 发现没有修改this.$data.a的值
console.log(vue.$data.b.bb); // 修改后的bb
```
+ 正确方式：再来一次数据劫持
管你赋值的是什么类型，如果你对this.a赋值，我就先劫持掉，把你赋的值转赋给this.$data.a。如果你要获取this.a的值，那我就把this.$data.a的值拿来给你。
要注意的是：给this.$data.b.bb赋值也就是给this.$data.b赋值，通过"."方式操作对象的的过程其实也是一层一层寻找的过程。
```javascript
class Vue {
  constructor (options = {}) {
    this.$options = options;
    this.$data = options.data;
    this.defineProperty(this.$data);
    for (let key in this.$data) {
      Object.defineProperty(this, key, {
        enumerable: true, // 这里加不加可枚举属性,vue.$data的属性都是可枚举的
        get () {
          return this.$data[key];
        },
        set (newVal) {
          this.$data[key] = newVal;
        }
      })
    }
  }
  defineProperty (obj) {
    let seft = this;
    for (let property in obj) {
      let val = obj[property];
      Object.defineProperty(obj, property, {
        get () {
          document.getElementById('div').innerHTML = val;
          return val;
        }, 
        set (newVal) {
          if (newVal !== val) {
            val = newVal;
          }
          
        }
      })
      if (typeof val == 'object') {
        this.defineProperty.call(seft, val);
      }
    }
  }
}
let vue = new Vue({
  el: '#app',
  data: {
    a: 'aaa',
    b: {
      bb: 'bb'
    },
    c: 'ccc'
  }
})
console.log(vue.$data.a); // aaa
console.log(vue.$data.b.bb); // bb
vue.a = '修改后的a'
vue.b.bb = '修改后的bb';
console.log(vue.$data.a); // 修改后的a 发现可以修改基本类型的this.$data.a了
console.log(vue.$data.b.bb); // 修改后的bb
console.log(vue.a) // 修改后的a
console.log(vue.b.bb) // 修改后的bb
```
### 编译模板
Vue中的v-model、{{}}是如何编译的？
1. 通过el拿到里面的所有dom元素，遍历每个dom元素的节点（文本节点、属性节点、以及元素节点），并且把这些dom对象移入内存
  1. 文本节点
    1. 判断存在{{}}，并且解析
  2. 属性节点
    2. 判断存在"v-"，并且获得包含"v-"的指令列表，并且解析
  3. 元素节点
    1. 递归遍历子元素
2. 通过正则表达式解析“v-model、{{}}”这样的指令或者插值表达式模板\
  1. dom元素中属性中的指令
    1. 指令本身
      1. show
      2. model
      3. text
      4. html
      5. ...
    2. 指令的value
      1. 一般变量
      2. 表达式
  2. dom元素文本节点中的{{}}
   1. 一般变量
   2. 表达式
      1. {{flag? '真': '假'}}
3. 分离出模板中的js表达式，并且得到执行结果，比如取当前实例的属性值"shool.name"等
  1. 一般变量
  2. 表达式
4. 在内存中，使用拿到的属性值重新填充模板
5. 把填充好的模板再写入dom
总结： 当前实例vm，和需要操作的节点贯穿整个函数执行栈，因为要做的事情只有一件，就是找到节点对应的内存空间，并把this.$data中的相对应的值放进去。