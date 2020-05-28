---
title: let关键字使用
categories:
  - ES6
tags:
  - ES6
abbrlink: 13f9823b
date: 2019-04-28 10:00:00
---

### let关键字定义之后不能修改，let关键字有作用域限制
<!-- more -->
```html
<div class="nav">
  <button>选项一</button>
  <button>选项二</button>
  <button>选项三</button>
</div>
<div class="con">
  <p>内容一</p>
  <p>内容二</p>
  <p>内容三</p>
</div>
```

```javascript
// 使用常规的方式，变量声明和定义全部使用var关键字
var buttons = document.querySelectorAll('button');
var ps = document.querySelectorAll('p');

for (var i = 0; i < buttons.length; i ++) {
  buttons[i].index = i;
  buttons[i].onclick = function () {
    console.log(this.index); // 在这里只能通过自定义属性index来拿到外部循环的i
    for (var i = 0; i < buttons.length; i ++) {
      buttons[i].className = '';
      ps [i].className = '';
    }
    this.className = 'active';
    ps[this.index].className = 'active'
  }
}


// 使用const关键限制i的作用域
var buttons = document.querySelectorAll('button');
var ps = document.querySelectorAll('p')

for (let i = 0; i < buttons.length; i ++) {
  buttons[i].onclick = function () {
    console.log(i) // 这里可以拿到循环中的i
    for (let i = 0; i < buttons.length; i ++) {
      buttons[i].className = '';
      ps[i].className = '';
    }
    this.className = 'active';
    ps[i].className = 'active';
  }

}
```