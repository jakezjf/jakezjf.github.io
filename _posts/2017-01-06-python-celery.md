---
layout:     post
title:      "Python Celery 任务调度模块的应用"
subtitle:   "Python Celery 任务调度模块的应用"
date:       2017-01-06
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - Python
    - Celery
---

> 本篇将介绍 Python Celery 任务调度模块 的相关知识

# Celery 任务调度
Celery 是 Python 开发的分布式任务调度模块，Celery 使用起来非常方便，可使用在大量消息的分布式系统。Celery 的使用需要第三方中间人（Broker）的支持，例如redis、RabbitMQ。在实际开发中使用上述两个中间人比较稳定，当然 Celery 也支持MongoDB、ZeroMQ、CouchDB、SQLAlchemy、Django ORM、Amazon SQS。


## RabbitMQ
在 OS X 上安装 RabbitMQ

Mac 没有自带的 apt-get和yum，不过我们可以用 brew 代替他们。

首先使用 git 把 gitHub 上的 homebrew clone 下来

	git clone git://github.com/mxcl/homebrew/lol
	
	
##### Brew 安装
Brew 是包含一个简单的工具称为 brew，用于安装、移除和查询包。为了使用它，需要先把它添加到 PATH 中。

可以把下面的这行添加到你的 ~/.profile 末尾来实现：

	export PATH="/lol/bin:/lol/sbin:$PATH"

保存并重新加载：

	$ source ~/.profile
 
这时候就能使用 brew 了

##### RabbitMQ 安装

	$ brew install rabbitmq
	
等待一下就安装好了

##### 启动 RabbitMQ

	$ sudo rabbitmq-server
	
	
## 实例

### 测试任务调度

创建一个任务

##### 新建一个 task.py

	from celery import Celery
	
	app = Celery('task', broker='amqp://guest@localhost//')
	
	@app.task
	def add(x, y):
	    return x + y
	    
##### 启动任务
将任务交给中间人 broker 

在控制台输入：

	$ celery -A task worker --loglevel=info 
	
该命令启动的是Worker
	
task 是任务的 名字 

loglevel 日志级别


##### 调用任务

调用任务十分简单

我们在 shell 环境下进行测试：

	>>>from createData import add
	>>>add.delay(1,2)
	<AsyncResult: e590d7f7-1758-4552-b2bc-472f6ae4b581>

这时候我们可以在控制台看到以下信息：

	[2017-01-06 14:59:09,229: INFO/MainProcess] Connected to amqp://guest:**@127.0.0.1:5672//
	[2017-01-06 14:59:09,239: INFO/MainProcess] mingle: searching for neighbors
	[2017-01-06 14:59:10,249: INFO/MainProcess] mingle: all alone
	[2017-01-06 14:59:10,265: WARNING/MainProcess] celery@zhongjianfengdeMacBook-Pro.local ready.
	[2017-01-06 15:01:01,313: INFO/MainProcess] Received task: createData.add[e590d7f7-1758-4552-b2bc-472f6ae4b581]
	[2017-01-06 15:01:01,314: WARNING/Worker-5] 3

成功执行任务，返回3




















