---
layout:     post
title:      "搭建crm及单机版云遇到的坑"
subtitle:   "搭建crm及单机版云遇到的坑"
date:       2017-01-05
author:     "JianFeng"
header-img: ""
catalog: true
tags:
    - python
---

> 搭建crm及单机版云遇到的坑


＃ python 云单机版环境下遇到的坑

#### six 包BUG

six==1.8.0 依赖包有bug，更新为six==1.10.0

	Traceback (most recent call last):
	  File "./manage.py", line 10, in <module>
	    execute_from_command_line(sys.argv)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/core/management/__init__.py", line 338, in execute_from_command_line
	    utility.execute()
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/core/management/__init__.py", line 312, in execute
	    django.setup()
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/__init__.py", line 18, in setup
	    apps.populate(settings.INSTALLED_APPS)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/apps/registry.py", line 85, in populate
	    app_config = AppConfig.create(entry)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/apps/config.py", line 86, in create
	    module = import_module(entry)
	  File "/usr/local/Cellar/python/2.7.13/Frameworks/Python.framework/Versions/2.7/lib/python2.7/importlib/__init__.py", line 37, in import_module
	    __import__(name)
	  File "/Users/zhongjianfeng/pro/crm/asynctask/__init__.py", line 9, in <module>
	    from asynctask.task import *
	  File "/Users/zhongjianfeng/pro/crm/asynctask/task/__init__.py", line 14, in <module>
	    from asynctask.task.edm import (email_notify, email_resend,
	  File "/Users/zhongjianfeng/pro/crm/asynctask/task/edm.py", line 4, in <module>
	    from edm.models import EmailRecord, SMSRecord
	  File "/Users/zhongjianfeng/pro/crm/apps/edm/models.py", line 17, in <module>
	    import bleach
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/bleach/__init__.py", line 7, in <module>
	    import html5lib
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/html5lib/__init__.py", line 16, in <module>
	    from .html5parser import HTMLParser, parse, parseFragment
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/html5lib/html5parser.py", line 2, in <module>
	    from six import with_metaclass, viewkeys, PY3
	ImportError: cannot import name viewkeys

	
six==1.8.0 依赖包有bug，更新为six==1.10.0

使用pip进行安装：

	pip install six==1.10.0
	

#### html5lib 依赖包引发的问题

	Traceback (most recent call last):
	  File "./manage.py", line 10, in <module>
	    execute_from_command_line(sys.argv)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/core/management/__init__.py", line 338, in execute_from_command_line
	    utility.execute()
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/core/management/__init__.py", line 312, in execute
	    django.setup()
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/__init__.py", line 18, in setup
	    apps.populate(settings.INSTALLED_APPS)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/apps/registry.py", line 85, in populate
	    app_config = AppConfig.create(entry)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/apps/config.py", line 86, in create
	    module = import_module(entry)
	  File "/usr/local/Cellar/python/2.7.13/Frameworks/Python.framework/Versions/2.7/lib/python2.7/importlib/__init__.py", line 37, in import_module
	    __import__(name)
	  File "/Users/zhongjianfeng/pro/crm/asynctask/__init__.py", line 9, in <module>
	    from asynctask.task import *
	  File "/Users/zhongjianfeng/pro/crm/asynctask/task/__init__.py", line 14, in <module>
	    from asynctask.task.edm import (email_notify, email_resend,
	  File "/Users/zhongjianfeng/pro/crm/asynctask/task/edm.py", line 4, in <module>
	    from edm.models import EmailRecord, SMSRecord
	  File "/Users/zhongjianfeng/pro/crm/apps/edm/models.py", line 17, in <module>
	    import bleach
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/bleach/__init__.py", line 8, in <module>
	    from html5lib.sanitizer import HTMLSanitizer
	ImportError: No module named sanitizer
	
	
	
报错：

	from html5lib.sanitizer import HTMLSanitizer
		ImportError: No module named sanitizer


解决方案：pip install --upgrade bleach


	pip install --upgrade bleach
	Collecting bleach
	  Downloading bleach-1.5.0-py2.py3-none-any.whl
	Collecting html5lib!=0.9999,!=0.99999,<0.99999999,>=0.999 (from bleach)
	  Downloading html5lib-0.9999999.tar.gz (889kB)
	    100% |████████████████████████████████| 890kB 429kB/s
	Requirement already up-to-date: six in ./env/lib/python2.7/site-packages (from bleach)
	Building wheels for collected packages: html5lib
	  Running setup.py bdist_wheel for html5lib ... done
	  Stored in directory: /Users/zhongjianfeng/Library/Caches/pip/wheels/6f/85/6c/
	Successfully built html5lib
	Installing collected packages: html5lib, bleach
	  Found existing installation: html5lib 0.999999999
	    Uninstalling html5lib-0.999999999:
	      Successfully uninstalled html5lib-0.999999999
	  Found existing installation: bleach 1.4
	    Uninstalling bleach-1.4:
	      Successfully uninstalled bleach-1.4
	Successfully installed bleach-1.5.0 html5lib-0.9999999
	
	
#### 数据库问题
在使用django批量建表时报错，出现无法创建一些表，这些表都包含有外键，看报错发现是这些表无法创建外键的原因。

	Traceback (most recent call last):
	  File "./manage.py", line 10, in <module>
	    execute_from_command_line(sys.argv)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/core/management/__init__.py", line 338, in execute_from_command_line
	    utility.execute()
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/core/management/__init__.py", line 330, in execute
	    self.fetch_command(subcommand).run_from_argv(self.argv)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/core/management/base.py", line 393, in run_from_argv
	    self.execute(*args, **cmd_options)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/core/management/base.py", line 444, in execute
	    output = self.handle(*args, **options)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/core/management/commands/syncdb.py", line 25, in handle
	    call_command("migrate", **options)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/core/management/__init__.py", line 120, in call_command
	    return command.execute(*args, **defaults)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/core/management/base.py", line 444, in execute
	    output = self.handle(*args, **options)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/core/management/commands/migrate.py", line 179, in handle
	    created_models = self.sync_apps(connection, executor.loader.unmigrated_apps)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/core/management/commands/migrate.py", line 317, in sync_apps
	    cursor.execute(statement)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/db/backends/utils.py", line 79, in execute
	    return super(CursorDebugWrapper, self).execute(sql, params)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/db/backends/utils.py", line 64, in execute
	    return self.cursor.execute(sql, params)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/db/utils.py", line 97, in __exit__
	    six.reraise(dj_exc_type, dj_exc_value, traceback)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/db/backends/utils.py", line 62, in execute
	    return self.cursor.execute(sql)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/django/db/backends/mysql/base.py", line 124, in execute
	    return self.cursor.execute(query, args)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/MySQLdb/cursors.py", line 205, in execute
	    self.errorhandler(self, exc, value)
	  File "/Users/zhongjianfeng/pro/crm/env/lib/python2.7/site-packages/MySQLdb/connections.py", line 36, in defaulterrorhandler
	    raise errorclass, errorvalue
	django.db.utils.IntegrityError: (1215, 'Cannot add foreign key constraint')
	
报错：

	django.db.utils.IntegrityError: (1215, 'Cannot add foreign key constraint')

解决方案：

在批量建表时我们常常会忽略建表顺序的问题，默认情况下，MySQL建表，外键会在该数据库其他表中判断这个外键是否在其他表作为主键，如果找到了，就能正常创建；若发现该主键不存在就会出现像上面的情况。

- 使用 SHOW ENGINE INNODB STATUS; 命令可以查看 LATEST FOREIGN KEY ERROR 外键的创建情况。
- 检查表中外键与主键对应类型是否一样
- 同时,在查询运行DDL之前设置set foreign_key_checks = 0，这样就可以以任意顺序创建表而不是之前需要创建所有父表相关的子表。

⚠️ 数据库表的引擎类型也要注意，MySQL最常用的数据存储引擎有 **InnoDB** 、 **MyISAM** ，在导表的时候要注意原存储引擎和现在的存储引擎。

在 **MyISAM** 转 **InnoDB** 情况下转化，可以在 Django 配制 OPTIONS 属性。
添加：

	 'OPTIONS':{'init_command':'SET storage_engine=MyISAM'







