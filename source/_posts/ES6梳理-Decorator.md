---
title: ES6梳理-Decorator
date: 2019-07-11 10:48:33
categories:
  - ES6
  - Decorator
tags:
  - ES6
  - Decorator
---

# Decorator（修饰器的意思）
+ 基本概念
+ 基本用法
<!-- more -->
### 基本概念
Decorator是一个函数，用来修改类的行为。
### 基本用法
需要额外安装包 babel-plugin-transform-decorators-legacy 
```javascript
// 修饰器放在方法前面
{
  // 限制某个属性是只读的
  let readonly = function (target, name, descriptor) {
                          // target是目标类本身
                          // name是要修改的属性
                          // descriptor是要改动的配置
    descriptor.writable = false;
    return descriptor
  }

  class Test {
    @readonly
    time () {
      return '2017-03-11';
    }
  }
  
  let test = new Test();
  console.log(test.time()); // '2017-03-11'
  test.time = function () {} // 不允许修改
}  
// 修饰器放在类前面
{
  let typename = function (target, name, descriptor) {
    target.myName = 'hello';
  }

  @typename
  class Test {

  }

  console.log('类修饰符', Test.myName);// 类修饰符 hello
}
```
### 使用第三方修饰器的js库：core-decorators
npm i core-decorators
```javascript
// 日志系统
 {
  let log = (type) => {
    return function (target, name, descriptor) {
      let src_method = descriptor.value; // 原始的方法体
      descriptor.value = (...arg) => {
        src_method.apply(target, arg);
        console.info(`log ${type}`);
      }
    }
  }

  // 广告类
  class AD {
    @log('show')
    show () {
      console.info('ad is show');
    }
    @log('click')
    click () {
      console.info('ad is click');
    }
  }
}
```