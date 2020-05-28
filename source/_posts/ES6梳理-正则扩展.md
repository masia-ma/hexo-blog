---
title: ES6梳理-正则扩展
categories:
  - ES6
  - 正则扩展
tags:
  - ES6
  - 正则扩展
abbrlink: 13f9823b
date: 2019-04-29 14:00:00
---

# 正则扩展
1. 正则对象
ES5中的正则对象
<!-- more -->
```javascript
{
  // 第一种写法： 第一个参数是字符串，第二个参数是修饰符
  let regex = new RegExp('xyz', 'i'); // 第二个参数i是忽略大小写的意思
  // 第二种写法：只有一个参数：正则表达式
  let regex2 = new RegExp(/xyz/i);
  // 当然还可以使用字面量的方式去定义一个正则对象 let regex2 = /xyz/i;
  console.log(regex.flags) // flags是ES6新增加的一个获取正则对象修饰符的属性，输出正则对象的修饰符
  console.log(regex2.flags) // 输出正则表达式的修饰符
  console.log(regex.test('xyz123'), regex2.test('xyz123'))
  // 输出： true true
}
```
ES6中增加了一种写法：
```javascript
// 第一个参数是正则表达式，还能有第二个参数
// ES6允许第一个参数是正则表达式，也允许第二个参数去覆盖正则表达式的修饰符
let regex3 = new RegExp(/xyz/ig, 'i');
console.log(regex3.flags) // 输出 'i'
```
2. 修饰符
ES5中常见的修饰符是`i`和`g`,ES6增加了`y`和`u`
+ `y`修饰符
```javascript
{
  let s = 'bbb_bb_b';
  let a1 = /b+/g;
  let a2 = /b+/y;
  console.log('one', a1.exec(s), a2.exec(s)); // 第一次执行
  // 输出：都匹配到了第一个字符'bbb'
  // ["bbb", index: 0, input: "bbb_bb_b", groups: undefined] 
  // ["bbb", index: 0, input: "bbb_bb_b", groups: undefined]
  console.log('two', a1.exec(s), a2.exec(s)); // 第二次执行
  // 输出：使用g修饰符的匹配到了 'bb', 使用y修饰符的没有匹配到，并且返回了null，即没有匹配成功
  // ["bb", index: 4, input: "bbb_bb_b", groups: undefined]
  // null
  console.log('three', a1.exec(s), a2.exec(s)); // 第三次执行
  // 输出：使用g修饰符的匹配到了 'b', 使用y修饰符的输出结果和第一次相同
  //["b", index: 7, input: "bbb_bb_b", groups: undefined]
  //["bbb", index: 0, input: "bbb_bb_b", groups: undefined]


  // ES6中的sticky属性, sticky 黏(性)的;一面带黏胶的;闷热的
  console.log(a1.sticky, a2.sticky) // 来判断正则对象是否开启了带y的修饰符的作用
  //          false      true
}
```
+ `u`修饰符
u其实是unicode的缩写
```javascript
{
  console.log('u-1', /^\uD830/.test('\uD830\DC2A')); // true
                              // 不加u会把后面的内容当两个字符去解析
  console.log('u-2', /^\uD830/u.test('\uD830\DC2A')); // false
                            //加u会把后面test中的一连串内容当作一个字符去解析


  console.log(/\u{61}/.test('a'));
  console.log(/\u{61}/u.test('a')); // 如果{}内放的是unicode编码，且你要识别unicode编码，则一定要加u
}
```
在unicode中，一个编码组合为两个字节，表示一个字符
+ 小知识点
在ES5中,`.`代表任意一个字符，但是其实他只能匹配两个字节的字符 
```javascript
{
  console.log(`\u{20BB7}`) // "𠮷"
              // unicode的编码超过了4位，那也就意味着这个字符的大小超过了2字节
  let s = '𠮷';
  console.log(/^.$/.test(s)) // false, 说明"."并没有成功匹配到"𠮷"这个字符，那也就验证了"."只能匹配2字节的unicode字符
  console.log(/^.$/u.test(s)) // true, 但是加上正则表达式修饰符"u","."就可以匹配这个超过4位的unicode字符了
}
{
  console.log(/𠮷{2}/.test('𠮷𠮷')) // false ,不加u修饰符，识别不了超过2字节的字符
  console.log(/𠮷{2}/u.test('𠮷𠮷')) // true ,加了u修饰符，能够识别超过2字节的字符
}

// 总结："."只能匹配2字节的unicode字符，如果我们要匹配的字符串中有超过2字节的unicode字符，请加u
```
