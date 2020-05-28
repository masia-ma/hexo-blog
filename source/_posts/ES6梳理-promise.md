---
title: ES6梳理-Promise
date: 2019-07-10 10:48:33
categories:
  - ES6
  - Promise
tags:
  - ES6
  - Promise
---

# Promise
<!-- more -->
### ES5常规回调和Promise回调做对比
```javascript
var fs = require('fs')
var path = require('path')

// 读取文件函数
function getfile (fpath, callback) {
	fs.readFile(path.join(__dirname, fpath), 'utf8', (err, data) => {
		// if (err) throw err //终止程序执行，并且在控制台答应错误信息。这种方式不太好用，代码健壮性不好，不能因为获取不到文件而耽误整个程序的执行
		if (err) return callback(err) //把错误通过回调函数返回，程序依然可以正常进行下面的，但其实这种方式也是不好的，因为你的回调函数并不知道你返回的是错误信息还是真正所需的数据
		callback(data)
	})
}
// 1、异步读取文件，不能决定读取文件的顺序
getfile('./5.txt', function (data) {
	console.log(data)
})
getfile('./2.txt', function (data) {
	console.log(data)
})
getfile('./3.txt', function (data) {
	console.log(data)
})

// 2、按照先后顺序读取多个文件
getfile('./5.txt', function (data) {
	console.log(data)
	getfile('./2.txt', function (data) {
		console.log(data)
		getfile('./3.txt', function (data) {
			console.log(data)
		})
	})
})

// 3、使用promise对象封装一个函数，优化代码结构，文件读取顺序执行
function promiseGetfile (fpath) {
	return new Promise(function (resolve, reject) {
		fs.readFile(path.join(__dirname, fpath), 'utf8', function (err, data) {
			if (err) {
				reject(err)
			} else {
				resolve(data)
			}
		})
	})
}

promiseGetfile('./1.txt')
	.then(function (data) {
		console.log(data)
		return promiseGetfile('./2.txt')
	})
	.then(function (data) {
		console.log(data)
		return promiseGetfile('./3.txt')
	})
	.then(function (data) {
		console.log(data)
	})
```

### Promise使用场景
+ 对axios进行二次封装，统一post和get请求的操作，该层面的封装属于系统层面的封装
```javascript
import axios from 'axios'

export default function ajax (url, data = {}, type = 'GET') {
  // 这里统一了post和get方式请求接口传参的形式，即都为对象。而且还可以设置默认参数
  return new Promise((resolve, reject) => {
    let promise
    if (type === 'GET') {
      let dataStr = ''
      Object.keys(data).forEach(key => {
        dataStr += key + '=' + data[key] + '&'
      })
      if (dataStr !== '') {
        dataStr = dataStr.substring(0, dataStr.lastIndexOf('&'))
        url = url + '?' + dataStr
      }
      promise = axios.get(url)
    } else {
      promise = axios.post(url, data)
    }
    promise.then(res => {
      resolve(res.data)
    }).catch(err => {
      reject(err.data)
    })
  })
}
```
+ 在index.js进行第三次封装
对请求进行第三次封装，该层面属于业务层面的封装，封装了请求接口地址，只需要传入查询参数对象formData，使用reqRegister
```javascript
import ajax from './ajax'

const BASE_URL_API = '/api'
const BASE_URL_USERS = '/users'

// 1.注册
export const reqRegister = (formData) => ajax(`${BASE_URL_USERS}/register`, formData, 'post') 
(formData)调用即可
// 2.登录
export const reqLogin = (formData) => ajax(`${BASE_URL_USERS}/login`, formData, 'post')
// 3.个人中心信息
export const reqProfile = () => ajax(`${BASE_URL_USERS}/profile`)

// 4.获取位置
export const reqPosition = (position) => ajax(`${BASE_URL_API}/position/${position}`)

// 5.获取产品列表
export const reqShopLists = () => ajax(`/shopLists`)

// 6. 获取店家商品列表
export const reqGoodsLists = () => ajax('/goods')
```
### catch使用
```javascript
{
  let ajax = function (num) {
    return new Promise((resolve, reject) => {
      if (num > 5) {
        resolve(num);
      } else {
        throw new Error('出错了');
      }
    })
  }

  // 1. 使用catch
  ajax(1)
    .then((data) => {
      console.log(data+"正确");
    })
    .catch((err) => {
      console.log(err);
      /*
        Error: 出错了
        at promise.html:17
        at new Promise (<anonymous>)
        at ajax (promise.html:13)
        at promise.html:21
      */
    })
  
  // 2. 不使用catch
  ajax(1)
    .then((data) => {
      console.log(data+"正确");
    }, (err) => { 
      /*虽然没有在Promise对象中合适的位置调用reject()，
        但是调用了throw new Error('出错了');
        所以中的err就是new Error('出错了')
      */
      console.log(err);
      /*
        Error: 出错了
        at promise.html:17
        at new Promise (<anonymous>)
        at ajax (promise.html:13)
        at promise.html:21
      */
    })

  // 3. then中的第二个回调和catch共存，则只会执行then的第二个回调。
  ajax(1)
    .then((data) => {
      console.log(data+"正确");
    }, (err) => { 
      console.log(err);
      /*
        Error: 出错了
        at promise.html:17
        at new Promise (<anonymous>)
        at ajax (promise.html:13)
        at promise.html:21
      */
    })
    .catch((err) => { // 不执行
      console.log(err);
    }) 
} 
```
### Promise高级用法
+ Promise.all
+ Promise.rece()

