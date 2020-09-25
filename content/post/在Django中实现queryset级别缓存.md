---
title: 在Django中实现queryset级别缓存
date: 2018-11-16 11:50:19
description: django中实现queryset级别缓存
draft: false
tags: ["Django"]
categories: ["Django"]
series: []
url: /2018/11/16/django-queryset-cache/
---
## 介绍

实现queryset级别的缓存，不是view层面的，相当于缓存sql查询结果。

## 使用

### 首先在你的django项目中安装依赖的模块

```text
pip install django-cache-machine
```

### 创建queryset_cache.py文件,文件内容如下

```python
#! /usr/bin/env python
# -*- coding: utf-8 -*-
# __author__ = "Breakering"
# Date: 18-8-29
"""
依赖django-cache-machine，并在此基础上实现了轻松切换使用queryset级别缓存以及count等缓存
"""
import contextlib

from caching import config
from caching.base import CachingQuerySet, cached_with
from django.db.models.sql import query


def queryset_cache_decorator(always_cached=True):
    """queryset级别缓存的装饰器，可以使得queryset直接从缓存中获取数据"""
    def wrapper(func):
        def inner(self, *args, **kwargs):
            queryset = func(self, *args, **kwargs)
            if always_cached:  # 此装饰器默认从cache中获取数据
                queryset = queryset.from_cache()
            else:
                with contextlib.suppress(Exception):
                    queryset_cache_time = self.request.query_params.get('queryset_cache_time', '')
                    if queryset_cache_time and queryset_cache_time.isdigit():
                        queryset = queryset.from_cache(int(queryset_cache_time))
            return queryset
        return inner
    return wrapper


def queryset_cache_count_decorator(always_cached=True):
    """queryset count缓存的装饰器，可以使得queryset直接从缓存中获取count的值"""
    def wrapper(func):
        def inner(self, *args, **kwargs):
            queryset = func(self, *args, **kwargs)
            if always_cached:  # 此装饰器默认从cache中获取数据
                queryset = queryset.cache_count()
            else:
                with contextlib.suppress(Exception):
                    queryset_cache_time = self.request.query_params.get('queryset_cache_time', '')
                    if queryset_cache_time and queryset_cache_time.isdigit():
                        queryset = queryset.cache_count(int(queryset_cache_time))
            return queryset
        return inner
    return wrapper


class CachedQuerySet(CachingQuerySet):
    """
    Return queryset from cache if query_key in cache
    """
    def __init__(self, *args, **kwargs):
        super(CachedQuerySet, self).__init__(*args, **kwargs)
        self.timeout = config.NO_CACHE  # 默认直接从数据库取数据
        self.cache_count_timeout = config.NO_CACHE  # 自定义queryset count的缓存时间

    def _clone(self, *args, **kw):
        qs = super(CachedQuerySet, self)._clone(*args, **kw)
        qs.cache_count_timeout = self.cache_count_timeout
        return qs

    def from_cache(self, timeout=60*60):
        """在queryset中调用此函数则是从缓存中获取,且调用之后返回的仍是queryset"""
        self.timeout = timeout
        return self._clone()

    def cache_count(self, cache_count_timeout=60*60):
        """实现queryset count的缓存,且调用之后返回的仍是queryset"""
        self.cache_count_timeout = cache_count_timeout
        return self._clone()

    # todo values目前的实现方式有BUG，现已取消
    # def values(self, *fields, **expressions):
    #     """rewrite queryset's values"""
    #     if self.timeout == config.NO_CACHE:  # 默认情况下values直接从数据库获取数据
    #         return super(CachedQuerySet, self).values(*fields, **expressions)
    #
    #     clone = self._clone()
    #     clone.query.set_values(fields)
    #     key = make_key('values:{key}'.format(key=clone.query_key()))
    #     val = cache.get(key)
    #     if val is None:
    #         val = super(CachedQuerySet, self).values(*fields, **expressions)
    #         cache.set(key, val, self.timeout)
    #     return val

    def count(self):
        """自定义queryset的count"""
        super_count = super(CachingQuerySet, self).count
        try:
            query_string = 'count:%s' % self.query_key()
        except query.EmptyResultSet:
            return 0
        if self.cache_count_timeout:
            return cached_with(self, super_count, query_string, self.cache_count_timeout)
        elif self.timeout == config.NO_CACHE or config.TIMEOUT == config.NO_CACHE:
            return super_count()
        else:
            return cached_with(self, super_count, query_string, config.TIMEOUT)

```

### 改造您的model

```python
from django.db import models
from queryset_cache import CachedQuerySet
from caching.base import CachingMixin

class ModelClassManger(models.Manager):

    def get_queryset(self):
        return CachedQuerySet(self.model)
    
class ModelClass(CachingMixin, models.Model):
    objects = ModelClassManger()
```

### view层只需在get_queryset上加上装饰器即可

```python
@queryset_cache_count_decorator()
@queryset_cache_decorator()
def get_queryset(self):
    pass
```

如果添加了always_cached=False

```python
@queryset_cache_count_decorator(always_cached=False)
@queryset_cache_decorator(always_cached=False)
def get_queryset(self):
    pass
```

则需要在query参数中加上queryset_cache_time=180,参数后面的数字即为缓存的时间。
