---
layout:     post
title:      "MySQL配置文件"
subtitle:   "掌握MySQL配置文件的配置"
date:       2016-09-17
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - java
    - 构造器
---

> 本篇将介绍MySQL配置文件的相关知识


数据库：是物理操作系统文件或其他形式文件的集合。数据库文件可以是frm、MYD、MYI、ibd结尾的文件

实例：数据库实例可以操作数据库文件

在通常情况下，一个数据库对应一个实例，一个实例对应一个数据库。但在集群下，存在一个数据库可能对于多个数据实例的情况。数据库是操作系统和用户之间的一种数据管理软件，用户对数据库进行操作，包括对数据库的定义、对数据的增删改查、对数据的维护等等都是通过数据库实例进行完成的。应用程序只能通过数据库实例对数据库进行操作。

当启动数据库实例时，MySQL数据库会读取配置文件，根据配置文件里的参数来配置启动数据库实例。当MySQL数据库没有找到相应的配置文件时，MySQL会按照编译时的默认参数启动。



## windows下MySQL配置文件的位置


<table class="table table-bordered table-striped table-condesed">

	<tr>
		<td>File Name</td>
		<td>Purpose</td>
	</tr>
	<tr>
		<td>%PROGRAMDATA%\MySQL\MySQL Server 版本号\my.ini, %PROGRAMDATA%\MySQL\MySQL Server 版本号\my.cnf</td>
		<td>Global options</td>
	</tr>
	<tr>
		<td>%WINDIR%\my.ini, %WINDIR%\my.cnf</td>
		<td>Global options</td>
	</tr>
	<tr>
		<td>C:\my.ini, C:\my.cnf</td>
		<td>Global options</td>
	</tr>
	<tr>
		<td>BASEDIR\my.ini, BASEDIR\my.cnf</td>
		<td>Global options</td>
	</tr>
	<tr>
		<td>defaults-extra-file</td>
		<td>The file specified with --defaults-extra-file, if any</td>
	</tr>
	<tr>
		<td>%APPDATA%\MySQL\.mylogin.cnf</td>
		<td>Login path options (clients only)</td>
	</tr>

</table>


％PROGRAMDATA％表示包含主机上所有用户的应用程序数据的文件系统目录，比如win10是C:\Program Files。较老版本的系统在C:\Documents and Settings\All Users\Application Data下。

％WINDIR％表示Windows目录的位置。这通常是C：\ WINDOWS。

BASEDIR表示MySQL基本安装目录。

在Windows下运行 mysql --help 可以查找到MySQL配置文件路径

	Default options are read from the following files in the given order:
	C:\WINDOWS\my.ini C:\WINDOWS\my.cnf C:\my.ini C:\my.cnf C:\Program Files\MySQL\MySQL Server 5.7\my.ini C:\Program Files\MySQL\MySQL Server 5.7\my.cnf  
    
MySQL数据库是按照C:\WINDOWS\my.ini **→** C:\WINDOWS\my.cnf **→** C:\my.ini **→** C:\my.cnf **→** C:\Program Files\MySQL\MySQL Server 5.7\my.ini **→** C:\Program Files\MySQL\MySQL Server 5.7\my.cnf 的顺序进行读取配置文件，如果多个配置文件都对同一个参数进行配置，那么依哪一个为准呢？将按照最后一个配置文件的参数信息为准。


