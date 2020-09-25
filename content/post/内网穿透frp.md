---
title: 内网穿透frp
description: 简单地说，内网穿透依赖于 NAT 原理，根据 NAT 设备不同大致可分为以下 4 大类(前3种NAT类型可统称为cone类型)
date: 2018-09-28 20:58:27
draft: false
tags: ["frp"]
categories: ["网络编程"]
series: []
url: /2018/09/28/frp/
---
## 一、内网穿透原理
简单地说，内网穿透依赖于 NAT 原理，根据 NAT 设备不同大致可分为以下 4 大类(前3种NAT类型可统称为cone类型)：
* 全克隆(Full Cone)：NAT 把所有来自相同内部 IP 地址和端口的请求映射到相同的外部 IP 地址和端口上，任何一个外部主机均可通过该映射反向发送 IP 包到该内部主机
* 限制性克隆(Restricted Cone)：NAT 把所有来自相同内部 IP 地址和端口的请求映射到相同的外部 IP 地址和端口；但是，只有当内部主机先给 IP 地址为 X 的外部主机发送 IP 包时，该外部主机才能向该内部主机发送 IP 包
* 端口限制性克隆(Port Restricted Cone)：端口限制性克隆与限制性克隆类似，只是多了端口号的限制，即只有内部主机先向 IP 地址为 X，端口号为 P 的外部主机发送1个 IP 包,该外部主机才能够把源端口号为 P 的 IP 包发送给该内部主机
* 对称式NAT(Symmetric NAT)：这种类型的 NAT 与上述3种类型的不同，在于当同一内部主机使用相同的端口与不同地址的外部主机进行通信时， NAT 对该内部主机的映射会有所不同；对称式 NAT 不保证所有会话中的私有地址和公开 IP 之间绑定的一致性；相反，它为每个新的会话分配一个新的端口号；导致此种 NAT 根本没法穿透

内网穿透的作用就是利用以上规则，创建一条从外部服务器到内部设备的 “隧道”，具体的 NAT 原理等可参考 内网打洞、网络地址转换NAT原理。

## 二、环境准备
实际上根据以上 NAT 规则，基本上大部分家用设备和运营商上级路由等都在前三种规则之中，所以只需要借助成熟的内网穿透工具即可，以下为本次穿透环境

* 最新版本 frp
* 一台公网 VPS 服务器
* 内网一台服务器，最好 Linux 系统

## 三、服务端搭建
服务器作为公网访问唯一的固定地址，即作为 server 端；内网客户端作为 client 端，会主动向 server 端创建连接，此时再从 server 端反向发送数据即可实现内网穿透

