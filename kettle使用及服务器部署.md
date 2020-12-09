# kettle简介

## ETL

ETL（extract-transform-load）工具是用来对数据进行抽取、转换、加载的，kettle是一款开源的ETL工具，由java编写。

### kettle分为三个主要功能：

windows端：

- spoon.bat
- pan.bat
- kitchen.bat

linux端：

- spoon.sh
- pan.sh
- kitchen.sh

其中spoon是图形化操作界面，便于定义转换（.ktr）以及作业（.kjb），pan是用来执行转换文件（.ktr）的，kitchen是用来执行（.kjb）文件的。

### 下载及安装：

下载pdi-ce-7.1.0.0-12文件后直接解压，即可运行相关脚本文件，无需安装。

### 配置：

下载mysql驱动，放入解压目录下的lib子目录下。

# kettle使用（spoon图形界面）

## 转换

kettle的转换包含非常多的功能，目前只涉及到了输入、输出、记录排序及合并、数据同步这几种。

转换主要是定义输入表、输出表，以及数据转换的一系列规则。

转换的文件后缀为.ktr

下面以company表的同步为例

<img src=".\img\kettle-transform-graph.png" style="zoom: 80%;" />

其中source和target分别表示源数据表和待同步的数据表，后面跟的是表名，sort是对查到的记录进行排序，然后将两张表的排序好的记录进行合并，最后的merge是进行数据同步。

定义表输入的时候需要指定数据库连接以及对应的表，如下图所示。

<img src=".\img\kettle-table.png" style="zoom:80%;" />

如果没有已有的连接需要新建连接，新建连接的方式和navicat差不多。

记录排序需要指定排序字段，一般都为主键id。

<img src=".\img\kettle-sort.png" style="zoom:80%;" />

记录合并的时候需要指定旧数据源和新数据源，分别对应待更新的表和源数据表，标志字段在后面数据同步的步骤会用到，下面匹配关键字为主键id，数据字段为要更新的字段。

<img src=".\img\kettle-merge-record.png" style="zoom:80%;" />

数据同步步骤需要定义数据库连接及目标数据表，就是待同步的target表

<img src=".\img\kettle-merge-table1.png" style="zoom:80%;" />

下面的字段也是匹配字段和更新字段

<img src=".\img\kettle-merge-table2.png" style="zoom:80%;" />

在高级栏需要定义同步更新的规则，其中flagfield字段为记录合并中的flagfield。

<img src=".\img\kettle-merge-table3.png" style="zoom:80%;" />

## 作业

一个作业中可以包含多个转换，作业可以实现定时执行的功能。

作业的后缀为.kjb

作业的简单流程图定义如下：

<img src=".\img\kettle-job.png" style="zoom:80%;" />

定义定时执行在start中设置：

<img src=".\img\kettle-job-time.png" style="zoom:80%;" />

# kettle的服务器部署

上传压缩文件到linux的kettle目录下，解压，驱动除了放在lib目录下，还需要放在libswt\linux\x86_64目录下。

将windows端保存的转换.ktr文件和作业.kjb文件上传至服务器的transform-file目录下。

查看data-integration目录下的.sh文件是否有执行权限，没有的话修改成可执行文件。

vim 打开.kjb文件，找到filename，修改.ktr文件的路径，建议使用绝对路径。

执行nohup kitchen.sh -file=xx.kjb -logfile=xx.log

nohup表示后台执行进程，用户退出登录了服务器也保持进程的执行状态。

在/root/kettle目录下有run.sh执行脚本文件，服务器运行时直接运行此脚本文件即可。