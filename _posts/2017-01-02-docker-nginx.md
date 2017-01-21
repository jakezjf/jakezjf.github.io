---
layout:     post
title:      "Docker 下安装Nginx"
subtitle:   "Docker 下安装Nginx"
date:       2017-01-02
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Docker
    - Nginx
---

> 本篇将介绍 Docker 下安装Nginx 的相关知识
	
# Docker
Docker 是一个开源的应用容器引擎，Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。像我们在 Windows 下使用的虚拟软件 Vmware ，Docker 使用的场景也是将资源虚拟化分。而 Docker 与传统的 VM 不同的是：

- 传统 VM 是虚拟出一整套它所需的硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程。
- Docker 是建立在宿主机内核上的，它的应用进程直接运行在宿主机内核，没用对硬件资源进行虚拟化分，所以 Docker 比传统 VM 更加轻量、消耗资源少。

在宿主机上使用传统的 VM 技术，也只能虚拟化几十台虚拟机，但 Docker 可以虚拟化上千个容器甚至更多。

Docker 更大的优点在于它贴近面向对象的思想，它有类-镜像、实例-容器，对于运维人员及后台开发人员简直是福音。镜像相当于对一个软件、程序、运行环境的一个封装，在编写完应用程序时，我们可能要发布到几十台、几百台、甚至上千上万台。以往的方式我们可能要一台一台的为机器安装操作系统，安装运行环境，最后将应用程序部署，而在部署中肯定会遇到很多意想不到的问题，如 linux 版本内核不兼容，JDK 版本不一致等问题。运维人员和后台开发人员可能会在部署上花很多的时间，但如果我们将我们所需的应用程序及运行环境一起“打包”，类似于 Maven 的 war 包，直接在机器上虚拟化，只需简单的几步就能将程序部署在大规模的集群下。集群里用的环境、应用程序都一致，使我们更方便的管理。使用 Docker 可以通过定制应用镜像来实现持续集成、持续交付、部署。

# Nginx
Nginx 是一个高性能的HTTP和反向代理服务器，在集群的环境下，我们常用 Nginx 作为负载均衡，使得服务压力可以合理的分担到集群相应的服务器。

#在 Docker 下安装 Nginx

##### docker images 命令可以查询已经下载的镜像

	[root@jianfeng ~]# docker images
	REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
	docker.io/ubuntu        14.04               b969ab9f929b        16 hours ago        187.9 MB
	docker.io/nginx         latest              a39777a1a4a6        3 days ago          181.6 MB
	docker.io/hello-world   latest              48b5124b2768        7 days ago          1.84 kB
	
REPOSITORY 是仓库的名称，TAG 相应的版本号，例如：ubuntu 14.04 ，IMAGE ID 是唯一的标识ID，CREATED 是创建的时间，SIZE 是该镜像德1大小。

##### docker pull 可以从远程仓库拉取镜像，这里我已经下好nginx的了

##### 创建容器

docker run --name nginx01 -d -p 80:80 nginx

创建容器名为 nginx01 的容器，访问端口设置为80，用的镜像是nginx
 
	[root@jianfeng ~]# docker run --name nginx01 -d -p 80:80 nginx
	b15047346752b440916666f6569ab15e757a13c88799c76cf3be558fc7e4063d
	
##### 查看创建容器产生了什么变化

docker diff

	[root@jianfeng ~]# docker diff nginx01
	C /run
	A /run/nginx.pid
	A /run/secrets
	C /var
	C /var/cache
	C /var/cache/nginx
	A /var/cache/nginx/proxy_temp
	A /var/cache/nginx/scgi_temp
	A /var/cache/nginx/uwsgi_temp
	A /var/cache/nginx/client_temp
	A /var/cache/nginx/fastcgi_temp
	
##### 访问

通过curl 命令对80端口访问

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
	[root@jianfeng ~]#

成功访问


# 总结

使用 Docker 我们能方便高效的部署我们的服务应用，让我们更懒，一次创建、配置，可以在任意地方正常运行，让我们更专注于开发。


