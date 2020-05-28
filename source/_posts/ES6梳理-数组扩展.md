---
title: ES6梳理-数组扩展
date: 2019-07-08 10:48:33
categories:
  - ES6
  - 数组扩展
tags:
  - ES6
  - 数组扩展
---

## 数组扩展
<!-- more -->
### 常用方法
+ Array.of()把一组数据组合成一个数组
```javascript
{
  let arr = Array.of(3, 4, 7, 9, 11);
  console.log(arr) // 数组扩展

  // 如哦Array.of()中不传任何参数,那么返回一个空数组
  let arr2 = Array.of();
  console.log(arr2)  // []
} 
```
+ Array.from(),此方法不会改变原来的数组
  - 只写一个参数（对象），把一个伪数组转化成一个真正的数组
```javascript
{ 
  let p = document.getElementsByTagName('p');
  
  let pArr = Array.from(p)
  console.log(p);
  console.log(pArr);
}
```
{% asset_img Array.from().png Array.from()把一个伪数组转化成一个真正的数组 %}
 - 第一个参数是数组，第二个参数是函数(这个函数起了一个map的作用，对前面的数组的每一个数组项都做了处理)
```javascript
// 只对数组项为数字活数字字符串生效，不然就处理为NaN
{
  console.log(Array.from([1, 3, 5], function (item) { return item*2 }));
  // (3) [2, 6, 10]
  console.log(Array.from([1, 3, '5'], function (item) { return item*2 }));
  // (3) [2, 6, 10]
  console.log(Array.from([1, { a: 2 }, 'fasf'], function (item) { return item*2 }));
  // (3) [2, NaN, NaN]
}
```
+ 数组对象.fill()
 - 只有第一个参数，把数组中的每一个值都替换为目标值，此方法会改变原来的数组
```javascript
{
  let arr = [1, 'masia0', { a: 0 }];
  let newArr = arr.fill(7);
  console.log(arr); // (3) [7, 7, 7]
  console.log(newArr); // (3) [7, 7, 7]
}
```
 - 有三个参数， 第一个参数为目标值，第二个参数是其实位置下标，第三个位置是结束位置下标，
