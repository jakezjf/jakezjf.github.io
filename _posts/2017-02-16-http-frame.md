---
layout:     post
title:      "X-Frame-Options 响应头的应用"
subtitle:   "学习X-Frame-Options 响应头的应用"
date:       2017-02-016
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Http
---

> 本篇将介绍 X-Frame-Options 响应头 的相关知识



# X-Frame-Options 响应头

### 背景
在进行前端开发中，经常会遇到页面与页面之间的嵌套问题，在写管理系统时表现尤甚哈哈。而 X-Frame-Options 响应头能够有效的控制网页是否内被嵌套，有很多导航网站喜欢集成嵌套各种网站，如果你不想自己的网站被其他网站嵌套，可以使用X-Frame-Options 响应头。

### X-Frame-Options HTTP 响应头的作用 

X-Frame-Options HTTP是用来给浏览器指示允许一个页面可否在 <frame>, <iframe> 或者 <object> 中展现的标记。网站可以使用此功能，来确保自己网站的内容没有被嵌到别人的网站中去，也从而避免了点击劫持 (clickjacking) 的攻击。怎么样？X-Frame-Options HTTP是不是很强大。

### X-Frame-Options HTTP 的使用   

X-Frame-Options HTTP 主要有三个参数可选用：

- DENY

表示该页面不允许在 frame 中展示，即便是在相同域名的页面中嵌套也不允许。这是最安全的，禁止一切嵌套。

- SAMEORIGIN

表示该页面可以在相同域名页面的 frame 中展示。使用了该参数值，可以在自己的域名下进行嵌套。安全性较高。

- ALLOW-FROM uri

ALLOW-FROM 允许指定 url ，允许该 url 能进行嵌套 ，表示该页面可以在指定来源的 frame 中展示。

### 在各类服务器下的配置

- Apache

配置 Apache 服务器，使所有页面都发送 X-Frame-Options 响应头，在 Apache 的’site‘ 配置中进行如下配置：

	
	Header always append X-Frame-Options SAMEORIGIN
	
- Nginx

Nginx 是一个支撑高并发用于负载均衡的服务器，配置 Nginx 发送 X-Frame-Options 响应头，把下面这行添加到 'http', 'server' 或者 'location' 的配置中:

	
	add_header X-Frame-Options SAMEORIGIN;
	

- IIS

配置 IIS 发送 X-Frame-Options 响应头，添加下面的配置到 Web.config 文件中:

	<system.webServer>
	  ...
	
	  <httpProtocol>
	    <customHeaders>
	      <add name="X-Frame-Options" value="SAMEORIGIN" />
	    </customHeaders>
	  </httpProtocol>
	
	  ...
	</system.webServer>
	
# 总结

在实际开发中，如果使用了 X-Frame-Options 响应头限制网页嵌套，在嵌套该网页时会显示出about:blank的空白页面，没有报错信息，不太好，所以建议价格判断，并弹出错误提示信息。



