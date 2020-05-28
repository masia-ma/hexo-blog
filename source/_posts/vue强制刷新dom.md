---
title: vue强制刷新dom
categories: 
  - vue
  - vue-awesome-swiper
tags: 
  - vue
  - vue-awesome-swiper
date: 2019-06-01 09:00:00
---

## 有这样一个业务需求，实现一个轮播效果，如果后台数据项小于4个，那么就让这几个数据项一组显示出来，并且水平flex排列。如果后台数据项大于等于4个，那么让数据4个为一组显示，并且让这组数据项实现轮播效果，点击一次导航键，只滑动一个数据项。
<!-- more -->
分析：
1. 涉及到轮播图插件的使用，vue-awesome-swiper
2. 轮播图组规划问题，几个为一组，组内项的间隔是多少，点击一次导航滑动一组，还是滑动一个组内项
3. 有两个选项卡，点击任意一个选项卡，只会引起params参数的变化，从而发起请求，而不是因为跳转路由而发起的数据请求，所以页面只在第一次加载的时候执行一次mounted()。
4. 根据后台数据，动态规划vue-awesome-swiper的配置（注意：这里不是动态给swiper嵌套数据，而是动态配置swiper的参数）

## 解决思路：
首先想到的是，点击选项卡，实现的不是路由的跳转，而实现的是`/solution/3`与`/solution/4`这样的路由变化，但是仅有这样的路由变化不会再次引起页面的mounted()。
然后想到为后台接收到的数据设置监听，然后根据后台数据的数组长度，从而修改 swiper 配置参数。
还需要在修改swiper配置参数之后，重新渲染页面或者swiper代码段，不然修改的swiper参数不会生效。

## 之前的失败尝试：
```javascript
// vue 组件
<template>
    <!--轮播图-->
    <div class="swiper-con">
      <swiper :options="swiperOption1" id="swiper1"> // swiperOption1为配置参数
        <swiper-slide v-for="item of msg" :key="item.index">
          <img src="@/assets/computer_icon.png" alt="">
          <h1>{{item.title}}</h1>
          <p class="ellipsis">{{item.con}}</p>
        </swiper-slide>
      </swiper> 
      <div class="swiper-button-next swiper-button-next1"></div>
      <div class="swiper-button-prev swiper-button-prev1"></div>
      <div class="swiper-pagination"></div>
    </div>

</template>

<script>
import axios from 'axios'
import "swiper/dist/css/swiper.min.css";
import { swiper, swiperSlide } from "vue-awesome-swiper";

export default {

  name: 'swiper_bussiness',
  data () {
    return {
      msg: [{
        title: '政府：资源分布不均',
        con: '资源分布不均，专家不能充分利用。中国30%大城市集中了80%的优质医疗资源，同时，跨科室跨机构协同业务开展困难，突发情况下，医疗救援效率低。'
      }, {
        title: '医院：急需培养人才',
        con: '医疗从业人员短缺，优质专家资源短缺，医院间、医院与医学院校间临床教学和研讨开展困难。'
      },{
        title: '政府：资源分布不均',
        con: '资源分布不均，专家不能充分利用。中国30%大城市集中了80%的优质医疗资源，同时，跨科室跨机构协同业务开展困难，突发情况下，医疗救援效率低。'
      }]
    }
  },
  computed: {
    swiperOption1 () {
      if (this.msg.length<4) { // 监听接收后台数据的数组长度的变化，从而动态修改swiper参数，事实上，swiperOption1 会因为this.msg的变化而执行这个计算属性，但不会根据新的 swiperOption1 作为参数去自动渲染dom，解决此问题必须让其在新生成的 swiperOption1 之后去强制渲染dom
        return {
          slidesPerView: this.msg.length, // 数组长度小于四个时，一组全部显示
          spaceBetween: 70, // 数组项间隔
          freeMode: true,
          navigation: {
            nextEl: '.swiper-button-next1',
            prevEl: '.swiper-button-prev1'
          }
        } 
      } else {
        return {
          slidesPerView: 4,
          spaceBetween: 32,
          freeMode: true,
          navigation: {
            nextEl: '.swiper-button-next1',
            prevEl: '.swiper-button-prev1'
          }
        } 
      } 
    }
  },
  components: {
    swiper,
    swiperSlide
  },
  mounted () { // 页面第一次加载进来，会执行数组赋值操作，并且swiperOption1会根据this.msg.length计算之后去配置swiper
    axios.get('/api/detail/'+ this.$route.params.id).then(res => {
      this.msg = res.data 
    })
  },
  methods: {
    
  },
  watch: {
    '$route.params.id' () { // 监听params参数的变化，如有变化，则重新发起请求，并且为this.msg赋值
      axios.get('/api/detail/'+this.$route.params.id).then(res => {
        console.log(res.data)
        this.msg = res.data[0].val
      })
    }
  }
}
</script>
```

