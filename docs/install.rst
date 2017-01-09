检查配置要求
==================

推荐配置：

======  =============
配置    要求
======  =============
CPU     asdfasdf
内存    asdfasdfasdf
网卡    asdfasdfasdf
======  =============

最低配置：

======  =============
配置    要求
======  =============
CPU     asdfasdf
内存    asdfasdfasdf
网卡    asdfasdfasdf
======  =============

磁盘空间：

+--------------+--------------+----------+
| 监控节点数量 | 数据保存时间 |   空间   |
+==============+==============+==========+
| 1～10个      | 3个月        | 500M     |
+--------------+--------------+----------+
| 11～50个     | 3个月        | 3G       |
+--------------+--------------+----------+
| 51～100个    | 3个月        | 50G      |
+--------------+--------------+----------+
| 1～10个      | 9个月        | 1.5G     |
+--------------+--------------+----------+
| 11～50个     | 9个月        | 9G       |
+--------------+--------------+----------+
| 51～100个    | 9个月        | 150G     |
+--------------+--------------+----------+

TODO：完善信息


在 RedHat 6.5 上安装OET
============================

注意：如没有特殊说明，本文档中所有命令都是通过 root 用户来执行的

TODO：上面的注意用引用块来表示

配置本地Yum源
----------------------------

进入yum源配置文件所在的目录::

  # cd /etc/yum.repos.d

创建备份文件夹，将不需要的配置文件移动到备份文件夹内::

  # mkdir bak
  # mv * bak

建立新的配置文件，配置好本地yum源::

  # touch local-source.repo

向配置文件中添加如下内容并保存::

  [local-source]
  name=server
  baseurl=file:///mnt
  enabled=1
  gpgcheck=0

挂载光盘，并 mount 至 /mnt 路径下::

  # mount -o ro /dev/cdrom /mnt

通过yum命令，测试是否配置成功::

  # yum clean all
  # yum list mysql*

如果看到类似下面的输出，则说明配置成功::

  .........
  Available Packages
  MySQL-python.x86_64           1.2.3-0.3.c1.1.el6   local-source
  mysql.x86_64                  5.1.71-1.el6         local-source
  mysql-bench.x86_64            5.1.71-1.el6         local-source
  mysql-connector-java.noarch   1:5.1.17-6.el6       local-source
  mysql-connector-odbc.x86_64   5.1.5r1144-7.el6     local-source
  mysql-devel.i686              5.1.71-1.el6         local-source
  mysql-devel.x86_64            5.1.71-1.el6         local-source
  mysql-libs.i686               5.1.71-1.el6         local-source
  mysql-server.x86_64           5.1.71-1.el6         local-source
  mysql-test.x86_64             5.1.71-1.el6         local-source
  .........

安装并配置MySQL
-------------------------

使用yum快速安装MySQL::

  # yum install mysql mysql-server mysql-devel -y

初始化并启动MySQL::

  # service mysqld start

为MySQL的超级用户 root 设置密码，本例中密码为123456，读者可以自行修改::

  # mysqladmin -u root password 123456

使用root用户登录MySQL，如果成功登录，说明配置成功::

  # mysql -uroot -p123456 -s
  mysql> 

使用yum快速安装所需的包
-------------------------

上传并解压 oet 包
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

建立路径，用以存放程序相关文件，本文档中以 /software 为例::

  # mkdir /software

使用SSH或FTP工具，将文件上传到 /software 路径中，并解压释放相关文件::

  # cd /software
  # tar -xf tar -xf oet-1.0.0-rhel6.tar

将解压后的 oet 文件夹放入要安装的路径中，本文档中以 /oet 为例::

  # mv oet /oet

进入 /oet 路径，查看路径下的内容::

  # cd /oet
  # ls
  collected  common  data  docs  etc  manage.py  oet  perfmon  setup  util

安装Redis
^^^^^^^^^^^^^^



安装Influx TICK
-------------------------


安装Oracle Instant Client
----------------------------


确保Python版本大于2.7.9
--------------------------------------------------
**注意！Python2.7.9以上的版本不包括Python3！OET为了兼容厂商的旧服务器，是运行在Python2的环境下的。**

检查python版本
^^^^^^^^^^^^^^^^^

安装pyenv
^^^^^^^^^^^^^^^^^
`下载离线安装包 <http://www.baidu.com>`_

使用pyenv安装python 2.7.11
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


通过pip安装Python组件
-------------------------
pip install -r 



安装产品
---------------


在RedHat上安装OET
============================



