---
title: Ubuntu安装Neo4j
date: 2019-07-09T15:42:32+08:00
draft: false
tags: ["Neo4j"]
categories: ["Neo4j"]
series: []
url: /2019/07/09/ubuntu-neo4j/
---

## 安装步骤

### 下载 Neo4j
官网下载地址：https://neo4j.com/download/ 

![image](/images/image.png)

### 解压
将 Neo4j 安装到一个目录 `/usr/local/neo4j`，直接压缩包复制到该目录下解压 
`tar -xzvf neo4j-community-3.5.3-unix.tar.gz `

### 启动
在 Neo4j 目录下启动，具体路径和命令为： 
`/usr/local/neo4j/neo4j-community-3.5.3/bin# ./neo4j start`

> 开机自启请自行google

如果提示如下信息 
ERROR: Unable to find Java executable. 

* Please use Oracle(R) Java(TM) 8, OpenJDK(TM) or IBM J9 to run Neo4j. 
* Please see https://neo4j.com/docs/ for Neo4j installation instructions. 

说明 java 版本不对，Neo4j 要求 java 版本为8，所以根据提示信息要使用 java8

### 安装 JAVA 8

#### 下载 Java
Java 8 下载地址：http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html 
![image](/images/image-1562565267975.png)

同意条款后下载红框中的版本

#### 解压
将 java 放在目录`/usr/lib/jvm`目录下，路径和命令为： 
`tar -xzvf jdk-8u131-linux-x64.tar.gz`

#### 配置环境变量
这里通过修改bashrc文件来配置环境变量 
`vim ~/.bashrc`

在文件末尾添加如下信息 
```
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_131 
export JRE_HOME=${JAVA_HOME}/jre 
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib 
export PATH=${JAVA_HOME}/bin:$PATH
```

具体内容可根据自己实际文件稍微改动

#### 生效配置文件
`source ~/.bashrc`

#### 检查 Java 版本
`java -version `

### 启动 Neo4j
`/usr/local/neo4j/neo4j-community-3.5.3/bin# ./neo4j start`

### 打开 Web 端
根据启动时的提示，在浏览器中打开：http://localhost:7474/ 
![image](/images/image-1562565297724.png)

这里输入默认用户名“neo4j”和密码“neo4j”。

![image](/images/image-1562565308224.png)

这里输入自己的新密码,本地开发请设置密码`111111`。

设置好后就会看到如下界面 
![image](/images/image-1562565320433.png)

可以点击页面上的“Start Learning”进行初步的学习 Neo4j。

### 安装[apoc](https://github.com/neo4j-contrib/neo4j-apoc-procedures)插件

apoc插件是一个neo4j的方法(function)和储存过程(Procedures)库，里面封装了很多有用的复杂图算法和数据操作的方法

1.  下载apoc的jar包 [github release](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases)
2.  复制jar文件到neo4j的plugins目录下
3.  修改neo4j的conf文件, 增加`dbms.security.procedures.unrestricted=apoc.*`
4.  重启neo4j服务 `neo4j restart`
5.  在neo4j浏览器中输入 RETURN apoc.version() 正常返回则安装成功

至此，Neo4j 已经在 Ubuntu 上安装好了。

### neo4j开启远程访问

在安装目录的 $NEO4J_HOME/conf/neo4j.conf 文件内，找到下面一行，将注释#号去掉就可以了
`dbms.connectors.default_listen_address=0.0.0.0`

![image](/images/image-1562565419439.png)

> 如果开启了防火墙等防护软件，一定要放开7474端口和7687端口，否则依然无法远程访问neo4j的web界面