```javascript
{
  let arr = [1, 'masia0', { a: 0 }, 33, 55];
  let newArr = arr.fill(7, 1, 3); // 替换index为0（包含0）到index为2（包含2）位置上的值
  console.log(arr); // [1, 7, 7, 33, 55]
  console.log(newArr); // [1, 7, 7, 33, 55]
}
```
+ 数组的keys(), values(), entyies()
```javascript
{
  let arr = [1, 'masia0', { a: 0 }, 33, 55];
  console.log(arr.keys()); // Array Iterator {} 返回了一个数组迭代对象
  console.log(arr.values()); // Array Iterator {}
  // 这两个迭代对象可以使用for in取出来
  for (let index of arr.keys()) {
    console.log('key', index);
    /*
      key 0
      key 1
      key 2
      key 3
      key 4
    */
  }
  for (let value of arr.values()) {
    console.log('value', value);
    /*
      value 1
      value masia0
      value {a: 0}
      value 33
      value 55
    */
  }
  for (let [index, value] of arr.entries()) {
    console.log(index, value);
    /*
      0 1
      1 "masia0"
      2 Object
      3 33
      4 55
    */
  }
}
```
+ copyWithin()在当前数组内部，把指定位置上的成员复制到其他位置上(会改变原来的数组)
```javascript
{
  let arr = [1, 2, 3, 4, 5, 6];
  let newArr = arr.copyWithin(0, 2, 5)
  // 从0位置开始替换，读取的数据从位置2开始（包含2位置）到4位置（包含4位置）
  console.log(arr); // [3, 4, 5, 4, 5, 6]
  console.log(newArr); // [3, 4, 5, 4, 5, 6]
}
```
+ find()和findIndex()，检查数组中是否包含满足某个条件的值，find()返回该值，findIndex()返回该值的数组下标
```javascript
{
  let arr =  [1, 2, 3, 4, 5, 6];
  console.log(arr.find(function (item) { return item > 3 })); 
  // 返回数组中第一个满足条件的值 4
  console.log(arr.findIndex(function (item) { return item > 3 })); 
  // 返回数组中第一个满足条件的值的下标 3
  console.log(arr); // [1, 2, 3, 4, 5, 6] 不会改变原数组的值
}
```
+ includes(),检查数组中是否包含具体的某个值
比find和findIndex更省事，而且还解决了NaN的问题
```javascript
{
  const obj = { a: 1 };
  let arr =  [1, 2, 3, 4, 5, 6, NaN, 'masia', obj];
  console.log(arr.includes(1)); // true
  console.log(arr.includes('masia')); // true
  console.log(arr.includes(NaN)); // true
  console.log(arr.includes(obj)); // true
  console.log(arr.includes(15)); // false
}
```
### 复习常用的数组方法
1. ArrayObj.forEach(),第一个参数是数组项本身，第二个参数为数组项下标
```javascript
{
  let arr = [1, 2, 3, 4];
  arr.forEach(function (item, index) {
    console.log(index, item);
    /*
      0 1
      1 2
      2 3
      3 4
    */
  })
}
```
2. ArrayObj.map()对每一项值位数字的数组项做一个遍历操作，并不会改变原数组  
```javascript
{
  let arr = [1, 2, 3, 4, '5', 'masia', {}];
  let newArr = arr.map(function (item) {
    return item*2;
  })
  console.log(arr); //  [1, 2, 3, 4, "5", "masia", {…}]
  console.log(newArr); // [2, 4, 6, 8, 10, NaN, NaN]  可以看出对值为数字的字符串仍然适用
}
```
3. ArrayObj.filter()
此方法是将所有元素进行判断，将满足条件的元素作为一个新的数组返回
```javascript
{
  let arr = [1, 2, 3, 4, '5', 'masia', {}];
  let newArr = arr.filter(function (item) {
    return item > 2;
  })
  console.log(arr); //  [1, 2, 3, 4, "5", "masia", {…}]
  console.log(newArr); // [3, 4, "5"] 可以看出对值为数字的字符串仍然适用
  let newArr2 = arr.filter(function (item) {
    return item == 2;
  })
  console.log(newArr2);  // [2] 不管有多少个满足条件的数组项，返回的值一定是一个数组
  let newArr3 = arr.filter(function (item) {
    return item == 0;
  })
  console.log(newArr3);  // [] 不管有多少个满足条件的数组项，返回的值一定是一个数组
}
```
4. ArrayObj.every()
此方法是将所有元素进行判断返回一个布尔值，如果所有元素都满足判断条件，则返回true，否则为false。
```javascript
{
  let arr = [1, 2, 3, 4, '5', 'masia', {}];
  let flag = arr.every(function (item) {
    return item > 0;
  })
  console.log(flag); // false
  console.log(arr); // [1, 2, 3, 4, "5", "masia", {…}] 不会改变原数组
  let arr2 = [1, 2, 3, 4];
  let flag2 = arr2.every(function (item) {
    return item > 0;
  })
  console.log(flag2); // true
}
```
5. ArrayObj.some()
此方法是将所有元素进行判断返回一个布尔值，如果存在元素都满足判断条件，则返回true，若所有元素都不满足判断条件，则返回false。
```javascript
{
  let arr = [1, 2, 3, 4, '5', 'masia', {}];
  let flag = arr.some(function (item) {
    return item > 0;
  })
  console.log(flag); // true
  console.log(arr); // [1, 2, 3, 4, "5", "masia", {…}] 不会改变原数组
  let arr2 = [1, 2, 3, 4];
  let flag2 = arr2.some(function (item) {
    return item < 0;
  })
  console.log(flag2); // false
}
```
6. ArrayObj.reduce()
此方法是所有元素调用传入的回调函数，返回值为最后结果
```javascript
{
  let arr = [1, 2, 3, 4, '5', 'masia', {}];
  let res = arr.reduce(function (a, b) {
    console.log(a, b)
    return a + b;
  })
  console.log(res); 
  console.log(arr); 
  let arr2 = [1, 2, 3, 4];
  let res2 = arr2.reduce(function (a, b) {
    console.log(a, b)
    return a + b;
  })
  console.log(res2); 
}
```
{% asset_img ArrayObj.reduce().png ArrayObj.reduce() %}

