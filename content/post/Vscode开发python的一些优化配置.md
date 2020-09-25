---
title: "Vscode开发python的一些优化配置"
date: 2019-10-16T11:55:27+08:00
draft: false
tags: ["vscode"]
categories: ["Tools"]
series: []
toc: false
url: /2019/10/16/vscode-python/
---

# Vscode开发python的一些优化配置

## 首先安装vscode

直接官网下载安装即可 <https://code.visualstudio.com/>

## 安装一些有用插件

### vscode-python

点击插件按钮，搜索`python`第一个就是，安装即可

![image.png](http://images.breakering.com:9080/images/2019/10/16/image.png)

### importmagic

跟上个插件安装一样，搜索安装即可,可以设置快捷键`Alt+Enter`来快速导入模块

![presentation.gif](http://images.breakering.com:9080/images/2019/10/16/presentation.gif)

### aiXcoder

参考官网即可<https://www.aixcoder.com/#/DownloadManual>

## 一些有用的配置

在`settings.json`添加如下内容:

```
"aiXcoder.enableTelemetry": false,
"python.linting.flake8Enabled": true,
"python.formatting.provider": "yapf",
"python.linting.flake8Args": [
    "--max-line-length=248"
],
"python.linting.pylintEnabled": false
```

需要安装一些模块

```
pip install flake8
pip install yapf
```

其中`flake8`用来检查代码是否规范；`yapf`用来自动格式化代码，可以设置快捷键`Shift+Alt+F`来方便快速格式化代码。

## 一些美化插件

### Material Theme

直接插件库搜索即可，推荐主题`Ocean High Contrast`

### Material Theme Icons

直接插件库搜索即可，推荐 `Icons Ocean`

### Atom One Dark Theme

![image462b5b7be6d7268b.png](http://images.breakering.com:9080/images/2019/10/16/image462b5b7be6d7268b.png)

### vscode-icons

![image9aaeda3e2eddeff7.png](http://images.breakering.com:9080/images/2019/10/16/image9aaeda3e2eddeff7.png)

## 额外的辅助插件

### PostgreSQL

![imagec58f6d0da4112b1e.png](http://images.breakering.com:9080/images/2019/10/16/imagec58f6d0da4112b1e.png)

### Code Runner

选中代码块运行

### filesize

显示文件大小

### Git History

查看git历史记录

### GitLens

可以看到该行是谁提交的

### Path Autocomplete

自动补全文件路径

### Settings Sync

同步所有插件到github

## 推荐字体

### Monaco

下载地址<http://www.xiazaiziti.com/250657.html>,下载好直接安装

**设置vscode的字体**

![image8ffe1802b37c3bb5.png](http://images.breakering.com:9080/images/2019/10/16/image8ffe1802b37c3bb5.png)

**展示**

![image7b2826e0615eedcd.png](http://images.breakering.com:9080/images/2019/10/16/image7b2826e0615eedcd.png)