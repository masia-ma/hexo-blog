---
title: Object原型对象
categories:
  - javascript
  - Object
tags:
  - javascript
  - Object
date: 2019-04-29 11:00:00
---

# javascript Object
+ 方法Object.assign()，用类给一个构造函数或者说类的原型对象添加属性
<!-- more -->
```javascript
{
  function Person (name) {
    this.name = name
  }
  const obj = new Person();
  console.log(Person.prototype.constructor) // ƒ Person (name) {..}
  // 如果我们直接覆盖Person.prototype，那样prototype原来有constructor属性将会丢失。
  Person.prototype = {
    getName: function () {
      return this.name;
    }
  }
  console.log(Person.prototype.constructor) // 再次打印，只能查找到了Object的中constructor, ƒ Object() { [native code] }

  // 当然我们也不需要通过Person.prototype.getName = .. 的方式一个个去添加属性，直接使用Object.assign(),这样只是添加了属性，并没有改变构造函数原型对象的指向

  function Animal (name) {
    this.name = name
  }
  Object.assgin({
    getName: function () {
      return this.name;
    },
    setName (name) {
      this.name = name;
    }
  })
}
```
+ 方法Object.create()
```javascript
{
  const obj = {} // 由Object创建的对象，他的 __proto__ 指向Object.prototype
  function Person () {}
  const person = new Person() // 
  console.log(person) // 由Person创建，他的 __proto__ 指向Person.prototype

  const obj2 = Object.create(null); // 创建了一个没有 __proto__ 属性的空对象，那么自然这个对象不能使用Object原型对象上面的属性
  obj2.a = 'masia' // 单可以为他添加属性
  console.log(obj2.a) // 而且可以访问

  const methods = {
    getName: function () {
      return this.name
    }
  }
  const obj3 = Object.create(methods) // 把obj3的__proto__属性指向methods对象
  obj3.name = 'masia'; // 可以添加属性
  console.log(obj3.getName()); // masia 

  // 请不会给Object.create()的参数传了一个函数
}

// 使用工厂模式手写一个Object.create() 方法
{
  function ObCreate (proto) {
    const a = {}
    a.__proto__ = proto;
    return a;
  }
  const proto = {
    getName: function () {
      return this.name;
    }
  }
  const obj = ObCreate(proto);
  obj.name = 'masia';
  console.log(obj.getName()); // masia
}
</script>
```