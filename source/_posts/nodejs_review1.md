---
title: node.js复习1
categories:
  - node.js
tags:
  - node.js
abbrlink: f9876e6a
date: 2019-04-26 09:00:00
---
## 深入研究module.exports与exports各种组合用法，其意义在于更加深入理解这两个对象的本质特点，以便以后灵活使用。
<!-- more -->
## 归根结底是探究module.exports与exports的关系，以及对象引用的指向问题
系统默认exports = module.exports，也就是说他们两个指向了同一个对象，下面来看三种情况：
假设在文件demo.js中：
```javascript
var a = 2;
function fun () {
  console.log('this is a function')
}
var b = 3
```
+ 情况一：
    ```javascript
    module.exports.a = a
    module.exports.fun = fun
    exports.b = b
    ```
    那么module.exports和exports向外暴露的是同一个对象，在test.js文件中访问demo.js：
    ```javascript
    var demo = require('./demo')
    console.log(demo)
    ```
    并在控制台打印，得到：
    ```bash
    {a: a, fun: [Function: fun], b: 3}
    ```

+ 情况二：
    ```javascript 
    module.exports = { //这样的写法，实际上已经将module.export指向一个新的对象了，但是exports指向的还是原来默认的对象
      a: a,
      fun: fun
    }
    exports.b = b //但是exports指向的是之前默认的对象
    ```
    最后向外暴露的是module.exports实际指向的对象，此时exports.x也失去了向外部暴露属性的功能，在test.js文件中访问demo.js
    ```javascript
    var demo = require('./demo')
    var {b} = require('./demo')
    console.log(demo)
    console.log(b)
    ```
    在控制台得到的结果为：
    ```bash
    {a: a, fun: [Funtion: fun]}
    undefined
```

+ 情况三：
    使用module.exports整体暴露
    ```javascript
    module.exports = {
      a: a,
      fun: fun,
      b: b
    }
    ```
    在test中按需引入
    ```javascript
    var {a, fun} = require('./demo')
    console.log(a)
    console.log(fun)
    ```
    在控制台得到的结果为：
    ```bash
    12
    [Function: fun]
```
+ 注意：
  1. 如果要使用module.exports，在使用时尽量整体引入。
  2. 如果要使用exports.x，千万不要在同一个文件中module.exports = {}，但是可以使用module.exports.xxx
