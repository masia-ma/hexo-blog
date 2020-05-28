---
title: vue技术栈梳理-路由篇
categories: 
  - vue
  - vue-router
tags: 
 - vue
 - vue-router
date: 2019-06-11
---
# 对平时使用的vue技术栈进行系统的梳理，有助于以后更加全面和系统的掌握vue技术栈，并且提高编码的熟练度，加快以后的开发速度
<!-- more -->
## vue-router
1. router-link和router-view组件
2. 路由配置
  1. 动态路由
  2. 嵌套路由
  3. 命名路由
  4. 命名视图
3. js操作路由
4. 重定向和别名

### router-link和router-view组件
  router-link在没有参数tag的情况下面会被渲染成为a标签
### 路由配置
```javascript
// 在router.js中
import Home from '@/views/Home'

export default {
  routes: [
    { 
      path: '/home',
      name: 'home',
      component: Home,
    },
    {
      path: '/about',
      name: 'about',
      component: () => import('@/views/About')
    },
    {
      path: '/argu/:name',
      name: 'argu',
      component: () => import('@/views/argu')
    },
    {
      path: '/parent',
      name: 'parent',
      component: () => import('@/views/parent'),
      children: [
        {
          path: '/child1',
          name: 'child1',
          components: {
            right: () => import('@/views/child2'),
            left: () => import('@/views/child1')
          } 
        },
        {
          path: '/child2',
          name: 'child2',
          components: {
            right: () => import('@/views/child1')
          }
        }
      ]
    }
  ]
}
```
#### 简单路由 
`/about`和`/home`为最简单的两种路由配置，因为home页是访问平时访问比较多的页面，所以我们选择在router.js 文件在加载的时候就引入，而about页是平时访问频率较低的页面，所以我们选择按需引入，等到有路由请求的时候，再去动态引入。

#### 传参路由
`/argu/:name`是一种常见的传参路由，比如使用router-link来发送一个路由请求，并且给$route对象中传入params参数name。
`<router-link to="/argu/masia">传入参数name: 'masia'</router-link>`

需要注意的是： 
  1. 如果选择上述 `to="/argu/masia"` 这种方式传参数，系统会把路由中的参数写在地址栏中，比如：`localhost:8080/argu/masia`，这样用户在刷新页面的时候，系统会拿 地址栏的中的url地址，并且截取端口号后面的字符串，进行路由匹配。
  2. 如果路由匹配规则是
  ```javascript
  routes: [
    {
      path: '/argu',
      name: 'argu',
      component: () => import('@/views/argu')
    }
  ]
  ```
  选择 `:to="{ name: 'argu', params: { id: 2 }}"` 或者使用编程式路由 `this.$router.push({ name: 'argu', params: { id: 2 }})`这种方式传递参数，系统不会把params中的参数写在地址栏中，比如：`localhost:8080/argu`，第一次进入页面时，可以拿到params参数，但这样用户在刷新页面的时候，系统会拿地址栏的中的url端口号后面的字符串`/argu`，进行路由匹配。如果路由匹配规则中有path为`/argu`的路由，则会跳转到path为`/argu`这个页面，但是在这一次，系统在也拿不到params参数了。
  所以，要使用name进行路由匹配，并且传参，务必在路由匹配规则中将path上面的形参定义完整，正确规则定义如下：
  ```javascript
  routes: [
    {
      path: '/argu/:id',
      name: 'argu',
      component: () => import('@/views/argu')
    }
  ]
  ```
  3. 还遇到了好多不规范的路由规则定义，问题总结解决方案如下：
   1. 同名不同p，名匹配，谁前听谁，p匹配，正常
   2. 不同名同p，名匹配，正常，p匹配，谁前听谁
   3. 不同名不同p，正常
   4. 同p否，看p串
   5. 路由形参未定义，用名匹配并传参，刷新丢参

