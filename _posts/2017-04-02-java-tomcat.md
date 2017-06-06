---
layout:     post
title:      "Tomcat 项目部署形式"
subtitle:   "Tomcat war 和 war exploded 模式的区别"
date:       2017-04-02
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Java
---

> Tomcat war 和 war exploded 模式的区别

# Tomcat 配置

在配置 Tomcat 时，我们需要将项目发布，这时候项目发布包含两种形式，一种是 war ，另一个是 war exploded。

这两者有什么区别呢？

war 模式：是将项目工程以打包的形式上传到服务器

war exploded 模式：是将项目工程以自身文件的形式上传到服务器