---
title: django修改request对象
subtitle: modify-request
original: true
date: 2018-12-12 18:43:17
description: django修改request对象
tags: Django
categories: Django
photos:
---

**Remove immutability:**

```python
if not request.GET._mutable:
   request.GET._mutable = True

# now you can spoil it
request.GET['pwd'] = 'iloveyou'
```

