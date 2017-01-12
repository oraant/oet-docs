在 RedHat 6.5 上安装OET
============================

注意：如没有特殊说明，本文档中所有命令都是通过 root 用户来执行的

TODO：上面的注意用引用块来表示

系统的基本配置
----------------------------

配置本地Yum源
^^^^^^^^^^^^^^^^^^^^^^^

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

关闭防火墙与selinux
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
临时关闭防火墙服务::

  # service iptables stop

防止防火墙开机自启::

  # chkconfig iptables off

  # chkconfig --list iptables
  iptables       	0:off	1:off	2:off	3:off	4:off	5:off	6:off

临时关闭selinux::

  # getenforce 
  Enforcing

  # setenforce 0

  # getenforce 
  Permissive

防止selinux开机自启::

  # grep '^SELINUX=' /etc/selinux/config
  SELINUX=enforcing
  
  # sed -i 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
  
  # grep '^SELINUX=' /etc/selinux/config
  SELINUX=disabled

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
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
您可以手动来操作这一步骤，来了解pyenv的简单用法。或者您也可以跳过这一步骤，直接进行到下一步骤：[通过pip安装Python组件]，在这一步骤中，如果您没有安装过2.7.9，安装脚本会自动帮您安装。

TODO：上面的改为链接

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

进入相关路径下，执行安装脚本。注意，此步骤会做以下操作：

- 通过pyenv安装2.7.9版本的python。
- 将默认的python版本改为2.7.9。
- 建立名为 oet 的虚拟python环境，
- 将依赖的组件安装到此虚拟环境中。

