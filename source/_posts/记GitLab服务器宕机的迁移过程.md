---
title: 记GitLab服务器宕机的迁移过程
original: true
comments: true
date: 2019-06-17 11:23:25
subtitle: gitlab-migrations
description: GitLab服务器宕机的迁移
tags: GitLab
categories: GitLab
photos:
---

# 问题描述

公司gitlab服务器的硬件比较老旧，内存也相对较少，在一次增加内存条的时候，由于和主板不兼容的问题，导致系统引导出错，无法再进入系统并运行gitlab了，而公司所有项目都在该gitlab服务器上面，开发小组的工作受到了很大的影响，所以修复gitlab服务器变得刻不容缓。

# 解决问题

## 第一次：尝试修复系统

从u盘启动，并挂载之前gitlab服务器的系统盘，查找原因是系统引导出现问题，用网上找到的一些办法来尝试修复引导均失败了，多次尝试无果之后开始尝试其他方法，由于gitlab兼容性较强，所以想着可不可以在gitlab服务无法启动的情况下通过转移文件来迁移整个gitlab,在网上查了一些资料之后发现的确可行，话不多说，开始实践吧。

## 第二次：尝试转移gitlab

### 1. 查看gitlab的版本

默认安装在/opt/gitlab/, 找到version-manifest.txt文件，文件第一行记录gitlab的版本：

```
gitlab-ce 11.0.3
```

### 2. 在新服务器上安装该版本的gitlab

安装之后，执行

```
sudo gitlab-ctl reconfigure
```

将gitlab服务正常启动，然后将其服务关闭`sudo gitlab-ctl stop`，然后再进行下面的操作。

>   **注意：版本一定保持一致，否则迁移将会失败。**

### 3. 将旧服务器上的仓库拷贝到新服务器上

#### 3.1 将旧服务器上位于相对位置的**/var/opt/gitlab/git-data/**目录下面的repositories目录打包，拷贝到新服务器上（如果旧服务的硬盘直接挂载在新服务器上，则可以直接拷贝）。

```
# 本次演示，旧服务器挂了，需要通过U盘启动，因此旧服务器和新服务器直接传文件，只能通过网络。
# 当前位于旧服务器的/var/opt/gitlab/git-data/目录下
# gitlab使用默认配置时，仓库在repositories目录下面
tar -zcvf repositories.tar.gz repositories/

# 将包通过网络拷贝到新服务器的当前用户的家目录下面，保证有权限向新服务器写入数据
scp repositories.tar.gz 新服务器用户@新服务器的IP地址:~/
```

#### 3.2 将数据解压，并修改权限，移动到新服务器的/var/opt/gitlab/git-data/下面。

```
# 解压包
tar -zxvf repositories.tar.gz

# 修改属主属组
sudo chown  -R git.root repositories

# 移动仓库
sudo mv repositories /var/opt/gitlab/git-data/ 
```

### 4. 将旧服务器上的存储在postgresql（默认安装配置的gitlab的数据，存储在postgresql中）中的数据拷贝到新服务器上。

#### 4.1 将旧服务器上位于相对位置的/var/opt/gitlab/postgresql/目录下面的data目录打包，拷贝到新服务上。

```
# 打包
tar -zcvf data.tar.gz data/

# 发送到新服务器的用户的家目录中
scp data.tar.gz 新服务器的用户@新服务器的IP地址:~/
```

#### 4.2 解压，并修改属主属组，移动到新服务器的/var/opt/gitlab/postgresql/目录下面。

```
# 解压
tar -xvf data.tar.gz

# 修改属主属组
sudo chown -R gitlab-psql.root data

# 移动数据（移动数据较快，又可以保留文件的原始权限）
sudo mv data /var/opt/gitlab/postgresql/

# 或者使用带权限的拷贝
sudo cp -rp data /var/opt/gitlab/postgresql/
```

### 5.将旧服务器上的配置文件拷贝到新服务器上

```
cd /etc/gitlab
scp gitlab.rb 新服务器的用户@新服务器的IP地址:~/
```
新服务器上:
```
sudo mv /etc/gitlab/gitlab.rb /etc/gitlab/gitlab.rb.bck  # 备份下配置文件
sudo mv gitlab.rb /etc/gitlab  # 使用旧服务器的配置文件
```

### 6.将旧服务器上的上传文件拷贝到新服务器上。

#### 6.1 将旧服务器上位于相对位置的/var/opt/gitlab/gitlab-rails/目录下面的uploads目录打包，拷贝到新服务上。

```
# 打包
tar -zcvf uploads.tar.gz uploads/

# 发送到新服务器的用户的家目录中
scp uploads.tar.gz 新服务器的用户@新服务器的IP地址:~/
```

#### 6.2 解压，并修改属主属组，移动到新服务器的/var/opt/gitlab/gitlab-rails/目录下面。

```
# 解压
tar -xvf uploads.tar.gz

# 修改属主属组
sudo chown -R git.root uploads

# 移动数据（移动数据较快，又可以保留文件的原始权限）
sudo mv uploads /var/opt/gitlab/gitlab-rails/

# 或者使用带权限的拷贝
sudo cp -rp uploads /var/opt/gitlab/gitlab-rails/
```

### 7. 重新配置gitlab

```
sudo gitlab-ctl reconfigure  
```

### 8.启动服务

```
# 启动服务
sudo gitlab-ctl start

# 查看启动状态
sudo gitlab-ctl status

# 查看实时启动状态
sudo gitlab-ctl tail
```

# 参考

1.  [gitlab旧服务器停机状态下，迁移gitlab服务](<https://my.oschina.net/airship/blog/1574250>)