+ 编程式路由
```javascript
// 一般路由，使用router-link
<router-link to="/test/33">实参为id: 33</router-link>
<router-link :to="{ path: '/test/33' }">实参为id: 33</router-link>
<router-link :to="{ name: 'test', params: { id: 3 }}">实参为id: 33</router-link>
// 编程式路由
this.$router.push('/test/33') // 可以直接写path字符串
this.$router.push({ // 也可以传入一个包含了path的配置对象
  path: '/test/33'
})
this.$router.push({ // 也可以传入包含name和params参数的配置对象
  name: 'test',
  params: {
    id: 33
  }
})
// 还有一个常见的路由方法，
this.$router.replace() // 替换当前的路由，假如路由从A跳到B，然后从B到C使用的此方法，那么从C执行this.$router.go(-1)，退回的是A。
```
#### 嵌套路由
```javascript
routes: [
  {
    path: '/home',
    name: 'home',
    component: () => import('@/views/home'),
    children: [
      {
        path: '/child1',
        name: 'child1',
        component: () => import('@/views/child1')
      },
      {
        path: '/child2',
        name: 'child2',
        component: () => import('@/views/child2')
      }
    ]
  }
]
```
需要注意的是：
  1. 如果在children中的path中写`/child1`，那么访问child1子组件的path实际就为`/chlid`，在地址栏中也写为`/child`
  2. 如果在children中的path中写`child`，那么访问child1子组件的path实际就为`/parent/child`，在地址栏中也写为`/parent/child`
口诀：有杠为己p，无杠拼其父

#### 命名视图
```javascript
// 路由匹配规则
routes: [
  {
    path: '/home',
    name: 'home',
    components: {
      default: () => import('@/components/center')
      left: () => import('@/components/left'),
      right: () => import('@/components/right'),
    }
  }
]
```
对应的router-view
```html
<router-view name="left"/>
<router-view/> // 没有命名，代表填充的是default对应的路由组件
<router-view name="right"/>
```

#### 命名路由
在组件定义的时候加上name属性，在调用的时候通过$route中的name属性去调用。

