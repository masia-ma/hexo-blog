---
title: vue技术栈梳理-状态管理篇
categories: 
  - vue
  - vuex
tags: 
 - vue
 - vuex
date: 2019-06-12
---
#### 首先复习父子组件间的通信
<!-- more -->
编写组件实现 input的 v-model 功能
+ myInput 组件
```javascript
<template>
  <div>
    <input :type="type" @input="handleInput" :value="value"/>
  </div>
</template>

<script type="text/ecmascript-6">
export default {
  data () {
      return {

      }
  },
  props: {
    type: {
      type: String,
      default: 'text'
    },
    value: {
      type: [String, Number],
      default: ''
    }
  },
  components: {
    
  },
  methods: {
    handleInput (event) {
      const value = event.target.value
      this.$emit('input', value)
    }
  }
}
</script>
```
在页面中使用myInput组件
```javascript
<template>
  <div>
    <p>input component</p>
    <div>
      <myInput @input="getInputVal" :value="inputValue"/>
      // 这个组件就相当于 <input v-model="inputValue"/>
      // 因为自定义的事件名比较特殊，为input，
      // 组件接收的参数名也比较特殊，为value，
      // 所以可以直接改写为 <myInput v-model="inputValue"/>
      // 当然下面也就省去了写getInput函数了
      <div>{{inputValue}}</div>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">4
import myInput from '_c/myInput' // 导入 自己编写的 输入组件
export default {
  data () {
    return {
      inputValue: 'this is input value'
    }
  },
  components: {
    myInput
  },
  methods: {
    getInputVal (inputValue) {
      this.inputValue = inputValue
    }
  }
}
</script>
```

要注意的是： 
父子组件传值是单向数据流，不能在子组件中直接去修改父组件中的值，而必须通过emit向父组件提交一个自定义事件，然后把要修改的值，传在这个自定义事件函数的参数  中，让父组件自己去修改。

#### 兄弟组件通信
+ 当然也可以这样做，child1 组件使用自定义事件向父组件提交一个参数，然后在父组件中使用事件接收函数用这个参数去修改绑定给child2组件的参数值
这种情况适用于，两个兄弟组件依赖同一个父组件
+ 那么两个同时存在的路由组件如何通行呢？
比如：
```javascript
<div>
  <router-view name="default"/> // 组件A
  <router-view name="tel"/> // 组件B
</div>
```
当然，如果这两个router-view写在App.vue中，也可以借助App.vue 作为桥梁完成这两个兄弟组件间的传值（router-view组件的属性绑定和事件绑定和一般的自定义组件的操作方式相同）。因为router-view是一个用于替换内容的通用容器，如果给router-view上绑定很多属性和自定义事件函数，如：<router-view :value="value" :tel="tel" :desc="desc" @toSilbing="toSibling"/>，一方面会给我们的App.vue这个根组件中写入很多帮助兄弟组件间传值的中间属性和中间方法，污染全局的根组件，不符合高内聚低耦合的编程思想，另一方面在某个单独的组件中也需要定义大量的可能用到的属性和传值事件函数，代码量冗余，工作量也会加大。

#### 使用bus完成兄弟组件间的传值
文件`lib/bus/index.js`
```javascript
import Vue from 'vue'
const Bus = new Vue()
export default Bus
```
在main.js中引入bus
```javascript
import Vue from 'vue'
import App from './App.vue'
import router from './router'
import bus from './lib/bux'

Vue.config.productionTip = false
Vue.prptotype.$bus = bus // 在vue的原型对象上添加$bus

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')

```
组件child1.vue
```javascript
<template>
  <div>
    <p>child1 component</p>
    <p>使用bus传递来自child2的值 {{ fromSiblingValue }}</p>
  </div>
</template>

<script type="text/ecmascript-6">
export default {
  data () {
      return {
        fromSiblingValue: ''
      }
  },
  components: {

  },
  mounted () { // 在mountd的时候，名为为bus的vue实例绑定事件，因为bus实例永远不会消失，所以使用$on为bus绑定事件，等待$emit去触发事件。
    this.$bus.$on('use-bus', mes => {
      this.fromSiblingValue = mes
    })
  }
}
</script>
```
组件child2.vue
```javascript
<template>
  <div>
    <p>child2 component</p>
    <button @click="useBus">使用bus</button>
  </div>
</template>

<script type="text/ecmascript-6">
export default {
  data () {
      return {

      }
  },
  methods: {
    useBus () { // 使用$emit去触发 已经在bus中绑定的 'use-bus' 事件。
      this.$bus.$emit('use-bus', 'this is came from sibling')
    }
  }
}
</script>
```
注意： 使用$emit和$on，触发和绑定事件，必须只能在同一个vue实例中完成。

