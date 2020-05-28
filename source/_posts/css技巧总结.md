---
title: css技巧总结
categories:
 - html
 - css
tags: 
 - html
 - css
date: 2019-05-10 18:00:00
---

## 垂直水平居中
<!-- more -->
  + 使用css3中的transform(内容高度和宽度都可以是自动)
```css
.parent
  position relative
  .children 
    position absolute
    top 50%
    left 50%
    transform translate(-50%, -50%)
    width 不固定
    height 也可以不固定
```