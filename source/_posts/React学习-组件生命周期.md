---
title: React学习-组件生命周期
categories:
  - js
  - React
  - component
tags:
  - js
  - React
  - component
date: 2019-09-02 22:00:00
---

# React的组件生命周期
+ 从下面的图可以看出，这里嵌套组件渲染是递归实现的。父组件不管是在初次挂载还是更新，都需要把内部的孩子组件全部挂载或者更新完成之后，自己才完成挂载和更新
<!-- more -->
+ 特别要注意的是
  componentWillReceiveProps
  shouldComponentWillUpdate
  componentWillUpdate
  componentDidUpdate
  这四者的执行顺序

# 图解
{% asset_img component_life.png React组件的生命周期 %}

# 代码
```javascript
class Child extends React.Component {
  constructor (props) {
    super(props);
    console.log('Child constructor')
  }

  componentWillMount () { 
    console.log('Child componentWillMount')
  }

  componentDidMount (prevProps, prevState) {
    console.log('Child componentDidMount')
  }

  componentWillReceiveProps (nextProp) {
    console.log('Child componentWillReceiveProps')
  }

  shouldComponentUpdate (nextProps, nextState) {
    console.log('Child shouldComponentUpdate')
    return true
  }

  componentWillUpdate (nextProps, nextState) {
    console.log('Child componentWillUpdate')
  }

  componentDidUpdate (prevProps, prevState) {
    console.log('Child componentDidUpdate')
  }

  componentWillUnmount () {
    console.log('Child componentWillUnmount')
  }

  render () {
    console.log('Child render')

    return <div>{this.props.test}</div>
  }
}

class App extends React.Component {
  constructor () {
    super()
    this.state = {
      test: new Date().toLocaleTimeString(),
      count: 0,
      clockTimer: null
    }
    console.log('App constructor')
  }
  
  componentWillMount () {
    console.log('App componentWillMount')
  }

  componentDidMount () {
    console.log('App componentDidMount')

    let {clockTimer, count} = this.state
    clockTimer = setInterval(function () {
      console.log(this.state.test)
      if (count > 0) clearInterval(clockTimer)
      this.setState({
        test: new Date().toLocaleTimeString()
          })
          count++
        }.bind(this), 1000)
      }

      shouldComponentUpdate (nextProps, nextState) {
        console.log('App shouldComponentUpdate')
        return true
      }

      componentWillUpdate (nextProps, nextState) {
        console.log('App componentWillUpdate')
      }

      componentDidUpdate (prevProps, prevState) {
        console.log('App componentDidUpdate')
      }

      render () {
        console.log('App render')
        let {test} = this.state

        return (
          <Child test={test}/>
        )
      }
    }

    ReactDOM.render(
      <App />,
      document.getElementById('app')
    )
```