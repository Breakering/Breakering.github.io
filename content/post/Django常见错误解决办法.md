---
title: Django常见错误解决办法
date: 2018-11-19 14:10:51
description: Django常见错误解决办法
draft: false
tags: ["Django"]
categories: ["Django"]
series: []
url: /2018/11/19/django-errors/
---
1. ProgrammingError: relation "default_cache_table" does not exist

```text
......
django.db.utils.ProgrammingError: relation "default_cache_table" does not exist
LINE 1: SELECT cache_key, value, expires FROM "default_cache_table" WHERE ca...
```

类似上述这种错误，可以用下面这句命令解决:

```text
python manage.py createcachetable
```