#### 路由重定向
```javascript
routes: [
  {
    path: '/', // 在一级路由中设置重定向，则访问网站域名，直接重定向到/parent
    redirect: '/parent'
  },
  {
    path: '/parent',
    name: 'parent',
    component: () => import('@/views/parent'),
    children: [
      {
        path: '/', // 在子级路由设置'/'或者''，那么访问父级路由'/parent'，则会直接重定向到'/parent/child'
                  //  如果在子级路由中设置 '/fisrt'，任意的带斜杠的path，则会将当前重定向的路由按照一级路由处理
                  //  不能重定向有params参数定义的路由
                  //  重新定向可以传query参数，字符串方式和对象方式都可以
        redirect: 'child'
      },
      {
        path: 'child',
        name: 'child',
        component: () => import('@/views/child')
      }
    ]
  }
]
```
#### 路由组件接收props
非路由父组件给子组件传值，一般使用的是自定义属性，然后子组件中使用props接收。路由组件在路由跳转后，需要传递的参数值一般保存在$route中。
但是，其实路由组件也可以使用props来传值。
1. 布尔模式：路由中的props为一个布尔值：
路由匹配规则：
```javascript
routes: [
  {
    path: '/list/:name',
    name: 'name',
    component: () => import('@/components/list'),
    props: true // 启用props传参，给props传一个boolean类型的值
  }
]
```
list路由组件
```javascript
<template>
 <div>
   {{ name }}
 </div>
</template>

<script type="text/ecmascript-6">
export default {
  date () {

  },
  props: {
    name: {
      type: String
    }
  }
}
</script>
```
2. 对象模式：路由匹配规则中的props值为一个对象
```javascript
routes: [
  {
    path: '/list', // 这里不需要定义形参 :food
    name: 'name',
    component: () => import('@/components/list'),
    props: { // 给props传一个对象
      food: 'banner' // 不用的路由可以给food不同的值，虽然我不知道这个能用来干什么
    } 
  }
]
```
list路由组件
```javascript
<template>
 <div>
   {{ food }}
 </div>
</template>

<script type="text/ecmascript-6">
export default {
  date () {

  },
  props: {
    food: {
      type: String
    }
  }
}
</script>
```
3. 函数模式：适合根据当前的路由来做一些处理逻辑 ，从而设置传入组件的属性值
```javascript
export default [
  {
    path: '/list', // 这里不需要定义形参 :food
    name: 'name',
    component: () => import('@/components/list'),
    props: route => ({
      food: route.query.food // 把$route对象中query对象的food属性值拿出来，动态赋值给props中的food
                              // 但是好像这么做是多此一举，在路由组件中干嘛不直接取this.$route.query.food
    }) 
  }
]
```
list路由组件
```javascript
<template>
 <div>
   {{ food }}
 </div>
</template>

<script type="text/ecmascript-6">
export default {
  date () {

  },
  props: {
    food: {
      type: String
    }
  }
}
</script>
```
#### html5 history模式
文件 `/router/index.js`
```javascript
import Vue from 'vue'
import Router from 'vue-router'
import routes from './router'

Vue.use(Router)

const router = new Router({
  // mode: 'hash', // 默认值 是 hash模式，就是在url里面有一个＃号，在#号后面做路由的变化，页面是不会刷新的，本来就是单页面，只在这一个页面中变化
  mode: 'history', // history模式是利用history的一些api来做无刷新页面的路由跳转
  routes
})

// 在这里可以写路由守卫

export default router
```
使用history模式，必须后端做一些配置，因为当你访问localhost:8080的时候，后台会自动访问index.html，但是当你访问localhost:8080/list的时候，后端会把这个url当作静态资源来匹配，如果后台不做处理，浏览器会报404错误。
所以，后端需要配置如果没有匹配到指定的url，那么就返回index.html
但是当你既匹配不到静态资源，又匹配不到路由时，那就出问题了，所以一般需要在路由列表中最底下配置一个通用路由匹配规则
```javascript
routes: [
  // 上面都是具体的匹配规则
  {
    path: '*',
    component: () => import('@/views/error_404')
  }
]
```

