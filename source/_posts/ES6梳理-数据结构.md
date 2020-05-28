---
title: ES6梳理-数据结构
date: 2019-07-08 16:57:54
tags:
 - ES6
 - 数据结构
categories: 
 - ES6
 - 数据结构
---
# 数据结构
+ set用法
+ Map用法
+ WeakSet用法
+ WeakMap用法
<!-- more -->

set可以理解为数组，但是该集合中的元素不能重复
map可以理解成为对象，但是普通对象中的key一定是一个字符串，map中的key可以是其他数据类型，比如一个数组、一个对象都可以做他的key

### set用法
+ 基本用法
```javascript
{
  // Set的普通定义方法
  let list = new Set();
  list.add(5); // 向set中增加值，要使用add()方法
  list.add(7);
  console.log(list);
  console.log('size', list.size); // size 2
} 
```
{% asset_img set.png Set()的基本用法 %}
+ Set(arr)将传入数组转化为一个集合
```javascript
{
  let arr = [1, 2, 3, 4];
  let list = new Set(arr); // 将arr数组转化为数组集合
  console.log(list);
  console.log(list.size); // 4
}
```
{% asset_img Set(arr).png console.log(list); %}
+ Set类型的元素必须是唯一的，不能重复
```javascript
{
  let list = new Set();
  list.add(5); 
  list.add(7);
  list.add(5); // 没有添加进去，并不会报错
  console.log(list); // Set(2) {5, 7}
  console.log('size', list.size); // size 2
}
// 利用这个特性，我们可以做数组去重
{
  arr = [1, 1, 2, 2, 3, 3, 4, '4'];
  let list = new Set(arr);
  console.log('unique', list) // unique Set(5) {1, 2, 3, 4, "4"} 字符串4和数字4不算重复
  let newArr = Array.from(list);
  console.log(newArr); // [1, 2, 3, 4, '4']
}
```
+ Set实例对象的常用方法：
```javascript
{
  let arr = ['add', 'delete', 'clear', 'has']
  let list = new Set(arr);
  console.log(list.add('masia')); // 返回值为添加之后的集合：{"add", "delete", "clear", "has", "masia"}
  console.log(list.has('add')); // true
  console.log(list.delete('add')); // true
  console.log(list); // Set(3) {"delete", "clear", "has"}
  console.log(list.clear('clear')); // undefined
  console.log(list); // Set(0) {}
}
```
+ Set实例对象的遍历
```javascript
{
  let arr = ['add', 'delete', 'clear', 'has']
  let list = new Set(arr);
  for (let key of list.keys()) {
    console.log('keys', key); // 但好像打印的是value值
    /*
      keys add
      keys delete
      keys clear
      keys has
    */
  }
  for (let value of list.values()) {
    console.log('keys', value); // 结果仍然是value值
    /*
      keys add
      keys delete
      keys clear
      keys has
    */
  }
  for (let value of list) {
    console.log('keys', value); // 直接遍历list，结果也是value值
    /*
      keys add
      keys delete
      keys clear
      keys has
    */
  }
  for (let [key, value] of list.entries()) {
    console.log('keys', key, 'value', value); // 当然也可以调用entries()方法，结果也是value值
    /*
      keys add value add
      keys delete value delete
      keys clear value clear
      keys has value has
    */
  }
  list.forEach(function (value, key) {
    console.log('key', key, 'value', value) // 当然也可以使用forEach()方法
    /*
      key add value add
      key delete value delete
      key clear value clear
      key has value has
    */
  });
}
```
### WeakSet用法
+ WeakSet基本用法：
与Set的区别：
1. WeakSet的元素的key值必须是对象，而Set的元素可以是任意类型
2. WeakSet的元素的key值是一个弱引用(元素值都是地址的引用，而且并不会检测地址所对应的对象是否被垃圾回收了)
3. WeakSet没有size属性
4. WeakSet没有clear()方法
5. WeakSet不能遍历
与Set的相同点: has(), delete(), add()方法使用相同
```javascript
{
  let obj = { a: 1 };
  let obj2 = { b: 2 };
  let weakList = new WeakSet();
  weakList.add(obj);
  weakList.add(obj2);
  console.log(weakList);
}
```
{% asset_img WeakSet().png console.log(weakList); %}
### Map的用法
Map的特性：key可以是任何数据类型
+ Map的基本用法
```javascript
{
  let map = new Map();
  let arr = ['123'];
  map.set(arr, 456) // Map的实例对象添加元素使用set()
  // 使用数组arr做key， value为456
  console.log(map); // Map(1) {Array(1) => 456}
  console.log(map.get(arr)); // 456  使用get()获取属性值
}
```
+ Map的第二种定义方式
Map()，参数是数组，数组中又有两个数组
```javascript
{
  let arr = ['a', 123]; // 第一个数组必须包含两项值
  let arr2 = ['b', 456];
  let map = new Map([arr, arr2]);
  console.log(map) // Map(2) {"a" => 123, "b" => 456}
  console.log(map.set(arr, 999)); // Map(3) {"a" => 123, "b" => 456, Array(2) => 999}
  console.log(map.size); // 2
  console.log(map.get('a')  ); // 123
  console.log(map.delete('a'), map); // true Map(2) {"b" => 456, Array(2) => 999}
  console.log(map.clear(), map); // undefined Map(0) {}
}
```
{% asset_img Map([]).png console.log(map) %}
+ Map实例对象的遍历和Set一模一样
### WeakMap
与Map的区别：（即Set和WeakSet的区别）
1. WeakMap的元素的key值必须是对象，而Set的元素可以是任意类型
2. WeakMap的元素的key值是一个弱引用(元素值都是地址的引用，而且并不会检测地址所对应的对象是否被垃圾回收了)
3. WeakMap没有size属性
4. WeakMap没有clear()方法
5. WeakMap不能遍历
```javascript
{
  let obj1 = { a: 1 }
  let obj2 = { b: 1 }
  let weakMap = new WeakMap();
  weakMap.set(obj1, 'aa');
  weakMap.set(obj2, 'bb');
  console.log(weakMap); // WeakMap {{…} => "aa", {…} => "bb"}
  console.log(weakMap.get(obj1)); // aa
}
```