在此过程中，由于编译安装python等步骤需要大量消耗CPU资源，可能会导致系统短暂卡顿几分钟，耐心等待即可。::

  # cd /oet/setup/wheel/
  # ./install.sh 
  === make sure pyenv is installed ===
  system (set by /root/.pyenv/version)
  yes,we do have pyenv
  === make sure python 2.7.9 is installed ===
  pyenv: version `2.7.9' not installed
  no,we don't have python 2.7.9,try to install it.
  Installing Python-2.7.9...
  patching file ./Lib/site.py
  patching file ./Lib/ssl.py
  Installed Python-2.7.9 to /root/.pyenv/versions/2.7.9
  
  keep handling
  ====== check packages ======
  /oet/setup/wheel/packages find,handing...
  ====== create and switch virtualenv ======
  please wait a little minutes
  ..............
  ..............
  ====== install packages ======
  ..............
  ..............
  Successfully installed Django-1.9.7 MySQL-python-1.2.5 amqp-1.4.9 anyjson-0.3.3 billiard-3.3.0.
  23 celery-3.1.24 cx-Oracle-5.2.1 django-markdown-deux-1.0.5 docutils-0.12 gunicorn-19.6.0 influxdb-3.0.0 ipython-2.4.1 kombu-3.0.37 markdown2-2.3.2 meld3-1.0.2 psutil-5.0.0 python-dateutil-2.6.0 pytz-2016.10 redis-2.10.5 requests-2.12.4 six-1.10.0 supervisor-3.3.1 virtualenv-15.1.0python wheels install success
  ====== out of virtualenv ======
  ==== shell job done,please help as behind ====
  
  Please execute the command:"source /root/.bash_profile"
  
  you can test with behind commands."
  exec "pyenv activate oet" command to active virtual environment.
  exec "pip list" command to check how many packages installed.
  exec "pyenv deactivate" command to out virtual environment.

安装完成后，查看安装的组件，验证安装成果::

  [root@mini-server-a wheel]# pip list
  pip (1.5.6)
  setuptools (7.0)
  virtualenv (15.1.0)
  wsgiref (0.1.2)
  
  [root@mini-server-a wheel]# pyenv activate oet
  
  (oet) [root@mini-server-a wheel]# pyenv deactivate  
  amqp (1.4.9)
  anyjson (0.3.3)
  billiard (3.3.0.23)
  celery (3.1.24)
  cx-Oracle (5.2.1)
  Django (1.9.7)
  django-markdown-deux (1.0.5)
  docutils (0.12)
  gunicorn (19.6.0)
  influxdb (3.0.0)
  ipython (2.4.1)
  kombu (3.0.37)
  markdown2 (2.3.2)
  meld3 (1.0.2)
  MySQL-python (1.2.5)
  pip (9.0.1)
  psutil (5.0.0)
  python-dateutil (2.6.0)
  pytz (2016.10)
  redis (2.10.5)
  requests (2.12.4)
  setuptools (28.8.0)
  six (1.10.0)
  supervisor (3.3.1)
  wheel (0.29.0)
  
  (oet) [root@mini-server-a wheel]# pyenv deactivate  

可以看到，所有的包都是安装在了虚拟环境 oet 中，而全局的python 2.7.9的环境没有受到影响。

安装产品
---------------

安装并配置MySQL
^^^^^^^^^^^^^^^^^^^^^^^^

在MySQL数据库中建立django数据库及相关用户::

  # mysql -uroot -p123456
  
  mysql> create database oet character set utf8;
  Query OK, 1 row affected (0.00 sec)
  
  mysql> create user 'django'@'%' identified by 'django';
  Query OK, 0 rows affected (0.02 sec)
  
  mysql> grant all on oet.* to django;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> flush privileges;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql> Bye

使用刚建立的用户，测试是否执行成功::

  # mysql -udjango -pdjango oet
  mysql> 

TODO: 这里很可能无法登陆，需要修改前面初始化mysql的步骤


配置并测试Django
^^^^^^^^^^^^^^^^^^^^^^^^

进入相关目录并激活虚拟环境::

  # cd /oet
  # pyenv activate oet

在数据库中创建相关的表结构并导入数据::

  # ./manage.py migrate
  Operations to perform:
    Apply all migrations: admin, contenttypes, perfmon, auth, sessions
  Running migrations:
    Rendering model states... DONE
    Applying contenttypes.0001_initial... OK
    Applying auth.0001_initial... OK
    Applying admin.0001_initial... OK
    Applying admin.0002_logentry_remove_auto_add... OK
    Applying contenttypes.0002_remove_content_type_name... OK
    Applying auth.0002_alter_permission_name_max_length... OK
    Applying auth.0003_alter_user_email_max_length... OK
    Applying auth.0004_alter_user_username_opts... OK
    Applying auth.0005_alter_user_last_login_null... OK
    Applying auth.0006_require_contenttypes_0002... OK
    Applying auth.0007_alter_validators_add_error_messages... OK
    Applying perfmon.0001_initial... OK
    Applying perfmon.0002_auto_20161130_0723... OK
    Applying perfmon.0003_auto_20161130_0734... OK
    Applying perfmon.0004_auto_20161130_0809... OK
    Applying perfmon.0005_auto_20161207_0706... OK
    Applying perfmon.0006_auto_20161207_0914... OK
    Applying perfmon.0007_auto_20161209_0813... OK
    Applying sessions.0001_initial... OK

  # ./manage.py loaddata perfmon/fixtures/init.json 
  Installed 30 object(s) from 1 fixture(s)

  TODO: 数据里面最起码给一个假的target，否则无法进入主页
  TODO: 数据里要把users中的表中的数据导出来，否则还需要自己

尝试开启django服务::

  # ./manage.py runserver 0.0.0.0:8000
  Performing system checks...
  
  System check identified no issues (0 silenced).
  January 11, 2017 - 07:44:51
  Django version 1.9.7, using settings 'oet.settings'
  Starting development server at http://0.0.0.0:8000/
  Quit the server with CONTROL-C.


打开浏览器访问项目主页，假设如果您的主机IP地址为 192.168.18.128，则可以访问 http://192.168.18.128:8000 来查看效果。

注意，请务必使用chrome或火狐浏览器，如果您的浏览器为360之类的，请把访问模式从兼容模式改为极速模式（一般来说，按钮在地址栏的最右面。如果您不知道如何修改，可以联系我们，或者在网络上搜索相关的内容）

如果您成功的打开了主页，则说明django安装成功。

现在，按下 Ctrl+C 组合键，停止 django 服务。

部署Django项目
^^^^^^^^^^^^^^^^^^^^^^^^

TODO: 是否要创建用户，还是把原来的配置文件中的user改为root?

进入相关目录，开启supervisor服务::

  # cd /oet
  # supervisord

查看supervisor服务开启的状态::

  # bash util/checkproc.sh 
  root         28      2  0 14:23 ?        00:00:00 [sync_supers]
  root      25079      1  0 17:53 ?        00:00:00 /root/.pyenv/versions/2.7.9/envs/oet/bin/python2.7 /root/.pyenv/versions/oet/bin/supervisord
  root      25110  25079  0 17:53 ?        00:00:00 /root/.pyenv/versions/2.7.9/envs/oet/bin/python2.7 /root/.pyenv/versions/oet/bin/gunicorn oet.wsgi -b 0.0.0.0:8000
  root      25120  25110  0 17:53 ?        00:00:00 /root/.pyenv/versions/2.7.9/envs/oet/bin/python2.7 /root/.pyenv/versions/oet/bin/gunicorn oet.wsgi -b 0.0.0.0:8000
  root      25275  25273  0 17:58 pts/0    00:00:00 egrep --color super|celery|gunicorn

将nginx的配置文件移动至相应路径下，并重启nginx服务::
  
  # mv /etc/nginx/conf.d/default.conf  /etc/nginx/conf.d/default.conf.bak
  # cp /oet/setup/nginx/default.conf /etc/nginx/conf.d/

  # service nginx restart
  Stopping nginx:                                            [  OK  ]
  Starting nginx:                                            [  OK  ]

  TODO: 注明如果路径改了，则需要修改相关的配置文件
  TODO: 测试gunicorn把debug模式关闭、静态文件的url拿掉后，应该怎样使其正常运行

检查端口是否开启::

  netstat -tunlp|egrep '8000 |80 '

通过浏览器访问该网址，注意，请务必使用chrome或火狐浏览器，如果您的浏览器为360之类的，请把访问模式从兼容模式改为极速模式（一般来说，按钮在地址栏的最右面。如果您不知道如何修改，可以联系我们，或者在网络上搜索相关的内容）

如果可以正常打开该页面，则说明您已经成功的部署了OET，接下来，请参考使用文档，开始根据用户的环境进行不同的配置。
