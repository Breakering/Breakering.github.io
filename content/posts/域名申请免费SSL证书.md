---
title: "域名申请免费SSL证书"
date: 2019-08-01T11:20:19+08:00
description: 
draft: false
tags: ["ssl"]
categories: ["ssl"]
series: []
url: /2019/08/01/free-ssl/
---

### 1.下载certbot-auto

```
[root@Breakering ~]# wget https://dl.eff.org/certbot-auto
[root@Breakering ~]# chmod a+x ./certbot-auto
```

这里，我们无需关心`Certbot`是什么，只需要知道，证书的申请和生成是通过`Certbot`完成的，而`certbot-auto`脚本封装了`Certbot`。

### 2.申请证书

```
./certbot-auto certonly --standalone --email 1079614505@qq.com -d blog.breakering.com
```

如上，`certbot-auto`命令会自动下载`Certbot`所需的依赖，并且为`ainizhi.xin`域名申请并生成证书。请更换`--email`和 `-d`后的参数，分别表示自己邮箱和域名。

另外，证书90天后就会到期，到时我们只需要使用`certbot-auto renew`命令免费续签即可（建议配合使用linux的crontab机制），可参考官方文档。

### 3.证书位置

```
[root@Breakering ~]# ll /etc/letsencrypt/live/blog.breakering.com/
total 4
lrwxrwxrwx 1 root root  43 Aug  1 10:48 cert.pem -> ../../archive/blog.breakering.com/cert1.pem
lrwxrwxrwx 1 root root  44 Aug  1 10:48 chain.pem -> ../../archive/blog.breakering.com/chain1.pem
lrwxrwxrwx 1 root root  48 Aug  1 10:48 fullchain.pem -> ../../archive/blog.breakering.com/fullchain1.pem
lrwxrwxrwx 1 root root  46 Aug  1 10:48 privkey.pem -> ../../archive/blog.breakering.com/privkey1.pem
-rw-r--r-- 1 root root 692 Aug  1 10:48 README
```

这里，我们使用的是`fullchain.pem`和`privkey.pem`，前者是证书，后者是私钥。

### 4.启用证书

```
# 访问http自动切换至https
server {
       listen         80;
       server_name    blog.breakering.com;
       return         307 https://$server_name$request_uri;
}

server {
  listen 443 ssl;
  server_name blog.breakering.com;
  ssl_certificate /etc/letsencrypt/live/blog.breakering.com/fullchain.pem;  # 证书位置
  ssl_certificate_key /etc/letsencrypt/live/blog.breakering.com/privkey.pem;  # 私钥位置
  ssl_session_cache shared:SSL:1m;
  ssl_session_timeout  10m;
  ssl_ciphers HIGH:!aNULL:!MD5;
  ssl_prefer_server_ciphers on;
  location / {
   proxy_pass https://breakering.github.io;
  }
}
```

修改后重启nginx即可,确保443端口处于开放状态。

### 一些问题

-   出现下面的原因是`nginx`占用了80端口，先停止`nginx`,等证书安装完毕再重启`nginx`即可

```
Problem binding to port 80: Could not bind to IPv4 or IPv6.
```

### 参考

1.  <https://www.jianshu.com/p/90128aba680e>