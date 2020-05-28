---
title: ES6梳理-字符串扩展
categories:
  - ES6
  - 字符串扩展
tags:
  - ES6
  - 字符串扩展
date: 2019-04-30 14:00:00
---

# 字符串新增的特性：
+ Unicode表示法
+ 遍历接口
+ 模板字符串
+ 新增方法（10种）
<!-- more -->

## Unicode表示法
+ 引入： 
```javascript
{
  console.log('a', '\u0061'); // a a
  console.log('s', '\u20BB7'); // s ₻7
                    //这个超过了0xffff，也就是超过了2字节，他会把20BB和7分别拿出来解析，20BB不是unicode的编码，所以不能识别 
} 
```
+ 在ES6中处理超过2字节的unicode编码，使用{}包起来，比如：
```javascript
{
  console.log('s', '\u{20BB7}'); // s 𠮷
} 
```
+ ES5和ES6处理码值： 
```javascript
{
  let a = 'a'; 
  let s = '𠮷';
  console.log(a.length) // 1
  console.log(s.length) // 2
  console.log('0', s.charAt(0)) // 0 � 取s的Unicode编码第1个位置的字符
  console.log('1', s.charAt(1)) // 1 � 取s的Unicode编码第2个位置的字符
  console.log('at0', s.charCodeAt(0).toString(16)) // at0 d842 
  console.log('at1', s.charCodeAt(1).toString(16)) // at1 dfb7
  console.log('a', a.charCodeAt(0).toString(16)) // a 61

//ES6中的codePointeAt()
let s1 = '𠮷a'
console.log(s1.length)
console.log(s1.codePointAt(0).toString(16)) // 20bb7 取了4字节的码值，codePointAt()方法会先去判断指定字符是2字节的还是4字节的，然后再去获得Unicode编码
console.log(s1.codePointAt(1).toString(16)) // dfb7
console.log(s1.codePointAt(2).toString(16)) // 61
}
{
  // ES5中方法
  console.log(String.fromCharCode("0x20bb7")); // ஷ
  // ES6中方法
  console.log(String.fromCodePoint("0x20bb7")) // 𠮷

  // 两者区别就是能否处理超过2字节的Unicode字符
}
```
## ES5和ES6的字符串遍历
```javascript
{
  // 常用的字符遍历器接口
  let str = '\u{20bb7}abc';
  // ES6处理
  for (let code of str) { 
    console.log('ES6', code) // ES6 𠮷
                              // ES6 a
                              // ES6 b  
                              // ES6 c
  }

  // ES5不能处理
  for (let i = 0; i < str.length; i++) {
    console.log('es5', str[i]) // es5 �
                                // es5 �
                                // es5 a
                                // es5 b
                                // es5 c
  }
  let arr = str.split("") // split("")也是按照2个字节为一个字符来处理的，所以把1个4字节的字符识别成了2个2字节的字符
  console.log(arr) // ["�", "�", "a", "b", "c"] 
  {
    let str2 = "你好";
    console.log(str2[1]) // "好"
  }
}
```
## ES6新增的经常使用的字符串处理API
```javascript
{
  let str = "string";

  // ES6对字符串进行简单查询
  {
    console.log("includes", str.includes("r")); // includes true
    console.log("start", str.startsWith('str')) // start true
    console.log("end", str.endsWith('ng')) // end true
  }
  
  // ES6重复字符串
  {
    // ES6重复字符串
    console.log(str.repeat(2)); // stringstring
    // ES5重复字符串
    let str2 = str
    str2 += str;
    console.log(str2) // stringstring
    
    // 为String添加一个mRepeat原型方法
    String.prototype.mRepeat = function (n) {
    let str = this.valueOf();
    let str2 = "";
    for (let i = 0; i < n; i ++) {
      str2 += str
    }
    return str2;
    }
    console.log(str.mRepeat(2));
  }

  //模板字符串
  {
    let name = "list";
    let info = "hello world";
    let s = `I am ${name},${info}`;
    console.log(s); // I am list,hello world
  }

  // String.raw()
  {
    console.log(String.raw`Hi\n${1+2}`); // Hi\n3 换行符并没有生效，也就是在"\"前面又隐式的加了一个"\", \n  => \\n
    // 等价于
    console.log(`Hi\\n${1+2}`) // Hi\n3

    console.log(`Hi\n${1+2}`); // Hi
                                // 3 
    console.log('ma\\n') // ma\n 
  }

  // ES7草案
  {
    // padStart和endStart方法非常实用，比如日期补白: 2019-6-2 => 2019-06-02
    console.log('1', padStart(2, '0')); // 01
    // padStart()谷歌识别不了，第一个参数指定字符串长度，如果不够用第二个字符参数往前补白
    console.log('1', padEnd(2, '0')); // 10
    // padEnd()向后补白
  }
}
```
## 标签模板
作用：
1. 防止xss攻击
2. 多语言支持
```javascript
{
  let user = {
    name: 'list',
    info: 'hello world'
  };
  let str = abc`I am ${user.name},${user.info}`;
  console.log(str); // I am ,,,listhello world
  function abc (s, v1, v2) {
    console.log(s, v1, v2); // (3) ["I am ", ",", "", raw: Array(3)] "list" "hello world"
    return s + v1 + v2;
  }
}
```