---
title: vue使用axios
subtitle: vue-use-axios
original: false
date: 2018-11-27 10:23:02
description: vue使用axios发起http请求
tags: vue
categories: vue
photos:
---

## 安装

```reStructuredText
npm install --save axios vue-axios
```

## 引入

```vue
import Vue from 'vue'
import axios from 'axios'
import VueAxios from 'vue-axios'

axios.defaults.baseURL='http://localhost:8000';  // 可以设置baseURL
Vue.use(VueAxios, axios)
```

## 使用

```vue
getNewsList(){
      this.axios.get('api/getNewsList').then((response)=>{
        this.newsList=response.data.data;
      }).catch((response)=>{
        console.log(response);
      })
}
```

## 参考

[vue全局使用axios的方法](https://segmentfault.com/a/1190000013128858)

[vue-axios](https://www.npmjs.com/package/vue-axios)

[vue添加axios，并且指定baseurl](https://blog.csdn.net/wild46cat/article/details/78006280)