---
title: Markdown奇淫技巧
original: false
comments: true
date: 2019-03-01 17:26:11
subtitle: markdown-tips
description: Markdown奇淫技巧
tags: Markdown
categories: Markdown
photos:
---

Markdown的基础语法这里就不介绍了，比较简单，下面介绍的都是些能玩出花样的小技巧。

>   ps：如下技巧大多是利用 Markdown 兼容部分 HTML 标签的特性来完成，不一定在所有网站和软件里都完全支持，主要以 GitHub 支持为准。

## 目录自动生成

示例代码：

```
[TOC]
```

示例效果:

[TOC]

## 在表格单元格里换行

借助于 HTML 里的 `<br />` 实现。

示例代码：

```
| Header1 | Header2                          |
|---------|----------------------------------|
| item 1  | 1. one<br/>2. two<br/>3. three |
```

示例效果：

| Header1 | Header2                        |
| ------- | ------------------------------ |
| item 1  | 1. one<br/>2. two<br/>3. three |

## 控制图片大小和位置

标准的 Markdown 图片标记 `![]()` 无法指定图片的大小和位置，只能依赖默认的图片大小，默认居左。

而有时候源图太大想要缩小一点，或者想将图片居中，就仍需要借助 HTML 的标签来实现了。图片居中可以使用 `<div>` 标签加 `align` 属性来控制，图片宽高则用 `width` 和 `height` 来控制。

示例代码：

```
**图片默认显示效果：**

![](http://blog.breakering.com/images/avatar/Breakering.jpg)

**加以控制后的效果：**

<div align="left"><img width="65" height="75" src="http://blog.breakering.com/images/avatar/Breakering.jpg"/></div>
```

示例效果：

**图片默认显示效果：**

![](http://blog.breakering.com/images/avatar/Breakering.jpg)

**加以控制后的效果：**

<div align="left"><img width="90" height="75" src="http://blog.breakering.com/images/avatar/Breakering.jpg"/></div>

## 使用 Emoji

这个是 GitHub 对标准 Markdown 标记之外的扩展了，用得好能让文字生动一些。

示例代码：

```
我和我的小伙伴们都笑了。:smile:
```

示例效果：

我和我的小伙伴们都笑了。:smile:

更多可用 Emoji 代码参见 <https://www.webpagefx.com/tools/emoji-cheat-sheet/>。

## 行首缩进

直接在 Markdown 里用空格和 Tab 键缩进在渲染后会被忽略掉，需要借助 HTML 转义字符在行首添加空格来实现，`&ensp;` 代表半角空格，`&emsp;` 代表全角空格。

示例代码：

```
&emsp;&emsp;春天来了，又到了万物复苏的季节。
```

示例效果：

&emsp;&emsp;春天来了，又到了万物复苏的季节。

## 展示数学公式

如果是在 GitHub 项目的 README 等地方，目前我能找到的方案只能是贴图了，以下是一种比较方便的贴图方案：

1.  在 <https://www.codecogs.com/latex/eqneditor.php> 网页上部的输入框里输入 LaTeX 公式，比如 `$$x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}$$`；

2.  在网页下部拷贝 URL Encoded 的内容，比如以上公式生成的是 `https://latex.codecogs.com/png.latex?%24%24x%3D%5Cfrac%7B-b%5Cpm%5Csqrt%7Bb%5E2-4ac%7D%7D%7B2a%7D%24%24`；

3.  在文档需要的地方使用以上 URL 贴图，比如

    ```
    ![](https://latex.codecogs.com/png.latex?%24%24x%3D%5Cfrac%7B-b%5Cpm%5Csqrt%7Bb%5E2-4ac%7D%7D%7B2a%7D%24%24)
    ```

    示例效果：

    ![img](https://latex.codecogs.com/png.latex?%24%24x%3D%5Cfrac%7B-b%5Cpm%5Csqrt%7Bb%5E2-4ac%7D%7D%7B2a%7D%24%24)

## 任务列表

在 GitHub 和 GitLab 等网站，除了可以使用有序列表和无序列表外，还可以使用任务列表，很适合要列出一些清单的场景。

示例代码：

```
**购物清单**

- [ ] 一次性水杯
- [x] 西瓜
- [ ] 豆浆
- [x] 可口可乐
- [ ] 小茗同学
```

示例效果：

**购物清单**

- [ ] 一次性水杯

- [x] 西瓜

- [ ] 豆浆

- [x] 可口可乐

- [ ] 小茗同学

## 待续.....