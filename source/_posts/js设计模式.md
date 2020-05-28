---
title: js设计模式
categories: 
  - js
  - 设计模式
  - 面向对象
tags: 
   - js
   - 设计模式
   - 面向对象
date: 2019-05-15 10:00
---

## 定义类或者对象
<!-- more -->
1. 工厂模式

  + 创建对象
```javascript
var person = new Object(); // new Object() 和 new Object 是一样的效果
person.id = 1;
person.name = 'masia';
person.showName = function () {
  alert(this.name);
}
```
  + 创建多个对象
```javascript
function createPerson (id, name) {
  var obj = new Object;
  obj.id = id;
  obj.name = name;
  obj.showName = function () { // 每次创建一个实例，都要在实例中保存一个showName方法，在实例对象中复制了一份showName，为了解决这个重复复制函数的问题，可以使用给showName这个属性复制一个引用。
  // obj.showName = outerShowName;
    alert(this.name); // 这个this是obj
  }
  return obj;
}
function outerShowName () {
  alert(this.name)
}
var person1 = createPerson('1', 'masia');
var person2 = createPerson('2', 'laoliu'); 
```
2. 构造函数方式
```javascript
function Person (id, name) {
  this.id = id;
  this.name = name;
  this.showName = function () { // 这样写，在实例中也是会重复生成函数的，所以也可以使用引用赋值的方法，将一个函数的引用赋值给showName这个属性
  // this.showName = outerShowName;
    alert(this.name)
  }
}

var person1 = new Person('1', 'masia');
var person2 = new Person('2', 'laoliu');
```
3. 原型与构造函数混合方式
```javascript
fucntion Person (id, name) {
  this.id = id;
  this.name = name;
}
Person.prototype.showName = function () {
  alert(this.name);
}
// 这样的方式，在创建构造函数的时候，showName保存在构造函数的原型链中，没有浪费内存空间，不同的实例通过隐式原型链来调用showName方法。
```
+ 引入知识点：对象的方法借用
```javascript
var obj = {
  name: 'masia',
  age: 19,
  getName: function () {
    console.log(this.name)
  },
  modifyInfo: function (newName, newAge) {
    this.name = newName
    this.age = newAge
  }
}
var obj2 = {
  name: 'laoliu',
  age: 23
}
// obj2借用obj1中的方法，无参数时：
obj.getName.call(obj2)
obj.getName.apply(obj2)
// obj2借用obj1中的方法，有参数时：
obj.modifyInfo.call(obj2, 'Lawrence', 25) // 使用call，依次传参
obj.modifyInfo.apply(obj2, ['Lawrence', 25]) // 使用apply, 传入一个参数数组
```