#### 使用vuex
在main.js中
```javascript
import Vue from 'vue'
import Vuex from 'vuex'
import App from './App'
import router from './router'
import Bus from './lib/bus'

Vue.config.profuctionTip = false
Vue.prototype.$bus = Bus

new Vue({
  router,
  store,
  render: h => h(App)
}).$mount('#app')
```
文件`/store/index.js`
```javascript
import Vue from 'vue'
import Vuex from 'vuex'
import state from './state'
import getters from './getters'
import actions from './actions'
import mutations from './mutations'
import user from './modules/user'

import config from '@/config' // 引入配置对象
Vue.use(Vuex)

export default new Vuex.Store({ // 这里要注意使用的Vuex.Store
  state,
  getters,
  mutations,
  actions,
  mudules: {
    user
  }
})

```
需要注意的是：在如何操作在模块中定义的状态，比如user模块
```javascript
const state = {
  userName = 'masia'
}
const getters = {

}
const actions = {

}
const mutations = {

}

export default {
  state,
  getters,
  mutations,
  actions
}
```
在组件中的使用方式
```javascript
<template>
  <div>
    <p>{{ userName }}</p>
  </div>
</template>

<script type="text/ecmascript-6">
export default {
  data () {

  },
  computed: {
    userName () {
      return this.$store.state.user.userName
                              // 这里得写模块名
    }
  }
}
</script>
```
使用mapState等工具函数来操作模块中的状态
```javascript
<template>
  <div>
    <p>{{ userName }}</p>
    <p>{{ appName }}</P>
  </div>
</template>

<script type="text/ecmascript-6">
import { mapState } from 'vuex' 
export default {
  data () {

  },
  computed: {
    ...mapState(['appName']) // ... 是es6中的展开操作符，也是剩余操作符
    /* 这里用作展开操作，会展开一个对象，这里这样写，相当于
     appName () {
      return this.$store.state.appName
    }
    
    // 当然也可以传入一个对象
    ...map({
     appName: state => state.appName 
     userName: state => state.user.userName 
    })
    */
    userName () {
      return this.$store.state.user.userName
                              // 这里得写模块名
    }
  }
}
</script>
```
+ 注意
  1. 使用`...mapState(['appName', 'userName'])`这种方式，获取不到在user模块中state中的userName，在chrome dev tools中可以看到userName: undefined
  2. 使用对象方式可以获取到 userName
  ```javascript
  ...mapState({
    userName: state => state.user.userName,
    appName: state => state.appName 
  })
  ```
  3. 在user模块中使用命名空间，在组件中使用createNamespacedHelpers重新为mapState绑定模块
```javascript
// user模块
const state = {
  userName = 'masia'
}
const getters = {

}
const actions = {

}
const mutations = {

}

export default {
  namespaced: true, // 设置命名空间
  state,
  getters,
  mutations,
  actions
}
```
  在组件中使用：
