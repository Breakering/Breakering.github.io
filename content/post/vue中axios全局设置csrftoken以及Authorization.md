---
title: vue中axios全局设置csrftoken以及Authorization
draft: false
date: 2018-11-29 18:48:17
description: vue中axios全局设置csrftoken以及Authorization
tags: ["Vue"]
categories: ["Vue"]
series: []
url: /2018/11/29/vue-axios-set/
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



## 考虑封装axios

### 封装进api

-   创建api文件夹,其中创建一个`api.js`
-   编辑plugins下的`axios.js`

```js
import Vue from 'vue'
import axios from 'axios'
import VueAxios from 'vue-axios'
import store from '../store'

Vue.use(VueAxios, axios);

// base url
// Vue.axios.defaults.baseURL = 'http://127.0.0.1:9001';

// 请求超时时间
Vue.axios.defaults.timeout = 10000;

// 请求拦截器
Vue.axios.interceptors.request.use(
    config => {
        // 设置登录验证token
        const token = store.state.userInfo.Authorization;
        if (token) {
            config.headers.Authorization = token;
        }
        // 设置csrftoken
        const csrftoken = Vue.cookies.get('csrftoken');
        if (csrftoken) {
            config.headers['X-CSRFTOKEN'] = csrftoken;
        }
        console.log(token, csrftoken);
        return config
    },
    error => {
        return Promise.reject(error)
    });


// 响应拦截器
Vue.axios.interceptors.response.use(
    // 请求成功
    res => Promise.resolve(res),
    // 请求失败
    error => {
        // 请求已发出，但是不在2xx的范围
        console.log(error);  // 这儿可以用UI插件做个弹窗提醒
        return Promise.reject(error);
    });
```

-   编辑`api.js`

```js
import Vue from 'vue'

// 登录接口
export const login = data => Vue.axios.post(
    '/api/xxxx/login/', data
);

// 获取新闻列表接口
export const getNews = params => Vue.axios.get(
    '/api/xxxx/newsflashmaterial/?ordering=-create_time', params
);
```

-   我的`main.js`

```js
import Vue from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import './plugins/element.js'
import './plugins/cookies.js'
import './plugins/axios.js'


.....
```

-   使用

```js
<script>
    import {login} from "../api/api"

    export default {
        name: "Login",
        data() {
            return {
                formLabelAlign: {
                    username: '',
                    password: ''
                }
            }
        },
        methods: {
            submitForm() {
                login(this.formLabelAlign).then((response) => {
                    // 登录成功之后的操作
                })
            },
        }
    }
</script>
```

## 完成