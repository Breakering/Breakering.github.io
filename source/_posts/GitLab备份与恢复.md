---
title: GitLab备份与恢复
subtitle: gitlab-backup
original: true
date: 2018-11-19 16:15:50
description: GitLab备份与恢复
tags: GitLab
categories: GitLab
photos:
---
# 一、 备份gitlab
gitlab的备份比较简单，我们直接使用gitlab本身提供的命令进行备份即可。

## 1.1 通过gitlab-rake命令备份gitlab
gitlab提供的备份命令为gitlab-rake，备份命令使用如下:

```shell
gitlab-rake gitlab:backup:create
```

该命令会备份gitlab仓库、数据库、用户、用户组、用户密钥、权限等信息。

备份完成后备份文件会出现在`/var/opt/gitlab/backups/`
![](/images/2018-11-19/1.png)

当然备份的位置可以更换,使用如下命令：

```shell
vim /etc/gitlab/gitlab.rb
```

![](/images/2018-11-19/2.png)

修改上图`backup_path`的值即可，之后使用`gitlab-ctl reconfigure`使得配置生效

**ps：备份文件的名称中1537261122_2018_09_18_9.2.5是此次备份的编号。该编号我们会在后续恢复gitlab数据使用到。**

## 1.2 定时备份gitlab
如果要使ｇitlab自动进行备份的话，我们可以通过crontab命令来实现自动备份。强烈建议使用系统crontab命令，而不是用户crontab。

以实现每天凌晨4点进行一次自动备份为例，系统的crontab配置如下:

```shell
vim /etc/crontab
```

`0 4 * * * root /opt/gitlab/bin/gitlab-rake gitlab:backup:create CRON=1`

然后重启crontab服务，如下：

```shell
systemctl restart crond
```

## 1.3 保留部分备份文件
随着时间的推移gitlab备份文件越来越多，服务器的磁盘空间也不够大。

此时我们就要删除部分旧的备份文件，gitlab也提供了删除旧的备份文件功能。该功能在gitlab的配置文件中，进行配置即可。

在此以保留7天之前的备份文件为例，如下：

```shell
vim /etc/gitlab/gitlab.rb
```

`gitlab_rails[‘backup_keep_time’] = 604800`

其中backup_keep_time是以秒为单位进行计算的，然后执行命令`gitlab-ctl reconfigure`即可。

# 二、gitlab仓库恢复
要验证gitlab备份的有效性，我们可以把该备份文件复制到已经安装好gitlab服务器的/var/opt/gitlab/backups/目录下。然后进行数据恢复，最后访问并查看其数据完整性即可。

通过gitlab备份文件可以恢复gitlab所有的信息，包括仓库、数据库、用户、用户组、用户密钥、权限等信息。

**ps：新服务器上的gitlab的版本号必须与创建备份时的gitlab版本号相同。**

gitlab数据恢复比较简单，具体步骤如下：

## 2.1 停止相关数据连接服务
在gitlab服务器上停止相关数据连接服务，命令如下：

```shell
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
```

## 2.2 恢复gitlab仓库
现在我们要从1537261122_2018_09_18_9.2.5这个备份编号中，恢复数据，命令如下：

```shell
gitlab-rake gitlab:backup:restore BACKUP=1537261122_2018_09_18_9.2.5
```

如果出现多个done的信息，说明整个gitlab数据就已经正常恢复完毕。

## 2.3 启动gitlab服务
恢复完毕以后，我们现在来启动gitlab，使用以下命令：

```shell
gitlab-ctl start
```

**强烈建议：重启该新服务器。**

# 三、References:
1. [gitlab的备份与恢复](https://www.ilanni.com/?p=13890)
