---
title: async和await的妙用-异步方法同步执行
date: 2019-07-08 10:49:30
categories:
  - js
  - async-await
tags:
  - js
  - async-await
---

# 在项目中常常需要在调用接口前后执行一些操作，来对一系列的异步方法执行顺序进行控制
<!-- more -->
我们一般的方法是通过回调函数，在调用接口成功的回调函数中执行一些我们之后的操作
1. 如果不使用promise，那就比较恐怖了，常见的就是普通回调的地狱
2. 如果使用了promise对象的`.then()`方法控制异步执行，虽然可以控制异步方法的执行顺序，但是会将整个业务流程打散，我们要想理清整个业务流程就得从一个.then()中去找下一个业务流程节点
3. 利用async-await保证方法执行流程的完整性，这样在一个async方法中就可以清晰的看见整个业务流程的执行顺序

# 代码对比
业务流程：我们在点击提交之后：
(1) 先去判断用户有没有输入评价（评价可选字段，但是我们希望让用户输入），如果没有，那么弹出确定框提示用户还没有输入评价内容，是否确定提交，
(2.1) 如果用户点确定，调用save接口且确认框消失
(2.2) 如果用户点取消，不调用save且确认框消失
(3) save调用成功，通知工单列表组件，再次获取新的工单列表
(4) 详情框消失

2. promise对象的.then()方法实现方式
```javascript 
// 评价弹框组件
methods: {
  submitComment () {
    if (this.comment == '') {
      // (1)
      this.$confirm('您还没有输入评价内容，确认提交吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消'
      }).then(() => {
        // (2.1)
        this.dealMark()
      }).catch(() => { // (2.2) 点击了确认框的取消，确认框会单纯的消失 })
    }
  },
  dealMark () {
    saveComment().then(res = > {
      if (res.retCode == 200) {
        this.$message({
          type: 'success',
          message: '评价成功！'
        })
        // (3)
        this.$emit('commentCompleted') 
      } else {
        this.$message({
          type: 'warning',
          message: '评价失败，请稍后重试'
        })
      }
      // (4)
      this.dialogFormVisible = false
    }).catch(err => {
      this.$message({
        type: 'error',
        message: '系统异常'
      })
      this.dialogFormVisible = false
    })
    // this.dialogFormVisible = false // 弹框会在saveComment.thne()方法执行之前就关闭
  }
}
```
可以看出(3)和(4)步骤等于放在了(2.1)步骤里面执行的，缺点：如果整个业务流程非常长且复杂，那么就势必会有很多的步骤需要嵌套在前一个步骤中，虽然整个流程是整个一颗树型结构，单这颗树却是打散的，你很难在一个完整的代码块中看清整个流程。

3. async和await的实现方式
+ 带try catch 写法
```javascript
// 评价弹框组件
methods: {
  async submitComment () { // 这里的async 是针对里里面的await而言的，里面要使用await去执行await目标promise对象.then()的回调函数，外部必须在函数名前面加async，别人调它还是this.submitComment()
    if (this.comment == '') {
      try {
        // 这个代码块中相当于.then(res => {})回调函数的函数体
         // (1)
        await this.$confirm('您还没有输入评价内容，确认提交吗？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消'
        }) // 这里的await是正对this.$confirm()返回的promise对象的.then()方法而言的
        try {
          // (2.1)
          let { commentSuccess } = await this.dealMark()
          if (commentSuccess) {
            // (3)
            this.$emit('commentCompleted')
          } 
        } 
      } 
    }
    // (2.2) (4)
    this.dialogFormVisible = false 
  },
  dealMark () {
    return new Promise((resolve, reject) => {
      saveComment().then(res => {
        if (res.retCode == 200) {
          this.$message({
            type: 'success',
            message: '评价成功！'
          })
          resolve({
            commentSuccess: true
          })
        } else {
          this.$message({
            type: 'warning',
            message: '评价失败，请稍后重试'
          })
          resolve({
            commentSuccess: false
          })
        }
      }).catch(err => {
        this.$message({
          type: 'error',
          message: '系统异常'
        })
        reject()
      })
    })
  }
}
```