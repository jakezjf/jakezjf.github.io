---
layout:     post
title:      "Docker 常用命令(一)"
subtitle:   "Docker 的常用命令"
date:       2017-01-03
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Docker
---

> 本篇将介绍 Docker 的常用命令
    
# Docker

##### docker history

docker history name
    
    [root@jianfeng ~]# docker history nginx
    IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
    a39777a1a4a6        3 days ago          /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon    0 B
    <missing>           3 days ago          /bin/sh -c #(nop)  EXPOSE 443/tcp 80/tcp        0 B
    <missing>           3 days ago          /bin/sh -c ln -sf /dev/stdout /var/log/nginx/   0 B
    <missing>           3 days ago          /bin/sh -c apt-key adv --keyserver hkp://pgp.   58.55 MB
    <missing>           3 days ago          /bin/sh -c #(nop)  ENV NGINX_VERSION=1.11.8-1   0 B
    <missing>           3 days ago          /bin/sh -c #(nop)  MAINTAINER NGINX Docker Ma   0 B
    <missing>           4 days ago          /bin/sh -c #(nop)  CMD ["/bin/bash"]            0 B
    <missing>           4 days ago          /bin/sh -c #(nop) ADD file:89ecb642d662ee7edb   123 MB 
    
    
docker history 可以查看镜像的历史情况。

##### docker exec

    [root@jianfeng ~]# docker exec --help
    
    Usage:    docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
    
    Run a command in a running container
    
      -d, --detach         Detached mode: run command in the background
      --detach-keys        Override the key sequence for detaching a container
      --help               Print usage
      -i, --interactive    Keep STDIN open even if not attached
      --privileged         Give extended privileges to the command
      -t, --tty            Allocate a pseudo-TTY
      -u, --user           Username or UID (format: <name|uid>[:<group|gid>])
      
docker exec 命令可以进入已经运行的容器

docker exec -it nginx01 bash

以交互的方式进入 nginx01 容器，并使用bash 脚本运行。

    [root@jianfeng ~]# docker exec -it nginx01 bash
    root@b15047346752:/#



Nginx 默认的欢迎主页

    [root@jianfeng ~]# curl http://localhost:80
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>
    
    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>
    
    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>

修改 Nginx 首页


	root@b15047346752:/# echo '<h1>Hello JianFeng</h1> ' > /usr/share/nginx/html/index.html
	root@b15047346752:/# exit
	exit
	
再次访问 Nginx 主页

	[root@jianfeng ~]# curl http://localhost:80
	<h1>Hello JianFeng</h1>
	[root@jianfeng ~]#

已经修改完毕

##### docker diff

docker diff 用来查看容器的修改状况

	
	[root@jianfeng ~]# docker diff --help
	
	Usage:	docker diff [OPTIONS] CONTAINER
	
	Inspect changes on a container's filesystem
	
	  --help             Print usage
	  
	 
	[root@jianfeng ~]# docker diff nginx01
	C /var
	C /var/cache
	C /var/cache/nginx
	A /var/cache/nginx/fastcgi_temp
	A /var/cache/nginx/proxy_temp
	A /var/cache/nginx/scgi_temp
	A /var/cache/nginx/uwsgi_temp
	A /var/cache/nginx/client_temp
	C /root
	A /root/.bash_history
	C /run
	A /run/nginx.pid
	A /run/secrets
	C /usr
	C /usr/share
	C /usr/share/nginx
	C /usr/share/nginx/html
	C /usr/share/nginx/html/index.html
	
可以看到我们刚刚修改了主页的信息