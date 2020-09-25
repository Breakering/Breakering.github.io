---
title: GitLab升级
date: 2018-11-19 16:18:49
description: GitLab升级
draft: false
tags: ["GitLab"]
categories: ["GitLab"]
series: []
url: /2018/11/19/gitlab-update/
---
# 更新 GitLab
> 我们用的是 GitLab Omnibus 7.10.5 版本，查到[Doc](http://docs.gitlab.com/omnibus/update/README.html)（6.x.x 等低版本区别对待，详见文档）。
按照文档：
```
# To update to a newer GitLab version, all you have to do is:
# Debian/Ubuntu
sudo apt-get update
sudo apt-get install gitlab-ce
# Centos/RHEL
sudo yum install gitlab-ce
```

看起来太简单了！事实上，也就是这么简单。

但是，问题来了，`sudo apt-get install gilab-ce` 默认所用的源是 *packages-gitlab-com.s3.amazonaws.com*，然后你懂的，被墙了！

解决办法有两个：
1. 给 apt 加代理；
2. 换源。

## 1). 给 apt 加代理
考虑到换源可能产生其他的依赖问题，先尝试 加代理。结果是加了代理还是不行！原因可能是代理连接速度问题，总是超时。


这里参考的是 [打造Linux 终端翻墙环境](http://www.a-ho.com/2016/01/16/%E6%89%93%E9%80%A0Linux-%E7%BB%88%E7%AB%AF%E7%BF%BB%E5%A2%99%E7%8E%AF%E5%A2%83/)  使用 `shadowsocks + privoxy` 。

## 2). 换源解决！

Docs 里已经有声明其实：

![](/images/2018-11-19/3.png)

- 首先，添加信任 GitLab 里的 GPG 公钥：

```
curl https://packages.gitlab.com/gpg.key 2> /dev/null | sudo apt-key add - &>/dev/null
```

- 然后把 `/etc/apt/sources.list.d/gitlab_gitlab-ce.list` 文件中默认的源换成 *deb https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu trusty main*

![](/images/2018-11-19/4.png)

- 最后：

```
sudo update
sudo apt-get install gitlab-ce
```

>>> 安装完成！

# 对于更新版本跨度较大的情况

## 1). 关闭部分gitlab服务
升级之前，我们首先要关闭gitlab部分服务，如下：

```
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
gitlab-ctl stop nginx
```

## 2). 选择要升级的版本
[版本查看地址](https://packages.gitlab.com/gitlab/gitlab-ce?filter=debs)

然后执行命令：

```
apt-get install gitlab-ce=11.0.3-ce.0
```

其中`11.0.3`替换为你要升级的版本号。

**ps:版本跨度过大，请务必一个小版本一个小版本的更新**

另外，附上一次成功的更新过程对应的版本号：

`9.2.5-->9.5.6-->10.0.6-->10.8.5-->11.0.3 `

## 3). 重启gitlab
```
gitlab-ctl reconfigure
gitlab-ctl restart
```

# References:
1. http://www.a-ho.com/2016/01/16/%E6%89%93%E9%80%A0Linux-%E7%BB%88%E7%AB%AF%E7%BF%BB%E5%A2%99%E7%8E%AF%E5%A2%83/
2. https://about.gitlab.com/downloads/#ubuntu1404
3. https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/
4. http://docs.gitlab.com/omnibus/update/README.html
5. https://about.gitlab.com/upgrade-to-package-repository/
6. https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/trusty/gitlab-ce_8.9.5-ce.0_amd64.deb
7. https://www.ilanni.com/?p=13917
8. https://www.58jb.com/html/189.html