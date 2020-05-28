---
title: ES6梳理-数值扩展
categories:
  - ES6
  - 数值扩展
tags:
  - ES6
  - 数值扩展
date: 2019-05-01 10:00:00
---

## ES6把一些全局的属性移植到内置的类上，比如把全局的parseInt()方法移到了Number的原型对象上
```javascript
{
  let num = 66.3;
  console.log(parseInt(num))
  console.log(Number.hasOwnProperty('parseInt'))
} 
```
<!-- more -->
### ES6新增的数值操作：
```javascript
// js二进制八进制的表示方法
{
  console.log('二进制', 0B111111); // 二进制 63
  console.log('八进制', 0o17); // 十六进制 15
  console.log('十六进制', 0xff); // 十六进制 255
} 
// 判断一个数是否是有限的
{
  console.log('15', Number.isFinite(15)); // 15 true
  console.log('NaN', Number.isFinite(NaN)); // NaN false
  console.log('1/0', Number.isFinite(1/0)); // 1/0 false

  console.log('NaN', Number.isNaN(NaN)); // true
  console.log('0', Number.isNaN(0)); // false
} 
// 判断是不是整数
{
  console.log('25', Number.isInteger(25)); // 25 true
  console.log('25.0', Number.isInteger(25.0)); // 25.0 true
  console.log('25.1', Number.isInteger(25.1)); // 25.1 false
  console.log('25.1字符串', Number.isInteger('25.1')); // 25.1字符串 false
}
// 数值范围是-2的53次幂（不包含）到2的53次幂（包含）
{
  console.log(Number.MAX_SAFE_INTEGER) // 9007199254740991 表示数的最大的上限
  console.log(Number.MIN_SAFE_INTEGER) // -9007199254740991 表示数的最小的下限
  console.log(Number.NaN) // NaN
}
// 判断一个数是否在安全范围之内
{
  console.log('10', Number.isSafeInteger(10)); // 10 true
  console.log('10字符串', Number.isSafeInteger('10')); // 10字符串 false
  console.log('NaN', Number.isSafeInteger(NaN)); // NaN false
}
// 返回小数的整数部分
{
  console.log('4.1', Math.trunc(4.1)); // 4.1 4
  console.log('4.5', Math.trunc(4.5)); // 4.5 4
}
// 判断一个是正数、负数还是0， 
{
  // Math.sign()的返回值是四种情况，分别为： 0 1 -1 NaN
  console.log('-5', Math.sign(-5)); // -5 -1
  console.log('-5', Math.sign(0)); // -5 0
  console.log('5', Math.sign(5)); // 5 1
  console.log('5字符串', Math.sign('5')); // 5字符串 1
  console.log('NaN', Math.sign(NaN)); // NaN NaN
}
// 求一个数的立方根
{
  console.log('8的立方根', Math.cbrt(8)); // 8的立方根 2
  console.log('2的立方根', Math.cbrt(2)); // 2的立方根 1.2599210498948732
  console.log('-1的立方根', Math.cbrt(-1)); // -1的立方根 -1
}
// 还新增了三角函数和对数API等
```

