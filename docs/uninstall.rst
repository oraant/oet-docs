从RedHat6.5上卸载OET
=======================

关闭进程，退出python虚拟环境等
---------------------------------------

关闭产品相关进程::

  # cd /oet/
  # supervisorctl shutdown

关闭相关服务::

  # service mysqld stop
  # service redis stop
  # service nginx stop
  # service influxdb stop

删除不需要的软件
-------------------------

通过yum卸载相关软件::

  yum remove -y redis nginx telegraf influxdb chronograf kapacitor oracle-instantclient11.2-basic oracle-instantclient11.2-sqlplus oracle-instantclient11.2-devel

删除pyenv软件::

  rm -rf /root/.pyenv

还原 bash profile 的配置::

  sed -i '/OICCODEFORSEDDELETE/d' /root/.bash_profile

还原配置后，退出当前终端，然后重新登录一次::

  exit

如果您不需要，也可以卸载 MySQL::

  yum remove -y mysql-server

删除软件产品
--------------------

直接删除相关目录::

  # rm -rf /oet

至此，产品删除成功
