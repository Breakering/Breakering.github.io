---
title: vue中axios全局设置csrftoken以及Authorization
original: true
date: 2018-11-29 18:48:17
description: vue中axios全局设置csrftoken以及Authorization
tags: vue
categories: vue
photos:
---

## 说在前面

我们都知道，用django做后端服务时，对于post请求提交表单时总是需要csrftoken的验证，那么我们如何在vue中使用axios发起请求时全局在headers里面设置csrftoken呢？以及全局设置Authorization?

## 设置

其实非常简单，在main.js中设置下即可，示例代码如下:

```js
import Vue from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import './plugins/axios.js'
import './plugins/cookies.js'

Vue.config.productionTip = false;

new Vue({
    router,
    store,
    render: h => h(App),
	//进入页面时
    created() {
        // 拦截axios请求
        this.axios.interceptors.request.use(
            config => {
                // 设置登录验证token
                const token = this.$store.state.userInfo.Authorization;
                if (token) {
                    config.headers.Authorization = token;
                }

                // 设置csrftoken
                const csrftoken = this.$cookies.get('csrftoken');
                if (csrftoken) {
                    config.headers['X-CSRFTOKEN'] = csrftoken;
                }
                return config
            },
            error => {
                return Promise.reject(error)
            });
    }
}).$mount('#app');
```

另外贴下我的store.js:

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex);

export default new Vuex.Store({
    state: {
        userInfo: {
            username: "",  // 用户名
            Authorization: "",  // 用户登录的验证Token
        },
    },
    mutations: {
        // 更新用户信息
        updateUserInfo(state, userInfo) {
            state.userInfo.Authorization = userInfo.Authorization;
            state.userInfo.username = userInfo.username;
        }
    },
    actions: {
        updateUserInfo({commit}, userInfo) {
            commit('updateUserInfo', userInfo)
        }
    },
})
```



## 完成