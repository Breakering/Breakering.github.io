---
title: "使用gdrive同步GoogleDrive"
date: 2020-03-16T16:21:36+08:00
description: "使用gdrive同步GoogleDrive"
draft: false
tags: ["Google"]
categories: ["Google"]
series: []
url: /2020/03/16/gdrive/
---

gdrive自带的证书认证已经失效，本文是使用自建的own Google credential来进行认证

## 安装go

[使用源码安装go](<https://blog.breakering.com/2020/03/16/install-go/>)

## 创建own Google credential

-   打开链接 <https://console.developers.google.com/apis/dashboard>登录你的google账号

如果你的账号是第一次打开此链接，你需要同意google的一些条款

![image.png](http://images.breakering.com:9080/images/2020/03/16/image.png)

-   创建一个新的项目

![image6cc8238b67493bf5.png](http://images.breakering.com:9080/images/2020/03/16/image6cc8238b67493bf5.png)

-   输入项目名并点击创建

![imagebc78c83627a3b14b.png](http://images.breakering.com:9080/images/2020/03/16/imagebc78c83627a3b14b.png)

-   启用API和服务

![image416dad121fb24b89.png](http://images.breakering.com:9080/images/2020/03/16/image416dad121fb24b89.png)

-   选择*Google Drive API*

![image454fcbd11275bc38.png](http://images.breakering.com:9080/images/2020/03/16/image454fcbd11275bc38.png)

-   点击启用

![image147d34a3ca59d6ff.png](http://images.breakering.com:9080/images/2020/03/16/image147d34a3ca59d6ff.png)

-   Configure consent screen

![imagec744f5a49adec90e.png](http://images.breakering.com:9080/images/2020/03/16/imagec744f5a49adec90e.png)

-   输入项目名并保存

![imaged420c0a117a2e4c8.png](http://images.breakering.com:9080/images/2020/03/16/imaged420c0a117a2e4c8.png)

-   创建OAuth客户端ID

![imagec5ad3743f0a66729.png](http://images.breakering.com:9080/images/2020/03/16/imagec5ad3743f0a66729.png)

-   选择桌面应用然后加名称创建即可

![imagec090a9e3ce51e388.png](http://images.breakering.com:9080/images/2020/03/16/imagec090a9e3ce51e388.png)

-   创建成功

![image03911f8f0c2da84b.png](http://images.breakering.com:9080/images/2020/03/16/image03911f8f0c2da84b.png)

## 下载gdrive项目并设置证书

从<https://github.com/gdrive-org/gdrive>下载gdrive项目,打开根目录的`handlers_drive.go`文件，将上文创建的客户端ID以及客户端秘钥填入以下对应位置:

```
const ClientId = "367116221053-7n0v**.apps.googleusercontent.com"
const ClientSecret = "1qsNodXN*****jUjmvhoO"
```

## 构建gdrive项目

```
go get github.com/prasmussen/gdrive
```

```
go build
```

构建完成后，根目录会有个`gdrive`的文件

![image569304771c4387a9.png](http://images.breakering.com:9080/images/2020/03/16/image569304771c4387a9.png)

此文件即为可执行文件，对于ubuntu可以放入`/usr/bin/`目录下

```
sudo mv ./gdrive /usr/bin
sudo chmod 700 /usr/bin/gdrive
```

## gdrive认证

按上述配置完成之后，可以输入`gdrive about`命令来进行认证，打开相关链接，登录google账号，有不安全提示可以不用管，直接允许，然后获取verification code输入即可，成功会显示如下信息:

![imageb32456f380ab4518.png](http://images.breakering.com:9080/images/2020/03/16/imageb32456f380ab4518.png)

>   需要配合一些命令翻墙工具使用<https://github.com/hmgle/graftcp>

## gdrive使用方法

```
gdrive [global] list [options]                                 List files
gdrive [global] download [options] <fileId>                    Download file or directory
gdrive [global] download query [options] <query>               Download all files and directories matching query
gdrive [global] upload [options] <path>                        Upload file or directory
gdrive [global] upload - [options] <name>                      Upload file from stdin
gdrive [global] update [options] <fileId> <path>               Update file, this creates a new revision of the file
gdrive [global] info [options] <fileId>                        Show file info
gdrive [global] mkdir [options] <name>                         Create directory
gdrive [global] share [options] <fileId>                       Share file or directory
gdrive [global] share list <fileId>                            List files permissions
gdrive [global] share revoke <fileId> <permissionId>           Revoke permission
gdrive [global] delete [options] <fileId>                      Delete file or directory
gdrive [global] sync list [options]                            List all syncable directories on drive
gdrive [global] sync content [options] <fileId>                List content of syncable directory
gdrive [global] sync download [options] <fileId> <path>        Sync drive directory to local directory
gdrive [global] sync upload [options] <path> <fileId>          Sync local directory to drive
gdrive [global] changes [options]                              List file changes
gdrive [global] revision list [options] <fileId>               List file revisions
gdrive [global] revision download [options] <fileId> <revId>   Download revision
gdrive [global] revision delete <fileId> <revId>               Delete file revision
gdrive [global] import [options] <path>                        Upload and convert file to a google document, see 'about import' for available conversions
gdrive [global] export [options] <fileId>                      Export a google document
gdrive [global] about [options]                                Google drive metadata, quota usage
gdrive [global] about import                                   Show supported import formats
gdrive [global] about export                                   Show supported export formats
gdrive version                                                 Print application version
gdrive help                                                    Print help
gdrive help <command>                                          Print command help
gdrive help <command> <subcommand>                             Print subcommand help
```