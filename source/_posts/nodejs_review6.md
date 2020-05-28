---
title: node.js复习6：数据模型设计
categories:
  - node.js
  - mongodb
  - mongoose
tags:
  - node.js
  - mongodb
  - mongoose
abbrlink: '806847e1'
date: 2019-05-02 13:00:00
---

数据模型一般存放在model目录中
<!-- more -->
## 用户数据模型设计(user.js)

+ 连接数据库
```javascript
const mongoose = require('mongoose')
mongoose.connect('mongodb://localhost:27017/node_template', {
  useCreateIndex: true,
  useNewUrlParser: true
})
```

+ 定义模型
```javascript
const UserSchema = new mongoose.Schema({
  email: { // 登录账号
    type: String,
    unique: true
  },
  nickname: { // 昵称
    type: String,
    required: true
  },
  password: { // 密码
    type: String,
    set (a) {
      // 使用bctypt包，实现原理很复杂，但是实现起来很简单 ，直接调库就好
      return require('bcrypt').hashSync(a, 10)
                              // 同步方法     指数，指数越高，生成的散列值越复杂，但是用的时间也长时，但是设置的太低，安全性又很低
    }
  },
  created_time: { // 创建时间
    type: Date,
    // 传给数据库模块的时候，数据库模块发现这里是个方法，所以它就会调用这个方法，这时候，就会有真正的创建时间了
    // 如果这里改为Date.now(), 就会在user.js执行的时候立即调用这个方法,此时拿到的是一个相对在数据库模块创建时
    // 的过去时间,也就是相当于一个无效的死值
    // 所以注意不要写成Date.now()
    default: Date.now
  },
  last_modified_time: { // 最后修改时间
    type: Date,
    default: Date.now()
  },
  avatar: { // 头像
    type: String,
    default: '/public/img/youxiang.jpg'
  },
  bio: { // 个人简介 biography
    type: String,
    default: ''
  },
  gender: { // 性别
    type: Number,
    enum: [-1, 0, 1],
    default: -1
  },
  birthday: { // 生日
    type: Date,
    default: ''
  },
  status: { // 权限状态
    type: Number,
    // 0 没有权限限制
    // 1 不可以评论
    // 2 不可以登录
    // 是否可以评论
    //是否可以登录使用
    enum: [0, 1, 2], 
    default: 0
  }
})
```

+ 创建模型
```javascript
const User = mongoose.model('User', UserSchema)
// 或者直接导出, 适用于这个模块中只有一个模型
// module.exports = mongoose.model('User', UserSchema)
// 使用模块需要一个变量接收
// const User = require('./user')
```

+ 导出对象
```javascript
module.exports = {
  User
}
// 使用模块使用对象来接收，适用于这个模块中含有多个模型，使用模块可以按需导入
// const { User } = require('./user')
```