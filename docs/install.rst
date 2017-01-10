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
  # tar -xf oet-1.0.0-el6.tar

将解压后的 oet 文件夹放入要安装的路径中，本文档中以 /oet 为例。 然后进入 /oet 路径中，查看路径下的内容::

  # mv oet /
  # cd /oet
  # ls
  collected  common  data  docs  etc  manage.py  oet  perfmon  setup  util

安装Redis
^^^^^^^^^^^^^^

进入相关路径下，执行安装脚本，输入服务器发行版本，安装其rpm包::

  # cd /oet/setup/redis
  # ./install.sh 
  ==== input your system version ====
  if it's RedHat 5, then input 5
  if it's RedHat 6, then input 6
  if it's RedHat 7, then input 7
  please input your redhat release version[5/6/7]: 6
  your choice is 6
  ==== installing rpms ====
  ...........
  ...........
  Installed:
    redis.x86_64 0:2.4.10-1.el6

  Complete!
  rpms install success

开启redis服务，并登陆验证安装是否成功::

  # service redis start
  Starting redis-server:                                     [  OK  ]

  # redis-cli
  redis 127.0.0.1:6379> 

安装Nginx
^^^^^^^^^^^^^^

进入相关路径下，执行安装脚本，输入服务器发行版本，安装其rpm包::

  # cd /oet/setup/nginx
  # ./install.sh 
  ==== input your system version ====
  if it's RedHat 5, then input 5
  if it's RedHat 6, then input 6
  if it's RedHat 7, then input 7
  please input your redhat release version[5/6/7]: 6
  your choice is 6
  ==== installing rpms ====
  ..................
  ..................
  
  Installed:
    nginx.x86_64 0:1.10.0-1.el6.ngx                                                          
  
  Complete!
  rpms install success

开启 nginx 服务，并访问主页，验证安装是否成功::

  # service nginx start
  Starting nginx:                                            [  OK  ]

  # curl localhost 2>/dev/null|grep title
  <title>Welcome to nginx!</title>

安装TICK
^^^^^^^^^^^^^

进入相关路径下，执行安装脚本::

  # cd /oet/setup/tick
  # ./install.sh
  ==== installing rpms ====
  ........
  ........
  Installed:
    chronograf.x86_64 0:0.13.0-1   influxdb.x86_64 0:0.13.0-1   kapacitor.x86_64 0:0.13.1-1  
    telegraf.x86_64 0:0.13.1-1
  
  Complete!
  rpms install success

开启influxdb服务，并使用 influx 命令登陆数据库，验证安装是否成功::

  # service influxdb start
  Starting influxdb...
  influxdb process was started [ OK ]
  
  # influx
  Visit https://enterprise.influxdata.com to register for updates, InfluxDB server
   management, and monitoring.Connected to http://localhost:8086 version 1.1.1
  InfluxDB shell version: 1.1.1
  > 
  > 

安装OIC
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
TODO: 解释 Oracle Instant Client

进入相关路径下，执行安装脚本::

  # cd /oet/setup/oic
  # ./install.sh 
  ==== installing oic rpms to system ====
  find oic files,handing..
  ...............
  ...............
  Installed:
    oracle-instantclient11.2-basic.x86_64 0:11.2.0.4.0-1                          
    oracle-instantclient11.2-devel.x86_64 0:11.2.0.4.0-1                          
    oracle-instantclient11.2-sqlplus.x86_64 0:11.2.0.4.0-1                        
  
  Dependency Installed:
    libaio.x86_64 0:0.3.107-10.el6                                                
  
  Complete!
  basic oic files install success
  ==== echo environments to /root/.bash_profile ====
  echo environments complete.
  ==== shell job done,please help as behind ====
  Please execute the command:"source /root/.bash_profile",and test with "sqlplus" command.

按照提示运行相关命令，使root用户 bash_profile 文件中新添加的环境变量生效，并测试安装结果::

  # source /root/.bash_profile
  # sqlplus -v
  SQL*Plus: Release 11.2.0.4.0 Production

如果您执行 sqlplus -v 时不能立即响应，则需要正确配置 /etc/hosts 文件中的主机名，保持hostname与hosts文件中127.0.0.1对应的名称一致即可，本例中主机名为 mini-server-a ::

  # hostname
  mini-server-a
  
  # vim /etc/hosts
  127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
  ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
  127.0.0.1  mini-server-a

  # sqlplus -v
  SQL*Plus: Release 11.2.0.4.0 Production

确保Python版本大于2.7.9（不支持Python3）
--------------------------------------------------

检查python版本
^^^^^^^^^^^^^^^^^

检查python当前的版本::

  # python --version
  Python 2.6.6

如果您的版本为 2.7.9, 2.7.10, 2.7.11等，且不是python3，可以跳过此步骤，直接安装相关组件

如果发现python版本低于2.7.9，则需要对pyhon进行升级。然而有些系统组件只依赖旧版本。因此，我们采用安装python版本管理器的方式，来管理多个版本的python。

安装pyenv
^^^^^^^^^^^^^^^^^

进入相关路径下，执行安装脚本::

  # cd /oet/setup/pyenv
  # ./install.sh
  ==== installing basic softwares with yum ====
  .................
  .................
  Complete!
  basic install success
  ==== copying basic files to /root/.pyenv ====
  find files,handing..
  basic files copy success
  ==== echo environments to /root/.bash_profile ====
  echo environments complete.
  ==== shell job done,please help as behind ====
  Please execute the command:"source /root/.bash_profile",and test with "pyenv" command.

按照提示运行相关命令，使root用户 bash_profile 文件中新添加的环境变量生效，并测试安装结果::

  # source /root/.bash_profile
  # pyenv -v
  pyenv 20160509

使用pyenv安装python 2.7.9
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
检查操作系统当前有哪些版本的python::

  # pyenv versions
  * system (set by /root/.pyenv/version)

通过安装2.7.9版本的python，这一步骤可能需要耐心等待几分钟::

  # pyenv install 2.7.9
  Installing Python-2.7.9...
  patching file ./Lib/site.py
  patching file ./Lib/ssl.py
  Installed Python-2.7.9 to /root/.pyenv/versions/2.7.9

验证一下是否安装成功，可以看到，版本列表里多了2.7.9，不过系统的默认版本仍然是系统自带版本::

  # pyenv versions
  * system (set by /root/.pyenv/version)
    2.7.9

  # python --version
  Python 2.6.6

通过pyenv，将系统默认的python版本改为2.7.9，并验证效果::

  # pyenv global 2.7.9

  # pyenv versions
    system
  * 2.7.9 (set by /root/.pyenv/version)

  # python --version
  Python 2.7.9

通过pip安装Python组件
-------------------------
TODO: 添加用户的python版本本来就够的情况下，应该怎么做
pip install -r 



安装产品
---------------


在RedHat上安装OET
============================



