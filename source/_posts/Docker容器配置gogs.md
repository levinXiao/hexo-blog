---
title: Docker容器配置gogs
date: 2016-08-06 21:14:52
tags: [docker,gogs,git]
categories: linux
---
#gogs介绍
https://gogs.io/

#docker方式部署gogs
https://github.com/gogits/gogs/tree/master/docker

```
# Pull image from Docker Hub.
$ docker pull gogs/gogs

# Create local directory for volume.
$ mkdir -p /var/gogs

# Use `docker run` for the first time.
$ docker run --name=gogs -p 10022:22 -p 10080:3000 -v /var/gogs:/data gogs/gogs

# Use `docker start` if you have stopped it.
$ docker start gogs
```

将gogs运行端口绑定到3000端口后就ok了
这样当nginx启动完成后反向代理就成功了
