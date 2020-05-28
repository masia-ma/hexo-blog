---
title: ES6梳理-对象扩展
date: 2019-07-08 14:15:45
tags: 
 - ES6
 - 对象扩展
categories: 
 - ES6
 - 对象扩展
---
# 对象扩展
+ 简洁表示法
+ 属性表达式
+ 扩展运算符
+ Object新增方法
<!-- more -->
### 简洁表达
```javascript
{
  let name = 'masia';
  let age = 14;
  // ES5中创建对象
  const es5 = {
    name: name,
    age: age,
    level: 6,
    getName: function () {
      console.log(this.name)
    }
  }
  console.log(es5) // {name: "masia", age: 14, level: 6, getName: ƒ}
  // ES6创建对象
  const es6 = {
    name,
    age,
    level: 6,
    getName () {
      console.log(this.name)
    }
  }
  console.log(es6) // {name: "masia", age: 14, level: 6, getName: ƒ}
}
```
### 属性表达式
```javascript
{
  // 在ES5中对象的key值必须是一个字符串，固定的
  let a = 'aa';
  let fnName = 'fn';
  const es5 = {
    aa: 'value',
    fn: function () {}
  }
  // 但在ES6中对象的key值可以是一个表达式，用[]包起来
  const es6 = {
    [a]: 'value',
    [fnName]: function () {}
  }
  console.log(es5); // {aa: "value", fn: ƒ}
  console.log(es6); // {aa: "value", fn: ƒ}
}
```
### 扩展运算符
```javascript
{
  const obj = {
    a: 1, 
    b: 2,
    c: 3,
    d: 4,
  };
  let {a, b, ...c} = obj;
  console.log(a); // 1
  console.log(b); // 2
  console.log(c); // {c: 3, d: 4}
}
```
### 新增的常用Api
+ Object.is()，判断两个值是否完全相等，在功能上与"==="完全相同
```javascript
{
  let str1 = 'abc';
  let str2 = 'abc';
  console.log('字符串', Object.is(str1, str2), str1 === str2); // 字符串 true true
  console.log('数组', Object.is([], []), [] === []); // 数组 false false
}
```
+ Object.assign()浅复制
对象拷贝，使用第二个参数对象覆盖第一个参数对象，并且改变两个参数对象的内容，返回值为第一个参数对象的引用
```javascript
 {
  let obj1 = {
    a: 1,
  }
  let obj2 = {
    a: 11,
    b:2
  }
  let obj3 = Object.assign(obj1, obj2);
  console.log(obj1) // {a: 11, b: 2}
  console.log(obj2) // {a: 11, b: 2}
  console.log(obj3) // {a: 11, b: 2}
  console.log(obj1 == obj2) // false
  console.log(obj1 == obj3) // true
}
```
+ Object.entries()
方法的返回值是一个可迭代对象
需要注意的是调用方式，这个方法用作数组是ArrayObj.entries(),而一般对象则为Object.entries(obj)
```javascript
{
  let obj = {
    a: 111,
    b: '222',
    c: function () {},
    d: {}
  }
  for (let [key, value] of Object.entries(obj)) {
    console.log(key, value);
    /*
      a 111
      b 222
      c ƒ () {}
      d {}
    */
    console.log(Object.entries(obj));
  }
}
```
{% asset_img Object.entries().png ES5构造函数继承和原型链继承组合继承 %}
