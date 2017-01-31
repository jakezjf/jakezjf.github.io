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


##### docker commit

docker commit 命令可以定制自己的镜像

docker commit 命令参数：

	[root@jianfeng mynginx]# docker commit --help
	
	Usage:	docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
	
	Create a new image from a container's changes
	
	  -a, --author        Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
	  -c, --change=[]     Apply Dockerfile instruction to the created image
	  --help              Print usage
	  -m, --message       Commit message
	  -p, --pause=true    Pause container during commit


- -a 是修改人
- -m 类似git的-m，修改信息

镜像的定制实际上是定制每一层的文件、配置信息。

	[root@jianfeng ~]# docker commit -a "zhongjianfeng@meituan.com" -m "修改了index.html" nginx01 nginx:v1
	sha256:01012fd88816dc84714595c93e38417dba89fd5476155cc89249b4c18a620f32
	[root@jianfeng ~]# docker images
	REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
	nginx                   v1                  01012fd88816        18 seconds ago      181.6 MB
	docker.io/ubuntu        14.04               b969ab9f929b        29 hours ago        187.9 MB
	docker.io/nginx         latest              a39777a1a4a6        4 days ago          181.6 MB
	docker.io/hello-world   latest              48b5124b2768        8 days ago          1.84 kB
	
使用 commit 定制镜像是不是有点繁琐了？我们希望能写个像 shell 脚本一样，简洁方便呢？可以使用 build 命令，可以把每一层的修改、安装、配置、构建都写在脚本里，更加透明。
	
	
	
##### docker build

docker build 命令主要是用来定义自己的镜像

docker build 命令参数：


	[root@jianfeng mynginx]# docker build --help
	
	Usage:	docker build [OPTIONS] PATH | URL | -
	
	Build an image from a Dockerfile
	
	  --build-arg=[]                  Set build-time variables
	  --cpu-shares                    CPU shares (relative weight)
	  --cgroup-parent                 Optional parent cgroup for the container
	  --cpu-period                    Limit the CPU CFS (Completely Fair Scheduler) period
	  --cpu-quota                     Limit the CPU CFS (Completely Fair Scheduler) quota
	  --cpuset-cpus                   CPUs in which to allow execution (0-3, 0,1)
	  --cpuset-mems                   MEMs in which to allow execution (0-3, 0,1)
	  --disable-content-trust=true    Skip image verification
	  -f, --file                      Name of the Dockerfile (Default is 'PATH/Dockerfile')
	  --force-rm                      Always remove intermediate containers
	  --help                          Print usage
	  --isolation                     Container isolation level
	  -m, --memory                    Memory limit
	  --memory-swap                   Swap limit equal to memory plus swap: '-1' to enable unlimited swap
	  --no-cache                      Do not use cache when building the image
	  --pull                          Always attempt to pull a newer version of the image
	  -q, --quiet                     Suppress the build output and print image ID on success
	  --rm=true                       Remove intermediate containers after a successful build
	  --shm-size                      Size of /dev/shm, default value is 64MB
	  -t, --tag=[]                    Name and optionally a tag in the 'name:tag' format
	  --ulimit=[]                     Ulimit options
	  -v, --volume=[]                 Set build-time bind mounts


建立一个空文件夹，用来定制我们的镜像

	[root@jianfeng feng]# mkdir docker
	[root@jianfeng feng]# cd docker/
	[root@jianfeng docker]# mkdir mynginx
	[root@jianfeng docker]# cd mynginx/
	[root@jianfeng mynginx]# touch Dockerfile
	[root@jianfeng mynginx]# ls
	Dockerfile
	[root@jianfeng mynginx]# vi Dockerfile
	
创建一个文本文件：Dockerfile ，用来写脚本文件

使用 vim 进行编辑
	
	FROM nginx
	RUN echo "Hello zhongjianfeng" > /usr/share/nginx/html/index.html
	
FROM name   就是要指定构建镜像的基础镜像，比如：FROM nginx 就是我们定制的镜像是基于 nginx 的；

RUN 指令是用来执行命令的，是构建镜像的常用命令。
	
我们在写 docker 构建镜像脚本时，不能像 shell 一样写，之前说过 docker 是一层一层构建的，每使用一次 RUN 命令相当于在 docker 镜像上加一层镜像。

docker 构建镜像的层数是有限制的，之前 docker 允许构建 42 层镜像，目前最高可支持到 127 层，可见构建镜像时要考虑 RUN 命令使用的合理性。

RUN 命令可以使用 “&&” 符号，将多条命令的执行写到一个 RUN 中，节约了资源。

同时 docker 脚本也是支持注释的，可使用“#”进行注释。

	
##### 
	
	
	