7. ArrayObj.push()
此方法是在数组的后面添加新加元素，此方法改变了原数组，返回值为修改过后的数组的长度
```javascript
{
  let arr = [1, 2, 3];
  let res = arr.push(4);
  console.log(arr); // [1, 2, 3, 4]
  console.log(res); // 4 返回值为修改过后的数组的长度
  let res2 = arr.push(5, 6);
  console.log(arr); // [1, 2, 3, 4, 5, 6]
  console.log(res2); // 6 返回值为修改过后的数组的长度
}
```
8. ArrayObj.pop()
此方法删除了数组的最后一个元素，改变了原数组，返回值为删除的数组项的值
```javascript
{
  let arr = [1, 2, 6];
  let res = arr.pop(); // pop()没有参数，传参数没用
  console.log(arr); // [1, 2]
  console.log(res); // 6 返回值为删除的数组项的值
}
```
9. ArrayObj.shift()
此方法删除了数组的第一个元素，改变了原数组，返回值为删除的数组项的值
```javascript
{
  let arr = [99, 2, 6];
  let res = arr.shift(); // shift()没有参数，传参数没用
  console.log(arr); // [2, 6]
  console.log(res); // 99 返回值为删除的数组项的值
}
```
10. ArrayObj.unshift()
此方法是将一个或多个元素插入到数组的开头，改变了原数组，返回值为修改后的数组长度
```javascript
{
  let arr = [99, 2, 6];
  let res = arr.unshift(1, 2); 
  console.log(arr); // [1, 2, 99, 2, 6]
  console.log(res); // 5 返回值为修改后的数组长度
}
```
11. ArrayObj.concat()
此方法是一个可以将多个数组拼接成一个数组，返回值为合并后的新数组，此方法不会改变原来的数组
```javascript
{
  let arr = [1, 1, 1];
  let arr2 = [2, 2, 2];
  let arr3 = [3, 3, 3];
  let newArr = arr.concat(arr2, arr3).concat(arr3);
  // 或者 let newArr = arr.concat(arr2).concat(arr3);
  console.log(newArr); // [1, 1, 1, 2, 2, 2, 3, 3, 3]
  console.log(arr); // [1, 1, 1] 此方法不会改变原来的数组
  console.log(arr2); // [2, 2, 2] 此方法不会改变原来的数组
  console.log(arr3); // [3, 3, 3] 此方法不会改变原来的数组
}
```
12. ArrayObj.toString()
此方法将数组转化为字符串，不会改变原数组
```javascript
{
  let arr = [1, 1, 1];
  let str = arr.toString();
  console.log(str); // 1,1,1
  console.log(arr); // [1, 1, 1]
}
```
13. ArrayObj.join()
此方法将数组转化为字符串，与toString()的区别是，join('#')可以设置元素之间的间隔
此方法与StringObj.split()方法正好相反
```javascript
{
  let arr = [1, 1, 1];
  let str = arr.join(); // 不加参数，返回的结果个toString()一样
  let str2 = arr.join(''); // 加参数空
  let str3 = arr.join('##'); // 加参数##
  console.log(str); // 1,1,1
  console.log(str2); // 111
  console.log(str3); // 1##1##1
  console.log(arr); // [1, 1, 1] 不改变原数组
}
```
14. ArrayObj.splice(开始位置， 删除的个数，元素)
万能方法，可以实现增删改，会改变原数组，返回删除元素组成的数组
```javascript
// 删除一组数组项
{
  let arr = [1, 2, 3, 4, 5];
  let arr1 = arr.splice(2, 3)
  console.log('arr', arr) // [1, 2]
  console.log('arr1', arr1) // [3, 4, 5]
}
// 删除一组数组项，并且在原数组删除的位置开始添加一组元素
{
  let arr = [1, 2, 3, 4, 5, 6];
  let arr1 = arr.splice(2, 3, 'haha', 'lala')
  console.log('arr', arr) // [1, 2, "haha", "lala", 6]
  console.log('arr1', arr1) // [3, 4, 5]
}
// 实现指定位置数组元素替换
{
  let arr = [1, 2, 3, 4, 5, 6];
  let arr1 = arr.splice(2, 1, 'haha')
  console.log('arr', arr) // 1, 2, "haha", 4, 5, 6]
  console.log('arr1', arr1) // [3]
}
// 实现指定位置插入一个元素
{
  let arr = [1, 2, 3, 4, 5, 6];
  let arr1 = arr.splice(2, 0, 'haha')
  console.log('arr', arr) // [1, 2, "haha", 3, 4, 5, 6]
  console.log('arr1', arr1) // []
}
```
15. Array.isArray()
判断一个对象是不是数组，返回的是布尔值
```javascript
{
  let arr = [99, 2, 6];
  let res = Array.isArray(arr);
  console.log(res); // true
}
```
### 附加
+ 把一个一维数组转为二维数组
```javascript
{
  arr = [1, 1, 1, 2, 2, 2, 3, 3];
  function twoDimensionalArray (n) {
    let newArr = [];
    let sArr = [];
    arr.forEach(function (item) {
      if (sArr.length == n) { // 如果小数组满了，就让他指向一片新的空间
        sArr = [];
      }
      if (sArr.length == 0) { // 如果小数组为空，就把他添加到大数组中
        newArr.push(sArr);
      }
      sArr.push(item);
    })
    return newArr;
  }
  console.log(twoDimensionalArray(3)); // (3) [Array(3), Array(3), Array(2)]
}
```