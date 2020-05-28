---
title: vue中axios的封装
categories: 
  - vue
  - axios
  - ajax
tags: 
  - vue 
  - axios
  - ajax
date: 2019-05-17 09:00:00
---

## 在vue中我们可以把调用接口的代码封装起来，一方面减少重复代码的编写，另一方面我们可以使本身的vue页面变得简洁易读。
<!-- more -->
+ 引入，原本的axios使用方式是: 
1. 首先在main.js文件中修改Vue原型链

```javascript
import axios from 'axios'
Vue.prototype.$http = axios
```
2. 在其他.vue文件中使用

```javascript
// post方式
this.$http({
  method: 'post', // 请求方式
  url: '/users/register', // 请求地址
  data: form.data // 传入参数
})
.then(res => {
  console.log(res) // 成功返回的数据
})
.catch(err => {
  console.log(err) // 失败返回的数据
})
// get方式
this.$http({
  method: 'get',
  url: 'products?id=1&name=maisa'
})
.then(res => {
  console.log(res)
})
.catch(err => {
  console.log(err)
})
```
+ 封装axios使用方式
在`src`文件夹中创建一个`api`文件夹，在这个文件下面有两个文件，一个是`ajax.js`，另外一个是`index.js`
1. 在`ajax.js`中：
```javascript
import axios from 'axios'

export default function ajax (url, data = {}, type = 'GET') {
  return new Promise((resolve, reject) => {
    let promise
    if (type === 'GET') {
      let dataStr = ''
      Object.keys(data).forEach(key => {
        dataStr += key + '=' + data[key] + '&'
      })
      if (dataStr !== '') {
        dataStr = dataStr.substring(0, dataStr.lastIndexOf('&')) // 删除字符串中最后一个&
        url = url + '?' + dataStr 
      }
      promise = axios.get(url)
    } else {
      promise = axios.post(url, data)
    }
    promise.then(res => {
      resolve(res.data) // 直接返回res中的data
    }).catch(err => {
      reject(err)
    })
  })
}
```

2. 在`index.js`中
```javascript
import ajax from './ajax'
const BASE_URL = ''

// 1.注册
export const reqRegister = (formData) => ajax('/users/register', formData)
// 2.登录
export const reqLogin = (formData) => ajax('/users/login', formData)
```

3. 在`.vue`中使用

```javascript
<script>
import {reqRegister} from '@/api/index' // 按需导入

export default {
  data () {
    return {
      form: {
        email: '',
        nickname: '',
        birthday: '',
        gender: '',
        bio: ''
      }
    }
  }
  methods: {
    submit () {
      reqRegister(this.form)
        .then(res => {
          console.log(res)
        })
        .catch(err => {
          console.log(err)
        })
    }
  }
};
</script>
```