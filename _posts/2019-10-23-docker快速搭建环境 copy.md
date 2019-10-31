---
layout: mypost
title: docker快速搭建环境
categories: [Linux]
---

## docker安装
```
https://docs.docker.com/install/linux/docker-ce/centos/
```

## 创建网络
```
docker network create --help
```
创建一个其他的网段(docker默认使用172.17.0.1网段)，使用如下命令。
```
docker network create --driver bridge --subnet 172.18.0.0/24 --gateway=172.18.0.1 prod
docker network create --driver bridge prod
```
docker 列出每个容器的IP
```
docker inspect 容器ID | grep IPAddress
```
## 下载镜像
配置使用国内镜像：修改Docker配置文件/etc/default/docker如下

```
DOCKER_OPTS="--registry-mirror=https://..."
```
```
docker pull redis
docker pull ...
```

## 启动

进入容器
```
docker exec -it 9df70f9a0714 /bin/bash
```


redis
如果需要映射出IP加上: -p 6379:6379 \
```

docker run -it --network prod --network-alias redis -v /home/docker/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf -v /home/docker/redis/data:/var/lib/redis \
-v /home/docker/redis/log:/var/log/redis \
--restart=always \
--name redis redis redis-server /usr/local/etc/redis/redis.conf --appendonly yes
```

mysql
```
先运行这个镜像copy出配置
docker run --name mysql -e MYSQL_ROOT_PASSWORD=Mhmt123! -d mysql
等初始化完成以后再复制
移动出mysql data
docker cp mysql:/var/lib/mysql/ /home/docker/mysql/data
copy出 配置文件
docker cp mysql:/etc/mysql/my.cnf /home/docker/mysql/conf/my.cnf
停止容器 删除容器
docker rm -f mysql

docker run --name mysql -e MYSQL_ROOT_PASSWORD=Mhmt123! --privileged=true --network prod --network-alias mysql --restart=always -v /home/docker/mysql/conf/my.cnf:/etc/mysql/my.cnf \
-v /home/docker/mysql/data:/var/lib/mysql \
-d mysql
```


redis
```
先运行这个镜像copy出配置
docker run --name nginx -d nginx
等初始化完成以后再复制
docker cp nginx:/etc/nginx/ /home/docker/nginx/
mv nginx/ conf
docker cp nginx:/usr/share/nginx/html /home/docker/nginx
docker rm -f nginx
运行

docker run --name nginx -d -p 80:80 -p 443:443 --network prod --network-alias nginx --restart=always -v /home/docker/nginx/html:/usr/share/nginx/html:ro -v /home/docker/nginx/conf:/etc/nginx -v /home/docker/nginx/logs:/var/log/nginx nginx

```
