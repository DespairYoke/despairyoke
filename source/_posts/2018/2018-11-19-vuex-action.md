---
layout: post
title: vuex进阶篇

category: vue
tags: [vue] 
date: 2018-11-19
---

[项目地址](https://github.com/DespairYoke/vue-examples/tree/master/vuex-action-example)

### state 对参数的初始化，供后期使用

``` js
import Vue from "vue";
import Vuex from "vuex"

import mutations from './mutations'
import actions from './action'
Vue.use(Vuex)

const state = {
    name: "张三",
    sex: '',
    age: '5'
}
export default new Vuex.Store({
    state,
    mutations,
    actions
})
```

### mutaion和actions用来改变state中的值，不同点后面代码演示

*<b>mutations.js</b>*
``` js
import {UPDATE_SEX,ADD_AGE} from './mutation-types'

export default {
    [UPDATE_SEX](state, sex) {
        state.sex = sex;
    },
    [ADD_AGE](state,age) {
        console.log(age,1111)
        state.age = age
    }
}
```
*<b>action.js</b>*
``` js
import {ADD_AGE} from './mutation-types'

export default {
    [ADD_AGE](state,age) {
        console.log(age,1111)
        state.age = age
    }
}
```
`这里mutation-types是来用简写方法名使用`
*<b> mutation=types.js</b>*
``` js
export const ADD_AGE = 'ADD_AGE'
export const UPDATE_SEX = 'UPDATE_SEX'
```

#### 演示效果
![](https://despairyoke.github.io/assets/images/2018/vue/vuex-1.png)

`张三，5是state初始化时的名字和年龄，男是组件创建时，调用mutations的update_sex进行初始化。此演示是想通过单选按钮事件调用mutation改变sex参数值；通过输入框来调用action来改变age。`

``` html
<template>
  <div>
    <p>{{name}}</p>
    <div>
      <p>{{sex}}</p>
      <p>{{age}}</p>
      <input name="sexes" type="radio" checked @click="changeSex"/>男
      <input name="sexes" type="radio" @click="changeSex"/>女
      <div>
      <input type="text" v-model="ages" placeholder="请输入年龄" @keyup.enter="test_Action_age"/>
      </div>
      <div>
      <input type="text" v-model="ages2" placeholder="请输入年龄" @keyup.enter="test_Mutation_age"/>
      </div>
    </div>
  </div>
</template>
<script>

import {mapState,mapMutations,mapActions} from 'vuex'
export default {
  data() {
    return {
      ages:'',
      ages2:'',
    }
  },
  computed: {
    ...mapState(['name','sex','age']),
    
  },
  created() {
    this.initData()
  },
  methods: {
    ...mapMutations(['UPDATE_SEX','ADD_AGE']),
    initData() {
      this.UPDATE_SEX("男")
    },
    ...mapActions({'UPDATE_AGE':'ADD_AGE'}),
    changeSex() {
      console.log(this.sex)
      if(this.sex==='男'){
      this.UPDATE_SEX("女")
      }else{
      this.UPDATE_SEX("男")
      }
    },
    test_Action_age() {
      this.UPDATE_AGE(this.ages)
    },
    test_Mutation_age() {
      this.ADD_AGE(this.ages2)
    }
  }
  
}
</script>
```

`二个输入框的目的是为了解释action和mutation不同点异步。当使用action中的改变事件时，age是不会发生改变，而使用mutation时，age是同步改变的。`