---
layout: post
title:  "Docker 容器化服务笔记"
date:   2019-10-22 09:14:00 +0800
categories: default
tags: Linux
comments: 1
---

容器化服务笔记



# 安装和启用 docker

> yum install docker

> systemctl enable docker.service

> systemctl start docker.service



1、运行时时区设置问题：

FROM openjdk:8-jdk-alpine

RUN apk add --no-cache tzdata

ENV TZ Asia/Shanghai



RUN date -R



运行时字体加载失败：NPE at sun.awt.FontConfiguration.getVersion(FontConfiguration.java:1264)

RUN apk add --no-cache fontconfig





2、进入容器执行 bash 命令

# container id 不用输全

docker exec -t -i {container id} /bin/bash



查询日志

docker logs 9573ada



服务的方式 docker 运行 nginx

sudo docker run --name my-nginx -p 80:80 -v /Users/stevenkang-mac/docker-nginx/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx



# 打包镜像命令

mvn clean package docker:build -D maven.test.skip=true



# 打包并上传镜像

mvn clean package docker:build -DpushImage -D maven.test.skip=true



# docker-maven-plugin 无法推送时，清除 ~/.docker/config.json 中的 autos 配置信息后重拾

Failed to push registry



# MySQL 容器化运行

 > docker run --name some-mysql -v /my/own/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

 > docker run --rm --name mysql -v /var/lib/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pwd -p 3306:3306 -d mysql:5.7

 > docker run --rm --name mysql -v /var/lib/mysql:/var/lib/mysql -p 3306:3306 -d mysql:5.6

 > sudo docker run --rm --name mysql -v /Users/stevenkang-mac/mysql:/var/lib/mysql -p 3306:3306 -m 256m --memory-swap=512m -d mysql:5.6



# Spring Config Server 容器化运行

 > docker run —rm --name config-server -p 192.168.1.145:8888:8888 -v /etc/spring/config_server-application.yml:/application.yml -d springcloud/configserver



# Redis 容器化运行

 > docker run --name redis -d redis:4-alpine

 > docker run --name redis -p 6379:6379 -d --rm redis --requirepass "password"



# Nginx 容器化运行

 > docker run --name my-nginx -v /etc/nginx.conf:/etc/nginx/nginx.conf:ro -v /etc/nginx:/v-nginx -p 80:80 -p 443:443 --rm -d nginx



# 编译 kms 服务：Dockerfile

```

FROM alpine:latest



RUN echo 'https://mirrors.aliyun.com/alpine/latest-stable/main/' > /etc/apk/repositories \

    && echo 'https://mirrors.aliyun.com/alpine/latest-stable/community/' >> /etc/apk/repositories \

    && apk update \

    && apk upgrade \

    && apk add --no-cache build-base gcc abuild binutils cmake git \

    && cd / \

    && git clone https://github.com/Wind4/vlmcsd.git vlmgit \

    && cd vlmgit \

    && make \

    && chmod +x bin/vlmcsd \

    && mv bin/vlmcsd / \

    && cd / \

    && apk del build-base gcc abuild binutils cmake git \

    && rm -rf /vlmgit  \

    && rm -rf /var/cache/apk/*



EXPOSE 1688



CMD ["/vlmcsd", "-D", "-d", "-t", "3", "-e", "-v"]

```



# 阿里云日志服务

```

> docker pull registry.cn-hangzhou.aliyuncs.com/log-service/logtail

> docker run -d -v /:/logtail_host:ro -v /var/run/docker.sock:/var/run/docker.sock --env ALIYUN_LOGTAIL_CONFIG=/etc/ilogtail/conf/cn-hangzhou/ilogtail_config.json --env ALIYUN_LOGTAIL_USER_ID=1553473*******38 --env ALIYUN_LOGTAIL_USER_DEFINED_ID=userdefined registry.cn-hangzhou.aliyuncs.com/log-service/logtail 

```



# 参考文献

 > [Docker 官方使用指南](https://docs.docker.com/get-started/)

 > [Github: docker-nginx issues #92](https://github.com/nginxinc/docker-nginx/issues/92#issuecomment-224974379)

 > [Docker 官方镜像之 Nginx](https://hub.docker.com/_/nginx)

 > [Docker 官方镜像之 MySQL](https://hub.docker.com/_/mysql)

 > [A Maven plugin for building and pushing Docker images.](https://github.com/spotify/docker-maven-plugin)