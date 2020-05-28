---
title: scss使用心得
tags: 
  - css
  - scss
categories: 
  - css
  - scss
---
# 在做前端页面的时候，需要提取公共的样式，利用scss可以快速的规定项目主题。
<!-- more -->
通过过去对css样式编写以及查看element-variables.scss，得出如下总结：
```scss


// main-color
$color-primary: #de4306 !default;
$color-white: #FFFFFF !default;
$color-black: #000000 !default;
$color-success: #67C23A !default;
$color-warning: #E6A23C !default;
$color-danger: #F56C6C !default;
$color-info: #909399 !default;



// Background
/// color|1|Background Color|4
$background-color-base: #F5F7FA !default;

// 文本颜色
/// color|1|Font Color|2
$color-text-primary: #303133 !default;
/// color|1|Font Color|2
$color-text-regular: #606266 !default;
/// color|1|Font Color|2
$color-text-secondary: #909399 !default;
/// color|1|Font Color|2
$color-text-placeholder: #C0C4CC !default;



// 通用样式定义
.color-orange {
  color: $orange_color;
}
.color-gray {
  color: $gray_color;
}
.color-blue {
  color: $blue_color;
}
.color-black {
  color: $black_color;
}
.fl {
  float: left;
}
.fr {
  float: right;
}
.clearFix:after {
  display: block;
  content: '';
  height: 0;
  clear: both;
  overflow: hidden;
}
.ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

// 通用mixins定义
@mixin clearFix () {
  *zoom: 1;
  &::after {
    content: '';
    display: block;
    clear: both;
    height: 0;
    overflow: hidden;
  }
}
@mixin fl () {
  float: left;
}
@mixin fr () {
  float: right;
}
@mixin ellipsis () {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
@mixin divisionBar ($bg:#a52705) {
  height: 40px;
  background: $bg;
  box-sizing: border-box;
  padding: 5px 0;
  line-height: 30px;
} 
```