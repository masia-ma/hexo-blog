---
title: node.js复习5：bcrypt加密和md5加密
categories:
  - node.js
  - 加密方式
tags:
  - node.js
  - 加密方式
abbrlink: dbf676e6
date: 2019-05-01 14:00:00
---

## 注意：
+ md5加密方式：每个字母对应的hash值是相同的，md5有密文对应表。两个人有两个相同的密码，对应的加密后的hash也是一样的。
+ bcrypt加密方式：对于相同的字符串，加密几次的hash值都是不一样的。也就是说两个人有相同的密码，对应加密后的hash是不一样的。bcrypt更加安全
<!-- more -->
+ md5使用注册时加密密码相同的算法，再次对登录时的密码做加密处理，然后当作查询条件去做组合查询，实质是email和password做多条件组合查询。
+ bcrypt先根据email查出用户的信息，在使用`bcrypt.compareSync( req.body.password, user.password)`方法对用户登录时输入的密码和查出的用户信息中的密码做一个比对，比对成功则返回true，证明验证成功。实质上在验证密码是否合法之前，已经根据email将所需要的个人信息查出来了。

## md5方式：
+ 安装及导入

```bash
npm i blueimp-md5
```
```javascript
var md5 = require('blueimp-md5')
const MD5SECRET = 'masia66666' // 配置一个密钥
```
+ 注册

```javascript
app.post('/register', async (req, res) => {
  req.body.password = md5(md5(req.body.password + MD5SECRET)) // 把密钥拼在密码后面，并且加密两次
  const user = await User.create({
    email: req.body.email,
    password: req.body.password
  })
  res.send(user)
})
```
返回结果为：

```javscript
{
  "_id": "5cce5d7d4a6a314744d31a30",
  "email": "user20",
  "password": "cf147405dd20224eae703c23ecc3fa8d",
  "date": "2019-05-05T03:50:21.842Z",
  "__v": 0
}
```
+ 登录

```javascript
app.post('/login', async (req, res) => {
  req.body.password = md5(md5(req.body.password  + MD5SECRET)) // 把登录时输入的密码加密了一遍，再与数据库中注册时间加密过的密码作比较
  const user = await User.findOne({
    email: req.body.email,
    password: req.body.password  
  })
  res.send(user)
})
```
+ 可以对密码加密的这一步写一个中间件

```javascript
const md5MiddleWare = (req, res, next) => {
  var password = req.body.password
  req.password = md5(md5(password + MD5SECRET))
  next()
}
```
+ 注册使用中间件

```javascript
app.post('/register', md5MiddleWare, async (req, res) => {
  const user = await User.creat({
    email: req.body.email,
    password: req.password
  })
  res.send(user)
})
```
+ 登录使用中间件

```javascript
app.post('/login', md5MiddleWare, async (req, res) => {
  const user = await User.findOne({
    email: req.body.email,
    password: req.password  
  })
  res.send(user)
})
```

## bcrypt加密方式：
+ 安装及其导入

```bash
npm i bcrypt
```
```javascript
const bcrypt = require('bcrypt')
```
+ 在创建模型时使用，对password进行加密

```javascript
const UserSchema = new mongoose.Schema({
  email: {
    type: String, 
    unique: true
  },
  password: {
    type: String,
    set (a) { // 在模型调用password属性时，调用这个set方法，a就是password的值
      // 使用bctypt包，实现原理很复杂，但是实现起来很简单 ，直接调库就好
      return bcrypt.hashSync(a, 10)
                              // 同步方法     指数，指数越高，生成的散列值越复杂，但是花费的时间也长时，但是设置的太低，安全性又很低
    }
  }
})
const User = mongoose.model('User', UserSchema)
```
+ 注册

```javascript
app.post('/register', async (req, res) => {
  const user = await User.create({
    email: req.body.email,
    password: req.body.password 
  })
  res.send(user)
})
```
结果：

```javascript
{
  "_id": "5cce6f15b7e8724438c8d621",
  "email": "user26",
  "password": "$2b$10$DE.qpGfozZnUJs2r50C/XOw5NdIrg9Hdf2GJen3lzxa3vCoLhMVU6", // 密码已经经过处理
  "date": "2019-05-05T05:05:25.817Z",
  "__v": 0
}
```
+ 登录

```javascript
app.post('/login', async (req, res) => {
  <!-- 先根据email找用户 -->
  const user = await User.findOne({
    email: req.body.email
  })
  if (!user) {
    // 一般用户提交的数据有问题，状态吗是422
    return res.status(422).send({
      "message": "用户名不存在"
    })
  }
  // 使用bcrypt模块的 compareSync() 方法，对密码进行校验比对，该方法的返回值是true或者false
  const isPasswordValid = bcrypt.compareSync(
    req.body.password, // 第一个参数是客户端发起请求的明文密码值
    user.password // 第二参数是数据库中的密文密码值
  )
  if (!isPasswordValid) {
    return res.status(422).send({
      "message": "输入密码错误"
    })
  }
  res.send(user)
})
```
结果：

```javascript
{
  "_id": "5cce6f15b7e8724438c8d621",
  "email": "user26",
  "password": "$2b$10$DE.qpGfozZnUJs2r50C/XOw5NdIrg9Hdf2GJen3lzxa3vCoLhMVU6",
  "date": "2019-05-05T05:05:25.817Z",
  "__v": 0
}
```