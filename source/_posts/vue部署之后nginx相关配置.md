---
title: vue部署之后nginx相关配置
original: true
date: 2018-11-30 15:16:11
description: vue部署之后nginx相关配置
tags: vue
categories: vue
photos:
---

```nginx
server {
   listen 80;
   server_name www.breakering.com;  # 你的域名
   location / {
        index index.html;
        root /home/jacob/study/licaishi_pc/dist;  # vue buil之后dist文件夹位置
        try_files $uri $uri/ /index.html;  # 可以让浏览器在子页面也能刷新，主要是vue-router的路径不是真实路径导致
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

