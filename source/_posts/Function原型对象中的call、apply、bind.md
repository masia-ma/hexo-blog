---
title: Function原型对象中的call、apply、bind
categories:
  - javascript
  - Function
tags:
  - javascript
  - Function
date: 2019-04-29 12:00:00
---
# Function.prototype
+ call、apply、bind都是Function.prototype的属性
<!-- more -->
```javascript
{
  console.log(Function.prototype.hasOwnProperty('apply')); // true
  console.log(Function.prototype.hasOwnProperty('call')); // true
  console.log(Function.prototype.hasOwnProperty('bind')); // true
}
```
### call、apply方法
```javascript
{
  function fn (a, b) {
    console.log(this.name, a, b)
  }
  var obj = {
    name: 'masia'
  }
  fn.call(obj, 1, 2); // masia 1 2  fn函数体中this是obj
  fn.apply(obj, [11, 22]); // masia 11 22  fn函数体中的this是obj
  var getFun = fn.bind(obj); // masia 111 222  fn函数体中的this是obj
  getFun(111, 222);
}
```
### bind方法用来重新定义方法中的this
与call、apply不同的是，使用fn.bind(self)并不是在调用的时候重新定义this的指向，而是在调用之前重新定义了方法中this的指向，fn.bind(self)这个表达式的值是一个方法体，所以需要一个变量去接收修改this后的fn方法体。fn.call(self)或者fn.apply(self)的值是fn执行的返回值。
```javascript
{
  function fn (arg) {
    console.log('arg',arg); // arg this is arg
    console.log(this.name); // obj1
  }
  let obj1 = {
    name: 'obj1'
  }
  let obj2 = {
    name: 'obj2',
    fn: fn.bind(obj1)
  }
  obj2.fn('this is arg');
}
```
```javascript
// 使用新的变量去接收改变this指向后的fn函数
{
  function fn () {
    console.log(this);
  }
  let obj = {
    name: 'obj'
  }
  let newFn = fn.bind(obj);
  fn(); // Window {postMessage: ƒ, blur: ƒ, focus: ƒ, close: ƒ, parent: Window, …}
  newFn(); // {name: "obj"}
}
```
