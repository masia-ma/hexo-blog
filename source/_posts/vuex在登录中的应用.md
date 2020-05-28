---
title: vuex在登录、获取用户信息状态的应用
categories:
  - vue
  - vuex
tags:
  - vue
  - vuex
date: 2019-05-30 14:30:00
---
## 问题分析：
之前的想法是后台只做一个登录接口，然后登录之后，获取用户的信息，并且保存在vuex中，供整个vue项目中的pages页面使用。但实践证明，我并没有真正了解vuex的基本使用和适用场景。
<!-- more -->
所以，首先要说的是，vuex的解决的是同一个页面中组件间的数据存储和公用问题，vuex并不是最初理解的对于整个vue项目全局的数据存储空间

显然，如果只有一个获取用户信息的方法并且封装在登录接口中执行是考虑不周全的，比如：在订单页面，个人中心我都需要用户信息，难道我还要调用登录接口来取数据吗？还需要再输入密码吗？还是需要将密码保存在某个地方，调用登录接口，自动去登录？这样做是不是会造成密码泄露？

## 解决方案：
+  将登录操作和获取用户信息状态分开执行：
1. 登录操作只获取token，并把token保存在session中
2. 获取个人用户信息的接口再根据session中的token去获取个人用户信息

需要注意的是：这里有一个细节，在用户正确输入密码，点击登录按钮并且拿到token之后，我们要监听token是否拿到，如果拿到，将页面跳转到我的订单或者个人中心等非登录页面。


后台接口代码：router/users.js
```javascript
const jwt = require('jsonwebtoken') 
const md5 = require('blueimp-md5')
const { User } = require('../models/users')
const SECRET = 'masia'
const MD5SECRET = 'masia66666'

router
  .post('/login', async (req, res) => {
    req.body.password = md5(md5(req.body.password + MD5SECRET))
    const user = await User.findOne().where({
      email: req.body.email
    })
    if (!user) { // 如果对象为空
      return res.send({ 
        code: 1,
        msg: '输入的用户名不存在'
      })
    } else { // 如果对象存在
      if (user.password != req.body.password) { 
        return res.send({
          code: 2,
          msg: '输入的密码错误'
        })
      } else {
        const token = jwt.sign({
          id: String(user._id) 
        }, SECRET)
        res.send({
          code: 0, // code为0时，登录成功
          msg: '登录成功',
          token
        })
      }
    }
```
```javascript
// state.js
export default {
  currentUser: null, // 未登录状态，当前用户为null,
  token: null, // 从后台返回到的token，一方面保存在vuex中，用来判断是否登录成功，另一方面，保存在session中，供其他页面获取用户信息使用
  loginStatus: {
    code: -1, // 默认状态为 -1 ，状态为未登录
    msg: '未登录'
  }
}
```
```javascript
// actions.js
import axios from 'axios'

import {
  reqLogin
} from '../api'

import {
  RECEIVE_PROFILE,
  RECEIVE_LOGIN,
  RECEIVE_LOGINSTATUS 
} from './mutation-types'

export default {
  async getLogin ({commit, state}, formData) { // 登录,formData中包含了用户名，密码等属性
    const res = await reqLogin(formData) // 已经封装好的登录方法，已经过滤掉了headers等信息，返回的是data对象
    if (res.code) { // 当 res.code 不为0时，均为登录失败
      return commit(RECEIVE_LOGINSTATUS, {res}) // 根据返回结果，更新登录状态
    }
    // code 为0，执行下面的代码
    let token = res.token // 拿到token
    sessionStorage.setItem('token', token) // 把token写入session中
    commit(RECEIVE_LOGIN, {res})
  },
   async getProfile ({commit}) { // 获取用户个人信息
    const token = sessionStorage.getItem('token') //获取session中的token
    if (token) {  // 如果token存在，发送获取个人用户信息请求
      const user = await axios({
        method: 'POST',
        url: '/users/profile',
        headers: {
          'Authorization': 'Bearer ' + token
        }
      })
      const currentUser = user.data
      commit(RECEIVE_PROFILE, {currentUser})
    }
    return;
  }
}
```
```javascript
// mutation-types.js
export const RECEIVE_LOGINSTATUS = 'receive_loginStatus' // 改变state.loginStatus
export const RECEIVE_LOGIN = 'receive_login' // 改变state.token
export const RECEIVE_PROFILE = 'receive_profile' // 接收个人信息
```
```javascript
// mutations.js
import {
  RECEIVE_LOGINSTATUS, 
  RECEIVE_LOGIN
} from './mutation-types'

export default {
  [RECEIVE_LOGINSTATUS] (state, {res}) {
    state.loginStatus.code = res.code
    state.loginStatus.msg = res.msg
  },
  [RECEIVE_LOGIN] (state, {res}) {
    state.token = res.token
  }，
  [RECEIVE_PROFILE] (state, {currentUser}) {
    state.currentUser = currentUser 
  },
}
```

在login.vue页面中：
```javascript
watch: {
  '$store.state.token' (newVal, oldVal) { // 必须监视到登录成功，且把token存入到session中之后，方可跳转页面
    console.log('masia')
    this.$router.push('/profile')
  }
},
methods: {
  login () {
    this.$store.dispatch('getLogin')
  }
}
```

在profile.vue页面中：
```javascript
mounted () {
  this.$store.dispatch('getProfile')
}
```