---
title: ES5继承
categories:
  - ES5
  - 继承
tags:
  - ES5
  - 继承
date: 2019-04-29 11:00:00
---

## ES5继承
<!-- more -->
{% asset_img ES5_extends.png ES5构造函数继承和原型链继承组合继承 %}
事实上，以下代码应该是实现了多态
```javascript
{
  function Person (name, age) {
    this.name = name;
    this.age = age;
    this.getName = function () {
      console.log(this.name);
    }
  }
  Object.assign(Person.prototype, {
    sleep () {
      console.log("sleepping");
    }
  })
  function Student (name, age) {
    Person.call(this, name, age) // 先继承Person构造函数中的属性，在这一句之后也可以改写Person构造函数中的属性，但这种方式不能继承父类原型对象上的属性。
  }
  Student.prototype = new Person(); // 更好的实现方式是，可以把Student的prototype对象变成Person的实例对象，
  // 则Student.prototype.__proto__ == Person.prototype，这样一来，Student的实例对象就可以通过查找
  // 两层原型链的方式来调用Person原型上的方法了，即stu.__proto__.__proto__  =  Person.prototype,
  // 这样的话，在Student中的原型链中也可以重写Person中的方法
  Object.assign(Student.prototype, {
    studentSleep () {
      console.log("studentSleep");
    }
  })
  var stu = new Student('masia', 66);
  console.log(stu)
  console.log(stu.constructor.name) // 输出Person，
  // 分析：stu是Student的实例对象，但是我们知道stu中没有constructor这个属性，
  // 所以就会顺着原型链往上查找，即找到stu.__proto__，查看这个对象中有没有，
  // 一般情况下是stu.__proto__对象就是Student.prototype对象，所以应该会有一个
  // 名为constructor的属性，并且值位Student这个函数本身，
  // 但是我们改变了Student.prototype的指向，并且把他指向了一个新的Person实例对象，
  // 那自然Student.prototype这个对象的__proto__属性就不是指向Object构造函数的Prototype了，
  // 而是Person的Prototype
  // stu.__proto__.__proto__指向了Person.prototype
  // 由于刚刚在stu.__proto__中没有找到constructor，拿就继续向上查找，并且
  // 找到了stu.__proto__.__proto__,很显然，在这个对象中找到了constructor，
  // 并且指向的是Person函数本身
}
```
通过修改子类.prototype.__proto__实现继承，并且保证了子类的实例对象的constructor属性是创建他的类
{% asset_img ES5_extends2.png ES5构造函数继承和原型链继承组合继承 %}

```javascript
{
  function Person (name, age) {
    this.name = name;
    this.age = age;
    this.getName = function () {
      console.log(this.name);
    }
  }
  Object.assign(Person.prototype, {
    sleep () {
      console.log("sleepping");
    }
  })
  function Student (name, age) {
    Person.call(this, name, age) // 先继承Person构造函数中的属性，在这一句之后也可以改写Person构造函数中的属性，但这种方式不能继承父类原型对象上的属性。
  }
  Student.prototype.__proto__ = new Person(); // 只需要把这里修改一下，就可以了，
  // 从而总结出一种思路：可以让子类的prototype.__proto__指向父类的实例对象从而实现继承，
  // 虽然这样做，浪费了一定的空间，比如父类实例对象中的this.name 或者 this.age 这些属性加到子类的prototype.__proto__，
  // 但其实我们在访问stu.name的时候，早在stu的实例对象就访问到了，所以也就意味着不会因为访问stu.name而一直顺着原型链网上找，
  // 所以Student.prototype.__proto__.name永远不会被访问，但他依然占据空间。
  Object.assign(Student.prototype, {
    studentSleep () {
      console.log("studentSleep");
    }
  })
  var stu = new Student('masia', 66);
  console.log(stu)
  console.log(stu.constructor.name) // Student
}
```