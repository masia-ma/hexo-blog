---
title: ES6梳理-模块化
date: 2019-07-11 10:48:33
categories:
  - ES6
  - 模块化
tags:
  - ES6
  - 模块化
---

# 模块化
<!-- more -->
+ 批量导出
文件A.js中
```javascript
export let A = 123;
export function test () {
  console.log('test');
}
export class Hello {
  test () {
    console.log('test');
  }
}
```
按需导入：文件A中的变量导入文件B.js中
```javascript
import {A, test, Hello} from './A.js'
console.log(A);
test();
```
全部导入：文件A中的变量导入文件c.js中
```javascript
import * as A form './A.js'
console.log(A.A);
A.test();
```
