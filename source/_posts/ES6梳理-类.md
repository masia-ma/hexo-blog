---
title: ES6梳理-类
date: 2019-07-08 19:57:54
tags:
 - ES6
 - 类
categories: 
 - ES6
 - 类
---
# 类
+ 基本概念
+ 基本语法
+ 类的继承
+ 静态方法
+ 静态属性
+ getter
+ setter
<!-- more -->
### 基本使用
```javascript
// ES6继承
{
  class Animal {
    constructor (name, age = 0) {
                // 当然这里也可以使用形参赋值的方式
      this.name = name;
      this.age = age;
    }
    getName () {
      console.log('Animal getName ' + this.name);
    }
  }
  class Cat extends Animal {
    constructor (name, age, voice) { 
      /* 
        在子类中要么不写构造方法，则使用子类创建对象时，会自动调用父类的构造方法 ，
        如果在子类中写了构造方法，则在创建子类实例的时候执行该方法，（实质就是原型
        链就近原则，js使用方法名来规定方法的唯一性，这点和java是不同的，java使用
        方法签名，也就是方法名和参数列表来规定方法的唯一性）
      */
      super(name, age);
      this.voice = voice;
    }
    getName () {
      super.getName();
      console.log('Cat getName ' + this.name));
    }
  }
  const cat = new Cat('mimi', 12, 'miao');
  console.log(cat); // Cat {name: "mimi", age: 12, voice: "miao"}
  cat.getName(); // Animal getName
                  // Cat getName
} 
// ES5继承
{
  function Animal (name, age) {
    this.name = name;
    this.age = age;
  }
  Object.assign(Animal.prototype, {
    getName () {
      console.log('Animal getName ' + this.name);
    }
  });
  function Cat (name, age, voice) {
    Animal.apply(this, [name, age]);
    this.voice = voice;
  }
  Object.assign(Cat.prototype, {
    getName () {
      // 子类public方法调用父类方法
      Cat.prototype.__proto__.getName.call(this); // 同样可以使用call()来改变方法内的this
      console.log('Cat getName ' + this.name)
    }
  })
  Cat.prototype.__proto__ = Animal.prototype;
  const cat = new Cat('lili', 2, 'miao'); 
  console.log(cat); // Cat {name: "lili", age: 2, voice: "miao"}
  cat.getName(); // Animal getName lili
                  // Cat getName lili
}
```
{% asset_img class_compare1.png 类的简单使用 %}
{% asset_img class_compare2.png ES6类的继承 %}
{% asset_img class_compare3.png ES5类的继承 %}

### getter和setter
```javascript
{
  class Animal {
    constructor (name = 'animal') {
      this.name = name;
    }

    // 这里你不要理解为方法，这里是非方法属性，相当于vue中的计算属性
    get longName () {
      return 'mk' + this.name;
    }

    set longName (value) {
      // 意为你给longName赋值，最终会给name属性赋值
      this.name =  value
    }
  }
  const ani = new Animal();
  console.log(ani.name); // animal
  console.log(ani.longName); // mkanimal
  ani.longName = 'cat';
  console.log(ani.name); // cat
  console.log(ani.longName); // mkcat
}
```
### 静态方法
+ 使用关键字static，这点和java相同
```javascript
{
  class Animal {
    constructor (name = 'animal') {
      this.name = name;
    }
    static tell () {
      console.log('this is a static method');
    }
    test () {
      tell();
    }
  }
  const ani = new Animal();
  console.log(ani);
  console.log(ani.name); // animal
  Animal.tell(); // this is a static method

  // js中的静态方法不能通过实例去调用，这主要与静态方法存放的位置有关，这点与java不同
  ani.tell(); // Uncaught TypeError: ani.tell is not a function 

  // 当然也不能通过原型链上的其他方法去调用(这实质上也是通过实例在调用)
  ani.test(); // Uncaught TypeError: ani.tell is not a function 
}
```
{% asset_img static1.png 静态方法%}
+ 当然，也可以使用一些骚操作来实现实例对静态方法的调用，只要搞清楚静态方法存放的位置，就可以为所欲为
```javascript
{
  class Animal {
    constructor (name = 'animal') {
      this.name = name;
    }
    static tell () {
      console.log('this is a static method');
    }
    static getThis () {
      console.log(this)
    }

    test () {
      tell();
    }
    test2 () {
      Animal.tell();
    }
    test3 () {
      Animal.getThis.call(this);
    }
  }
  const ani = new Animal();
  console.log(ani);
  console.log(ani.name); // animal
  Animal.tell(); // this is a static method
  Animal.getThis();
  /* 输出：
  class Animal {
    constructor (name = 'animal') {
      this.name = name;
    }
    static tell () {
      console.log('this is a static method');
    }
    static getThis () {
      console.log(this)
   …
  */

  // js中的静态方法不能通过实例去调用，这主要与静态方法存放的位置有关，这点与java不同
  ani.tell(); // Uncaught TypeError: ani.tell is not a function 

  // 当然也不能通过原型链上的其他方法去调用，因为在对象的原型链上找不到这个方法。
  ani.test(); // Uncaught TypeError: ani.tell is not a function 

  // 但其实你非要通过实例中的方法间接使用，也可以
  ani.test2(); // this is a static method

  // 甚至通过call来改变this的指向（call太强大了）
  ani.test3(); // Animal {name: "animal"}
}
```
+ 如果要用ES5实现，就直接给类加属性就可以了，如：Animal.getThis = function () {}，甚至可以使用这种方法定义静态属性，但static只能用来定义静态方法