---
title: ubuntu18.04设置开机启动脚本
original: false
comments: true
date: 2019-02-28 13:20:45
subtitle: ubuntu-boot-script
description: ubuntu18.04设置开机启动脚本
tags: Ubuntu
categories: Ubuntu
photos:
---

## 一、建立rc-local.service文件

```
sudo vi /etc/systemd/system/rc-local.service
```

## 二、将下列内容复制进rc-local.service文件

```
[Unit]
Description=/etc/rc.local Compatibility
ConditionPathExists=/etc/rc.local
 
[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99
 
[Install]
WantedBy=multi-user.target
```

## 三、创建文件rc.local

```
sudo vi /etc/rc.local
```

## 四、将下列内容复制进rc.local文件

```
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
echo "看到这行字，说明添加自启动脚本成功。" > /usr/local/test.log
exit 0
```

## 五、给rc.local加上权限,启用服务

```
sudo chmod +x /etc/rc.local
sudo systemctl enable rc-local
```

## 六、启动服务并检查状态

```
sudo systemctl start rc-local.service
sudo systemctl status rc-local.service
```

## 七、重启并检查test.log文件

```
cat /usr/local/test.log
```



>   PS:ubuntu16.04直接编辑/etc/rc.local文件即可