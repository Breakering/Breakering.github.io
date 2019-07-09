---
title: Linux安装supervisor
draft: false
date: 2019-01-27 15:07:49
description: Linux下安装supervisor
tags: ["supervisor"]
categories: ["supervisor"]
series: []
url: /2019/01/27/install-supervisor/
---

# 介绍

[Supervisor](https://github.com/Supervisor/supervisor) 是一个用 Python 写的进程管理工具，可以很方便的对进程进行启动、停止、重启等操作。

# 安装

## For Ubuntu

-   安装命令

```shell
$ sudo apt-get install supervisor
```

-   配置

安装成功后，会在`/etc/supervisor`目录下，生成`supervisord.conf`配置文件。

进程配置会读取`/etc/supervisor/conf.d`目录下的`*.conf`配置文件，我们在此目录下创建一个`product_celeryd.conf`进程配置文件：

```reStructuredText
[program:product_celeryd]
directory      = /webapps/product/
command        = /webapps/product/env/bin/celery -A licaishi worker -Q product
user           = root
stdout_logfile = /webapps/product/logs/product_celeryd.log
stderr_logfile = /webapps/product/logs/product_celeryd.log
autostart      = true
autorestart    = true
startsecs      = 10
stopwaitsecs   = 600
```

-   启动

接着就可以启动 Supervisord 了：

```shell
$ supervisord
```

`supervisorctl` 常用命令：

| 命令                               | 说明                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| supervisorctl stop program_name    | 停止某个进程                                                 |
| supervisorctl start program_name   | 启动某个进程                                                 |
| supervisorctl restart program_name | 重启某个进程                                                 |
| supervisorctl stop all             | 停止全部进程                                                 |
| supervisorctl reload               | 载入最新的配置文件，停止原有进程并按新的配置启动、管理所有进程 |
| supervisorctl update               | 根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启 |

## For Centos

-   **安装**

```shell
# yum install python-setuptools
# easy_install supervisor
```

-   **创建配置文件(supervisord.conf）**

使用root身份创建一个全局配置文件

```shell
# echo_supervisord_conf > /etc/supervisord.conf
# supervisord -c /etc/supervisord.conf
```

-   **修改配置文件(supervisord.conf）**

如果修改了 /etc/supervisord.conf ,需要执行` # supervisorctl reload `来重新加载配置文件，否则不会生效

```shell
# supervisord 是启动supervisor 
# supervisorctl 是控制supervisord
```

打开supervisord.conf 的 [include] 引入 files的配置.

```reStructuredText
[include]
files = /etc/supervisor/conf.d/*.conf
```

```reStructuredText
[include]
files = /etc/supervisor/conf.d/*.conf
```

需要创建`/etc/supervisor/conf.d/`

-   **重启**

```shell
# supervisorctl reload
```



