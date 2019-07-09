---
title: 使用pipenv管理python项目
date: 2018-11-19 14:19:10
description: 使用pipenv管理python项目
draft: false
tags: ["Python"]
categories: ["Python"]
series: []
url: /2018/11/19/pipenv/
---
## 安装pipenv

```bash
pip install pipenv
```

## 项目初始化

```bash
cd your_project
PIPENV_VENV_IN_PROJECT=true pipenv --python=3.6
```

将在项目目录中创建新文件`Pipfile`和一个虚拟环境`.venv`, `--python=3.6`则是使用`python3.6`来创建虚拟环境,`PIPENV_VENV_IN_PROJECT=true`则是让虚拟环境创建在该项目目录下，方便管理。

 如果你添加--two或--three标志到上面的最后一个命令，它分别使用Python 2或3来初始化你的项目。 否则将使用默认版本的Python。

## 激活开发环境

```bash
pipenv shell
```

## 退出开发环境

```bash
exit
```

## 使用说明(`pipenv -h`)
```bash
Usage: pipenv [OPTIONS] COMMAND [ARGS]...

Options:
  --where             Output project home information.
  --venv              Output virtualenv information.
  --py                Output Python interpreter information.
  --envs              Output Environment Variable options.
  --rm                Remove the virtualenv.
  --bare              Minimal output.
  --completion        Output completion (to be eval'd).
  --man               Display manpage.
  --support           Output diagnostic information for use in GitHub issues.
  --site-packages     Enable site-packages for the virtualenv.
  --python TEXT       Specify which version of Python virtualenv should use.
  --three / --two     Use Python 3/2 when creating virtualenv.
  --clear             Clears caches (pipenv, pip, and pip-tools).
  -v, --verbose       Verbose mode.
  --pypi-mirror TEXT  Specify a PyPI mirror.
  --version           Show the version and exit.
  -h, --help          Show this message and exit.


Usage Examples:
   Create a new project using Python 3.7, specifically:
   $ pipenv --python 3.7

   Remove project virtualenv (inferred from current directory):
   $ pipenv --rm

   Install all dependencies for a project (including dev):
   $ pipenv install --dev

   Create a lockfile containing pre-releases:
   $ pipenv lock --pre

   Show a graph of your installed dependencies:
   $ pipenv graph

   Check your installed dependencies for security vulnerabilities:
   $ pipenv check

   Install a local setup.py into your virtual environment/Pipfile:
   $ pipenv install -e .

   Use a lower-level pip command:
   $ pipenv run pip freeze

Commands:
  check      Checks for security vulnerabilities and against PEP 508 markers
             provided in Pipfile.
  clean      Uninstalls all packages not specified in Pipfile.lock.
  graph      Displays currently-installed dependency graph information.
  install    Installs provided packages and adds them to Pipfile, or (if no
             packages are given), installs all packages from Pipfile.
  lock       Generates Pipfile.lock.
  open       View a given module in your editor.
  run        Spawns a command installed into the virtualenv.
  shell      Spawns a shell within the virtualenv.
  sync       Installs all packages specified in Pipfile.lock.
  uninstall  Un-installs a provided package and removes it from Pipfile.
  update     Runs lock, then sync.
```

## 总结

`pipenv`使得开发和管理项目包的过程变成的简单，让我们尽早使用起来吧。


## 一些错误

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