---
title: node.js复习2
categories:
  - node.js
tags:
  - node.js
abbrlink: 608e3fd0
date: 2019-04-28 08:47:00
---
## 今天主要来复习一下node.js内置模块的用法
<!-- more -->
+ http模块
```javascript
var http = require('http')
// 支持链式调用
var server = http
  .createServer( function (req, res) {
    res.write('Hello World!')
    res.writeHead(404, 'not found', {
        'Content-Type': 'text/html;charset=utf-8',
        'set-Cookie': 'mycookie1=value1',
        'Access-Control-Allow-Origin', 'http://localhost'
      })
    res.end()
  })
  .listen(3000, function () {

  }) // 在listen中拿不到req 和 res
```
- url模块
```javascript 
var url = require('url')
var a = 'http://192.168.15.138:8080/courses/query?credit=2&name=Java'
var urlObj = url.parse(a)  // 将整个字符串url变成对象
var pathname = urlObj.pathname // '/courses/query'
var queryStr = urlObj.query // 'credit=2&name=Java'
var queryObj = url.parse(a, true).query // 将整个查询字符串'credit=2&name=Java'转化为对象 { crdit: 2, name: 'Java'}
```
- querystring模块
```javascript
var querystring = require('querystring')
var queryObj = querystring.parse(queryStr) // 将查询字符串'credit=2&name=Java'转化为参数对象
var queryStr2 = querystring.stringify(queryObj) // 同时，当然也可以将参数对象转化为字符串
```
- path模块
```javascript
var path = require('path')

var a = '/home/masia/a.txt'
console.log('basename:', path.basename(a)) // basename: a.txt
console.log('dirname:', path.dirname(a)) // dirname: c:/home/masia
console.log('extname:', path.extname(a)) // extname: .txt
console.log('isAbsolute', path.isAbsolute(a)) // isAbsolute true
console.log('__dirname', path.join(__dirname)) // __dirname 当前文件所在目录
```
- fs模块
```javascript
var fs = require('fs')
// 文件读取方法
fs.readFile(path.join(__dirname, './a.txt'), function (err, data) {
  if (err) {
    return res.end('read file err')
  } else {
    console.log(data) // 这样读取出来的文件buffer 的 16进制格式的
    console.log(data.toString()) // 需要使用x.toString()方法来将buffer格式的数据转化为字符串形式的数据
  }
})
// 目录读取方法
fs.readdir(path.join(__dirname), function (err, files) {
  if (err) {
    console.log('can not find this directory')
  } else {
    console.log(files) // 输出的是path.join(__dirname)这个目录下面的所有目录名以及文件名
  }
})
```
- res.writeHead()和res.setHeader()的区别
 + writeHead：该方法被调用时发送响应头
 + writeHead必须要在发送数据之前调用
```javascript
res.writeHead(200, 'normal', {
  'Content-Type': 'text/plain;charset=utf-8',
  'set-Cookie': ['mycookie1=value1', 'mycookie2=value2']
  'Access-Control-Allow-Origin': 'http://localhost'
})
```
 + setHeader：write方法第一次被调用时发送响应头
```javascript
res.statusCode = 200
res.setHeader('Content-Type', 'text/plain')
res.setHeader('Access-Contorl-Allow-Origin', 'http://localhost')
res.setHeader('Set-cookie', ['mycookie1=value1', 'mycookie2=value2'])
```
- res.end()与res.send()区别
 + res.end()只能发送string或者buffer类型的数据，是node.js的内置方法
 + res.send()能发送对象
