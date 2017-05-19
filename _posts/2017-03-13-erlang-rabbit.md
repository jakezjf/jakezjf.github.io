---
layout:     post
title:      "RabbitMQ 消息中间件的安装 "
subtitle:   "RabbitMQ 消息中间件的安装"
date:       2017-03-13
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - RabbitMQ
---

> RabbitMQ 消息中间件的安装



# RabbitMQ 

RabbitMQ 是著名的消息队列中间件，使用 Erlang 语言进行编写，支持多种语言进行交互，和许多消息中间件例如ActiveMQ、ZeroMQ 和 Apach Qpid 等一样提供了不同的开源消息队列解决方案。



# RabbitMQ 的安装

##### 语言环境

安装 RabbitMQ 首先要先安装 Erlang 的语言环境，我使用的系统是 Mac ，可以使用 Homebrew 安装 Erlang 环境。

```
brew install erlang    
```

更新 Erlang

```
brew upgrade erlang
```



##### RabbitMQ 下载

```
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.5.0/rabbitmq-server-mac-standalone-3.5.0.tar.gz
```



##### 解压压缩包

```
tar -xzvf rabbitmq-server-mac-standalone-3.5.0.tar.gz
```

```
cd rabbitmq-server-mac-standalone-3.5.0.tar.gz
```



##### 创建log目录

```
mkdir -p /var/log/rabbitmq
```

```
mkdir -p /var/log/rabbitmq/mnesia/rabbit
```



##### 运行 RabbitMQ

上述步骤完成后，我们可以开始运行 RabbitMQ 了，进入 RabbitMQ 的根目录。

运行 RabbitMQ

```
sbin/rabbitmq-server
```

出现如下提示，说明安装成功

```
              RabbitMQ 3.5.0. Copyright (C) 2007-2014 GoPivotal, Inc.
  ##  ##      Licensed under the MPL.  See http://www.rabbitmq.com/
  ##  ##
  ##########  Logs: sbin/../var/log/rabbitmq/rabbit@zhongjiengdeMBP.log
  ######  ##        sbin/../var/log/rabbitmq/rabbit@zhongjiengdeMBP-sasl.log
  ##########
              Starting broker... completed with 0 plugins.
```

检查 RabbitMQ 的运行状态：

```
$ sbin/rabbitmqctl status
```

出现如下提示说明运行正常：

```
[{pid,32797},
 {running_applications,[{rabbit,"RabbitMQ","3.5.0"},
                        {mnesia,"MNESIA  CXC 138 12","4.12.1"},
                        {os_mon,"CPO  CXC 138 46","2.2.15"},
                        {xmerl,"XML parser","1.3.7"},
                        {sasl,"SASL  CXC 138 11","2.4"},
                        {stdlib,"ERTS  CXC 138 10","2.1"},
                        {kernel,"ERTS  CXC 138 10","3.0.1"}]},
 {os,{unix,darwin}},
 {erlang_version,"Erlang/OTP 17 [erts-6.1] [source] [64-bit] [smp:8:8] [async-threads:30] [hipe] [kernel-poll:true]\n"},
 {memory,[{total,38213080},
          {connection_readers,0},
          {connection_writers,0},
          {connection_channels,0},
          {connection_other,2808},
          {queue_procs,2808},
          {queue_slave_procs,0},
          {plugins,0},
          {other_proc,13782304},
          {mnesia,61984},
          {mgmt_db,0},
          {msg_index,47680},
          {other_ets,891552},
          {binary,13280},
          {code,16889202},
          {atom,654217},
          {other_system,5867245}]},
 {alarms,[]},
 {listeners,[{clustering,25672,"::"},{amqp,5672,"::"}]},
 {vm_memory_high_watermark,0.4},
 {vm_memory_limit,6056111308},
 {disk_free_limit,50000000},
 {disk_free,174695522304},
 {file_descriptors,[{total_limit,4764},
                    {total_used,3},
                    {sockets_limit,4285},
                    {sockets_used,1}]},
 {processes,[{limit,1048576},{used,130}]},
 {run_queue,0},
 {uptime,86}]    
```



# 总结

RabbitMQ 的安装流程十分简易，是许多消息中间件中安装最简便的。



















