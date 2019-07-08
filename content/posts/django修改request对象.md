---
title: django修改request对象
date: 2018-12-12 18:43:17
description: django修改request对象
draft: true
tags: ["Django"]
categories: ["Django"]
series: []
url: /2018/12/12/modify-request/
---

**Remove immutability:**

```python
if not request.GET._mutable:
   request.GET._mutable = True

# now you can spoil it
request.GET['pwd'] = 'iloveyou'
```

