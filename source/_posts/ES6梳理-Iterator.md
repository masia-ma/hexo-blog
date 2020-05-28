---
title: ES6梳理-Iterator
date: 2019-07-10 18:48:33
categories:
  - ES6
  - Iterator
tags:
  - ES6
  - Iterator
---
# Iterator
数组和对象数据结构本来是不同的，那么怎么样使不同的数据结构操作变得统一起来
<!-- more -->
### for of 实现的原理实际上就是不断调用Iterator的过程
```javascript
// 数组对象默认定义了Iterator接口
{
  let arr = ['hello', 'world'];
  let map = arr[Symbol.iterator]();
  console.log(map.next()); // {value: "hello", done: false}
  console.log(map.next()); // {value: "world", done: false}
  console.log(map.next()); // {value: undefined, done: true}
}  
// obj本身没有默认定义Iterator接口，所以不能使用for of，但是自定了Iterator之后，就可以使用for of了
{
  let obj = {
    start: [1, 3, 2],
    end: [7, 8, 9],

    // 定义一个iterator接口方法
    [Symbol.iterator] () {
      let self = this;
      let index = 0;
      let arr = self.start.concat(self.end);
      let len = arr.length;
      return {
        next () {
          if (index < len) {
            return {
              value: arr[index++],
              done: false
            }
          } else {
            return {
              value: arr[index++],
              done: true
            }
          }
        }
      }
    }
  }
  // 输出obj，见图
  console.log(obj);
  // 使用for of 自动迭代
  for (let key of obj) {
    console.log(key); // 但是只输出了value
    /*
      1
      3
      2
      7
      8
      9
    */
  }

  // 手动迭代
  let objMap = obj[Symbol.iterator](); // 其实就是利用闭包，形成了一个不可销毁的作用域
  console.log(objMap.next()); // {value: 1, done: false}
  console.log(objMap.next()); // {value: 3, done: false}
  console.log(objMap.next()); // {value: 2, done: false}
  console.log(objMap.next()); // {value: 7, done: false}
  console.log(objMap.next()); // {value: 8, done: false}
  console.log(objMap.next()); // {value: 9, done: false}
  console.log(objMap.next()); // {value: undefined, done: true} 
}
```
{% asset_img Iterator_obj.png console.log(obj) %}

### 手动模拟一个Iterator
原理就是利用函数的闭包，把函数内部的引用成员保存到了函数的外部，是得函数本身的执行栈不可释放，进而还可以继续使用函数内部的变量实现的。
```javascript
{
  let obj = {
    start: [1, 2, 3, 4],
    end: [5, 6, 7],
    iterator () {
      let self = this;
      let arr = self.start.concat(self.end);
      let index = 0;
      return {
        next () {
          if (index < arr.length) {
            return {
              value: arr[index++],
              done: false
            }
          } else {
            return {
              value: arr[index++],
              done: true
            }
          }
        }
      }
    }
  }

  let iterator = obj.iterator();
  console.log(iterator.next());
  console.log(iterator.next());
  console.log(iterator.next());
  console.log(iterator.next());
  console.log(iterator.next());
  console.log(iterator.next());
  console.log(iterator.next());
  console.log(iterator.next());
}
```
### 利用高阶函数实现抽奖，并且封闭了抽奖次数变量
```javascript
{
  // 1. 实现抽奖次数限制
  function residue () {
    let count = 5; // 初始化抽奖次数
    return {
      next () {
        while (count>0) {
          return {
            count: --count,
            done: false
          }
        }
        return {
          count: --count, // 应该是-1了
          done: true
        }
      }
    }
  }

  // 1. 抽奖点击功能
  function prizeDraw (iteratorObj) {
    let btn = document.createElement('button');
    btn.innerText = "抽奖";
    btn.addEventListener('click', function () {
      remainder(iteratorObj.next());
    })
    document.body.appendChild(btn);
  } 

  // 2. 判断抽奖剩余次数
  function remainder (iteratorNext) {
    if (!iteratorNext.done) {
      console.log('还剩余'+iteratorNext.count+'次');
    } else {
      console.log('你没机会了')
    }
  }
  
  // * 初始化抽奖功能
  prizeDraw(residue());
  
}
```