## MySQL 配置文件详解
	
	[client]
	port		= 3306
	socket		= /var/run/mysqld/mysqld.sock
	
	[mysqld_safe]
	socket		= /var/run/mysqld/mysqld.sock
	nice		= 0
	
	[mysqld]
	#
	# * 基本设置
	#
	character-set-server=utf8
	user		= mysql
	pid-file	= /var/run/mysqld/mysqld.pid
	socket		= /var/run/mysqld/mysqld.sock
	port		= 3306
	basedir		= /usr
	datadir		= /var/lib/mysql
	tmpdir		= /tmp
	lc-messages-dir	= /usr/share/mysql
	skip-external-locking
	
	#
	# MySQL服务器在单个网络上侦听TCP / IP连接。这个套接字绑定到单个地址，但是地址可以映射到多个网络接口。
	# 要指定地址，请在服务器启动时使用--bind-address = addr选项，其中addr是IPv4或IPv6地址或主机名。
	# 如果addr是主机名，则服务器将该名称解析为IP地址并绑定到该地址。
	# 如果地址为*，则服务器在所有服务器主机IPv6和IPv4接口上接受TCP/IP连接。
	bind-address		= 127.0.0.1
	#
	# * Fine Tuning
	#

	# key_buffer 通过增加该值，以便为所有读取和多个写入获得更好的索引处理;
	# 是MySQL使用MyISAM存储引擎的主要功能，使用服务器内存占25％总内存时，是此变量的较优值。
	# 但是，如果使值太大（例如，超过服务器总内存的50％），可能会导致数据库查询变慢。
	key_buffer		= 16M

	# 允许接受包的最大值
	max_allowed_packet	= 16M


	# 设置每个线程的堆栈大小。默认值192KB，对于正常操作是足够的。如果线程堆栈太小，
	# 会限制服务器可以处理的SQL语句的复杂性，存储过程的递归深度以及其他内存消耗操作。
	thread_stack		= 192K

	# 设置服务器应缓存多少线程以供重复使用。当客户端断开连接时，如果线程少于thread_cache_size线程，
	# 客户端的线程将放入缓存中。
	# 如果可能的话，通过重用从高速缓存取得的线程来满足线程的请求，并且只有当高速缓存为空时，才创建新的线程。
	# 如果你有很多新的连接，这个变量可以增加以提高性能。在实际线上中，如果服务器每秒有数百个连接请求，
	# 应将thread_cache_size设置为足够高，以便大多数新连接使用缓存线程。
	thread_cache_size       = 8
	myisam-recover         = BACKUP
	#max_connections        = 100
	#table_cache            = 64
	#thread_concurrency     = 10

	#
	# query_cache_limit 不缓存大于此字节数的结果。默认值为1MB。
	#
	query_cache_limit	= 1M
	query_cache_size        = 16M
	#
	#general_log_file        = /var/log/mysql/mysql.log
	#general_log             = 1
	#
	# 配置文件路径，将错误和启动消息记录到此文件，如果文件名没有扩展名，则服务器将添加.err的扩展名。
	# 如果省略文件名，在Unix和Linux系统下的默认日志文件为data目录中的host_name.err
	# Windows下的默认值为data目录下的host_name.err
	#
	log_error = /var/log/mysql/error.log
	#
	# Here you can see queries with especially long duration
	#log_slow_queries	= /var/log/mysql/mysql-slow.log
	#long_query_time = 2
	#log-queries-not-using-indexes
	#
	#  server-id 表示是本机的序号为1,一般来讲就是master的意思
	#server-id		= 1
	#log_bin			= /var/log/mysql/mysql-bin.log

	# 自动删除日志文件的天数。默认值为0，表示“无自动删除”。
	# 删除可能发生在启动时和日志刷新时。
	expire_logs_days	= 10

	# max_binlog_size
	# 如果对二进制日志的写入导致当前日志文件大小超过此变量的值，则服务器将关闭当前文件并新建下一个日志。
	# 最小值为4096字节。最大和默认值为1GB。
	max_binlog_size         = 100M
	#binlog_do_db		= include_database_name
	#binlog_ignore_db	= include_database_name
	
	[mysqldump]
	quick
	quote-names

	# max_allowed_packet 设置一个数据包的最大值，默认值为4MB。
	max_allowed_packet	= 4M
	
	[mysql]
	# default-character-set 设置字符集
	default-character-set=utf8
	
	[isamchk]
	# key_buffer_size 是用于设置索引块的缓冲区大小
	key_buffer		= 16M
	
	!includedir /etc/mysql/conf.d/

