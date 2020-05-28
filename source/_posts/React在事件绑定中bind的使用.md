---
title: React在事件绑定中bind的使用
categories:
  - js
  - react
tags:
  - js
  - react
---

# 首先要知道的是bind的一些基本用法（骚操作 /滑稽）

+ 1. bind 在使用的时候，第一个参数 是 this
```javascript
 function demo (...arg) {
  arg.forEach(item => {
    console.log(item)
  })
}

let bindDemo = demo.bind(this, 1, 2, 3)

bindDemo()

// 输出了 1 2 3 ，却没有输出this ,那就证明 this 不是当作载荷传进去的
```
+ 2. 在被绑定的函数中主动打出 this
```javascript
function demo2 (...arg) {
  console.log(this)
  arg.forEach(item => {
    console.log(item)
  })
}

let bindDemo2 = demo2.bind(this, 1, 2, 3)

let obj = { a: 1 }

bindDemo2.call(obj)

// 再使用.call调用，设法改变bindDemo2中的this，但是
// 输出了 Window {postMessage: ƒ, blur: ƒ, focus: ƒ, close: ƒ, parent: Window, …}
//       1
//       2
//       3
// 那就证明，在.call调用bindDemo2之前，bindDemo2中的this早就被规定好了。
```
+ 3. 那给bind 的第一个参数随便传一个呢，事实上你给bind的第一个参数传什么this就是什么，不管是什么类型( 例如传个 数字 11 )

```javascript
function demo3 () {
  console.log(this) // 输出了了一个 Number 对象
  // 当然你可以使用 valueOf() 
  console.log(this.valueOf()) // 输出了一个数字 11
  // 你还可以使用 toString()
  console.log(this.toString()) // 输出了一个字符串 '11'
}

let bindDemo3 = demo3.bind(11)

bindDemo3()
```
+ 4. 如果在jsx中使用bind给onClick绑定事件回调函数，那么第二个参数默认是event
```javascript
function Button () {
  return (
    <div onClick={clickDemo.bind(this, event, ...args)}></div>
                              // 这里如果要传入this，和事件源对象，必须写this 和 event，2019年10月10日证明这种写法的错误的，因为event找不到
    // 应该改为如下，先找个闭包把事件源对象保存一下
    <div onClick={(e) => clickDemo.call(this, e, ...args)}></div>
  )
}

function clickDemo (this, event, ...args) {
  console.log(this)
  console.log(event)
  args.forEach(item => {
    console.log(item)
  })
}
```
# 在 React中绑定事件回调函数的方法一般有三种（目前我知道的3种）

第一种：在constructor去为事件回调函数绑定this，且将绑定好this的函数赋值给该函数本身(比较土，比较low)
第二种：在真正的事件回调函数外面再包一层箭头函数 onClick={() => this.testClick()}，使用箭头函数中this去定义testClick中的this
第三种：使用bind在定义事件回调函数的this（比较麻烦）

+ 第一种：在constructor去为事件回调函数绑定this，且将绑定好this的函数赋值给该函数本身(比较土，比较low)
```javascript
class Button extends React.Component {
  constructor () {
    super()
    this.clickButton = this.clickButton.bind(this)
  }
  clickButton (...args) {
    console.log(this)
    args.forEach(item => {
      console.log(item)
    })
  }

  render () {
    return (
      <button onClick={this.clickButton}>点击按钮</button>
    )
  }
}

ReactDOM.render(<Button />, document.getElementById('app'))

// 输出:
// Button {props: {…}, context: {…}, refs: {…}, updater: {…}, clickButton: ƒ, …}
// Class {dispatchConfig: {…}, _targetInst: FiberNode, _dispatchListeners: ƒ, _dispatchInstances: FiberNode, nativeEvent: MouseEvent, …}
```
如果你想要传入event，那么还是得在事件绑定函数的时候用到
`this.clickButton.bind(this, event)`
输出：
```javascript
// Button {props: {…}, context: {…}, refs: {…}, updater: {…}, clickButton: ƒ, …}
// Event {isTrusted: true, type: "DOMContentLoaded", target: document, currentTarget: null, eventPhase: 0, …}
// Class {dispatchConfig: {…}, _targetInst: FiberNode, _dispatchListeners: ƒ, _dispatchInstances: FiberNode, nativeEvent: MouseEvent, …}
```

+ 第二种：在真正的事件回调函数外面再包一层箭头函数 onClick={() => this.testClick()}，使用箭头函数中this去定义testClick中的this

```javascript
class Button extends React.Component {
  constructor () {
    super()
  }

  clickButton (...args) {
    console.log(this)
    args.forEach(item => {
      console.log(item)
    })
  }

  render () {
    return (
      <button onClick={() => this.clickButton()}>点击按钮</button>
    )
  }
}

ReactDOM.render(<Button />, document.getElementById('app'))
```

+ 第三种：使用bind在定义事件回调函数的this（比较麻烦）
```javascript
let args = [222, 333, 444]
class Button extends React.Component {
  clickButton (...args) {
    console.log(this) 
    args.forEach(item => {
      console.log(item)
    })
  }

  render () {
    return (
      <button onClick={this.clickButton.bind(this, event, ...args)}>点击按钮</button>
    ) 
  }
}

ReactDOM.render(<Button />, document.getElementById('app'))


// 点击按钮输出：
/*
  Button {props: {…}, context: {…}, refs: {…}, updater: {…}, _reactInternalFiber: FiberNode, …}
  Event {isTrusted: true, type: "DOMContentLoaded", target: document, currentTarget: null, eventPhase: 0, …}
  222
  333
  444
  Class {dispatchConfig: {…}, _targetInst: FiberNode, _dispatchListeners: ƒ, _dispatchInstances: FiberNode, nativeEvent: MouseEvent, …}
*/
```