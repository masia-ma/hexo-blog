---
title: ES6梳理-数据结构对比
date: 2019-07-08 16:58:54
tags:
---
# 数据结构对比
理解什么情况下用Map，什么情况下用Set
+ Map与Array对比
+ Set与Array对比 
+ Map与object对比
+ Set与object对比
<!-- more -->

### Map与Array对比
```javascript
// Map与Array对比，增删改查
{
  let map = new Map(); // 
  let array = [];
  // 增
  map.set('t', 1);
  array.push({t: 1});
  console.log('map', map); // map Map(1) {"t" => 1}
  console.log('array', array); // array [{ t: 1 }]

  // 查
  let map_exist = map.has('t');
  let array_exist = array.find(i => i.t);
  console.log('map_exist', map_exist); // map_exist true
  console.log('array_exist', array_exist); // array_exist {t: 1}

  // 改
  map.set('t', 2);
  array.forEach(i => {
    i.t && (i.t = 2);
  })
  console.log('修改后的map', map); // 修改后的map Map(1) {"t" => 2}
  console.log('修改后的Array', array) // 修改后的Array [{ t: 2 }]

  // 删
  map.delete('t'); 
  let index = array.findIndex(item => item.t);
  array.splice(index, 1);
  console.log('删除后的map', map); // 删除后的map Map(0) {}
  console.log('删除后的array', array); // 删除后的array []
}
```

### Set与Array对比
```javascript
// Set与Array对比，增删改查
{
  let set = new Set(); 
  let array = [];
  // 增
  let obj = {t: 1};
  console.log(set.add(obj)); // Set(1) {{…}} set.add()返回的是操作之后的Set对象
  console.log(array.push({t: 1})); // 1  array.push()返回的是操作之后的数组长度
  console.log(set); // Set(1) {Object {t: 1}}
  console.log(array); // [{t: 1}]
  // 查
  let set_exist = set.has(obj) // 返回值是true
  let array_exist = array.find(item => item.t);
  console.log(set_exist); // true
  console.log(array_exist); // {t: 1}
  // 改（这里的改对于数组，不是按照数组下标获取数组元素的值，而是获取数组元素对象中t的值，所以要先forEach查询）
  console.log(set.forEach(item => item.t ? item.t = 2 : '')); // undefined 方法forEach()的返回值是undefined
  console.log(array.forEach(item => item.t ? item.t = 2 : '')); // undefined undefined
  console.log(set); // Set(1) {Object {t: 2}} ，其中 {t: 2} ，则证明被修改了
  console.log(array); // [{t: 2}]
  // 删 （两者也同样通过forEach先找到，再删除）
  set.forEach(item => item.t ? set.delete(item) : '');
  let index = array.findIndex(item => item.t);

  array.splice(index, 1);
  console.log(set); // Set(0) {}
  console.log(array); // []
}  
```

### Map与object对比
```javascript
// Map、Set与object对比，增删改查
{
  let item = {t: 1};
  let map = new Map();
  let set = new Set();
  let obj = {};

  // 增
  map.set('t', 1);
  set.add(item);
  obj['t'] = 1;
  console.info({
    map, // map: Map(1) {"t" => 1}
    set, // set: Set(1) {Object {t: 1}}
    obj // obj: {t: 1}
  })
  // 查
  console.info({
    map_exist: map.has('t'), // map_exist: true
    set_exist: set.has(item), // obj_exist: true
    obj_exist: 't' in obj // set_exist: true
  })

  set.forEach(item => {
    item.t && console.log(item);
  })
  // 改
  map.set('t', 2); // 覆盖了原来的值
  set.forEach(item => item.t && (item.t = 2));
  obj['t'] = 2;
  console.info({
    map, // map: Map(1) {"t" => 2}
    set, // set: Set(1) {Object {t: 2}}
    obj // obj: {t: 2}
  })
  // 删
  map.delete('t');
  set.delete(item); 
  // 或 set.forEach(item => item.t && (set.delete(item)));
  delete obj['t'];
  console.info({
    map, // map: Map(0) {}
    set, // set: Set(0) {}
    obj // obj: {} 
  })
}
```
总结：
1. 能使用Map，不使用数组，里面越复杂，越适合使用Map
2. 要求数据存储的唯一性，则使用Set
3. 优先使用Map，放弃使用Array和Object