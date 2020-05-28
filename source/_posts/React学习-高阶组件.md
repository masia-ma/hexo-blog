---
title: React学习-高阶组件
categories:
  - js
  - React
  - component
tags:
  - js
  - React
  - component
---

### 高阶组件和套子组件的区别是：
<!-- more -->
1. 套子组件是把写在其内部标签结构内的jsx元素通过this.children的方式拿到，并且解析
2. 高阶组件是函数执行后，返回一个组件类，函数的参数为你要渲染的组件类，这个函数会给你传递的组件类外面在加一层父组件我们用的时候实际上在用这个父组件，而且这个父组件以后会代理外部所要传给内部组件的props

### 代码
+ 给实际的ui组件类外加一层代理组件
connect.jsx
```javascript
import React from 'react'

export default function connect(mapStateToProps, mapActionsToProps) {
  return function connectHot(Component) {
    return class Proxy extends React.Component {

      state = {
        n: 'nnnn',
        m: 'mmmm'
      }

      componentDidMount() {
        
      }
      
      render() {
        return <Component {...this.state} {...this.props}></Component>
      }
    }
  }
}
```
+ 实际的ui组件
```javascript
import React, { Component } from 'react'
import connect from './connect'  // 导入connect 函数

class Vote extends Component {
  render() {
    console.log('vote-component', this)
    return (
      <div>
        <div>vote 组件</div>
        { this.props.children }
      </div>
    )
  }
}

export default connect()(Vote) 
// 返回的是一个Proxy组件，里面包裹了Vote组件
```
+ index.js
```javascript
import React from 'react'
import { render } from 'react-dom'
import Vote from './components/higher-comp/vote' // 这里引入的其实是一个Proxy组件，里面包裹了 Vote 组件

render(
  <Vote c={'ccc'}>
    <Vote c={'ccc'}>

    </Vote>
  </Vote>,
  document.getElementById('root')
)
```

# 图解
{% asset_img react_higher_component1.png 高阶组件 %}
{% asset_img react_higher_component2.png 高阶组件 %}