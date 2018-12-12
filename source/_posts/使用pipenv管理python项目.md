---
title: 使用pipenv管理python项目
subtitle: pipenv
date: 2018-11-19 14:19:10
description: 使用pipenv管理python项目
tags: Python
categories: Python
photos:
original: false
---
## 安装pipenv

```text
pip install pipenv
```

## 进入你的project目录，并install

```text
cd your_project
pipenv install
```

`pipenv install`将在项目目录中创建两个新文件Pipfile和Pipfile.lock，如果项目不存在，则为项目创建一个新的虚拟环境。 如果你添加--two或--three标志到上面的最后一个命令，它分别使用Python 2或3来初始化你的项目。 否则将使用默认版本的Python。

ps:有可能会出现如下错误，应该是pyenv造成的

```text
...
  File "/usr/lib/python3.7/site-packages/pipenv/vendor/pythonfinder/models/python.py", line 70, in get_version_order
    version_order = [versions[v] for v in parse_pyenv_version_order()]
TypeError: 'NoneType' object is not iterable
```

这时可以使用下面的命令解决:

```text
pyenv global 3.7.1
```

> `3.7.1换成你用pyenv安装过的环境`

## 管理Python依赖关系

Pipfile包含关于项目的依赖包的信息，并取代通常在Python项目中使用的requirements.txt文件。 如果你在具有requirements.txt文件的项目中启动了Pipenv，则在把它从项目中删除之前，应该使用Pipenv安装该文件中列出的所有依赖包。

要为你的项目安装Python包，请使用install关键字。 例如，

`pipenv install requests`将安装当前版本的`requests`包。 可以使用uninstall关键字以类似的方式删除包;

`pipenv uninstall requests`将安装当前版本的`requests`包。 可以通过更新Pipfile.lock来冻结软件包名称及其版本以及其自己的依赖关系的列表。 这是使用lock关键字完成的;

`pipenv lock`如果另一个用户克隆存储库，可以添加Pipfiles到你的Git存储库，

这样他们只需要在他们的系统中安装Pipenv，然后键入`pipenv install`Pipenv会自动找到Pipfiles，创建一个新的虚拟环境，并安装必要的软件包。

## 管理你的开发环境

通常有一些Python包只在你的开发环境中需要，而不是在你的生产环境中，例如单元测试包。 Pipenv将使用--dev标志保持两个环境分开。 例如，

`pipenv install --dev ipython`将安装ipython，但也将其关联为只在你的开发环境中需要的软件包。 这很有用，因为现在，如果你要在你的生产环境中安装你的项目，

`pipenv install`默认情况下不会安装ipython包。 但是，如果另一个开发人员将你的项目克隆到自己的开发环境中，他们可以使用--dev标志，

`pipenv install –dev`安装所有依赖项，包括开发包。

## 激活开发环境

`pipenv shell`

## 总结

`pipenv`使得开发和管理项目包的过程变成的简单，让我们尽早使用起来吧。