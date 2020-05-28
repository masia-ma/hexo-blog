---
title: node.js复习4：token和cookie的使用
categories:
  - node.js
tags:
  - node.js
abbrlink: 7b871db4
date: 2019-04-29 10:00:00
---

## 说明：
演示加密方式为 bcrypt 方式
<!-- more -->
## token方式

+ 安装和导入jsonwebtoken

```bash
npm i jsonwebtoken
```
```javascript
const jwt = require('jsonwebtoken')
```
+ 定义SECRET（这个secret不应该写在一个单独的文件中，比如.igonore等被忽略上传的文件中）

```javascript
const SECRET = 'fkdsjfkasjfd' // 相当于一段密码
```
+ 登录

```javascript
app.post('/login', async (req, res) => {
    const user = await User.findOne({
      email: req.body.email
    })
    if (!user) {
      // 一般用户提交的数据有问题，状态吗是422
      return res.status(422).send({
        "message": "用户名不存在"
      })
    }

    <!-- 校验密码是否正确 -->
    // 使用bcrypt模块的 compareSync() 方法，对密码进行校验比对，该方法的返回值是true或者false
    const isPasswordValid = require('bcrypt').compareSync(
      req.body.password, // 第一个参数是客户端发起请求的明文密码值
      user.password // 第二参数是数据库中的密文密码值
    )
    if (!isPasswordValid) {
      return res.status(422).send({
        "message": "输入密码错误"
      })
    }

    <!-- 根据id生成token -->
    const token = jwt.sign({
      id: String(user._id) // 这里需要注意的是，不要把密码放在token中，可以放一个其他的键，比如_id等。
      //这里虽然写的形式是对象键值对的形式，但是实际上，token不是一个对象，而是一段hash值，id不会以键值对的形式出现。
      // 解密时使用 jwt.verify(raw, SECRET) 方法，会得到包含id属性的对象，这里raw就是那段hash值。
    }, SECRET) // 第二个参数为secret，这个secret不应该出现在代码库中，应该是一个环境变量之类的东西，应该是在.ignore中被忽略上传的东西
          // 这个secret应该是全局保持唯一的
    res.send({
      user,
      token: token
    })
  })
```
返回给客户端的结果为

```javascript
{
  "user": {
    "_id": "5cce3ac57879e13fc45d3511",
    "email": "user3",
    "password": "$2b$10$/PJRuI8mnOUpMkB5an8iKOOYqUFAyhBrV6FImTUuU7mmGG6P9Ys5a",
    "__v": 0
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVjY2UzYWM1Nzg3OWUxM2ZjNDVkMzUxMSIsImlhdCI6MTU1NzAxOTMzNn0.22ImaDk_1dkMkEbezu0ByrwkBGXPrlc7Ox-ORJ75oUQ"
  // 后面的这段hash值就是上面jwt对user._id加密生成的，并且每次执行发起登录请求时，会根据相同的user._id生成不同的token
}
```
+ 进入个人中心

客户端

```javascript
post {{url}}/profile
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVjY2UzYWM1Nzg3OWUxM2ZjNDVkMzUxMSIsImlhdCI6MTU1NzAxOTMzNn0.22ImaDk_1dkMkEbezu0ByrwkBGXPrlc7Ox-ORJ75oUQ
```
服务端

```javascript
app.post('/profile', async (req, res) => {
  var raw = String(req.headers.authorization).split(' ').pop()
  console.log(jwt.verify(raw, SECRET))
  // { id: '5cce6f15b7e8724438c8d621', iat: 1557034387 }
  const { id } = jwt.verify(raw, SECRET) // 当初用_id做的密钥，所以现在解密_id，再用_id做条件查找
  const user = await User.findById(id)
  res.send(user)
})
```
  可以提取中间件 

```javascript
// 用户登录获取信息中间件
const authMiddleware = async (req, res, next) => {
  var raw = String(req.headers.authorization).split(' ').pop()
  const { id } = jwt.verify(raw, SECRET)
  req.user = await User.findById(id) // 将查询的结果以属性的方式保存在req中
  next() 
}
```
  服务端改写为

```javascript
app.post('/profile', authMiddleware, async (req, res) => {
  res.send(req.user)
})
```

## cookie方式

+ 安装和导入express-session

```bash
npm i express-session
```
```javascript
const session = require('express-session')
```
+ 配置中间件

```javascript
<!-- 中间件一定要放在使用之前 -->
app.use(session({ 
  secret: 'keyboard cat', // 密钥
  resave: false,
  saveUninitialized: true 
}))
```
+ 登录

```javascript
app.post('/login', async (req, res) => {
  const user = await User.findOne({
    email: req.body.email
  })
  if (!user) {
    // 一般用户提交的数据有问题，状态吗是422
    return res.status(422).send({
      "message": "用户名不存在"
    })
  }

  <!-- 校验密码是否正确 -->
  // 使用bcrypt模块的 compareSync() 方法，对密码进行校验比对，该方法的返回值是true或者false
  const isPasswordValid = require('bcrypt').compareSync(
    req.body.password, // 第一个参数是客户端发起请求的明文密码值
    user.password // 第二参数是数据库中的密文密码值
  )
  if (!isPasswordValid) {
    return res.status(422).send({
      "message": "输入密码错误"
    })
  }

  <!-- 成功验证后，将查询到的用户放入cookie -->
  req.session.user = user
  res.status(200).json({ // 返回成功信息
    err_code: 200,
    message: 'ok'
  })
})
```
+ 进入个人中心

```javascript
app.post('/profile', async (req, res) => {
  console.log(req.session)
  res.send(req.session)
})
```
+ 返回结果

```javascript
{ _id: '5cce56bbecd3734398851c80',
  email: 'user12',
  password: '123456',
  date: '2019-05-05T03:21:31.457Z',
  __v: 0 }
```
