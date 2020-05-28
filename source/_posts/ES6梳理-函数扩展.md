---
title: ES6梳理-函数扩展
categories:
  - ES6
  - 函数扩展
tags:
  - ES6
  - 函数扩展
date: 2019-04-29 11:00:00
---

# 函数扩展
+ 参数默认值
+ rest参数
+ 扩展运算符
+ 箭头函数
+ this绑定
+ 尾调用
<!-- more -->

## 函数参数默认值
```javascript
{
  {
    function fn (x, y = 'masia', z=3) { // 在设置了默认值参数的后面，必须加没有默认值的参数
      console.log(x, y, z); // 666 "masia"
    }
    fn(666, 222, 55);
  }

  // 注意点： 不要混着写，一般推荐把有默认值的参数都放在最后面，以免手动传默认值。
  {
    function fn (x, y = 'masia', z) { // 不要这样写
      console.log(x, y, z); // 666 "masia"
    }
    fn(666);
  }
  
}
```
## 作用域
```javascript
{
  let x = 'test';
  function test2 (x, y=x) {
                    // 参数y的值是参数x的值，还是外部x变量的值？
    console.log(x, y) 
  }
  test2('kill'); // kill kill 答案是参数x的值
  test2(); // undefined undefined
  function test3 (c, y=x) {
                // 单如果没有参数x，那么y的值是外部变量x的值
    console.log(c, y);
  }
  test3('kill'); // kill test
  test3(); // undefined "test"
}
```
### rest参数
```javascript
{
  function test (...arg) {
    // ...arg在传入的参数数量不确定的时候，这个操作符号，把你传入的参数组合成了一个数组，名为arg
    for (let v of arg) {
      console.log('rest', v);
    }
    console.log(arg) // [1, 2, 3, 4, "a"] 数组
    console.log(arguments) // 对象 ，Arguments(5) [1, 2, 3, 4, "a", callee: (...), Symbol(Symbol.iterator): ƒ]
  }
  test(1, 2, 3, 4, 'a');
  // rest 1
  // rest 2
  // rest 3
  // rest 4
  // rest a
}

// 逆用
{
  // "..."操作符把一个数组拆成了一个个离散的值  
  console.log(...[1, 2, 4]); // 1 2 4 
  console.log('a', ...[1, 2, 4]); // a 1 2 4
}

// 实质（arg和对arguments做了数组转化一个道理）
{
  function fn (...arg) {
    console.log(arg); // (4) [1, 2, 3, 4]
    console.log(arguments); // Arguments(4) [1, 2, 3, 4, callee: (...), Symbol(Symbol.iterator): ƒ]
    console.log(Array.from(arguments)); // (4) [1, 2, 3, 4]
  }
  fn(1, 2, 3, 4);
}
```
### 箭头函数
+ 简单使用
```javascript
{
  // 有参数
  let  arrow = arrow => console.log(arrow);
  arrow('你好'); // 你好

  // 等价写法： 
  let arrow2 = arrow2 => {
    return console.log(arrow);
  }
  arrow('你好'); // 你好

  //无参数
  let arrow3 = () => console.log(5);
}
```
+ 箭头函数中this到底指向谁？
由于箭头函数不绑定this， 它会捕获其所在（即定义的位置）上下文的this值， 作为自己的this值
```javascript
{
  function Person (name) {
    this.name = name;
    this.child = {
      name: 'ming',
      getFather: () => {
        console.log(this.name);
      },
      getSeftName: function () {
        console.log(this.name);
      }
    }
  }
  const person = new Person('masia');
  person.child.getFather(); // masia
  person.child.getSeftName(); // ming
}

{
  function Person () {
    this.age = 0;
    setTimeout(function () {
      console.log(this); // Window {postMessage: ƒ, blur: ƒ, focus: ƒ, close: ƒ, parent: Window, …}
    }, 1000)
    setTimeout(() => {
      console.log(this); // Person {age: 0}
    }, 1000)
  }
  const p = new Person();
}
```

### 尾调用
函数式编程，函数的最后一句话是不是函数调用，尾调用可以提升函数的性能，之前的
```javascript
{
  function tail (x) {
    console.log('tail', x);
  }
  function fx (x) {
    return tail(x);
  }
  fx('尾调用'); // tail 尾调用
}
```