<!-- 最终解决 -->

## 利用v-if去重新强制渲染dom

```javascript
// vue 组件
<template>
    <!--轮播图-->
    <div class="swiper-con">
      <swiper :options="swiperOption1" id="swiper1" v-if="showSwiper">
        <swiper-slide v-for="item of msg" :key="item.index">
          <img src="@/assets/computer_icon.png" alt="">
          <h1>{{item.title}}</h1>
          <p class="ellipsis">{{item.con}}</p>
        </swiper-slide>
      </swiper> 
      <div class="swiper-button-next swiper-button-next1"></div>
      <div class="swiper-button-prev swiper-button-prev1"></div>
      <div class="swiper-pagination"></div>
    </div>

</template>

<script>
import axios from 'axios'
import "swiper/dist/css/swiper.min.css";
import { swiper, swiperSlide } from "vue-awesome-swiper";

export default {

  name: 'swiper_bussiness',
  data () {
    return {
      msg: [{
        title: '政府：资源分布不均',
        con: '资源分布不均，专家不能充分利用。中国30%大城市集中了80%的优质医疗资源，同时，跨科室跨机构协同业务开展困难，突发情况下，医疗救援效率低。'
      }, {
        title: '医院：急需培养人才',
        con: '医疗从业人员短缺，优质专家资源短缺，医院间、医院与医学院校间临床教学和研讨开展困难。'
      },{
        title: '政府：资源分布不均',
        con: '资源分布不均，专家不能充分利用。中国30%大城市集中了80%的优质医疗资源，同时，跨科室跨机构协同业务开展困难，突发情况下，医疗救援效率低。'
      }],
      showSwiper: true // 显示 swiper
    }
  },
  computed: {
    swiperOption1 () {
      if (this.msg.length<4) { // 监听接收后台数据的数组长度的变化，从而动态修改swiper参数，事实上，swiperOption1 会因为this.msg的变化而执行这个计算属性，但不会根据新的 swiperOption1 作为参数去自动渲染dom，解决此问题必须让其在新生成的 swiperOption1 之后去强制渲染dom
        return {
          slidesPerView: this.msg.length,
          spaceBetween: 70,
          freeMode: true,
          navigation: {
            nextEl: '.swiper-button-next1',
            prevEl: '.swiper-button-prev1'
          }
        } 
      } else {
        return {
          slidesPerView: 4,
          spaceBetween: 32,
          freeMode: true,
          navigation: {
            nextEl: '.swiper-button-next1',
            prevEl: '.swiper-button-prev1'
          }
        } 
      } 
    }
  },
  components: {
    swiper,
    swiperSlide
  },
  mounted () { // 页面第一次加载进来，会执行数组赋值操作，并且swiperOption1会根据this.msg.length计算之后去配置swiper
    axios.get('/api/detail/'+ this.$route.params.id).then(res => {
      this.msg = res.data
    })
  },
  methods: {
    
  },
  watch: {
    '$route.params.id' () { // 监听params参数的变化，如有变化，则重新发起请求，并且为this.msg赋值
      axios.get('/api/detail/'+this.$route.params.id).then(res => {
        this.msg = res.data // 为this.msg赋值
      })
    },
    msg () { // 监视数据变化之后，在强制刷新
      this.showSwiper = false // 让 swiper 的display 变为none 
      this.$nextTick(() => { // 等到所有数据处理完成，当然也包括 swiperOption1 根据数组长度重新计算完成
        this.showSwiper = true // 让 swiper 的diaplay 变为block，即手动完成重新渲染
      })
    }
  }
}
</script>
```