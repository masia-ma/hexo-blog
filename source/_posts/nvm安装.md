---
title: nvm安装
categories: 
  - 安装
  - nvm
tags: 
  - 安装
  - nvm
time: 2019-04-26
---
### nvm用来管理多个版本的node
但是在安装nvm之前如果本机已经安装了node，那么需要先删除node所配置的环境变量，比如在系统环境变量中的NODE_HOME等
<!-- more -->
# 下载安装nvm
1. 
[nvm](https://github.com/coreybutler/nvm-windows/releases)，并且选择nvm-setup.zip，下载并解压

2. 选择安装目录，会让你选择两次
第一次选择的是nvm的系统目录，里面包含了nvm.exe等。我选择的目录是`D:\nvm\nvm`
第二次选择的nodejs是一个快捷方式目录，我选择的是`D:\nvm\nodejs`

  1. 其实nvm的原理就是在使用命令`nvm use nodejs版本`选择nodejs版本的时候，nvm帮你把`D:\nvm\nodejs`这个目录的指向换成了指定的版本号对应的nodejs目录。
  2. 所有的nodejs的目录全部放在nvm中，比如你执行 `nvm use 12.6.0`，他就会先去 `D:\nvm\nvm` 中找有没有v12.6.0这个目录，如果有的话，他就把`D:\nvm\nodejs`这个文件目录的变成`D:\nvm\v12.6.0`的一个快捷方式。
  3. 如果在`D:\nvm\nvm`中没有找到v12.6.0这个目录，那你必须先使用命令`nvm install 12.5.0`来把v12.5.0版本的nodejs安装包下载到`D:\nvm`中，然后再去使用`nvm use 12.6.0`修改`D:\nvm\nodejs`的指向。

{% asset_img nvm_install4.png nvm和nodejs目录说明 %}
{% asset_img nvm_install2.png D:\nvm中有两个版本的nodejs文件包 %}

3. 配置环境变量
  1. 用户环境变量
  先在用户环境变量中的`path`中配置nvm的环境变量，在path中加入`D:\nvm`,其实配置环境变量就是你在命令框中执行nvm指令的时候，他要去在环境变量中配置的目录路径去找有没有nvm.cmd或者nvm.exe，有的话才能在命令行中正常执行。否则他就会输出`nvm不是内部或者外部指令`
  2. 系统环境变量
  在系统环境变量中先定义两个变量`NVM_HOME=D:\nvm\nvm`和`NVM_SYMLINK=D:\nvm\nodejs`，然后再把这两个变量加到`path`中。（这里有点我有点疑惑的是网上的教程只在系统环境变量中写了`NVM_HOME=D:\nvm\nvm`并且加入path，然后在命令行执行`nvm -v`就正常执行了，而我这样配置却没有生效。所以我在用户环境变量中的path中写了`D:\nvm\nvm`，才可以正常执行`nvm`。但是执行node相关的命令是正常的，也就意味着`NVM_SYMLINK=D:\nvm\nodejs`生效了，系统可以通过`NVM_SYMLINK=D:\nvm\nodejs`这个环境变量来找到`D:\nvm\nodejs`目录下面的`node.exe`
{% asset_img nvm_install3.png 执行命令就是通过环境变量所指的目录去寻找该目录下面对应的可执行程序 %}

4. 配置npm的全局安装目录
  1. 这个目录里面放的是通过npm来安装的一些全局的包，比如通过`npm i webpack@3.6.0 -g`安装的全局的webpack@3.6.0目录就在这里。
  2. 执行命令`npm config set prefix "D:\nvm\npm"`，执行完成之后会在`C:\Users\MA`中出现一个名为`.npmrc`的文件，里面的内容是prefix=D:\nvm\npm，
  `npm config set prefix 目录`的意思是设置npm的全局安装目录，后面生成的那个文件暂时没明白是啥意思。
  3. 为这个npm的全局安装目录编写环境变量，在用户环境变量中的path中加入`D:\nvm\npm`
5. 使用npm全局安装nodemon等工具
  1. `npm i nodemon -g`
  2. `nodemon -v`，打印输出nodemon的版本
{% asset_img nvm_install5.png %}


### 总结：