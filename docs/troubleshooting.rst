问题排查
==================

如何查看日志
---------------

日志文件在 /oet/data/log/ 下，一般有以下文件::

  celery.log
  celery_worker0.log
  celery_worker1.log
  celery_worker2.log
  celery_worker3.log
  gunicorn.log
  supervisord.log

其中文件的作用分别为:

- celery.log: celery插件的调度日志
- celery_workerX.log: 从Oracle抓取数据的进程的日志
- gunicorn.log: 部署Django的进程的日志
- supervisord.log: 将进程变为后台守护进程的日志

其中，celery_workerX.log 这种格式的日志，每个任务都会产生进程，每个进程都会产生日志。前后两个任务，有可能会复用同一个日志文件，那么这时候，应该如何找到某一对象对应的监控日志呢？

1，结合时间判断，确定进程范围。

2，进入后台，查看Target的名称并通过grep命令查找。比如，想知道 test 目标同步 test_v$tablespace 这张表相关的信息，就可以这样做::

  # cd /oet/data/log
  # grep --color "mirror.test.test_v\$tablespace" *  （一般来说，日志里非常大，最好加上less命令，然后用查找并高亮要看的内容）

上面要过滤的字符串格式为 < mirror | gather > . < TARGET NAME > . < Table Name / Point Name > ，其中 mirror 代表同步表并缓存，gather代表收集数据并归档。

如何防止日志文件过大
-----------------------------------

稳定运行一段时间后，可以尝试在 celery 的配置文件中，将 loglevel 调整为 WARNING 或 ERROR

如何查看产品是否正常运行
--------------------------------

通过 supervisorctl status 查看两个后台进程是否正常启动::

  # supervisorctl status
  oet_celery                       RUNNING   pid 115484, uptime 22 days, 0:25:41
  oet_gunicorn                     RUNNING   pid 115483, uptime 22 days, 0:25:41

通过 ps 确认进程是否真正启动了::

  # bash util/checkproc.sh 
  root     115452      1  0 Feb07 ?        00:01:18 /root/.pyenv/versions/2.7.9/envs/oet/bin/python2.7 /root/.pyenv/versions/oet/bin/supervisord
  root     115483 115452  0 Feb07 ?        00:00:58 /root/.pyenv/versions/2.7.9/envs/oet/bin/python2.7 /root/.pyenv/versions/oet/bin/gunicorn oet.wsgi -b 0.0.0.0:8000
  root     115484 115452  0 Feb07 ?        00:10:19 /root/.pyenv/versions/2.7.9/envs/oet/bin/python2.7 /root/.pyenv/versions/oet/bin/celery worker -A oet --autoscale=99, 0 -Ofair --loglevel=INFO --logfile=/oet/etc/../data/log/celery_worker%i.log --statedb=/oet/etc/../data/run/celery.state
  root     115672 115484  1 Feb07 ?        00:56:14 /root/.pyenv/versions/2.7.9/envs/oet/bin/python2.7 /root/.pyenv/versions/oet/bin/celery worker -A oet --autoscale=99, 0 -Ofair --loglevel=INFO --logfile=/oet/etc/../data/log/celery_worker%i.log --statedb=/oet/etc/../data/run/celery.state
  root     120183 115484  1 Feb08 ?        00:35:24 /root/.pyenv/versions/2.7.9/envs/oet/bin/python2.7 /root/.pyenv/versions/oet/bin/celery worker -A oet --autoscale=99, 0 -Ofair --loglevel=INFO --logfile=/oet/etc/../data/log/celery_worker%i.log --statedb=/oet/etc/../data/run/celery.state
  root     126032 115483  0 09:22 ?        00:00:02 /root/.pyenv/versions/2.7.9/envs/oet/bin/python2.7 /root/.pyenv/versions/oet/bin/gunicorn oet.wsgi -b 0.0.0.0:8000

