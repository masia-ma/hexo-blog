---
title: ES6梳理-Symbol
date: 2019-07-08 15:11:21
tags: 
 - ES6
 - Symbol
categories: 
 - ES6
 - Symbol
---
# Symbol
+ Symbol概念：
生成一个独一无二的值
<!-- more -->
```javascript
{
  // 声明
  let a1 = Symbol(); // 并没有使用new关键字，而是直接当作函数去使用
  let a2 = Symbol(); // 并没有使用new关键字，而是直接当作函数去使用
  console.log(a1 === a2); // false
  console.log(a1) // Symbol()红色的
  console.log(a2) // Symbol()红色的

  let a3 = Symbol.for('a3'); // 如果a3变量声明过且类型是Symbol类型，那就返回变量a3的值
  // 否则则创建一个Symbol对象
  let a4 = Symbol.for('a3'); 
  console.log(a3); // Symbol(a3)
  console.log(a4); // Symbol(a3)
  console.log(a3 == a4); // true
}  
```
+ Symbol的作用：
注意： 如果一个对象中使用Symbol对象做对象的key时，使用for in或者let of是检索不到的
```javascript
{
  let a1 = Symbol.for('abc');
  let obj = {
    [a1]: '123',
    'abc': 345,
    'c': 456
  }
  for (let item in obj) {
    console.log(item)
    /*
      abc
      c
    */
  }
  for (let [key, value] of Object.entries(obj)) {
    console.log(key, value);
    /*
      abc 345
      c 456
    */
  }
  console.log(Object.keys(obj)); // ["abc", "c"]
}
```
{% asset_img Symbol.png Symbol的作用 %}
复习数组对象使用let of 和 entries()
```javascript
{
  let arr = [1, 2, 3, 4];
  console.log(arr.entries());
  for (let [index, value] of arr.entries()) {
    console.log(index, value);
    /*
      0 1
      1 2
      2 3
      3 4
    */
  }
}
```
+ Object.getOwnPropertySymbols(obj)
这个方法用来取对象的key值为Symbol类型的key值本身，返回值为一个数组
```javascript
{
  let a1 = Symbol.for('abc');
  let a2 = Symbol();
  let a3 = Symbol.for();
  let obj = {
    [a1]: '123',
    [a2]: 'maisa',
    [a3]: 'a3',
    'abc': 345,
    'c': 456
  }
  let arr = Object.getOwnPropertySymbols(obj);
  console.log(arr); // [Symbol(abc), Symbol(), Symbol(undefined)]
  arr.forEach(function (item) {
    console.log(item, obj[item]); 
    /*
      Symbol(abc) "123"
      Symbol() "maisa"
      Symbol(undefined) "a3"
    */
  })
}
```
+ Reflect.ownKeys(obj)
取出对象中的所有属性值（Symbol类型的key值），返回一个数组。
```javascript
{
  let a1 = Symbol.for('abc');
  let a2 = Symbol();
  let a3 = Symbol.for();
  let obj = {
    [a1]: '123',
    [a2]: 'maisa',
    [a3]: 'a3',
    'abc': 345,
    'c': 456
  }
  let arr = Reflect.ownKeys(obj);
  console.log(arr); //  ["abc", "c", Symbol(abc), Symbol(), Symbol(undefined)]
  arr.forEach(function (item) {
    console.log(item, obj[item]); 
    /*
      abc 345
      c 456
      Symbol(abc) "123"
      Symbol() "maisa"
      Symbol(undefined) "a3"
    */
  })
}
```
