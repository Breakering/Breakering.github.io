---
title: 内网穿透frp
description: 简单地说，内网穿透依赖于 NAT 原理，根据 NAT 设备不同大致可分为以下 4 大类(前3种NAT类型可统称为cone类型)
copyright: true
permalink: 1
top: 100
date: 2018-09-28 20:58:27
tags: 内网穿透
categories: 270 - Learning - Backend | 后端相关
---
# 一、内网穿透原理
简单地说，内网穿透依赖于 NAT 原理，根据 NAT 设备不同大致可分为以下 4 大类(前3种NAT类型可统称为cone类型)：
* 全克隆(Full Cone)：NAT 把所有来自相同内部 IP 地址和端口的请求映射到相同的外部 IP 地址和端口上，任何一个外部主机均可通过该映射反向发送 IP 包到该内部主机
* 限制性克隆(Restricted Cone)：NAT 把所有来自相同内部 IP 地址和端口的请求映射到相同的外部 IP 地址和端口；但是，只有当内部主机先给 IP 地址为 X 的外部主机发送 IP 包时，该外部主机才能向该内部主机发送 IP 包
* 端口限制性克隆(Port Restricted Cone)：端口限制性克隆与限制性克隆类似，只是多了端口号的限制，即只有内部主机先向 IP 地址为 X，端口号为 P 的外部主机发送1个 IP 包,该外部主机才能够把源端口号为 P 的 IP 包发送给该内部主机
* 对称式NAT(Symmetric NAT)：这种类型的 NAT 与上述3种类型的不同，在于当同一内部主机使用相同的端口与不同地址的外部主机进行通信时， NAT 对该内部主机的映射会有所不同；对称式 NAT 不保证所有会话中的私有地址和公开 IP 之间绑定的一致性；相反，它为每个新的会话分配一个新的端口号；导致此种 NAT 根本没法穿透

内网穿透的作用就是利用以上规则，创建一条从外部服务器到内部设备的 “隧道”，具体的 NAT 原理等可参考 内网打洞、网络地址转换NAT原理。

# 二、环境准备
实际上根据以上 NAT 规则，基本上大部分家用设备和运营商上级路由等都在前三种规则之中，所以只需要借助成熟的内网穿透工具即可，以下为本次穿透环境

* 最新版本 frp
* 一台公网 VPS 服务器
* 内网一台服务器，最好 Linux 系统

# 三、服务端搭建
服务器作为公网访问唯一的固定地址，即作为 server 端；内网客户端作为 client 端，会主动向 server 端创建连接，此时再从 server 端反向发送数据即可实现内网穿透