通过 netstat 查看端口是否正常开启::

  # netstat -tnlup|egrep "80 |8000 "
  tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      17876/nginx         
  tcp        0      0 0.0.0.0:8000                0.0.0.0:*                   LISTEN      115483/python2.7

登录 http://ip 查看网页是否能正常访问

查看首页监控对象的状态是否是 RUNNING

查看监控对象的图表中有没有数据

如何开启关掉的开启的服务
----------------------------------------------------

通过 supervisorctl shutdown 来关闭后台进程::

  # supervisorctl shutdown
  Shut down

没有数据怎么办
-------------------

可能是没有选择正确的数据库，此时可以点击右上角的 选择数据库 来选择要查看的数据库

可能是没有选择正确的参数，此时应多尝试几个不同的分类，多查看一些图表

可能是没有选择正确的时间范围，此时应尝试调整不同的时间范围，如查看全部（ALL）或最近一小时的信息

登录后台，确保相关的 Table，Point，Detail 配置项，都是被启用了的，即 Enabled

在确保没有以上低级错误的情况下，登录InfluxDB，进入相关的数据库中，查看对应的 measurement 中有没有生成数据。如果有数据，则根据 Details 中配置的 SQL 语句，尝试查询出数据来。

如果 measurement 中没有数据，则查看 MySQL 中是否成功的将相关的表映射了过来。如果映射了过来，则问题出在了抓取数据并归档这一环节。如果没有映射成功，则问题出在了同步表并缓存的环节。

这时，请根据后台配置，手工模拟同步表并缓存，或抓数据并归档相关的操作，确保相关语句是可以执行的通的。

如果是无法发现问题，请更改 /oet/etc/celeryd.conf 中的选项，将 --loglevel=INFO 改为 --loglevel=DEBUG，并重启，然后分析新产生的日志。

如何查看进程信息
------------------

通过 supervisorctl status 查看两个后台进程是否正常启动::

  # supervisorctl status
  oet_celery                       RUNNING   pid 115484, uptime 22 days, 0:25:41
  oet_gunicorn                     RUNNING   pid 115483, uptime 22 days, 0:25:41

通过 ps 确认进程是否真正启动了::

  # bash util/checkproc.sh 
  root     129938      1  0 15:01 ?        00:00:03 /root/.pyenv/versions/2.7.9/envs/oet/bin/python2.7 /root/.pyenv/versions/oet/bin/supervisord
  root     129969 129938  0 15:01 ?        00:00:01 /root/.pyenv/versions/2.7.9/envs/oet/bin/python2.7 /root/.pyenv/versions/oet/bin/gunicorn oet.wsgi -b 0.0.0.0:8000
  root     129970 129938  0 15:01 ?        00:00:13 /root/.pyenv/versions/2.7.9/envs/oet/bin/python2.7 /root/.pyenv/versions/oet/bin/celery worker -A oet --autoscale=99, 0 -Ofair --loglevel=INFO --logfile=/oet/etc/../data/log/celery_worker%i.log --statedb=/oet/etc/../data/run/celery.state
  root     130022 129969  0 15:01 ?        00:00:02 /root/.pyenv/versions/2.7.9/envs/oet/bin/python2.7 /root/.pyenv/versions/oet/bin/gunicorn oet.wsgi -b 0.0.0.0:8000

监控失败时应该如何排查
------------------------------

监控失败时，可以查看 STATUS 具体给出的错误信息。

一般来说，如果和 AUTH 有关系，则多为权限问题，此时，应确保 OET 可以正确的访问相关的数据库。并根据 MySQL Links，InfluxDB Links，Redis Links，Oracle Links 中的配置，在 OET 所在的机器上，通过实际命令测试能否正常登录相关数据库。

其他出现错误的原因
-----------------------

不严格按照文档操作，产品的放置目录，用户名和密码的配置，Redis / MySQL / InfluxDB 的配置有冲突。
