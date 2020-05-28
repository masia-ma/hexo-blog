---
title: mongoose使用
categories:
  - node.js
  - mongodb
  - mongoose
tags:
  - node.js
  - mongodb
  - mongoose
abbrlink: 62ea38d6
date: 2019-04-30 16:00:00
---

## 注意:
+ 以params方式传参的方式编写的接口，params本身也被看做是路由的一部分，所以params参数不能为空，为必传项，不然会找不到接口
+ 以query方式传参的方式编写的接口可以不传
<!-- more -->
+ get方式和pos方式，都可以使用json对象格式来传入参数，都用res.body来接收，注意在使用`req.body`之前要使用`app.use(express.json())`来配置一下，但是一般使用json对象来传入数据体，使用query方式或者params方式来传入查询条件。
+ 客户端（发送表单格式数据: `Content-Type: application/x-www-form-urlencoded`）
  服务端（处理客服端发送的表单格式的数据`app.use(express.urlencoded({extended: false}))`或`app.use(bodyParser.urlencoded({extended: false}))`）
+ 客户端（发送json对象格式数据: `Content-Type: application/json`）
  服务端（处理客服端发送的json对象格式的数据`app.use(express.json())`或`app.use(bodyParser.json())`）

## 准备: 
```javascript
const mongoose = require('mongoose') // 导包
mongoose.connect('mongodb://localhost:27017/express-test', {useNewUrlParser: true})
```
## 创建Schema：
```javascript
const ProductSchema = new mongoose.Schema({ // 创建模型1
  title: {
    type: String, // 类型
  },
  content: {
    type: String
  }
})
const Product = mongoose.model('Product', ProductSchema) // 创建模型2

module.exports = {
  Product // 把模型导出去
}
```
## 删除：
+ 删除集合
```javascript
Product.db.dropCollection('products') // 模型为Product ，但是在mongodb数据库中保存的集合是 products
```
## 查找：
+  在products集合中查找所有的记录
```javscript
const data = Product.find()
```
+  在products集合中查找所有的记录，并且只显示两条
```javscript
const data = Product.find().limit(2)
```
+  在products集合中查找所有的记录，并且只显示两条,并且跳过最前面的1条
```javscript
const data = Product.find().skip(1).limit(2) // 将skip()以及limit()结合起来，可以做分页
```
+ 使用where()进行条件查询(find()和findOne()后面都可以跟where()做组合查询)
```javascript
const data = Product.find().where({ // where()方法的传入参数为对象格式，即使只有一个查询条件，也得写成键值对形式
  title: '产品2'
})
```
+  使用sort进行排序，可以用来做产品列表页接口
```javascript
const data = Product.find().sort({
  _id: 1 // 1 表示正序(从小到大，一般默认都是正序，最新的排在最后面)， -1 表示倒序(从大到小)
        //这里的 _id 是一个16进制的值，也是可以比较大小的
})
```
+ 根据id查找，可以用来做详情页的接口
```javascript
app.get('/products/:id', async (req, res) => {
        // 这里使用params方式传参数
  const data = await Product.findById(req.params.id)
  res.send(data) // 在req.params中获取参数，不需要中间件再处理req
})
```
+ 根据条件查询一条数据
```javascript
app.post('/login', async (req, res) => {
  const user = await User.findOne().where(req.body)
  res.send(user)
})
```
+ 查询全部和条件查询通用接口
```javascript
app.get('products', await (req, res) => {
  if (JSON.strify(req.query) == '{}') { // 查询条件是否为空，若为空，则查询全部
    const products = await Product.find()
    return res.send(products)
  }
  const products = await Product.find().where(req.query) // 按照req.query中的约束条件查询
  res.send(products)
})
```
## 插入：
+ 往products集合中插入多条记录
```javascript
Product.insertMany([
  {title: '产品1', content: '这是产品1'},
  {title: '产品2', content: '这是产品2'},
  {title: '产品3', content: '这是产品3'}
])
```
+ 使用create() 往集合中插入一条记录
```javascript
const app = express()
app.use(express.json())
// 一般添加数据的请求都是post请求，所以要先使用app.use(express.json()) 处理传过来的json数据
app.post('/products', async (req, res) => {
  const data = req.body // 保存客户端传过来的json数据
  const product = await Product.create(req.body) 
  // 这里的await一定要加上，不然会先执行下面的res.send()
  res.send(product)
})
```
## 修改：（一般会使用put或者patch两种请求方式）
+ 先说两个请求方式，put和patch
 1. patch表示的是部分修改
 2. put表示的是直接覆盖
+ 先查，后改，再存储
```javascript
app.put('/products/:id', async (req, res) => {
  const product = await Product.findById(req.params.id) // 先找到指定产品
  product.title = req.body.title // 改值，记得在发送请求的时候，要在请求头中加上 "Content-Type": "application/json"，意思是，发送的数据类型是json格式的，这样req.body中才能拿到前台传来的json数据
  await product.save() //  保存到集合中也是异步的，所以要加await 
  res.send(product)
})
```
## 删除
+ 按照id删除
```javascript
<!-- 方法一 -->
app.delete('/product/:id', async (req, res) => {
  const product = await Product.findById(req.params.id)
  await product.remove()
  res.send({
    success: true
  })
})
<!-- 方法二 -->
app.post('/deleteProductById', async (req, res) => {
  await Product.remove({
    _id: req.body.id
  })
  res.send('delete success')
})
```
+ 按id批量删除
```javascript
app.post('/deleteByIds', async (req, res) => {
  await Product.remove({
    _id: {
      $in: req.body.ids // ids 是多个id组成的数组
    }
  })
})
```
+ 按照同一个title批量删除
```javascript
<!-- 方法一 -->
app.post('/products/:title', async (req, res) => {
  const products = await Product.find().where({
    title: req.params.title
  })
  console.log(products instanceof Array) // 查询出来的products是一个数组
  console.log(products) 
  for (var i = 0; i < products.length; i ++) {
    await  products[i].remove() // 遍历删除
  }
  res.send('remove products complete') // 返回删除成功
})
<!-- 方法二 -->
app.delete('productByTitle', async (req, res) => {
  await Product.remove({
    title: req.params.title
  })
  res.send(Delete by title is success.')
})
```