#### 导航守卫
执行生命周期：路由发生跳转 -- 导航结束
比如：在跳转到一个页面的时候，判断用户有没有登录，如果已经登录了，则正常跳转，如果没有登录，则跳转到登录页面。
还有判断是否有访问某个页面的权限
+ 全局守卫（三个：router.beforeEach，router.beforeResolve，router.afterEach）
`文件router/index.js`
```javascript
import Vue from 'vue'
import Router from 'vue-router'
import routes from './router'

Vue.use(Router)

const router = new Router({
  mode: 'hash'
  routes
})

const HAS_LOGIN = true
// 在这里可以写路由守卫
router.beforeEach((to, from, next) => {
  if (to.name !== 'login') {
    if (HAS_LOGIN) next()
    else next({ //如果还没有登录， 则跳转到login页面
    // next() 和 this.$router.push() 方法中传入的值是一样的，可以是一个path字符串，也可以是对象
      name: 'login'
    })
  } else { // 如果访问的是登录页面，但是之前已经登录了，那么我们把让它跳到首页
    if (HAS_LOGIN) next({ name: 'home' }) 
  }
})

// 导航被确认（导航被确认指的是：所有导航钩子被确认）之前，所有组件内守卫和异步路由组件被解析之后被调用
router.beforeResolve((to, from, next) => {
  next()
})

// 导航被确认之后，使用场景：可以在条状之前写一个loading的样式，而在跳转之后，取消这个loading样式
router.afterEach((to, from, next) => {
  // afterEach中没有next，所以可以不传
  console.log('afterEach next: ',nexts) // afterEach next: undefined
})
export default router
```
+ 路由独享守卫（只有一个：beforeEnter）
文件`/router/router.js`
```javascript
export default [
  {
    path: '/home', // 这里不需要定义形参 :food
    name: 'home',
    component: () => import('@/views/home'),
    beforeEnter: (to, from, next) => {
      // 在这里进行路径的处理
      if (from.name === 'login') alert('这是从登录页来的')
      else alert('这不是从登录页来的')
      next()
    }
  }
]
```
+ 组件内守卫
<!-- home组件 -->
```javascript
<template>
  <div>
    this is home
  </div>
</template>

<sciprt type="text/esmascript-6">4
export default {
  name: 'home',
  data () {
    return {

    }
  },
  beforeRouteEnter: (to, from, next) => {
    // 路由触发，还没进来的时候调用，页面还没有渲染，在这里使用this，是获取不到实例的，但是可以next中使用组件实例
    next(vm => { // vm 就是当前的组件实例
      console.log(vm)
    })
  }),
  beforeRouteLeave: (to, from, next) => {
    // 路由即将离开时调用
    // 在这里可以做提示，比如：提示当前编辑内容还未保存，是否选择离开？ 
    const leave = confirm('您确定要离开吗？')
    if (leave) next();
    else next(false) // 给next传入false就不会发生跳转

    // 这个方法一般适用与适用router-link做跳转的场景
    // 如果我用的this.$route.push('/xxx')这样的方式做跳转，那我完全可以在确定点击之后去执行this.$route.puth('/xxx')
  }),
  beforeRouteUpdate: (to, from, next) => {
    // 这个钩子发生在，路由发生变化，组件被复用的时候调用
    // 比如：路由发生了 '/list/33' 到 '/list/22' 这样变化的时候，
    // mounted钩子只在请求为'/list/33'时执行了一次，但是我们的数据请求写在mounted中，
    // 所以点击选项卡，改变参数，执行路由'/list/22'时，mounted没有被执行，那么数据请求就没有被执行，
    // 以前的做法是 监听 $route 对象，然后在监听函数中再去写数据请求
    // 有了这个钩子，就可以直接在这个里面写了
    this.getDate(to.params.id) // 只需要在给选项卡绑定click时间，去改变id值
    next()
  }
}
</script>
```

#### 在这三种守卫中访问this
```javascript
<template>
  <div>
    <div ref="testTmp">
      this is test component
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
export default {
  data () {
      return {
        list: [{
          id: 1
        },{
          id: 2
        },{
          id: 3
        }]
      }
  },
  components: {

  },
  beforeRouteEnter: (to, from, next) => {
    console.log('before', this) // 拿不到this，输出undefined
    next(vm => console.log('vm', vm)) // 能拿到当前组件实例，但是拿不到组件中的数据，即使组件数据还未初始化
  },
  create () {
    console.log(this.list) // 没执行
    console.log(this.$refs.testTmp) // 没执行
  },
  mounted () {
    console.log(this.list) // 能拿到数据
    console.log(this.$refs.testTmp) // 能拿到模板
  },
  beforeRouteUpdate: (to, from, next) => {
    console.log('update', this) // 能拿到当前组件实例，但是拿不到组件中的数据，即使组件数据还未初始化
    next()
  }
}
</script>
```

