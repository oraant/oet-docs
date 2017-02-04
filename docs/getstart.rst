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


尝试启动、停止监控对象的监控任务
-----------------------------------

尝试启动监控对象的监控任务
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
进入 Monitor Targets 的列表页面中，勾选刚才建立的监控对象，从上方的 Action 中选择 [[Create] start monitor tasks for selected processes]，然后点击 Go，留心查看 test 对象的 STATUS 列。

这一操作将会创建一个后台进程，持续不断的从监控对象中抽取数据。

查看监控对象的监控情况
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
等待10秒后，请刷新 Monitor Targets 的列表页面，留心查看 test 对象的 STATUS 列。

其中不同的效果含义如下：

===========  ================================
状态         含义
===========  ================================
No Task      此监控对象没有开启抓取任务
PENDING      任务已被挂起或无法获知状态
STARTED      正在尝试启动抓取任务
RUNNING      正在抓取数据
RETRY        抓取数据时出现异常，正在等待重试
FAILURE      抓取数据失败，详情请看日志
SUCCESS      抓取任务成功结束
===========  ================================

尝试停止监控对象的监控状态
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
进入 Monitor Targets 的列表页面中，勾选刚才建立的监控对象，从上方的 Action 中选择 [[Revoke] stop monitor tasks for selected processes]，然后点击 Go，留心查看 test 对象的 STATUS 列。

这一操作将会尝试停止后台抓取数据的进程。如果成功结束，STATUS 将会变为 SUCCESS。

尝试清空监控对象的任务状态
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
进入 Monitor Targets 的列表页面中，勾选刚才建立的监控对象，从上方的 Action 中选择 [[Clear] stop trace task's status for selected processes]，然后点击 Go，留心查看 test 对象的 STATUS 列。

这一操作将会停止对已有任务状态的追踪，主要用于用户的不正当操作引发的无法重启、无法停止任务的情况。在停止监控该任务前，请务必对其尝试 Revoke 操作，防止有多个任务同时运行的情况。

尝试使用产品
---------------

退出后台
^^^^^^^^^^^^^^^^^^^^^^
点击右上角的 VIEW SITE ，或者直接访问 http://ip ，查看产品效果

首页 - 仪表盘
^^^^^^^^^^^^^^^^^
这个页面的主要目的是展示当前监控对象的整体情况：如一共多少个监控对象，正在同步多少张表等等。

在任务状态里面，列出了当前所有的监控对象及监控状态。

在左上角，是当前监控对象的图标及状态。你可以通过点击不同的监控对象来切换当前展示的内容，或者通过右上角 切换数据库 来实现这一操作。

首页 - 服务器信息
^^^^^^^^^^^^^^^^^^^
这个页面的主要目的是展示监控服务器当前的性能情况，如CPU使用率，内存使用率，磁盘使用率等。你可以通过这个页面，方便的了解服务器当前的状态。

性能趋势图的查看与操作
^^^^^^^^^^^^^^^^^^^^^^^^^^
这个页面的主要目的，是展示监控对象各个性能参数的实时趋势。左侧是参数的分类及具体参数，右侧则是具体的图表。

**请注意查看你的左上角选择的监控对象是哪一个**，如果你想查看其它 监控对象的信息，请点击 切换数据库。

在趋势图中，你可以通过 Zoom 中的按钮来选择查看的时间范围，或通过右侧的时间选择器来选择具体的时间段。

除此之外，通过鼠标在图片上点划，或移动下方滑块，或调整滑块边界，都是非常高效的时间选择方式。

在趋势图的右上角有四个按钮，分别对应着四个功能：

- 查看本参数的简介，
- 刷新当前的数据，
- 临时删掉这个趋势图，
- 最小化这个趋势图，

**其中，“查看本参数的简介”这一功能你应该格外留心使用**，这可以使你在最短的时间内迅速的了解这个参数相关的内容。

值得注意的是，你会在图片中发现一些间隙。这很可能是服务器由于某次停机维护导致无法抓取数据。你要留意的是，OET并不会将这些空白时间展示为大段的空白，反应到图片上只会是一个简单的断裂。

更多
^^^^^^^^^^

更多 - 进入后台，可以使你进入到后台管理系统中

更多 - 帮助文档，可以使你进入到本文档中

更多 - 访问官网，可以使你进入到本公司的官方网站中
