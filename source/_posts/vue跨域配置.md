---
title: vue跨域配置
subtitle: vue-cross-domain
original: true
date: 2018-11-30 15:16:11
description: vue跨域配置
tags: vue
categories: vue
photos:
---

## 开发环境

**如果你使用的是vue-cli3的话，则可按如下配置**

-   在你的项目根目录创建`vue.config.js`文件
-   在文件中写入如下配置信息:

```js
// 配置proxy
module.exports = {
    devServer: {
        proxy: {
            '/api': {
                target: 'https://xxxx.xxxxxxxxxx.com',
                ws: true,
                changeOrigin: true
            },
        }
    }
};
```

>   参考[devserver-proxy](https://cli.vuejs.org/zh/config/#devserver-proxy)

## 线上环境

**线上通过nginx代理,实现跨域**

```nginx
server {
   listen 80;
   server_name www.breakering.com;  # 你的域名
   location / {
        index index.html;
        root /home/jacob/study/licaishi_pc/dist;  # vue buil之后dist文件夹位置
        try_files $uri $uri/ /index.html =404;  # 可以让浏览器在子页面也能刷新，主要是vue-router的路径不是真实路径导致
   }

   # 用/api来访问其他网站的接口，实现跨域
   location /api {
        # 下面三个是跨域的一些设置
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Methods 'GET, POST, PUT, PATCH, DELETE, OPTIONS';
        # Access-Control-Allow-Headers需要注意，会屏蔽一些headers，部署时需要注意
        add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,X-CSRFTOKEN';
        proxy_pass https://xxxx.xxxxxxxxxx.com/api;  # 其他网站的接口
   }
}
```



