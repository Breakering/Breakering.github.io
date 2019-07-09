---
title: django-celery实现定时任务
date: 2018-11-16 17:11:53
draft: false
tags: ["Django"]
categories: ["Django"]
series: []
url: /2018/11/16/django-celery/
---
## 介绍
我们知道celery可以直接用在django项目中，但是配置稍微繁琐，还有添加定时任务需要重启celery beat进程，实在蛋疼，好在找到了`django-celery`这个模块，话不多说，让我们用起来吧。

## 安装和配置

安装还是很简单的，直接pip即可

```text
pip install django-clery
```

> 此时会将一些依赖库一并安装，比如celery等

接下来是django项目中的配置，在settings中配置如下:

```text
# INSTALLED_APPS中加入djcelery
INSTALLED_APPS = [
    ....
    'djcelery'
]

# 配置djcelery相关参数，ResultStore默认存储在数据库可不必重写 ，
djcelery.setup_loader()
BROKER_URL = 'redis://127.0.0.1:6379/8'  # 配置你的redis地址和库
# 使用和Django一样的时区
CELERY_TIMEZONE = TIME_ZONE

# 以上为基本配置，以下为周期性任务定义
CELERYBEAT_SCHEDULER = 'djcelery.schedulers.DatabaseScheduler'
```

同步数据库

```text
python manage.py migrate
```

## 创建task

在你的app下面创建一个`tasks.py`文件，文件名必须一致，`django-celery`默认情况下会自动从各个app中寻找该模块。

```python
from celery import task

@task()
def send_msg(msg):
    print(msg)
```

> 注意：task装饰器的`name`参数最好和函数名一致或者干脆不指定;最好不指定，这样下方分发任务时好统一处理。

## 创建定时任务

接下来我们就可以在Django admin中创建定时任务了

![](/images/2018-11-16/QQ20170613-215907.jpg)
![](/images/2018-11-16/QQ20170613-220348.jpg)

## 启动beat和worker

```text
python manage.py celery worker -l info
```

```text
python manage.py celery beat
```

之后就可以观察日志了，另外可以使用`supervisor`来管理这两个进程。

## 利用queue分发任务

在settings中增加如下配置:

```reStructuredText
# 定义任务对应的queue
class TasksRouter(object):
    @classmethod
    def route_for_task(cls, task, args=None, kwargs=None):
        task_routes = {
            'algorithm.product.tasks.*': {'queue': 'product'},
            'algorithm.material.tasks.*': {'queue': 'material'},
        }
        for route_key in task_routes:
            if re.search(route_key, task):
                return task_routes[route_key]


CELERY_ROUTES = (TasksRouter(), )
```

配置完成之后，启动beat和worker

```
python manage.py celery beat
```

beat会实时检测任务的变化，在django admin界面对任务进行操作，均会刷新该进程，使得分派任务变得非常简单。

```reStructuredText
python manage.py celery worker -Q product
```

上述命令启动的worker只会监测并执行`product`这个queue中的任务，即只执行`algorithm.product.tasks`下面的任务。同理`python manage.py celery worker -Q material`只执行`algorithm.material.tasks`下面的任务。

另外queue可以添加多个,例如`python manage.py celery worker -Q product,material`。

若要不区分queue执行所有任务，只需`python manage.py celery worker`即可,但不推荐,开启任务分发之后，最好分开执行，日志方面也更好排查。