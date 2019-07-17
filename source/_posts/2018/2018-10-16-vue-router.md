---
layout: post
title: vue-router的简单使用
category: vue
tags: [vue] 
date: 2018-10-16
---

## vue-router学习使用

* 项目的简单结构

[项目地址](https://github.com/despairyoke/vue-examples/tree/master/hello-world)

![](https://despairyoke.github.io/assets/images/2018/vue/vue-router1-1.png)

* router下的index.js内容

``` js
import Vue from 'vue'
import Router from 'vue-router'
import Home from '@/pages/home'
import About from '@/pages/about'
import HelloWorld from '@/components/HelloWorld'
import Login from '@/components/Login'
import Child from '@/components/Child'
Vue.use(Router)

export default new Router({
    mode: 'history',
    routes: [
        {
            path: '/',
            name: 'Home',
            component: Home
        },
        {
            path: '/about',
            name: About,
            component: About
        },
        {
            path: '/hello',
            name: 'HelloWorld',
            component: HelloWorld,
            children: [
                {
                    path: '/hello/:name',
                    name: 'HelloWorld',
                    component: Child
                },
            ]
        },
       
        {
            path: '/login',
            name: 'Login',
            component: Login
        }
    ]
})
```

> `@/pages/home`和`@/components/HelloWorld`是对应文件夹下的组件
<h5>route下参数详情</h5>

* path表示组件访问的路径
* name路由的标志
* component组件的名称和顶部import对应

`提醒`：Vue.use(Router)不要忘记

##### 路由如何使用？
----
* `加载路由`

``` html
import Vue from 'vue'
import router from './router'
import App from './App.vue'

Vue.config.productionTip = false

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')

```

* `使用路由`

``` html
<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    
      <li><router-link to="/about">About</router-link></li>
      <li><router-link to="/">Home</router-link></li>
      <li><router-link to="/login">Login</router-link></li>
    <router-view></router-view>
  </div>
</template>
```

![](https://despairyoke.github.io/assets/images/2018/vue/vue-router1-2.png)

* `router-link`会映射成`<a>`标签，`to`对应的就是`route/index.js`配置`path`
* `router-view`会填充`route`中的组件,如当前访问地址为`8080/`,页面显示内容为`home`组件中的`i,m home page`

##### 路由参数传递
----
![](https://despairyoke.github.io/assets/images/2018/vue/vue-router1-3.png)

![](https://despairyoke.github.io/assets/images/2018/vue/vue-router1-4.png)


``` html
<template>
    <div>
       <div> <label>姓名</label><input type="text" v-model="username">
       </div>
       <div>
        <label>密码</label><input type="password" v-model="password">
       </div>
        <button @click="login">登录</button>
    </div>
</template>
<script>
export default {
  data() {
    return {
      username: '',
      password: ''
    }
  },
  methods: {
    login: function() {
      console.log(this.username, this.password);
      this.$router.push({path: '/hello',
      query: {username: this.username,
          password: this.password}
      }
      )
    }
  }
};
</script>
```

> `$router.push`把参数和地址放到路由中，再又路由进行跳转
> `参数的接收`：{{$route.query.username}},你的密码是：{{$route.query.password}}

##### 嵌套路由

``` js
        {
            path: '/hello',
            name: 'HelloWorld',
            component: HelloWorld,
            children: [
                {
                    path: '/hello/:name',
                    name: 'HelloWorld',
                    component: Child
                },
            ]
        },
       
```
当使用路由嵌套时，先要配置children属性，children属性同上

`父组件如何使用`
``` html
<div>
  <router-view></router-view>
</div>
```
通过url的变化填充组件内容