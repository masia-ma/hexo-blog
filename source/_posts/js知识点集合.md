---
title: js知识点集合
categories: 
  - js
tags: 
 - js
date: 2019-06-01
---
#### 检测数组的几种方式：
<!-- more -->
1. Array.isArray(); // es5
2. toString.call([]) // [Object Array] 这只是一个字符串
3. var arr = []
  console.log(arr.contructor.name) // Array
4. console.log([] instanceof Array) // 检测某个对象是不是属于某个类或者构造函数

拓展：
 toString.call(/[0-9]/) // [Object RegExp] // 判断正则表达式对象
```javascript
// 编写一个函数来判断输入对象的类型
function check (obj) {
  return toString.call(obj).split(' ').pop().slice(0, -1)
}
```

#### javascript 中实现继承的几种方式
+ 原型链继承
+ 借用构造函数继承
```javascript
function Animal () {
  this.name = 'masia',
  this.getName = function () {
    console.log(this.name)
  }
}
function Cat () {
  Animal.call(this) 
  this.age = 18
}
var cat = new Cat()
console.log(cat)
console.log(cat.name) // 'masia'
cat.getName() // 'masia'
```
+ 原型+构造函数继承
+ 寄生式继承(工厂模式的变种)
```javascript
function Japanese (name, language) {
  this.name = name
  this.language = language
}
function CreateChinese (name, language) {
  var obj = {};
  Japanese.call(obj, name, language)
  return obj
}
const chinese = CreateChinese('masia', 'chinese')
console.log(chinese)
console.log(chinese.constructor) // Object
```
#### 动态原型
在构造函数中，通过传入的属性来判断，是否让他的原型拥有方法的方式
```javascript
function Person (name, work) {
  this.name = name;
  work && Person.prototype.woking = function () {
    console.log(work)
  }
}
var obj = new Person('masia')
var obj2 = new Person('masia', 'font-engineer')
```
