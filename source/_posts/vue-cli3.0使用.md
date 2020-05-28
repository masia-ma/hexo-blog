---
title: vue-cli3.0使用
categories: 
  - vue
  - vue-cli3.0
tags: 
 - vue
 - vue-cli3.0
date: 2019-06-10
---
# 安装
<!-- more -->
 + 回顾vue-cli2.0的使用
```bash
npm install -g webpack // 全局安装webpack
npm install vue-cli -g // 全局安装vue-cli脚手架构建工具
vue --version // 或者 vue -V 查看安装vue-cli版本 ，我之前的版本为2.9.3
vue init webpack vue_practice // 创建项目（需要注意的是，项目名字不能包含“-”）
cd vue_practice // 进入刚刚创建好的项目目录
npm i // 安装项目所需要的依赖
npm run dev // 本地启动项目
npm run build // 项目打包
npm i serve // 安装一个本地模拟服务器，用来运行打包完成的项目
serve dist // 运行打包完成的项目
```
  + vue-cli3.0
```bash
npm i @vue/cli -g
vue ui // 启动图形化项目的创建
npm run serve // 本地项目启动
```