Promise.all()使用场景：
```javascript
/*使用场景:假如一个cell中有三张图，我们的三张图片是异步加载的，并且来自不同的接口，
如果每张图片一旦获取成功，就加载到页面中。由于三张的获取成功顺序将不能确定，
但是他们在页面上的先后位置顺序是固定的，那就极有可能出现第二张图加载完成，
第一张图还没有加载完成的现象，这样的用户体验很不好。
所以，我们要规定三张图的异步加载顺序，用户看不到图片加载的过程，也看不到闪动
我们需要做：三张图都获取到了之后，再一起加载到页面
*/
{
  // 所有图片全部获取完成之后在添加到到页面
  function loadImg (src) {
    return new Promise ((resolve, reject) => {
      let img = document.createElement('img');
      img.src = src;
      img.onload = function () {
        resolve(img); // 把创建好的img DOM元素对象传给成功的回调
      } 
      img.onerror = function (err) {
        reject(err)
      }
    })
  }

  function showImgs (imgs) {
    imgs.forEach(img => {
      document.body.appendChild(img);
    })
  }

  // 数组中传入多个Promise实例对象，当所有传入的Promise对象的状态发生变化后，这个Promise对象集合才会调用then方法
  Promise.all([
    loadImg('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1562759633416&di=41de4b9fc2024e278c77d5a4ae6135c2&imgtype=0&src=http%3A%2F%2Fwww.chinapoesy.com%2FUploadFiles%2FPoesy%2F20141015_92b4978b-973a-472c-b4e8-33d89e01853f.jpg'),
    loadImg('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1562760007124&di=f447518c90caf2859e2c5e1fb0fef39c&imgtype=jpg&src=http%3A%2F%2F5b0988e595225.cdn.sohucs.com%2Fimages%2F20190122%2Ff24ab1f776974b41bb83cbd86353f702.jpeg'),
    loadImg('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1562759633411&di=6b3012be166e8f06ecbe49bbc175a94f&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F09%2F20161009140358_3S8cz.jpeg')
  ]).then(showImgs)
}
```
```javascript
// 图片实现不了，用div模拟
{
  function loadDiv (text) {
    return new Promise ((resolve, reject) => {
      if (!text) {
        throw new Error("出错了"); 
        /*
          Error: 出错了
          at promise.html:43
          at new Promise (<anonymous>)
          at loadDiv (promise.html:41)
          at promise.html:70  
        */
        reject('出错了'); // 出错了
      }
      let div = document.createElement('div');
      div.style.color = 'red';
      div.innerText = text;
      resolve(div);
    })
  }

  function showDivs (divs) {
    divs.forEach(div => document.body.appendChild(div));
  }
  Promise.all([
    loadDiv('1 div'),
    loadDiv('2 div'),
    loadDiv('3 div'),
    loadDiv('4 div')
  ])
  .then(showDivs);
}
```
Promise.race()使用场景：
```javascript
/*
  假如有三张图片，来自不同的接口，哪个网速好，先获取到了，我就优先显示哪张图片
*/
{
  // 所有图片全部获取完成之后在添加到到页面
  function loadImg (src) {
    return new Promise ((resolve, reject) => {
      let img = document.createElement('img');
      img.src = src;
      img.onload = function () {
        resolve(img); // 把创建好的img DOM元素对象传给成功的回调
      } 
      img.onerror = function (err) {
        reject(err)
      }
    })
  }

  function showImg (imgs) {
    document.body.appendChild(img);
  }

  // 数组中传入多个Promise实例对象，当所有传入的Promise对象的状态发生变化后，这个Promise对象集合才会调用then方法
  Promise.race([
    loadImg('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1562759633416&di=41de4b9fc2024e278c77d5a4ae6135c2&imgtype=0&src=http%3A%2F%2Fwww.chinapoesy.com%2FUploadFiles%2FPoesy%2F20141015_92b4978b-973a-472c-b4e8-33d89e01853f.jpg'),
    loadImg('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1562760007124&di=f447518c90caf2859e2c5e1fb0fef39c&imgtype=jpg&src=http%3A%2F%2F5b0988e595225.cdn.sohucs.com%2Fimages%2F20190122%2Ff24ab1f776974b41bb83cbd86353f702.jpeg'),
    loadImg('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1562759633411&di=6b3012be166e8f06ecbe49bbc175a94f&imgtype=0&src=http%3A%2F%2Fb-ssl.duitang.com%2Fuploads%2Fitem%2F201610%2F09%2F20161009140358_3S8cz.jpeg')
  ]).then(showImg)

  // 页面上只显示了一张图片
}
```