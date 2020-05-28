---
title: sass的使用
categories:
  - css
  - sass
tags:
  - css
  - sass
---
+ 定义变量，使用$
<!-- more -->
```css
$mainColor: #ccc;
$siteWidth: 1024px;
$borderStyle: dotted;
body {
  color: $maincolor;
  width: $siteWidth;
  border: 1px $borderStyle $mainColor;
}
```
+ 运算符号
```css
body {
  margin: (14px/2);
  top: 50px + 100px;
  right: 80 * 10%;
}
```
+ 颜色函数
CSS 预处理器一般都会内置一些颜色处理函数用来对颜色值进行处理，例如加亮、变暗、颜色梯度等。
1.sass的颜色处理函数：
```css
lighten($color, 10%); 
darken($color, 10%);  
saturate($color, 10%);   
desaturate($color, 10%);
grayscale($color);  
complement($color); 
invert($color); 
mix($color1, $color2, 50%);
```
```css
$color: #0982C1;
h1 {
  background: $color;
  border: 3px solid darken($color, 50%);
}
```
+ 导入 (Import)
注意：导入文件中定义的混入、变量等信息也将会被引入到主样式文件中，因此需要避免它们互相冲突。 
scss中的导入只是将多个文件合并成了同一个文件，导入的文件在导入语句之后生效，如：@import "style/index.scss"
如下a.scss中引入了b.scss: 

a.scss:
```scss
$width : 100px;

.before {
  width: $width;
}

@import "b";

.after {
  width: $width;
}

.container {
  width: $width;
  height: $height;
  border: 1px solid;
}
```
b.scss:
```scss
$width : 200px;
$height : 200px
```

最后生成的
```css
.before {
  width: 100px; 
}
.after {
  width: 200px; 
}
.container {
  width: 200px;
  height: 200px;
  border: 1px solid;
}
```
默认值：$width : 200px !default
用法：可以在被引入的文件中定义默认值，主文件@import这个文件后，如果主文件中变量有定义，就不会被覆盖。
+ 继承
sass可通过@extend来实现代码组合声明，使代码更加优越简洁。
```scss
.message {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}
.success {
  @extend .message;
  border-color: green;
}
.error {
  @extend .message;
  border-color: red;
}
.warning {
  @extend .message;
  border-color: yellow;
}
```
结果：
```css
.message, .success, .error, .warning {
  border: 1px solid #cccccc;
  padding: 10px;
  color: #333;
}
.success {
  border-color: green;
}
.error {
  border-color: red;
}
.warning {
  border-color: yellow;
}
```
+ Mixins（混入）这个混入就好比在某个样式类中执行了一个定义好的函数，返回值就是这些函数里面的东西
Mixins 有点像是函数或者是宏，当某段 CSS 经常需要在多个元素中使用时，可以为这些共用的 CSS 定义一个 Mixin，然后只需要在需要引用这些 CSS 地方调用该 Mixin 即可。
1.Sass 的混入语法
sass中可用mixin定义一些代码片段，且可传参数，方便日后根据需求调用。比如说处理css3浏览器前缀：
```scss
@mixin error($borderWidth: 2px) {
  border: $borderWidth solid #F00;
  color: #F00;
}
.generic-error {
  padding: 20px;
  margin: 4px;
  @ include error(); //这里调用默认 border: 2px solid #F00;
}
.login-error {
  left: 12px;
  position: absolute;
  top: 20px;
  @ include error(5px); //这里调用 border:5px solid #F00;
}
```

# 使用场景
+ 3D文本
要生成具有 3D 效果的文本可以使用 text-shadows ，唯一的问题就是当要修改颜色的时候就非常的麻烦，而通过 mixin 和颜色函数可以很轻松的实现：
```scss
@mixin text3d($color) {
  color: $color;
  text-shadow: 1px 1px 0px darken($color, 5%),
               2px 2px 0px darken($color, 10%),
               3px 3px 0px darken($color, 15%),
               4px 4px 0px darken($color, 20%),
               4px 4px 2px #000;
}

h1 {
  font-size: 32pt;
  @ include text3d(#0982c1);
}
```
+ 高级语法
在sass中，还支持条件语句： @if可一个条件单独使用，也可以和@else结合多条件使用
```scss
$lte7: true;
$type: monster;
.ib{
    display:inline-block;
    @if $lte7 {
        *display:inline;
        *zoom:1;
    }
}
p {
  @if $type == ocean {
    color: blue;
  } @else if $type == matador {
    color: red;
  } @else if $type == monster {
    color: green;
  } @else {
    color: black;
  }
}
```
结果：
```scss
.ib{
    display:inline-block;
    *display:inline;
    *zoom:1;
}
p {
  color: green; 
}
```
+ for循环
以后再说