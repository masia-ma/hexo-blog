---
title: vuex中遇到的问题1
categories:
  - 总结
  - vue
  - vuex
tags:
  - 总结
  - vue
  - vuex
date: 2019-05-30 14:00:00
---
# 首先注意：
  vuex解决的是一个pages中组件间的数据存储数据存储问题和共同使用问题，而不是跨page，并不是最初理解的对于整个vue项目全局的数据存储空间
  <!-- more -->
+ vuex
  ① Vue Components 是我们的 vue 组件，组件会触发（dispatch）一些事件或动作，也就是图中的 Actions；
  ② 我们在组件中发出的动作，肯定是想获取或者改变数据的，但是在 vuex 中，数据是集中管理的，我们不能直接去更改数据，所以会把这个动作提交（Commit）到 Mutations 中；
  ③ 然后 Mutations 就去改变（Mutate）State 中的数据；
  ④ 当 State 中的数据被改变之后，就会重新渲染（Render）到 Vue Components 中去，组件展示更新后的数据，完成一个流程。