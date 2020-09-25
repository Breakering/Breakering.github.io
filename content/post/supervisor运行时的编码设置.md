---
title: "Supervisor运行时的编码设置"
date: 2020-03-16T13:49:44+08:00
description: "Supervisor运行时的编码设置"
draft: false
tags: ["Supervisor"]
categories: ["Supervisor"]
series: []
toc: false
url: /2020/03/16/supervisor-encode/
---

用supervisor运行celery时有时会出现如下错误

```
UnicodeEncodeError: 'ascii' codec can't encode characters in position 65-66: ordinal not in range(128)
```

这是由于supervisor在运行时没有使用`utf-8`编码格式

可以在supervisor配置文件`supervisord.conf`里面增加如下配置

```
[supervisord]
environment=LC_ALL='en_US.UTF-8',LANG='en_US.UTF-8'
```

