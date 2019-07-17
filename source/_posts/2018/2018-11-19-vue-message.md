---
layout: post
title: vue父子组件参数传递

category: vue
tags: [vue] 
date: 2018-11-19
---

# message-father-child

### 父子组件相互传值

> 父传子：是通过props数组接收数据，子传父是通过emit提交

`下面这个例子是父组件传递message在子组件中显示，而子组件中通过输入框输入内容传递给父组件显示`

<b>效果图</b>
![](https://despairyoke.github.io/assets/images/2018/vue/vue-message.jpg)


* 父组件
``` html
<template>
　　<div>
　　　　<p> father</p>
   　　 <hello :mes="loginJson.animal"  @sendiptVal='showChildMsg'></hello>
        <p>{{this.message}}</p>
　　</div>

</template>

<script>
import hello from '@/components/hello'
    
export default {
    name: 'app',
  data() {
    return {

      message:'',
      loginJson:{
          "animal":"dog"
      },

    }
  },
  components:{
    hello
      
  },
  methods: {
      showChildMsg(e) {
          console.log(e)
        this.message = e
      }
  }
}
</script>

<style>

</style>
```

* 子组件

``` html
<template>
  <div class="hello">
    <!-- 添加一个input输入框 添加keypress事件-->
    <input type="text" v-model="inputValue" @keypress.enter="enter">
    <p>{{mes}}</p>
  </div>
</template>

<script>
export default {
  props:['mes'],

  // 添加data, 用户输入绑定到inputValue变量，从而获取用户输入
  data: function () {
    return {
      inputValue: ''  
    }
  },
  methods: {
    enter () {
      this.$emit("sendiptVal", this.inputValue) 
      //子组件发射自定义事件sendiptVal 并携带要传递给父组件的值，
      // 如果要传递给父组件很多值，这些值要作为参数依次列出 如 this.$emit('valueUp', this.inputValue, this.mesFather); 
    }
  }
}
</script>
```