## 3.1). 下载并解压frp
可以查看[releases](https://github.com/fatedier/frp/releases)获取最新的版本,选好版本之后使用以下命令:

```
wget https://github.com/fatedier/frp/releases/download/v0.21.0/frp_0.21.0_linux_amd64.tar.gz
tar -zxvf frp_0.21.0_linux_amd64.tar.gz
cd frp_0.21.0_linux_amd64
```

## 3.2). 编辑frps.ini
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

## 3.3). 启动frp server
设置完成后执行 ./frps -c frps.ini 启动即可

**ps:当然也可以使用supervisor来管理**

# 四、客户端配置
客户端作为发起链接的主动方，只需要正确配置服务器地址，以及要映射客户端的哪些服务端口等即可

## 4.1). 下载并解压frp
```
wget https://github.com/fatedier/frp/releases/download/v0.21.0/frp_0.21.0_linux_amd64.tar.gz
tar -zxvf frp_0.21.0_linux_amd64.tar.gz
cd frp_0.21.0_linux_amd64
```

## 4.2). 编辑frpc.ini

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
local_port = 8080
subdomain = gitlab  # 这样只要访问http://gitlab.xxxx.com:8080即可访问到该客户端的gitlab服务
use_gzip = true

[gitlab_static_file]
type = tcp
remote_port = 8082
plugin = static_file
# 要对外暴露的文件目录
plugin_local_path = /opt/gitlab/embedded/service/gitlab-rails/public/assets/
# 访问 url 中会被去除的前缀，保留的内容即为要访问的文件路径
plugin_strip_prefix = assets
#plugin_http_user = abc
#plugin_http_passwd = abc

[gitlab_ssh]
type = tcp 
local_ip = 127.0.0.1
local_port = 22
remote_port = 8081
```
**其他具体配置说明请参考[frp README](https://github.com/fatedier/frp/blob/master/README_zh.md) 文档**

## 4.3). 启动frp client
设置完成后执行 ./frpc -c frpc.ini 启动即可

**ps:当然也可以使用supervisor来管理**

# 五、测试
服务端和客户端同时开启完成后，即可访问 http://127.0.0.1:7500 进入 frp 控制面板，如下
![](/images/1046366-20180927105622574-1652030646.png)
![](/images/1046366-20180927105631267-34167117.png)
此时通过 ssh root@127.0.0.1 -p 8081 即可ssh到gitlab，通过访问http://gitlab.xxxx.com:8080 即可访问gitlab服务

# 六、GitLab通过frp代理
通过上述配置，确实可以通过 http://gitlab.xxxx.com:8080 访问gitlab服务,但是你会发现缺少静态文件,因为gitlab的静态文件是nginx代理的，走的tcp协议,需要一种解决方案。

## 方案一、使用frp的static_file的插件
虽然可以成功，通过 http://127.0.0.1:8082 即可访问gitlab的静态文件，并且也可以通过nginx反向代理到gitlab.xxxx.com这个域名上，但是速度会很慢很慢,nginx配置如下:

```
server {                                                                                                                                                                                                    
    listen  80; 
    server_name  gitlab.xxxx.com;
    location / { 
        proxy_pass http://gitlab.xxxx.com:8080;
    }   
    location /assets {
        proxy_pass http://127.0.0.1:8082;
    }   
}
```

## 方案二、将gitlab静态文件移至服务器上，用nginx代理
gitlab静态文件在如下位置`/opt/gitlab/embedded/service/gitlab-rails/public/assets/`放至服务器，并配置nginx如下:
```
server {                                                                                                                                                                                                    
    listen  80;
    server_name  gitlab.xxxx.com;
    location / {
        proxy_pass http://gitlab.xxxx.com:8080;
    }
    location /assets {
        alias /webapps/gitlab/public/assets;
    }
}
```
这样即可通过 http://gitlab.xxxx.com 正常访问内网的gitlab了

但是这样还没结束，你会发现外网通过git clone http://gitlab.xxxx.com/zhuqian/licaishi.git ,根本没法正常克隆仓库，那有啥用啊，别急，咋们还可以用ssh方式啊。

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
![](/images/1046366-20180927105727421-1721871308.png)

要实现此效果，只需配置下`/etc/gitlab/gitlab.rb`即可：

```
...
external_url 'http://gitlab.xxxx.com'
...

gitlab_rails['gitlab_shell_ssh_port'] = 8081
...
```
>> 另外需要注意下`nginx['listen_addresses'] = ['192.168.10.60']`，需要对应到本地的ip地址

配置完之后:

```
gitlab-ctl reconfigure
```
然后通过域名访问gitlab即可实现上述效果了，只不过http方式目前还无法解决。

# 七、由mtu引起的无法访问的问题
如果frp的admin界面一切正常，但是就是无法获取数据

![](/images/1046366-20180927105741148-1074788234.png)

那么极有可能是你本地的网络最大分片小于服务器的最大分片，导致数据无法发送出去,解决办法是减小服务器的mtu:

```
sudo ifconfig eth0 mtu 1000 up
```

其他修改mtu的方式请自行google。

# 八、References:

1. [利用 frp 进行内网穿透](https://mritd.me/2017/01/21/use-frp-for-internal-network-wear/)
