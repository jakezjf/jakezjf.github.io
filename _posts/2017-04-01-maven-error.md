---
layout:     post
title:      "Maven遇到的问题"
subtitle:   "Maven 启动报错问题解决"
date:       2017-04-01
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Maven
---

> Maven 启动报错问题解决



# Maven 启动报错
Maven 配置环境配置好后，在控制台上运行 mvn 命令，发现报错了，错误如下：

	[ERROR] No goals have been specified for this build. You
	fecycle phase or a goal in the format <plugin-prefix>:<go
	>:<plugin-artifact-id>[:<plugin-version>]:<goal>. Availab
	: validate, initialize, generate-sources, process-sources
	rocess-resources, compile, process-classes, generate-test
	sources, generate-test-resources, process-test-resources,
	test-classes, test, prepare-package, package, pre-integra
	test, post-integration-test, verify, install, deploy, pre
	an, pre-site, site, post-site, site-deploy. -> [Help 1]
	    
# 解决问题
在pom.xml文件<build>标签里面加 上<defaultGoal>compile</defaultGoal>可以解决。