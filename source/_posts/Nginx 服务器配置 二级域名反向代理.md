---
title: Nginx 服务器配置 二级域名反向代理
date: 2016-08-06 21:14:52
tags: [nginx,ubuntu]
categories: linux
---
前段时间在搬瓦工买了个凤凰城的海外服务器用来翻墙用
翻墙主要是使用 shadowsocks 搬瓦工服务服务管理系统openvz上面自带会有安装shodowsocks的选项,后来又想着在服务器上装个docker 把我原先的博客转移过来结果发现班瓦工linux内核版本太低不能安装docker

后来就到digtalOcean 购买了一个服务器 配置了docker环境后
又碰到一个问题 我想在这个服务器上 配置git服务和ghost服务
但是域名指向又存在问题
于是在google后发现nginx 能很好的解决这个问题

现在将这个问题记录下来

Ubuntu安装 Nginx
```
$ sudo apt-get install nginx 
```

nginx 的启动和关闭
```
//启动
$ service nginx restart
//关闭
$ service nginx stop

# service nginx {start|stop|status|restart|reload|configtest|}
```

因为服务器是以docker容器为基础,所以我又部署了我的另外的git服务
#####现在问题来了
我想要将我的git服务绑定到二级域名git.xiaoyuyun.com 这时候nginx强大的地方就体现出来了

在/etc/nginx/sites-enabled 下新建文件 gogs
用vim编辑
```
server {
    listen 80;

    server_name git.xiaoyuyun.com; # Replace with your domain

    root /usr/share/nginx/html;
    index index.html index.htm;

    client_max_body_size 10G;

    location / {
     # 这里需要配置的是本地3000端口 
     # 表示是将本地3000端口反向代理到git.xiaoyuyun.com:80上面
     # 这样就实现了二级域名的端口转发 
     # 这样之后,还需要做的就是将gogs服务配置到本地的3000端口上面
        proxy_pass http://localhost:3000;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
    }
}

```

下面我将介绍docker环境下面gogs的详细配置和安装再将这两者联系起来