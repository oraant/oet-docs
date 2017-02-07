架构与设计及后台原理简述
==============================

注意，此文属于公司保密资料，仅限内部员工使用，不可外传。

架构设计与原理简述
------------------------

架构设计图如下：

.. image:: /_static/Topological.png

当前版本中，还有一些计划中的二期三期任务并没有被实现：

- 利用 Kapacitor 和 Reporter ，结合 各种API 来报警没有被实现，
- 利用 Logs 和 log parser 来展示日志没有被实现。
- 利用 Statsd 等组件来从操作系统抓取 目标OS 的系统信息没有被实现。

和配置相关的模块：

- admin：Django中的Admin模块，利用网页来操作MySQL数据库中的表，在此模块中，可以对展示列表时的很多地方做详细的配置。
- ORM Configs：Django中的Model模块，利用Python中的类，和MySQL中的表对应起来，并增加很多配置。如类型验证、选项、表链接引用等。

同步表的模块：

- mirror：抽象的一个概念，指用来同步一组表的几个方法。这些方法的主要作用，是将被监控的 Oracle 中，将关键的数据字典表从 Oracle 中抓取放入自己的 MySQL 服务器中，方便执行消耗大量性能的查询SQL。
- pull：从 Oracle 中把表中的数据全部查出来。
- cache：将 pull 中查出来的数据，缓存到 MySQL 的中，如果已缓存了数据，则清除这些旧数据。
- record：pull + cache 一个表，都将此信息覆盖记录到 redis 中，用来通知其他组件缓存的表中的数据是否是最新的。

收集数据的模块：

- gather：抽象的一个概念，指用来收集数据的几个方法。这些方法的主要作用，是从自己 MySQL 服务器里缓存的表中，通过where条件、表链接等，查询出要监控的性能点。
- verify：针对要收集的性能点，可能需要从多个表中连接查询。这个方法毁从 redis 数据库中读取用到的表的同步时间，确保这些表里的数据都是最新的，否则不会进行查询。
- pick：执行SQL语句，从对应的表中查出想要的数据的值。
- archive：将 pick 查出来的值，归档存储到 InfluxDB 中，用来方便后续的数据展示。

展示图片的模块：

- pages：pages是 Django 中 Templates 模块和网页模板 AdminLTE 及其他前端组件结合的产物。

本文所用到的模块和组件：
----------------------------------------

Python：主要用来编写程序逻辑等
Django：基于Python的网络开发框架，利用网络框架快速搭建网络服务
HTML/CSS/JS：网页三剑客，主要编写前端的内容展现
BootStrap：基于三剑客的前台框架，利用框架快速编写网页
AdminLTE：前端网页模板

Python模块：
^^^^^^^^^^^^^^^^^^

基础模块：

- celery：远程任务调度
- supervisor：管理后台进程
- gunicorn：部署Django任务

和数据库交互：

- cx_Oracle：连接Oracle数据库
- influxdb：连接InfluxDB数据库
- MySQL-python：连接MySQL数据库
- redis：连接Redis数据库
- psutil：获取操作系统信息

其他模块：

- APScheduler: 任务调度
- django-markdown-deux：使Django支持Markdown
- docutils：将文本处理为HTML

前端模块：
^^^^^^^^^^^^^^^^^^^^^

图表展现：

- HighStock：主要用于趋势图展现
- Echarts：主要用于系统性能展现
- DataTables：主要用于表格展现

其他模块：
^^^^^^^^^^^^^^^^^^^^^

- Nginx：轻量级的网络服务器，用来展示 Django 编写的网页及静态文件
- OIC：Oracle Instant Client，主要用于 cx_oracle 组件，用来被依赖的
- Pyenv：因为很多客户的机器过于老旧，自带光盘中的 Python 版本不支持 Django ，所以需要利用 Pyenv 来安装其他版本的 Python
- Redis：缓存数据库，主要用于记录同步表时的时间信息，确保抽取数据时表里的数据是最新的。另外 Celery 组件也需要利用 Redis 来通信
- TICK：Influx Data的一组工具，主要利用其中的 Influx DB 来存储时序数据，并用来展现数据
- Oracle：要监控的目标数据库。
- MySQL：主要用于缓存目标数据库中的数据字典表。另外 Django 中需要存储的配置表也在这里面。

代码逻辑
-----------------------------------

