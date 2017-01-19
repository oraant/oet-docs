开始使用
=========

访问后台管理面板（右上角"更多" --> "进入后台"），或者直接访问 http://ip/admin ，修改相关配置

创建用户
----------

点击 Users 右面的 Add，添加，为用户添加一个专属的管理用户，内容填写如下：

- Username: admin
- Password: oetoetoet
- Password confirmation: oetoetoet

点击 Save and continue editing ，保存这个用户，并配置相关权限：

- 在 Permissions 中，勾选 Active 和 Staff status 两个选项
- 在 User permissions 中，选择相关权限（见下表）

需要赋予的权限如下：（小技巧——你可以通过在左侧的搜索栏通过 Link 和 Target 两个单词来快速筛选出这些权限）

- perfmon | Influx Link | Can add Influx Link
- perfmon | Influx Link | Can change Influx Link
- perfmon | Influx Link | Can delete Influx Link
- perfmon | Mysql Link | Can add Mysql Link
- perfmon | Mysql Link | Can change Mysql Link
- perfmon | Mysql Link | Can delete Mysql Link
- perfmon | Oracle Link | Can add Oracle Link
- perfmon | Oracle Link | Can change Oracle Link
- perfmon | Oracle Link | Can delete Oracle Link
- perfmon | Redis Link | Can add Redis Link
- perfmon | Redis Link | Can change Redis Link
- perfmon | Redis Link | Can delete Redis Link
- perfmon | Monitor Target | Can add Monitor Target
- perfmon | Monitor Target | Can change Monitor Target
- perfmon | Monitor Target | Can delete Monitor Target

点击 Save 按钮，保存新建的用户，并点击右上角的 Logout，退出登录后，点击 Log in again，使用新建立的用户测试是否配置成功

- Username: admin
- Password: oetoetoet

登录后，若成功显示如下表格，则说明配置成功：

- 1.Monitor Targets	Add  Change
- 2.___ Oracle Links	Add  Change
- 3.___ Mysql Links	Add  Change
- 4.___ Redis Links	Add  Change
- 5.___ Influx Links	Add  Change

配置本地链接库中的内容
---------------------------

在以下四个表中，分别存储了OET需要用到的中间数据库的链接，请正确填写。为方便使用，OET内置了几个链接，请酌情修改或增删。

- Oracle Links: 存储了到达 Oracle 目标数据库的监听信息。
- Mysql Links: 用来缓存数据的 MySQL 数据库信息，一般是安装在本地的，如果是按照此文档安装的，则已经在本地安装完毕。
- Redis Links: 用来做 broker 的 Redis 信息，一般是安装在本地的，如果是按照此文档安装的，则已经在本地安装完毕。
- Influx Links: 用来存储归档数据的 InfluxDB 信息，一般是安装在本地的，如果是按照此文档安装的，则已经在本地安装完毕。

如果是按照前文的文档安装的，一般只需要改动 Oracle Links ，添加要监控的数据库的监听信息即可。

在 Monitor Targets 中，增加要监控的数据库信息。

=====================  ====================================
选项名字               选择如何添加
=====================  ====================================
Name                   为此监控对象起个名字
Desc                   添加描述
Reborn                 连接失败后隔多长时间自动重试
Scheme                 监控哪些参数
Oracle listener        选择要连接的 oracle 监听
Oracle service name    选择 Oracle 的服务名
Mysql server           选择要使用的 MySQL Server
Mysql database         选择要使用哪个库来缓存数据
Redis server           选择要使用的 Redis Server
Redis database         选择要使用哪个库来交互数据
Influx server          选择要使用的 Influx Server
Influx database        选择要使用哪个库来归档数据
=====================  ====================================

点击 Save 按钮，保存新建的监控对象。


尝试启动监控对象
-------------------------
进入 Monitor Targets 的列表页面中，勾选刚才建立的监控对象，从上方的 Action 中选择 [[Create] start monitor tasks for selected processes]，然后点击 Go，留心查看 test 对象的 STATUS 列。

等待10秒后，请刷新页面，再次查看变化。不同的效果含义如下：

===========  ===========================
状态         含义
===========  ===========================
No Task
PENDING
RUNNING
===========  ===========================

TODO: 完善这个表格
