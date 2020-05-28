---
title: ES6梳理-Generator
date: 2019-07-10 18:48:33
categories:
  - ES6
  - Generator
tags:
  - ES6
  - Generator
---

# Generator
+ 基本概念
+ next函数的用法
+ yield*的语法

<!-- more -->
### 基本概念
Generator通俗的来讲，就是异步编程的一种解决方案，比回调函数以及Promise更加高级
### next函数
```javascript
{
  let tell = function* () {
    yield 'a';
    yield 'b';
    return 'c';
  }
  let k = tell();
  console.log(k.next());
  console.log(k.next());
  console.log(k.next());
  console.log(k.next());
}
```
{% asset_img Generator1.png console.log(k.next()); %}
### yield*的语法

### 使用Generator也可以作为遍历器的返回值
```javascript
{
  let obj = {};
  // 不同通过手写Iterator的方式，而是通过创建Generator函数的方式实现对象的遍历器
  // 写法比自定义编写的形式简单
  obj[Symbol.iterator] = function* () {
    yield 1;
    yield 2;
    yield 3;
  }
  for (let value of obj) {
    console.log(value);
    /*
      1
      2
      3
    */
  }
}
```
### Generator状态机
Generator状态机的运用，假如只有A、B、C三种状态
```javascript
{
  let state = function* () {
    while (1) {
      yield 'A',
      yield 'B',
      yield 'C'
    }
  }
  let status = state();
  console.log(status.next()); // {value: "A", done: false}
  console.log(status.next()); // {value: "B", done: false}
  console.log(status.next()); // {value: "C", done: false}
  console.log(status.next()); // {value: "A", done: false}
  console.log(status.next()); // .. 
  console.log(status.next()); // {value: "A", done: false}
}
```
### 利用闭包模拟实现状态机
```javascript
{
  function mockStatemachine (arr) {
    let arrLen = arr.length;
    let index = 0;
    return {
      next () {
        return {
          value: arr[index++%arrLen],
          done: false
        }
      }
    }
  }
  let state = mockStatemachine(['A', 'B', 'C', 'D']);
  console.log(state.next()); // {value: "A", done: false}
  console.log(state.next()); // {value: "B", done: false}
  console.log(state.next()); // {value: "C", done: false}
  console.log(state.next()); // {value: "D", done: false}
  console.log(state.next()); // {value: "A", done: false}
  console.log(state.next()); // {value: "B", done: false}
  console.log(state.next()); // {value: "C", done: false}
  console.log(state.next()); // {value: "D", done: false}
}
```
### Generator中的async
```javascript
{
  let state = async function () {
    while (1) {
      await 'A',
      await 'B',
      await 'C'
    }
  }
  let status = state();
  console.log(status.next()); // {value: "A", done: false}
  console.log(status.next()); // {value: "B", done: false}
  console.log(status.next()); // {value: "C", done: false}
  console.log(status.next()); // {value: "A", done: false}
  console.log(status.next()); // .. 
  console.log(status.next()); // {value: "A", done: false}
}
```
### 什么情况下，Generator有他最大的作用，适用场景
+ 问题：
假如有一个抽奖的业务逻辑，比如当前用户下可抽奖的次数有5次，用户点击一个，可抽奖次数变为4次，再点一次，可抽奖次数变为3次，这个剩余抽奖次数的逻辑，前端和后端都要做限制。
+ 思路： 
之前的做法是设置一个全局变量，去保存抽奖的次数，
  1. 但其实是不安全的，因为一旦用户知道记录抽奖次数的变量，并且做一些修改，你就不能限制用户的抽奖次数了
  2. 尽量少写全局变量，影响性能
```javascript
{
  let draw = function (count) {
    // 具体抽奖逻辑，这里要注意的是，这里完全是验证是否中奖的逻辑，并且和可抽奖次数的判断完全隔离开，
    // 并且通过Generator来控制可抽奖次数
    console.info(`剩余${count}次`);
  }
  let residue = function* (count) {
    while (count) {
      count --;
      yield draw(count);
    }
  }
  let star = residue(5);
  let btn = document.createElement('button');
  btn.id = 'start';
  btn.innerText = '抽奖';
  btn.addEventListener('click', function () {
    star.next();
  })
  document.body.appendChild(btn);

  // 点击5次答应的结果
  // 剩余4次
  // 剩余3次
  // 剩余2次
  // 剩余1次
  // 剩余1次
}
```
+ 问题2:（长轮询）
服务端的某一个数据定期的发生变化，前端需要定期的去后端取数据，因为http是无状态的连接，我们要怎么去实时获取后端变化的数据呢？一种是长轮询，还有一种是webSocket，因为webSocket对浏览器支持的不好。原来的长轮询是利用定时器去实现，现在使用Generator函数去实现，会让程序变的更加优雅。
```javascript
{
  // 长论询
  let ajax = function* () {
    yield new Promise(function (resolve, reject) {
      setTimeout(function () { // 这部分模拟的是获取真实接口数据
        resolve({code: 1})
      }, 200);  
    })
  }

  let pull = function () {
    let generator = ajax(); // 对Generator实例化
    let step = generator.next();
    step.value.then(function (d) {
      if (d.code!=0) {
        setTimeout(function () {
          console.log('wait');
          pull();
        }, 1000)
      } else {
        console.log(d)
      }
    })
  }
  pull();
}

```

### 使用Generator和不使用Generator做对比
```javascript
// 使用Generator
{
  let state = function* (arr) {
    for (item of arr) {
      yield item;
    }
  }
  let k = state([111, 222, 333]);
  console.log(k.next()); // {value: 111, done: false}
  console.log(k.next()); // {value: 222, done: false}
  console.log(k.next()); // {value: 222, done: false}
  console.log(k.next()); // {value: undefined, done: true}
}
// 不使用Generator
{
  let state = function (arr) {
    let index = 0;
    return {
      next () {
        while (index < arr.length) {
          return {
            value: arr[index++],
            done: false
          } 
        }
        return {
          value: arr[index],
          done: true
        }
      }
    }
  }
  let k = state([111, 222, 333]);
  console.log(k.next()); // {value: 111, done: false}
  console.log(k.next()); // {value: 222, done: false}
  console.log(k.next()); // {value: 222, done: false}
  console.log(k.next()); // {value: undefined, done: true}
}
```
### yield使用对应关系
{% asset_img Generator2.png yield使用对应关系 %}
{% asset_img Generator3.png 使用Generator和不使用Generator %}