## 可选参数
	
	# 事务被提交后，其所用使用的undolog可能不再需要，可以使用PurgeThread来回收
	# 已经分配的undo页，可独立到单独的线程中进行，可以减轻Master Thread工作
	innodb_purge_threads = 1
	
	max_connections = 1000
	# MySQL的最大连接数，如果服务器的并发连接请求量比较大，建议调高此值，以增加并行连接数量，
	# 当然这建立在机器能支撑的情况下，因为如果连接数越多，介于MySQL会为每个连接提供连接缓冲区，
	# 就会开销越多的内存，所以要适当调整该值，不能盲目提高设值。
	# 可以过'conn%'通配符查看当前状态的连接数量，以定夺该值的大小。

	table_open_cache = 128
	# MySQL每打开一个表，都会读入一些数据到table_open_cache缓存中，
	# 当MySQL在这个缓存中找不到相应信息时，才会去磁盘上读取。默认值为64。
	# 假定系统有200个并发连接，则需将此参数设置为200*N(N为每个连接所需的文件描述符数目)；
	# 当把table_open_cache设置为很大时，如果系统处理不了那么多文件描述符，
	# 那么就会出现客户端失效，连接不上。

	query_cache_limit = 2M
	#指定单个查询能够使用的缓冲区大小，默认1M
	
	innodb_adaptive_flushing = on
	# 设置是否根据工作负载动态调整在InnoDB缓冲池中刷新脏页的速率。
	# 动态调整刷新可以避免I/O活动的突发状况。默认情况下启用此设置。
  
	innodb_adaptive_flushing_lwm = 0
	# 设置启用自适应刷新时重做日志容量的百分比，默认值为10，最小值为0，最大值为70.

	innodb_adaptive_hash_index = on
	# 设置是否启用或禁用InnoDB自适应散列索引。数据量庞大时，查询性能会降低，
	# 可能需要动态启用或禁用自适应哈希索引以提高查询性能。

	innodb_adaptive_hash_index_parts = 8
	# 分区自适应哈希索引搜索系统。把每个索引绑定到特定分区，每个分区由单独的锁存器保护。
	# 默认情况下，搜索系统被设置为8，最小值为0，最大设置为512。

	innodb_adaptive_max_sleep_delay = 150000
	# 设置最大的线程延时，允许InnoDB根据当前工作负载自动调整innodb_thread_sleep_delay的值。
	# 自动调整innodb_thread_sleep_delay值，直到调整到innodb_adaptive_max_sleep_delay
	# 选项中指定的最大值。
	# 此选项在繁忙的系统中使用，并且可以具有16个以上的InnoDB线程。 
	#（在实践中，它对于具有数百或数千个同时连接的MySQL系统是最有价值的。）

	innodb_additional_mem_pool_size = 8M
	# 设置InnoDB使用存储数据字典信息和其他内部数据结构的内存池大小。当数据库中有很多的表，
	# 那么分配的内存越多。如果InnoDB在此池中内存不足，
	# 它会开始从操作系统请求内存，并向MySQL错误日志写入警告消息。默认值为8MB。该参数在MySQL 5.6.3中已弃用。

	innodb_api_bk_commit_interval = 5
	# 设置InnoDB memcached接口自动提交空闲连接的频率（以秒为单位）。默认值为5，最小值为1，最大值为1073741824。


	innodb_log_buffer_size = 2M
	# 此参数确定些日志文件所用的内存大小，以M为单位。
	# 缓冲区更大能提高性能，但意外的故障将会丢失数据。MySQL开发人员建议设置为1－8M之间。

	innodb_log_file_size = 32M
	# 此参数确定数据日志文件的大小，更大的设置可以提高性能，
	#但也会增加恢复故障数据库所需的时间

	innodb_log_files_in_group = 3
	# 为提高性能，MySQL可以以循环方式将日志文件写到多个文件。推荐设置为3。

	innodb_max_dirty_pages_pct = 90
	# innodb主线程刷新缓存池中的数据，使脏数据比例小于90%。

	innodb_lock_wait_timeout = 120 
	# InnoDB事务在被回滚之前可以等待一个锁定的超时秒数。
	# InnoDB在它自己的锁定表中自动检测事务死锁并且回滚事务。
	# InnoDB用LOCK TABLES语句注意到锁定设置。默认值是50秒
	
	bulk_insert_buffer_size = 8M
	# 批量插入缓存大小， 这个参数是针对MyISAM存储引擎来说的。
	# 适用于在一次性插入100-1000+条记录时，提高效率。默认值是8M。
	#可以针对数据量的大小，翻倍增加。

	myisam_sort_buffer_size = 8M
	# MyISAM设置恢复表之时使用的缓冲区的尺寸，当在REPAIR TABLE或用CREATE INDEX创建索引或
	# ALTER TABLE过程中排序 MyISAM索引分配的缓冲区。





