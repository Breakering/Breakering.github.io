---
title: "使用源码安装go"
date: 2020-03-16T13:28:43+08:00
description: "使用源码安装go"
draft: false
tags: ["Go"]
categories: ["Go"]
series: []
url: /2020/03/16/install-go/
---

## 官网下载源码包

[官网地址](https://golang.google.cn/dl/)

获取最新的源码包

```
wget https://dl.google.com/go/go1.14.linux-amd64.tar.gz
```

![1584335385735.png](http://images.breakering.com:9080/images/2020/03/16/1584335385735.png)

## 解压到指定目录

```
sudo tar zxvf go1.14.linux-amd64.tar.gz -C /usr/local
```

## 配置环境变量

看你使用的是`bash`还是`zsh`

```
vim ~/.bashrc
```

添加

```
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$PATH::$GOROOT/bin:$GOPATH/bin
```

保存并使生效

```
source ~/.bashrc
```

## 检查Go的版本

![1584335857889.png](http://images.breakering.com:9080/images/2020/03/16/1584335857889.png)