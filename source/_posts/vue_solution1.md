---
title: vue问题小结
categories:
  - 总结
  - vue
tags:
  - 总结
  - vue
date: 2019-05-01 09:00:00
---

## 使用watch监听路由发生变化后，我们接下来要执行的动作
<!-- more -->
+ 问题：进入详情页面时，我们需要从路由中拿到参数id去请求数据，进而渲染数据。一般情况下，我们会在详情页面这样写：
```javascript
data () {
  return {
    article_id = ''
  }
}
mounted () {
  article_id = this.$route.params.id // 1、先从路由中获取传过来的id
  this.$http.post('/getDetail/', { // 2、发送请求
    artical_id: this.artical_id
  }).then((res) => {
    //数据请求成功的回调...
  }).catch((err) => {
    //失败的回调...
  })

}
```
我们会把请求获取路由中参数的操作放在mounted()中，但是mounted()只有在第一次点击此路由的时候执行，但是回退到之前的页面，再次点击进来，不会执行mounted()。
或者说有圣杯布局的网页，头部和底部固定，内容区域变化，头部有一个导航栏，里面有若干个选项，比如内容1、内容2、内容3。假定内容1、内容2、内容3的路由相同，params参数不同，分别对应为`/detail/1`, `/detail/2`,`/detail/3`点击内容1选项之后，路由变为`/detail/1`，执行详情组件中的mounted()，获取到数据之后，会在网站主体的内容区域显示相应的内容1。然后再点击内容2选项，页面没有发生变化并且没有显示选项内容2对应的内容。这是因为路由没有发生变化而只是params参数发生了变化，页面不会再次调用mounted()钩子函数，所以我们不因该把获取id的操作写在mounted()中。
+ 解决方案：
监听$route对象，如果发生变化，则执行接下来我们要的操作。要注意的是mounted()获取id的方法仍然要写，因为没有挂载之前，组件监听到不$route这个对象，所以第一次点击选项卡拿id的操作要在mounted()中执行。
```javascript
data () {
  return {
    article_id = ''
  }
},
mounted () {
  console.log(this.$route.params.id) // 拿到第一次点击选项卡，渲染页面时拿到的id
},
watch: {
  '$route': 'routerAlter' // watch中不用写this，而且变量名都要用''包起来
},
methods: {
  routerAlter () {
    console.log(this.$route.params.id)
  }
}
```

## 监听$route，解决当前页面记录上次页面浏览位置问题
一般给App.vue中添加一个监听
```javascript
watch: {
  '$route': function () {
    window.scrollTo(0, 0) // 此方法并不是在操纵html元素或者是body元素，而是包含html和body元素的整个文档，即使html或者body高度是0，只要整个文档有高度，此方法使用之后依然是有效果的。
  }
}
```
+ html和body设置height: 100%问题
  当html和body设置了height: 100%之后，高度就是浏览器可是区域的高度，高度再也不会随着内容的高度而撑开。反之，body和html不设置高度，则他们的高度会随着内容而撑大。
+ position: fixed
  设置position: fixed的元素，位置是相对浏览器可视区域而定的。而且他们也不包含的整个文档之中。