### 3.1). 下载并解压frp
可以查看[releases](https://github.com/fatedier/frp/releases)获取最新的版本,选好版本之后使用以下命令:

```
wget https://github.com/fatedier/frp/releases/download/v0.21.0/frp_0.21.0_linux_amd64.tar.gz
tar -zxvf frp_0.21.0_linux_amd64.tar.gz
cd frp_0.21.0_linux_amd64
```

### 3.2). 编辑frps.ini
```
[common]                                                                                                                                                                                                
# frp 监听地址
bind_addr = 0.0.0.0
bind_port = 7000

# 如果需要代理 web(http) 服务，则开启此端口
vhost_http_port = 8080
vhost_https_port = 4443

# frp 控制面板
dashboard_port = 7500
dashboard_user = user
dashboard_pwd = pwd

# 默认日志输出位置(这里输出到标准输出)
log_file = /tmp/frps.log
# 日志级别，支持: debug, info, warn, error
log_level = info
log_max_days = 3

# 是否开启特权模式(特权模式下，客户端更改配置无需更新服务端)
privilege_mode = true
# 授权 token 建议随机生成
privilege_token = cc23*********************d072734
# 特权模式下允许分配的端口(避免端口滥用)
privilege_allow_ports = 4000-50000

# 后端连接池最大连接数量
max_pool_count = 100

# 口令超时时间
authentication_timeout = 900

# 子域名(特权模式下将 *.xxxx.com 解析到公网服务器)
subdomain_host = xxxx.com
```

**其他具体配置说明请参考[frp README](https://github.com/fatedier/frp/blob/master/README_zh.md) 文档**

### 3.3). 启动frp server
设置完成后执行 ./frps -c frps.ini 启动即可

**ps:当然也可以使用supervisor来管理**

## 四、客户端配置
客户端作为发起链接的主动方，只需要正确配置服务器地址，以及要映射客户端的哪些服务端口等即可

### 4.1). 下载并解压frp
```
wget https://github.com/fatedier/frp/releases/download/v0.21.0/frp_0.21.0_linux_amd64.tar.gz
tar -zxvf frp_0.21.0_linux_amd64.tar.gz
cd frp_0.21.0_linux_amd64
```

### 4.2). 编辑frpc.ini

```
[common]
server_addr = 127.0.0.1
server_port = 7000
# console or real logFile path like ./frpc.log
log_file = /tmp/frpc.log

# debug, info, warn, error
log_level = debug

log_max_days = 3

# 特权模式，要和服务器端的配置一致
privilege_token = cc23*********************d072734


[gitlab]
type = http
local_port = 80
subdomain = gitlab  # 这样只要访问http://gitlab.xxxx.com:8080即可访问到该客户端的gitlab服务
use_gzip = true


[gitlab_ssh]
type = tcp 
local_ip = 127.0.0.1
local_port = 22
remote_port = 8081
```
**其他具体配置说明请参考[frp README](https://github.com/fatedier/frp/blob/master/README_zh.md) 文档**

### 4.3). 启动frp client
设置完成后执行 `./frpc -c frpc.ini `启动即可

**ps:当然也可以使用supervisor来管理**

## 五、frp监测
服务端和客户端同时开启完成后，即可访问 http://127.0.0.1:7500 进入 frp 控制面板，如下
![](http://blog.breakering.com//images/1046366-20180927105622574-1652030646.png)
![](http://blog.breakering.com//images/1046366-20180927105631267-34167117.png)
此时通过 ssh root@127.0.0.1 -p 8081 即可ssh到gitlab，通过访问http://gitlab.xxxx.com:8080 即可访问gitlab服务。

## 六、GitLab通过frp代理
内网的gitlab服务想要实现流畅的访问效果还是需要一番配置的，比如想直接通过 http://gitlab.xxxx.com 访问，想直接通过 http://gitlab.xxxx.com/xxx/xxx.git 来clone

### gitlab服务配置

#### 取消gitlab自带的nginx服务

```
sudo vim /etc/gitlab/gitlab.rb
# 找到如下位置取消注释并设置为false
nginx['enable'] = false
```

#### 创建gitlab的nginx配置文件

```nginx
# gitlab socket 文件地址
upstream gitlab {
  server unix:/var/opt/gitlab/gitlab-workhorse/socket;
}

server {
    listen 80;
    server_tokens off;
    root /opt/gitlab/embedded/service/gitlab-rails/public;

    # Increase this if you want to upload large attachments
    # Or if you want to accept large git objects over http
    client_max_body_size 250m;

    # individual nginx logs for this gitlab vhost
    access_log  /var/log/gitlab/nginx/gitlab_access.log;
    error_log    /var/log/gitlab/nginx/gitlab_error.log;


    location / {
      # serve static files from defined root folder;.
      # @gitlab is a named location for the upstream fallback, see below
      try_files $uri $uri/index.html $uri.html @gitlab;
    }

    # if a file, which is not found in the root folder is requested,
    # then the proxy pass the request to the upsteam (gitlab unicorn)
    location @gitlab {
      # If you use https make sure you disable gzip compression 
      # to be safe against BREACH attack

      proxy_read_timeout 300; # Some requests take more than 30 seconds.
      proxy_connect_timeout 300; # Some requests take more than 30 seconds.
      proxy_redirect     off;

      proxy_set_header   X-Forwarded-Proto $scheme;
      proxy_set_header   Host              $http_host;
      proxy_set_header   X-Real-IP         $remote_addr;
      proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
      proxy_set_header   X-Frame-Options   SAMEORIGIN;

      proxy_pass http://gitlab;
    }
    
    # Enable gzip compression as per rails guide: http://guides.rubyonrails.org/asset_pipeline.html#gzip-compression
    # WARNING: If you are using relative urls do remove the block below
    # See config/application.rb under "Relative url support" for the list of
    # other files that need to be changed for relative url support
    location ~ ^/(assets)/  {
      root /opt/gitlab/embedded/service/gitlab-rails/public;
      # gzip_static on; # to serve pre-gzipped version
      expires max;
      add_header Cache-Control public;
    }

    error_page 502 /502.html;
}

```

#### 确保nginx能够访问`/var/opt/gitlab/gitlab-workhorse/socket`

```
# 查看nginx的启动用户
sudo vim /etc/nginx/nginx.conf

# 开头第一句就是
user www-data;
```

```
# 查看/var/opt/gitlab/gitlab-workhorse/socket的用户及用户组
sudo ls -l /var/opt/gitlab/gitlab-workhorse/

-rw-r----- 1 root git  70 Jul 10 13:14 config.toml
srwxrwxrwx 1 git  git   0 Jul 17 10:51 socket  # 这个就是我们要查看的
-rw-r--r-- 1 root root 40 Jul 10 13:14 VERSION
```

```
# 将nginx的启动用户加入/var/opt/gitlab/gitlab-workhorse/socket的用户组
sudo usermod -aG git www-data
```

**查看gitlab访问日志,可以使用如下命令**

```
sudo gitlab-ctl tail
```

**如果还是有权限不足的情况,直接可以更改`nginx`的启动用户为`gitlab-www`即可**

```
sudo vim /etc/nginx/nginx.conf

# 开头第一句改为
user gitlab-www gitlab-www;
```

**重启`nginx`**

```
sudo service nginx restart
```

### 公网服务器nginx如下设置

```nginx
server {
    listen  80;
    server_name  gitlab.xxxx.com;
    location / {
        proxy_pass http://gitlab.geekfinancer.com:8080;
    }
}
```

这样即可通过 http://gitlab.xxxx.com 正常访问内网的gitlab了

### 接下来就可以愉快的使用gitlab了

上面我们已经配置gitlab的22端口映射到服务器的8081端口了，所以可以这样克隆:

```
git clone ssh://git@127.0.0.1:8081/zhuqian/licaishi.git
# 或者
git clone ssh://git@gitlab.xxxx.com:8081/zhuqian/licaishi.git
```

对于pip install的话，可以这样：

```
pip install git+ssh://git@127.0.0.1:8081/zhuqian/algorithm.git
# 或者
pip install git+ssh://git@gitlab.xxxx.com:8081/zhuqian/algorithm.git
```

你以为就这样完了，还没有，我们想要直接能在gitlab项目首页直接能够显示git访问方法，效果如下:
![](http://blog.breakering.com/images/1046366-20180927105727421-1721871308.png)

要实现此效果，只需配置下`/etc/gitlab/gitlab.rb`即可：

```
...
external_url 'http://gitlab.xxxx.com'
...

gitlab_rails['gitlab_shell_ssh_port'] = 8081
...
```
配置完之后:

```
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
```
然后通过域名访问gitlab即可实现上述效果了,当然通过http去clone或者push都是没问题的。

## 七、由mtu引起的无法访问的问题
如果frp的admin界面一切正常，但是就是无法获取数据

![](http://blog.breakering.com/images/1046366-20180927105741148-1074788234.png)

那么极有可能是你本地的网络最大分片小于服务器的最大分片，导致数据无法发送出去,解决办法是减小服务器的mtu:

```
sudo ifconfig eth0 mtu 1000 up
```

其他修改mtu的方式请自行google。

## 八、References:

1. [利用 frp 进行内网穿透](https://mritd.me/2017/01/21/use-frp-for-internal-network-wear/)

