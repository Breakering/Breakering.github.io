---
title: "Docker配置Chevereto图床"
date: 2019-09-18T13:29:24+08:00
description: 
draft: false
tags: ["chevereto", "图床"]
categories: ["图床"]
series: []
url: /2019/09/18/setup-chevereto/
---

### 环境配置

安装 Dockers 和 Dockers-Compose，新建`docker-compose.yml`, 参考[Dockerhub](https://hub.docker.com/r/nmtan/chevereto/)

```
version: '3'

services:
  db:
    image: mariadb
    volumes:
      - ./database:/var/lib/mysql:rw
    restart: always
    networks:
      - private
    environment:
      MYSQL_ROOT_PASSWORD: chevereto_root
      MYSQL_DATABASE: chevereto
      MYSQL_USER: chevereto
      MYSQL_PASSWORD: chevereto

  chevereto:
    depends_on:
      - db
    image: nmtan/chevereto
    restart: always
    networks:
      - private
    environment:
      CHEVERETO_DB_HOST: db
      CHEVERETO_DB_USERNAME: chevereto
      CHEVERETO_DB_PASSWORD: chevereto
      CHEVERETO_DB_NAME: chevereto
      CHEVERETO_DB_PREFIX: chv_
    volumes:
      - ./images:/var/www/html/images:rw
      - ./content:/var/www/html/content
      - ./conf/php.ini:/usr/local/etc/php/conf.d/php.ini
    ports:
      - 63080:80

networks:
  private:
```

新建目录

```
mkdir database images content conf
```

创建 conf/php.ini 文件,内容如下:

```
PHP:
max_execution_time = 60;
memory_limit = 256M;
upload_max_filesize = 256M;
post_max_size =  256M;
```

修改owner:

```
sudo chown -R www-data:www-data database images content conf
```

### 启动服务

直接通过docker-compose来安装镜像，并启动服务

```
docker-compose up [-d] # -d 表示后台
```

浏览器端口在yml文件中可以修改。

### 迁移

只需要把上面四个目录同步就可以了。