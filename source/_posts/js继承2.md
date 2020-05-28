---
title: js继承2
categories:
  - js
  - 继承
tags:
  - js
  - 继承
---

# 首先写出ES6的继承
<!-- more -->
```javascript
class Animal {
  constructor () {
    this.z = 'zzz'
  }
  getAnimal () {
    console.log("animal")
  }
}
class Cat extends Animal {
  constructor () {
    super();
    this.x = 'xx';
    this.y = 'yy';
  }
  getCat () {
    console.log('cat')
  }
}
console.log(new Cat());
```
{% asset_img extends.png ES6的继承 %}
# ES5实现继承
```javascript
function Animal () {
  this.z = 'zzz'
}
Animal.prototype.getAnimal = function () {
  console.log("animal")
}
function Cat () {
  Object.assign(this, new Animal); // 要注意的是，Object.assign() 只可以浅复制可枚举对象，
  // 所以this.__proto__是不会被改变的，但是Animal实例对象中的z会被加入这里的this中
  // 在new实例对象的时候， this一被创建就被会被执行this.__proto__ = Cat.prototype
  this.x = 'xxx';
  this.y = 'yyy';
}
Cat.prototype.getCat = function () {
  console.log("cat")
}
Cat.prototype.__proto__ = Animal.prototype
console.log(new Cat())
```
上述ES6创建和ES5创建对象执行的结果是一模一样