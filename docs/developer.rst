开发者应该知道的事
===========================

如何添加一个参数
-----------------------

OET获取数据的过程如下：将数据词典表从Oracle中抓取到MySQL中 --> 从表中抓取数据存储到InfluxDB中 --> 将InfluxDB中的数据通过HighStock展现出来

所以添加参数的思路整体如下：添加要同步哪一张表过来，添加要从同步过来的表中抓取什么数据，添加展示数据时的图片

添加表
^^^^^^^^^^^

进入后台，在Tables表中，新增加一项内容：

Basic，基础信息:

- Name: 监控这个表的名字是什么
- Enable: 是否启用
- Desc: 描述信息
- Period: 每隔多长时间同步一次这个表

Pull SQL in Target Oracle Database，从Oracle中抓取数据相关的SQL:

- Pull: 从Oracle中抓取表数据的SQL语句

Cache SQL in MySQL Server，向MySQL中缓存数据相关的SQL:

- Create: 在MySQL中创建对应的缓存表时，用什么SQL。注意这里一定要加上 IF NOT EXISTS, ENGINE=MEMORY, 确保不会重复创建，并且缓存表是在内存中
- Drop: 删除创建的缓存表时的SQL语句
- Insert: 把 Python 抓出来的数据，缓存入 MySQL 时，
- Delete: 把旧的缓存数据删除时的SQL语句

填写完成后，点击保存。

TODO：重启什么？
重启并生效后，你可以登录用做缓存的 MySQL 数据库，耐心等待一段时间后，查看是否正常同步了过来。

添加点
^^^^^^^^^^^^^

进入后台，在Points表中，新增加一项内容：

Basic:

- Name: 这个抓取的性能点的名字是什么
- Enable: 是否启用
- Desc: 描述信息
- Period: 每隔多长时间抓取一次

Pull SQL in Cached MySQL Database，从MySQL中抓取数据相关的内容:

- Pull: 从MySQL中，查询出性能数据的SQL
- Tables: 在执行上面的SQL前，需要先确保哪些表的数据是最新的

Archive data in Influx Database，向InfluxDB中归档数据相关的内容:

- Measurement: 抽取出来的数据，存入InfluxDB中的哪个表里面
- Tags: 向InfluxDB中存数据时，哪几列用做 tag，即建立了索引的列。这些数据一般不会随着时间的变化而动态变化，而是用来做选择用。比如（机器型号，主机名，服务器版本）等。
- Fields: 向InfluxDB中存数据时，哪几列用做 fields，即没有索引的列。这些数据一般都是实时变化的。比如（CPU繁忙度，内存使用率，磁盘IO）等。

添加完成后，点击保存。

TODO：怎么重启？
重启并生效后，你可以登录用做归档的 InfluxDB 数据库，耐心等待一段时间后，查看是否正常抓取了数据过来。

添加图
^^^^^^^^^^^^

进入后台，在Details表中，新增加一项内容：

- Name: 图表的ID，这里不支持空格，单词之间必须用短横线-来连接起来
- Enable: 是否展示
- Title: 图表在展示时的标题，一般这里填写参数的中文名称
- SubTitle: 图表在展示时的标题，一般这里填写参数的英文名称
- Unit: 图表所对应参数的单位
- Category: 图表所在的分类是什么
- Number: 图表在该分类下的排序是多少，最好之间相隔10个，方便将来插入新内容
- Influx sql: 从InfluxDB中，将对应的数据抽取出来的SQL
- Desc: 图表的帮助文档。注意，这里支持Markdown语法

添加完成后，点击保存。

将相关内容增加入对应的监控方案中
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

进入后台，在Schemes表中，新增加一项内容（如果您不需要新增，只想要更改已有的监控方案，则可以直接点击该方案进行修改）：

Name: 监控方案的名称
Tables: 应用此套监控方案时，将会同步哪些表
Points: 应用此套监控方案时，会从同步过来的表中抽取哪些数据
Details: 应用此套监控方案时，会在前端页面展示哪些图表

添加完成后，点击保存。

使配置生效
^^^^^^^^^^^^^^^^^^

有了新的监控方案后，在需要更改监控方案的监控目标中，将监控方案改为新创建的这个。

然后重启监控目标，即将其Revoke后，重新Create。


如何使用其他配置
--------------------------------

如果你想更改一些默认配置，如MySQL数据库中 oet 用户的密码，那么有以下地方需要注意。

Django项目本身需要存储配置文件，这些配置文件的存放位置在 MySQL 数据库中，连接方式在 /oet/oet/settings.py 中配置

Celery插件需要使用Redis用来通信（Broker），连接方式在 /oet/oet/settings.py 中配置，即其中和 BROKER 相关的配置选项

如果要想修改 /oet 的位置，则需要注意把 nginx 中指向静态文件的路径改掉，即 /etc/nginx/conf.d/default.conf 中 location /static 这一配置项的内容

无法进入主页的情况
------------------------------

如果无法正常进入主页，但是可以进入后台（即http://ip/admin），则很有可能是 Targets 表中没有数据导致的。

因此，要确保 Targets 表中只要有一个监控对象。
