---
title: Hexo同步
date: 2018-11-15 15:27:52
description: 其他电脑同步博客的方法!
draft: false
tags: ["Hexo"]
categories: ["Hexo"]
series: []
url: /2018/11/15/hexo-sync/
---

## 环境搭建

### 安装Node.js

用来生成静态页面, 到[Node.js官网](https://nodejs.org/en/)，下载最新版本, 根据提示一路安装即可

### 安装Git

```
sudo apt-get install git
```

### 安装Hexo

当Node.js和Git都安装好后就可以正式安装Hexo了，终端执行如下命令：

```
sudo npm install -g hexo
```

### 克隆hexo分支

```
git clone -b hexo https://github.com/Breakering/breakering.github.io.git
```

### 进入breakering.github.io.git

创建博客

```
hexo n '博客名'
```

发表博客

```
hexo d -g
```

### 主题配置更新相关

需要先清空缓存

```text
hexo clean
```

然后进行部署操作

```text
hexo d -g
```

### 一些问题

1. 报错一: 若执行命令hexo deploy仍然报错：无法连接git或找不到git，则执行如下命令来安装hexo-deployer-git：

```
npm install hexo-deployer-git --save
```

2. 报错二: 若执行命令hexo d报以下错误:

```
ERROR Plugin load failed: hexo-server 
//或者类似的错误 
ERROR Plugin load failed: hexo-renderer-sass
```

则执行响应的命令:
 
```
sudo npm install hexo-server
//或者
sudo npm install hexo-renderer-sass
```
