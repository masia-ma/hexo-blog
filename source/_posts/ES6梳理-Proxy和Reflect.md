---
title: ES6梳理-Proxy和Reflect
date: 2019-07-08 17:57:54
tags:
 - ES6
 - Proxy和Reflect
categories: 
 - ES6
 - Proxy和Reflect
---

# Proxy和Reflect
+ Proxy和Reflect的概念
+ Proxy和Reflect的适用场景 
<!--  more -->
## 概念：
+ Proxy是代理的意思，连接了用户和对象之间的层，Reflect是反射的意思
+ Proxy和Reflect的使用方法基本相同
## Proxy使用
+ 基本使用
```javascript
{
  // 创建一个类似于供应商的对象
  let obj = {
    time: '2017-01-11',
    name: 'net',
    _r: 123
  };
  // 创建一个Proxy对象
  let monitor = new Proxy(obj, {
    // 拦截对象属性读取
    get (target, key) {
      return target[key].replace('2017', '2018')
    },
    // 拦截设置对象属性
    set (target, key, value) {
      // 假如我们做一个只能修改指定的属性name
      if (key === 'name') {
        return target[key] = value;
      } else {
        return target[key];
      }
    }
  });
  // 用户操作的是monitor对象，不管用户对monitor对象执行什么操作，都是由Proxy传递给obj对象
  // 因为obj对象对于用户来说是不可见的，所以monitor.time这个操作被Proxy中的get方法拦截了，
  // 意为读取obj中的某个属性，并且将这个属性的值做一些修改
  console.log('get', monitor.time); // get 2018-01-11
  monitor.time = '2018';
  console.log(monitor.time); // 2018-01-11 发现没有修改为 '2018'
  monitor.name = 'masia'; 
  console.log(monitor.name); // masia 发现修改成功了
}
```
+ 常用属性
```javascript
{
  // 创建一个类似于供应商的对象
  let obj = {
    time: '2017-01-11',
    name: 'net',
    _r: 123
  };
  // 创建一个Proxy对象
  let monitor = new Proxy(obj, {
    // 拦截对象属性读取
    get (target, key) {
      return target[key].replace('2017', '2018')
    },
    // 拦截设置对象属性
    set (target, key, value) {
      // 假如我们做一个只能修改指定的属性name
      if (key === 'name') {
        return target[key] = value;
      } else {
        return target[key];
      }
    },
    // 暴露指定属性name,其他的都不暴露（这里暴露指的是使用in关键字找不到设置限制的属性）
    has (target, key) {
      if (key === 'name') {
        return target[key];
      } else {
        return false;
      }
    },
    // 拦截删除
    deleteProperty (target, key) {
      // 比如指定属性是"_"开头的，允许删除，其他的则不允许
      if (key.indexOf('_') > -1) {
        delete target[key];
      } else {
        return target[key];
      }
    },
    // 拦截Object.keys, ObjectgetOwnPropertySymbols, Object.getOwnPropertyNames
    ownKeys (target) {
      return Object.keys(target).filter(item=>item!='time')
    }
  });
  console.log(monitor.name); // masia 
  console.log(monitor.time); // 2018-01-11 设置了has,但是依然可以访问到
  console.log('name' in monitor) // true 
  console.log('time' in monitor) // false 但是使用in关键字检查对象中是否包含某个属性时，却找不到 
  
  // 尝试删除time属性
  delete monitor.time;
  console.log(monitor); // Proxy {time: "2017-01-11", name: "net", _r: 123} 发现time属性还在
  delete monitor._r;
  console.log(monitor); // Proxy {time: "2017-01-11", name: "net"} 发现_r属性被删除了

  // 
  console.log('ownKeys', Object.keys(monitor)); // ownKeys ["name"] 把name属性拦截掉了
}
```
## Reflect使用
```javascript
{
  // 创建一个类似于供应商的对象
  let obj = {
    time: '2017-01-11',
    name: 'net',
    _r: 123
  };

  // ES5 使用obj.time来取值
  // Proxy还要通过创建实例monitor来取值
  // Reflect要使用以下的方式来取值
  console.log('Reflect get', Reflect.get(obj, 'time')); // Reflect get 2017-01-11
  Reflect.set(obj, 'name', 'masia');
  console.log(obj.name); // masia
  console.log(Reflect.has(obj, 'name')); // true
}
```

## Proxy和Reflect的使用场景
```javascript
{
  // 在开发过程中经常要对一些信息进行校验，比如登录的时候手机号格式是否正确
  // 解耦校验模块
  function validator (target, validator) {
    return new Proxy (target, {
      _validator: validator,
      set (target, key, value, proxy) {
        // 判断当前对象有没有指定的key值，如果没有则不能对他赋值
        if (target.hasOwnProperty(key)) {
          let va = this._validator[key];
          if (!!va(value)) {
            return Reflect.set(target, key, value, proxy); 
          } else {
            throw Error(`不能设置${key}到${value}`)
          }
        } else {
          throw Error(`${key} 不存在`)
        }
      }
    })
  }

  const personValidators = {
    name (val) {
      return typeof val === 'string';
    },
    age (val) {
      return typeof val === 'number' && val > 18
    }
  }
  
  class Person {
    constructor (name, age) {
      this.name = name;
      this.age = age;
      return validator(this, personValidators); // Proxy对象代理了Person对象实例
    }
  }

  const person = new Person('lilei', 30);
  console.info(person); // Proxy {name: "lilei", age: 30}
  // person.name = 48; // Uncaught Error: 不能设置name到48
  // 因为有了代理，就不能对对象的实例属性做一些随意的操作了，这在ES5中是不能实现的
  person.name = 'han mei';
  console.info(person); // Proxy {name: "han mei", age: 30}
  /* 总结：传统的做法，对某个属性进行赋值的时候，要对赋值的属性进行一些判断，
    通常这些操作是在创建完了对象之后做的，并没有在使用类定义的时候做这件事情，
    这就好比这个对象创建出来的时候是完整且干净的，就好比一块原材料，而我们后天
    往上加饰的雕琢，使他有了更多的功能和限制。
    但使用Proxy代理的方式，相当于我们拿到的原材料已经是半成品甚至成品，他已经包含了
    我们需要的一些功能和限制。这样跟易于我们后期的维护和扩展，因为我们不需要
    在多个地方使用的时候，搬运我们后天添加的验证或者限制权限的一些方法。而需要做的仅仅是
    使用new关键字创建一个实例对象。这个实例对象一出生就已经自带了验证方法。
    使得代码复用性、健壮性更强
  */
}
```