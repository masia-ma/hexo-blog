---
title: node安装引发的惨案
categories:
  - 安装
tags:
  - install
abbrlink: 2a6653ec
date: 2019-04-25 16:10:10
---
# node安装
本人在学习node初期，按照网上的教程傻瓜式的给win10安装了node，当时没有真正理会每一步安装的目的和意义，直接导致后期在使用npm的时候出现了各种各样的问题（尤其是在npm全局安装的时候）
<!-- more -->
+ ps: 如果你的机器已经安装了nodejs，而且在npm全局安装是出现了各种各样的问题（比如找不到npm指向的全局安装目录，安装权限问题），那么这篇文章可能会找到一些问题的答案，或者可以完全卸载重新安装。

## 在windows中完全移除nodejs（npm包含在nodejs中）

1. 在 【windows设置>应用】 中，找到nodejs，然后点击卸载
2. 删除所有与nodejs相关的文件目录以及文件
C:\Program Files (x86)\Nodejs
C:\Program Files\Nodejs
C:\Users\{User}\AppData\Roaming\npm（或%appdata%\npm）
C:\Users\{User}\AppData\Roaming\npm-cache（或%appdata%\npm-cache） 
以及之前的nodejs的安装目录（比如我刚开始将nodejs安装在了F:\nodejs中，那就将这个目录删除）
3. 删除所有与node或者npm相关的环境变量
一般包括用户环境变量中的PATH和系统变量中的NODE_PATH、Path（注意：只要看见包含npm或者node就大胆把它给删除了，不管是大写的还是小写的）

## 在windows中安装nodejs
1. 打开nodejs官网 https://nodejs.org/en/
    选择Download for Windows (x64)，LTS版本，此版为稳定版，点击下载，得到一个msi格式的nodejs安装文件
2. 在你经常存放软件的磁盘中先建一个nodejs文件夹，比如：F:\nodejs
    点击刚刚下载的nodejs安装文件，记得在选择安装目录的时候选择F:\nodejs，一路next，直到finish
3. 此时，你使用win+R打开命令行工具，输入指令`node -v`,已经可以看见nodejs的版本号了，输入指令`npm -v`,同样可以看见npm的版本号,证明你已经成功安装了nodejs
4. 接下来我们需要配置一些nodejs环境变量，也就是设置一下你的npm工具在全局安装一些例如webpack工具的时候，安装在哪里。
   + 注意：这里的配置非常重要，因为你安装完nodejs之后，默认设置以后的全局安装目录在C:\Program Files\Nodejs这个文件夹中，所以我们在以后全局安装的时候，就会因为权限的问题而安装失败，亲测管理员身份安装也会失败。。。（这个问题困扰了我好久）
   1. 在F:\nodejs\中新建两个文件夹，一个名为node_global，另外一个名为node_cache
      + node_global就是以后全局安装的目录，node_cache就是以后全局安装时产生日志文件目录
   2. 设置环境变量
      + 点击用户环境变量中的变量PATH，找到C:\User\你的计算机用户名\AppData\Roaming\npm,点击编辑，编辑为`F:\nodejs\node_global `
      + 在系统环境变量中新建一个名为NODE_PATH的变量，设置变量值为`F:\nodejs\node_global\node_modules`后面这个node_modules目录会在全局安装时自动生成。

   3. 打开命令行工具输入`npm config set prefix "F:\nodejs\node_global"`以及`npm config set cache "F:\nodejs\node_cache"`
   4. 尝试全局安装nodemon，`npm i -g nodemon`，等待安装完成之后，再输入`nodemon -v`,在命令行显示版本号，证明全局安装成功
      + ps：打开【F:\nodejs\node_global】目录你会看到刚刚安装的nodemon相关的文件，再点击 其中的node_modules文件夹，你会看到有一个名为nodemon的文件夹。