```javascript
<template>
  <div>
    <p>login</p> 
    <p>{{ userName }}</p>
    <p>{{ appName }}</p>
  </div>
</template>

<script type="text/ecmascript-6">
import { createNamespacedHelpers } from 'vuex' // 使用createNamespaceHelpers
const { mapState } = createNamespacedHelpers('user')
export default {
  data () {
      return {

      }
  },
  mounted () {
    console.log(createNamespacedHelpers)
    /*
      var createNamespacedHelpers = function (namespace) { return ({
        mapState: mapState.bind(null, namespace),
        mapGetters: mapGetters.bind(null, namespace),
        mapMutations: mapMutations.bind(null, namespace),
        mapActions: mapActions.bind(null, namespace)
      }); };
      */
  },
  components: {
    
  },
  computed: {
    ...mapState(['appName', 'userName']), // 这里的appName就为undefined了，因为user中没有appName
    // ...mapState({
    //   userName: state => state.userName, // 则这里直接在state中获取uesrName
    // })
  }
}
</script>
```
也可以在组件中使用：
```javascript
<template>
  <div>
    <p>login</p> 
    <p>{{ userName }}</p>
    <p>{{ appName }}</p>
  </div>
</template>

<script type="text/ecmascript-6">
import { mapState } from 'vuex' 
export default {
  data () {
      return {

      }
  },
  computed: {
    ...mapState('user', ['appName', 'userName']),
    /**
     *  或者
     * ...mapState('user', {
     *  userName: state => state.userName,
     *  appName: state => state.appName
     * }) 
     */
  }
}
</script>
```

#### 使用getters
首先明确getters的使用场景，在同一个路由组件中，如果只有一个局部组件依赖state中的某个属性来得到一个计算值，那么完全可以把这个计算属性写在他自己的组件中。但是，如果页面上多个局部组件都要使用计算值，那么应该直接把计算属性写在vuex中，在局部组件中拿来使用即可。
getters中有四个参数，全为系统默认参数，没有留用户参数的位置
1. 第一个参数是state对象的引用
2. 第二个参数是项目vuex中全部（所有模块中）的state，不管是在哪个模块，都可以找到。比如有两个模块list和user，他可以找到list和user模块下面的所有具体属性，并且包括了每个属性的get和set，所以可以任意修改别的模块里面的state
3. 第三个参数是当前模块的getters，并且包括每一个getter的get
4. 第四个参数是vuex中全部（所有模块中）的getter，并且包括每个getter的get
+ 注意：只有第一个参数是当前模块state的引用，其他不是

#### 使用actions
actions中每一个函数在定义的时候只能有两个参数
1. 第一个参数是系统赋予的当前的模块的vuex对象；
2. 第二个参数可以供用户使用，做一些传参操作，可以使用对象传参的方式，但要注意传参的属性名和定义的属性名必须相同。

#### 使用mutations
mutations中每一个函数在定义的时候也只能有两个参数
1. 第一个参数是系统赋予的当前vuex对象的state对象，这个参数是state对象的引用值；
2. 第二个参数是可以供用户使用
以user模块为例：
```javascript
const state = {
  userName: 'masia',
  number: 15
}
const getters = {
  bigOrSmall (state, bothState, currentGetters, bothGetters) {
    console.log('state', state) // 当前模块的state的引用
    console.log('bothState', bothState) // 所有模块中的state
    console.log('currentGetters', currentGetters) // 当前模块中的getters
    console.log('bothGetters', bothGetters) // 所有模块中的getters
    console.log('第三个参数', sss)
    bothState.appName = '修改了吧' // 可以修改其他模块中的state中的某个属性
    return state.number > 10 ? '大' : '小'
  },
  testGetter (state) {

  }
}
const actions = {
  modifyNumber ({ state, commit }, {isAdd, size, cb}) {
    if (isAdd) {
      commit('addNum', { size })
    } else {
      commit('subNum', { size })
    }
    cb && cb()
  }
}
const mutations = {
  addNum (_state, { size }) {
    state.number += size
  },
  subNum (state, { size }, name ) {
    state.number -= size
  }
}

export default {
  namespaced: true,
  state,
  getters,
  mutations,
  actions
}

```