#### beforeRouteUpdate 钩子的使用示例子，取代以前监听$route对象那种方法
```javascript
<template>
  <div>
    <div>
      <template v-for="item in list">
        <!-- 这里假如是一组选项卡，点击不同的选项卡，对应列表的id不同，点击完成之后要使用新的id来当作参数获取数据 -->
        <a @click="$router.push('/list' + item.id)">id为{{item.id}}</a> |
      </template>
    </div>
    <div>
      params中的id值为: {{$route.params.id}}
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
export default {
  data () {
      return {
        list: [{
          id: 1
        },{
          id: 2
        },{
          id: 3
        }]
      }
  },
  components: {

  },
  mounted () {
    console.log(this.$route.params.id) // 只在页面第一次加载的时候输出
  },
  beforeRouteUpdate: (to, from, next) => {
    // 这个钩子发生在，路由发生变化，组件被复用的时候调用
    // 比如：路由发生了 '/list/33' 到 '/list/22' 这样变化的时候，
    // mounted钩子只在请求为'/list/33'时执行了一次，但是我们的数据请求写在mounted中，
    // 所以点击选项卡，改变参数，执行路由'/list/22'时，mounted没有被执行，那么数据请求就没有被执行，
    // 以前的做法是 监听 $route 对象，然后在监听函数中再去写数据请求
    // 有了这个钩子，就可以直接在这个里面写了
    // 只需要在给选项卡绑定click时间，去改变id值
    console.log(to.params.id) // 点击选项卡会在控制台输出对应的 id 值
    next()
  }
}
</script>

<style scoped>

 
</style>
```

#### 总结：整个执行顺序
1. 导航被触发
2. 在失活的组件（即将离开的页面组件）里调用离开守卫 beforeRouteLeave
3. 调用全局的前置守卫 beforeEach
4. 在重用的组件里调用 beforeRouteUpdate（如果没有重用组件，则跳过这步骤）
5. 调用路由独享的守卫 beforeEnter
6. 解析异步路由组件
7. 在被激活的组件（即将进入页面的组件）里调用 beforeRouteEnter
8. 调用全局的解析守卫 beforeResolve
9. 导航被确认
10. 调用全局的 后置守卫afterEach
11. 触发DOM更新
12. 用创建好的实例调用beforeEnter守卫中的next回调函数

#### 路由中meta源信息的使用
修改不同路由下面的对应不同的title值
工具文件`lib/tools.js`
```javascript
const setTitle = (title) => {
  window.document.title = title || 'admin' // 如果title没有的话，会设置默认的admin
}
```
文件`router/index.js`
```javascript
import Vue from 'vue'
import Router from 'vue-router'
import routes from './router'

Vue.use(Router)

const router = new Router({
  routes
})
// 在此文件中引入工具函数，setTitle
import { setTitle } from '@/lib/tools'

router.beforeEach((to, from, next) => {
  to.meta && setTitle(to.meta.title) // 这里必须判断meta存在吗，如果meta存在，但是title不存在，访问to.meta.title也不会出错，会报title is undefined 
  //但是meta不存在，访问to.meta.title就会出错，因为title是to.meta.title中的第三级，一般情况下，如果没有二级，但是去访问二级，不会报undefined，
  //但是连二级都没有，再去访问第三级，就是出错
  next();
})
export default router
```
文件`router/router.js`
```javascript
export default [
  {
    path: '/home',
    name: 'home',
    component: () => import('@/views/home')
  }
]
```
#### 组件的过渡动效
如果值给一个组件设置过渡，则只需要用<transition></transtion>包住
```javascript
<transition name="router">
  <router-view/>
</transtion>
```
如果给多个路由逐渐设置过渡，需要使用<transition-group></transition-group>保住多个<router-view/>, 并且为每一个<router-view/>设置一个key值
```javascript
<transition-group  name="router">
  <router-view key="default" name="default"/>
  <router-view key="email" name="email"/>
  <router-view key="tel" name="tel"/>
</transtion-group>
```
```css
.router-enter { // 从无到进入时
  opacity: 0;
}
.router-enter-active { 进入到完全显示，一般transition属性设置在这个阶段
  transition: opacity 1s ease;
}
.router-enter-to { 完全显示时
  opacity: 1;
}
.router-leave { // 从完全显示到开始离开时
  opacity: 1;
}
.router-leave-active { // 从开始离开到消失
  transition: opacity 1s ease;
}
.router-leave-to { // 完全消失
  opacity: 0;
}
```

#### keep-alive