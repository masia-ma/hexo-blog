---
title: vue本地代理配置
categories: 
  - vue
  - webpack
tags: 
  - vue
  - webpack
date: 2019-05-16 14:00:00
---

## 原来在网上找了好多设置的方法，比如：
由于没有真正理解里面的原理，所以在用的时候，这种所谓路由重写的方式带来了一些麻烦，如请求的地址无效等，让我误以为proxyTable没有设置成功。
<!-- more -->

+ 说明： 
```javascript
proxyTable: {
      '/api': {
        target: 'http://127.0.0.1:4000', // 要访问接口的域名
        // secure: false,  // 如果是https接口，需要配置这个参数
        changeOrigin: true, // 如果接口跨域，需要进行这个参数配置
        pathRewrite: { 
          '^/api': '/' // 或 '^/api': ''
        }
      }
    },
```
前端请求路由为： `/api/api/test`
后台实际的路由为： `/api/test`
上段代码打包之后的执行过程是这样的(实际上对`/api/api/test`使用了两次)：
1. 第一次，发起请求 `/api/api/test`，系统识别到`/api/api/test`中的第一个`/api`，就认为在请求`http://127.0.0.1:4000`下面的路由；
2. 第二次，在对`/api/api/test`路由进行重写，找到了第一个`/api`并且把他换成`/`，最终完整的请求路由为：`http://127.0.0.1:4000//api/test`，事实上，这里的`//api`和`/api`是一样的，并不会报错。
注意： 在浏览器中看到的请求url是：`http:127.0.0.1:8080/api/api/test`

+ 配置1：
```javascript
proxyTable: {
      '/api': {
        target: 'http://127.0.0.1:4000', // 要访问接口的域名
        // secure: false,  // 如果是https接口，需要配置这个参数
        changeOrigin: true, // 如果接口跨域，需要进行这个参数配置
        pathRewrite: { 
          '^/api': ''
        }
      }
    },
```
如果这样配置，那么在请求路径那里需要将原来的 /api/test 需要改成 /api/api/test

+ 配置2：
```javascript
proxyTable: {
      '/api': {
        target: 'http://127.0.0.1:4000', // 要访问接口的域名
        // secure: false,  // 如果是https接口，需要配置这个参数
        changeOrigin: true, // 如果接口跨域，需要进行这个参数配置
        pathRewrite: { 
          '^/api': '/'
        }
      }
    },
```
如果这样配置，那么在请求路径那里需要将原来的 /api/test 需要改成 /api/api/test

+ 配置3：
```javascript
proxyTable: {
      '/api': {
        target: 'http://127.0.0.1:4000', // 要访问接口的域名
        // secure: false,  // 如果是https接口，需要配置这个参数
        changeOrigin: true, // 如果接口跨域，需要进行这个参数配置
      }
    },
```
如果这样配置，那么在请求路径那里需要将原来的 /api/test 不需要修改

+ 配置4：
```javascript
proxyTable: {
      '/': {
        target: 'http://127.0.0.1:4000', // 要访问接口的域名
        changeOrigin: true,
      }
    },
```
使用：
假如接口是 `http://127.0.0.1:4000/api/list`,那么访问的地址是`/api/list`
假如接口是 `http://127.0.0.1:4000/users/register`, 那么访问的地址是`/users/register`