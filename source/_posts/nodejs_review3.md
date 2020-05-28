---
title: node.js复习3：使用原生ajax与jquery版本ajax向node服务器请求数据对比
categories:
  - node.js
  - ajax
tags:
  - node.js
abbrlink: 1b1af260
date: 2019-04-28 17:30:00
---

一直以来没有搞清楚使用原生js以post方式发起ajax请求时，<!-- more -->是怎么向服务端发送请求参数的。在使用$.ajax()方法时，不管是get方式，还是post方式，都是可以以对象的方式向服务端提交参数的。但原生js的两种发送请求的方式，发送的参数都是查询字符串方式的，应该叫做表单格式的数据吧。
## $.ajax()发送请求
+ get方式
```javascript
var data = {
  email: '916527363@qq.com',
  password: '123456'
}
$.ajax({
  url: 'http://localhost:8888/login', // 这里要注意的是，既然在已经在传入对象中写了的属性，不要在这里重复写，不会覆盖的，而是会追加
  type: 'get',
  data: data,
  success: function (data) {
    console.log(data)
  }
})
```
+ post方式
```javascript
var data = {
  email: '916527363@qq.com',
  password: '123456'
}
$.ajax({
  url: 'http://localhost:8888/login',
  type: 'post',
  data: data,
  success: function (data) {
    console.log(data)
  }
})
```
## 原生js发送请求
+ get方式
```javascript
function ajaxGet () {
  var xhr = new XMLHttpRequest()
  xhr.open("GET", "http://localhost:8888/login?email=916527363@qq.com&password=123456")
  // 原生get方式的ajax带参数发送请求的时候，只能将参数以查询字符串的方式缀在路由后面
  xhr.onreadystatechange = function () {
    if(xhr.readyState == 4 && xhr.status == 200) {
      console.log(xhr.responseText) // xhr.responseText为响应数据
    }
  }
  xhr.send() // 原生的get请求不能在这里传参数
}
```
+ post方式
```javascript
// 先收集数据
var email = "916527363@qq.com"
var password = "123456"
// 将收集到的数据组合成查询字符串
var msg = "email="+email+"&password="+password
function ajaxPost () {
  var xhr = new XMLHttpRequest()
  xhr.open("POST", "http://localhost:8888/login")
  xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded"); // 声明了数据格式是表单格式的
  // 发送post请求，必须要写这一句，不然xhr.send(msg)不生效
  xhr.onreadystatechange = function () {
    if (xhr.readyState == 4 && xhr.status == 200) {
      console.log(xhr.responseText) // xhr.responseText为响应数据
    }
  }
  xhr.send(msg) // 即使是post请求，这里传的参数依然为查询字符串形式，不能是对象形式的参数
}
```

## 在node服务器中
在node服务器中，完全不需要关心前台是提交的参数是字符串类型的还是对象类型的，只需要知道，前台发送的请求是get方式的，还是post方式的
+ 准备：创建一个基于express框架的node服务器
```javascript
var express = require('express')
var url = require('url')
var query = url.paese()
var bodyParser = require('body-parser') // 该模块用来解析post提交的参数对象


app.use(bodyParser.urlencoded({extended: false})) // 处理传过来的表单数据
app.use(bodyParser.json()) // 处理传过来的json数据

app.listen(3000, function () {
  console.log('app is running at 3000')
})
```
+ 如果前台提交的请求是get方式的，那么后台就需要使用
```javascript
app.get('/login', function (req, res) {
  // 将req.url做处理（其中参数true的作用是，直接将查询字符串转化为了对象）
  var urlObj = url.parse(req.url, true)
  // 拿到参数对象
  var queryObj = urlObj.query
  console.log(queryObj)
})
```
+ 如果前台提交的请求是get方式的，那么后台就需要使用
```javascript
app.post('/login', function (req, res) {
  // 拿到参数对象
  var queryObj = req.body
  console.log(queryObj)
})
```