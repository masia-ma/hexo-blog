---
title: ES6梳理-解构赋值
categories:
  - ES6
  - 解构赋值
tags:
  - ES6
  - 解构赋值
abbrlink: 13f9823b
date: 2019-04-29 10:00:00
---

# 解构赋值
对象的结构赋值按照key去匹配，数组的解构赋值按照位置去匹配 
<!-- more -->

## 对象的解构赋值

写法一：
```javascript
// 解构对象
const obj = {
	a: 1, 
  	b: 2
}
// 赋值变量
let {a, b} = obj;
console.log(a, b);
```

写法二:
```javascript
 {
   let a, b;
   ({a, b} = {a: 1, b: 2});
   console.log(a, b);
 }
// 输出: 1 2 
```
覆盖变量的默认值：
```javascript
{
  let {a=10, b=5} = {a:5}
  console.log(a, b)
}
```
起别名：
```javascript
{
  let metaData = {
    title: 'abc',
    num: 2222
  }
  let {title: t, num: n} = metaData;
  console.log(t, n)
  // 输出 'abc' 222 
  // 如果你要访问title，就会报错，title is not defined
}
```
使用场景一：服务端和前端通行使用的格式是json，metaData是从后端接收到的json对象，我们需要取出第一层的title和第二层的title 

```javascript
{
  let metaData = {
    title: 'abc',
    test: [{
      title: 'test1', // 只能取出第一个数组项的title
      desc: 'description'
    }, {
      title: 'test2',
      desc: 'description'
    }]
  }

  //  取出来的是冒号右边的值，在后续访问的也是冒号后面的值，如果你访问左边的值，则会报 xxx is not defined
  let {title: esTitle, test: [{title: cnTitle}]} = metaData;
            // 取出了第一层的title属性并且别名
  console.log(esTitle, cnTitle)
}
``

ps:对象简单赋值法：
```javascript
let c, d;
c = 3;
d = 4;
const obj2 = {c, d};
```

## 数组的解构赋值

0. 一般写法
```javascript
{
  let a = 1, b = 2;
  const arr = [a, b];
  console.log(a, b, arr);
}
```
1. 解构赋值写法
```javascript
{
  let a, b;
  const arr = [a, b] = [1, 2]
              // 对应写法
  console.log(a, b, arr);
}
```
2. 数组解构赋值+ "..."操作符
```javascript
{
  let a, b, rest;
  [a, b,...rest] = [1, 2, 3, 4, 5, 6]
  console.log(a, b, rest)
}
// 输出: 1 2 [3, 4, 5, 6]
```

3. 如果右边缺少与左边的配对值，则右边的值默认为undefined。反则，右边值覆盖左边值
```javascript
{
  let a, b=111, c, rest;
  		// 给b设置初值
  const arr = [a, b=222, c] = [1, 2]			// 修改b的值      //再次修改b的值
  console.log(a, b, c, arr)
}
//输出：1 2 undefined [1, 2]
```

4. 数组解构赋值默认值使用场景，以前es5不好实现的，现在非常容易

场景一：实现变量交换
```javascript
{
  let a = 1, b = 2;
  [a, b] = [b, a];
  console.log(a, b);
}
//输出： 2 1
```

场景二：某个产品订单函数，输出三个值，第一个是名称，第二个是数量，第三个值是单价。
```javascript 
{
  function fn () {
 	return ['手机', 2, 2000];
  }
  let [name, num, price] = fn();
  // 直接使用变量来接受值本身，不需要再通过数组的索引去取值。
  console.log(name, num, price);
  // 输出： "手机" 2 2000
}
```
场景三：函数执行返回一个数组，我只取出来我想要的
```javascript
{
  function fn () {
    return [1, 2, 3, 4, 5];
  }
  let a, b, c;
  [a, b, , ,c] = fn(); // 根据位置来去除自己想要的值
  console.log(a, b, c);
  // 输出： 1 2 5
}
```
场景四（特别有用）：函数执行返回一个数组，但是长度未知，我想要第一个元素，但是其余的元素我需要保存在一个数组中在未来使用
```javascript
{
  function fn () {
    return [1, 2, 3, 4, 5];
  }
  let a, b;
  [a, ...b] = fn(); // 根据位置来去除自己想要的值
  console.log(a, b);
  // 输出： 1 [2, 3, 4, 5]
}
```