# MySQL8新特性与高可用集群架构

# 一、MySQL8克隆复制

## 1、Clone Plugin克隆

8.0.17引入clone plugin，允许从本地或者远程的MySQL中克隆数据。克隆的数据包括schema、表、表空间、元数据等等，克隆的数据是一个完整的数据目录，插件可以使用克隆的目录配置和恢复一个MySQL Server，克隆分为本地克隆和远程克隆。

作用：① 数据文件备份（物理备份） ② 用于故障恢复

https://dev.mysql.com/doc/refman/8.0/en/clone-plugin.html

本地克隆：将启动克隆操作的MySQL克隆到该主机的一个指定目录下。

![1280X1280](1280X1280.PNG)

远程克隆：涉及到启动克隆操作的本地MySQL称为recipient（数据接收方），远端的源数据MySQL称为donor（捐赠者）。通过网络传输的方式将donor数据克隆到recipient的指定目录。如果不指定则会将接收方数据目录中的所有数据替换为克隆的数据。

![1280X1280 (1)](1280X1280 (1).PNG)

**插件安装**

克隆插件和其他插件一样，都位于mysql/lib/plugin下，命名为mysql_clone.so，安装方式也一样，可以通过配置文件也可以在线安装。

第一种方式：my.cnf配置文件加载：

```PowerShell
[mysqld]
在mysqld标签尾部追加如下内容：
plugin-load-add=mysql_clone.so
clone=FORCE_PLUS_PERMANENT  #启动时加载插件并防止它在运行时被删除

配置完成后，使用systemctl重启mysqld服务
systemctl restart mysqld
```

配置完成后，要重启mysql服务。

第二种方式：启动时加载（了解即可，推荐my.cnf修改）：

```PowerShell
mysqld_safe --defaults-file=/etc/my.cnf --plugin-load-add=mysql_clone.so &
```

第三种方式：在线加载（了解即可，推荐my.cnf修改）：

```PowerShell
mysql> INSTALL PLUGIN clone SONAME 'mysql_clone.so';
```

检查插件是否加载成功

```PowerShell
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS WHERE PLUGIN_NAME = 'clone';
+-------------+---------------+
| PLUGIN_NAME | PLUGIN_STATUS |
+-------------+---------------+
| clone    | ACTIVE     |
+-------------+---------------+
```

如果想要卸载或者重新加载 Clone 插件，可使用下方命令：

```SQL
mysql> uninstall plugin clone;
```

## 2、本地克隆

![1280X1280 (2)](1280X1280 (2).PNG)

本地 Clone 命令的语法如下：

```SQL
CLONE LOCAL DATA DIRECTORY [=] 'clone_dir';
```

本地克隆实现

第一步：创建克隆专用的用户

```SQL
-- 创建用户
create user backup_clone@'127.0.0.1' identified by '123';

-- 授予本地克隆权限
GRANT BACKUP_ADMIN ON *.* TO backup_clone@'127.0.0.1';

-- 授予克隆信息查看权限
GRANT SELECT ON performance_schema.clone_status TO backup_clone@'127.0.0.1';
```

第二步：创建克隆目录

```PowerShell
mkdir -p /data/clone_bak
chown -R mysql:mysql /data/clone_bak
```

第三步：执行克隆命令

```PowerShell
mysql> CLONE LOCAL DATA DIRECTORY = '/data/clone_bak/20250505';
```

这里的 `/data/clone_bak/20250505` 是克隆目录，它需要满足 3 个要求：

- 克隆目录必须是绝对路径。
- 其中 /data/clone_bak 必须存在，且 MySQL 对其有写权限。
- 最后一级目录 20250505 不能存在。

第四步：查看克隆目录的内容

```PowerShell
ll /data/clone_bak/20250505 

# 输出：
总用量 570376
drwxr-x--- 2 mysql mysql        89 11月  6 11:27 #clone
-rw-r----- 1 mysql mysql      4373 11月  6 11:27 ib_buffer_pool
-rw-r----- 1 mysql mysql 524288000 11月  6 11:27 ibdata1
drwxr-x--- 2 mysql mysql        23 11月  6 11:27 #innodb_redo
drwxr-x--- 2 mysql mysql         6 11月  6 11:27 mysql
-rw-r----- 1 mysql mysql  26214400 11月  6 11:27 mysql.ibd
drwxr-x--- 2 mysql mysql        28 11月  6 11:27 sys
-rw-r----- 1 mysql mysql  16777216 11月  6 11:27 undo_001
-rw-r----- 1 mysql mysql  16777216 11月  6 11:27 undo_002
```

可以直接基于这些数据文件启动 MySQL 实例，相较于 Xtrabackup 克隆插件无须 Prepare 阶段。

## 3、远程克隆

作用？答：对远程数据库服务器中的数据进行备份。

远程克隆的原理可参考下图，克隆角色分为接收者 (recipient) 与捐赠者 (donor)，默认情况下使用远程克隆会删除 “接收者” 数据目录中的数据，替换为 “捐赠者” 的克隆数据。当然也可以选择将克隆的数据分配到 “接收者” 的其它目录，避免删除 “接收者” 现有的数据。

![425ebd79-f49a-43c7-aa4b-85befaa0cc3e](425ebd79-f49a-43c7-aa4b-85befaa0cc3e.png)

远程克隆的语法如下：

```PowerShell
CLONE INSTANCE FROM 'user'@'host':port
IDENTIFIED BY 'password'
[DATA DIRECTORY [=] 'clone_dir']
[REQUIRE [NO] SSL];
```

参数含义介绍：

- user：登陆捐赠者实例的用户名。
- host：捐赠者实例的主机名或 IP。
- port：捐赠者实例的端口。
- password：捐赠者实例的密码。
- clone_dir：不指定 clone_dir 时，会清空接收者实例的 datadir 目录，并将数据放到 datadir 指定路径。如果指定了 data directory，则该路径需要不存在，mysql 服务需要有目录权限。
- REQUIRE [NO] SSL：指定传输数据是是否使用加密协议。

下面将进行一个具体演示，操作环境介绍：

| 主机           | 版本   | 角色   | Clone Pluge |
| -------------- | ------ | ------ | ----------- |
| 192.168.88.101 | 8.0.40 | 捐赠者 | ACTIVE      |
| 192.168.88.102 | 8.0.40 | 接收者 | ACTIVE      |

第一步：在捐赠者（88.101）实例上创建相关账号并授权

```PowerShell
CREATE USER 'donor_user'@'%' IDENTIFIED BY '123';
GRANT BACKUP_ADMIN on *.* to 'donor_user'@'%';
```

第二步：在接收者（88.102）实例上创建账号并授权

```PowerShell
CREATE USER 'recipient_user'@'%' IDENTIFIED BY '123';
GRANT CLONE_ADMIN on *.* to 'recipient_user'@'%';
```

这里 CLONE_ADMIN 权限，隐含有 BACKUP_ADMIN 和 SHUTDOWN 重启实例权限。

第三步：在接收者（88.102）实例上设置捐赠者白名单

开启88.102这台机器上的克隆插件

```PowerShell
vim /etc/my.cnf
[mysqld]
尾部追加
plugin-load-add=mysql_clone.so
clone=FORCE_PLUS_PERMANENT

以上设置完成后，使用systemctl重启mysql
systemctl restart mysqld
```

回到MySQL终端：

```PowerShell
SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS WHERE PLUGIN_NAME = 'clone';
+-------------+---------------+
| PLUGIN_NAME | PLUGIN_STATUS |
+-------------+---------------+
| clone    | ACTIVE     |
+-------------+---------------+

# 配置白名单
SET GLOBAL clone_valid_donor_list = '192.168.88.101:3306';
```

接收者只能克隆白名单列表内捐赠者的数据，如果有多个实例使用逗号分隔。

第四步：在接收者实例上发起远程克隆

在Linux终端提前创建目录并授权

```PowerShell
mkdir -p /data/clone_bak
chown -R mysql.mysql /data/clone_bak
```

回到MySQL终端，执行克隆操作：

```PowerShell
CLONE INSTANCE FROM 'donor_user'@'192.168.88.101':3306 IDENTIFIED BY '123'  data directory='/data/clone_bak/20250505';
```

获取备份锁（backup lock）备份锁与 DDL 互斥。捐赠者与接收者两个节点的备份锁都要获取。远程克隆结束后，会重启接收者节点（如果重启失败了，没有关系，手工重启MySQL即可）。如果克隆命令指定克隆目录 `DATA DIRECTORY` 则不会重启。

小结：

远程克隆需要有两个角色：（捐赠者）和（接收者）

远程克隆应用场景：（备份）

## 4、克隆任务监控（扩展）

> 作用？答：如果数据量比较大的情况，克隆任务时间会执行比较长，这个时候可以对正在执行的克隆任务进行监控！
>
> MySQL 提供两张表，监控克隆任务及查看任务状态。分别是 performance_schema 下的 clone_status 和clone_progress 表。
>
> 首先看看 clone_status 表，该表记录了克隆操作的状态信息。
>
> ```PowerShell
> select * from performance_schema.clone_status\G
> *************************** 1. row ***************************
>              ID: 1
>             PID: 0
>           STATE: Completed
>      BEGIN_TIME: 2024-11-06 14:57:26.647
>        END_TIME: 2024-11-06 14:57:39.406
>          SOURCE: 192.168.88.101:3306
>     DESTINATION: LOCAL INSTANCE
>        ERROR_NO: 0
>   ERROR_MESSAGE: 
>     BINLOG_FILE: mysql-bin.000001
> BINLOG_POSITION: 3185
>   GTID_EXECUTED: 1b03028c-76f7-11ee-ac46-faa7cd9c6a00:1-4,
> eccc6b43-b0fc-11ed-8e74-fa0e3cc40b00:1-3
> 
> 其中各字段含义如下：
> ID：任务 ID
> PID：对应 show processlist 中的 ID，如果要终止克隆任务，可以执行 KILL QUERY processlist_id
> STATE：克隆操作的状态，包括 Not Started（尚未开始）In Progress（进行中）Completed（成功）Failed（失败）
> BEGIN_TIME：克隆任务开始时间
> END_TIME：克隆任务结束时间
> SOURCE：Donor 实例的地址
> DESTINATION：克隆目录
> BINLOG_FILE & BINLOG_POSITION & GTID_EXECUTED：克隆操作对应的一致性位点信息，可利用这些信息搭建从库
> ERROR_MESSAGE：如果任务失败，该字段会显示报错内容
> ```
>
> 接下来查看 clone_progress 表，该表记录克隆任务的进度信息。
>
> ```PowerShell
> select * from performance_schema.clone_progress;
> ```
>
> | ID   | STAGE     | STATE     | BEGIN_TIME                 | END_TIME                   | THREADS | ESTIMATE  | DATA      | NETWORK   | DATA_SPEED | NETWORK_SPEED |
> | ---- | --------- | --------- | -------------------------- | -------------------------- | ------- | --------- | --------- | --------- | ---------- | ------------- |
> | 1    | DROP DATA | Completed | 2024-11-06 14:57:26.673385 | 2024-11-06 14:57:26.891470 | 1       | 0         | 0         | 0         | 0          | 0             |
> | 1    | FILE COPY | Completed | 2024-11-06 14:57:26.891645 | 2024-11-06 14:57:31.875850 | 1       | 583241456 | 583241456 | 583281243 | 0          | 0             |
> | 1    | PAGE COPY | Completed | 2024-11-06 14:57:31.876114 | 2024-11-06 14:57:31.882372 | 1       | 0         | 0         | 99        | 0          | 0             |
> | 1    | REDO COPY | Completed | 2024-11-06 14:57:31.882550 | 2024-11-06 14:57:31.885697 | 1       | 2560      | 2560      | 2901      | 0          | 0             |
> | 1    | FILE SYNC | Completed | 2024-11-06 14:57:31.885818 | 2024-11-06 14:57:32.775465 | 1       | 0         | 0         | 0         | 0          | 0             |
> | 1    | RESTART   | Completed | 2024-11-06 14:57:32.775465 | 2024-11-06 14:57:37.769087 | 0       | 0         | 0         | 0         | 0          | 0             |
> | 1    | RECOVERY  | Completed | 2024-11-06 14:57:37.769087 | 2024-11-06 14:57:39.405806 | 0       | 0         | 0         | 0         | 0          | 0             |
>
> 其中各字段含义如下：
>
> - STAGE：一个克隆任务有 7 个阶段，分别是 DROP DATA、FILE COPY、PAGE COPY、REDO COPY、FILE SYNC、RESTART、RESTART 当前阶段结束后才会开始进入下一阶段。
> - STATE：当前阶段状态。
> - BEGIN_TIME & END_TIME：当前阶段的开始时间和结束时间。
> - THREADS：当前阶段使用的并发线程数。
> - ESTIMATE：预估数据量。
> - DATA：已拷贝的数据量。
> - NETWORK：通过网络传输的数据量。
> - DATA_SPEED：当前任务拷贝的速率。
> - NETWORK_SPEED：当前任务网络传输的速率。
>
> > 网络传输和带宽息息相关，100M带宽（理论）=> 100/8 = 实际传输速率（理论值）

## 5、克隆插件实现（理解 => 原理类似Xtrabackup）

作用：让我们更好理解克隆实现原理

![e0405326-a2f2-41ec-a5eb-7ec6313175ee](e0405326-a2f2-41ec-a5eb-7ec6313175ee.png)

克隆插件可以细分 5 个阶段：

#### 5.1 Init 阶段

初始化一个克隆对象（一个克隆线程）。

#### 5.2 File Copy

拷贝数据文件。在拷贝之前，会将当前的检查点 LSN 记为 CLONE START LSN，同时启动 Page Tracking。

Page Tracking 会跟踪 CLONE START LSN 之后发生修改的页，记录这些页面的元数据信息 tablespace ID 和 page ID。数据文件拷贝结束后，会将当前检查点的 LSN 记为 CLONE FILE END LSN。

File Copy 期间对源文件所有的改动，都会被 Page Tracking 记录，将在 Page Copy 阶段进行 “覆盖” 订正处理。所以不用担心 Copy 期间源文件发生改变，出现数据文件内部不一致的问题。

> 就是把数据文件备份，备份和文件大小有关，数据量越大，越耗时（2小时）

#### 5.3 Page Copy

该阶段的主要目的是订正覆盖 FILE COPY 阶段，源文件有改动的地方，相当于处理 FILE COPY 阶段的增量数据。执行拷贝之前，会基于 tablespace ID 和 page ID 对这些页进行排序，以避免 PAGE COPY 过程中的随机读写。

因为对数据文件的拷贝已经结束，那 PAGE COPY 阶段的增量数据，将通过归档 redo log 来处理。

所以，在 PAGE COPY 阶段启动前，会开启 Redo Archiving 归档线程，将 redo log 的内容按块拷贝到归档文件中。通常来讲，归档线程的拷贝速度会快于 redo log 的生成速度。即便 redo log 生成速度要快于归档线程，在写入 redo log 时，也会等待归档线程完成拷贝，不会覆盖还未拷贝的 redo log。

Page Tacking 中的页面拷贝完成后，会获取实例的一致性位点信息，停止 Redo Archiving 同时将此时的 LSN 记为 CLONE LSN。

> 在文件备份阶段（2小时），数据页也会随着改变（增删改），文件克隆后，也要克隆Page（这阶段改变的数据页）=> （0.5小时）

#### 5.4 Redo Copy

拷贝归档文件中 CLONE FILE END LSN 与 CLONE LSN 之间的 redo log，通过重用归档 redo log 就可以将数据库恢复到一个一致的时间点 Clone LSN，也就是停止 Redo Archiving 时的那一刻。

> 把Page Copy（0.5小时）这个阶段的增删改，写入到redo log，然后执行与备份文件合并！

#### 5.5 Done

调用 snashot_end() 销毁克隆对象。

克隆插件图解演示：

![d46ff714-9a05-4532-bbc7-1712ed11f578](d46ff714-9a05-4532-bbc7-1712ed11f578.png)

## 6、克隆技术限制条件

在使用 Clone 插件时，需注意有如下限制：

① 克隆期间，会堵塞 DDL 同样 DDL 也会堵塞克隆命令的执行。不过从 MySQL 8.0.27 开始，克隆命令就不会堵塞捐赠者上的 DDL 了。

② 克隆插件只会拷贝 innodb 引擎表中的数据，对于其他存储引擎，只会拷贝表结构。

③ 克隆插件不会拷贝配置参数和 Binlog。

④ 捐赠者和接受者的版本需要保持一致。不仅大版本要一样，小版本也要一样。可使用 show variables 命令查看版本。

⑤ 远程克隆，主机操作系统和位数必须一致，可通过 version_compile_os 与 version_compile_machine 查看。

⑥ 捐赠者和接收者都需要安装克隆插件。

⑦ 捐赠者和接受者字符集需要一样，可通过 character_set_server 与 collation_server 查看。

⑧ 捐赠者和接受者的参数 innodb_page_size 与 innodb_data_file_path 需要一样。

⑨ 默认情况下，远程 clone 会在完成数据 clone 后，关闭接受者实例。需要有控制进程（如 mysqld_safe 脚本、systemctl 等）来拉起接受者实例。如果缺少控制进程，则接受者实例关闭后，无法自动启动，需要手动拉起。

⑩ 无论是捐赠者还是接收者，同一时间只能执行一个克隆操作。

小结：

本地克隆 还是 远程克隆都是为了实现MySQL的（备份）操作

另外克隆操作只能克隆InnoDB引擎表中的结构和数据，其他引擎只负责表结构复制！

# 二、MGR组复制（MySQL高可用集群）

作用：

最早期高可用架构都是基于MHA设计的，主要针对MySQL5.5 ~ MySQL5.7，但是由于MHA官网很久没有更新，导致没有办法针对MySQL8 + MHA实现高可用。

MySQL8.0以后版本，除了引入克隆复制以外，还引入了MGR（组复制），实际上：MySQL5.7也支持MGR组复制，提供了MySQL高可用架构解决方案。

![3b6508b4-15d3-4235-af11-2704e97c8e9b](3b6508b4-15d3-4235-af11-2704e97c8e9b.png)

## 1、场景说明

master：1、2、3、4、5

slave：1、2、3

在准备同步4、5过程中，master突然宕机了！

由于传统异步复制的缺陷，可能会导致主从数据不一致的问题，在主节点异常宕机时从节点可能造成数据丢失。基于这个缺陷，Mysql5.7.17推出了一个高可用与高扩展的解决方案Mysql Group Replication(简称MGR)，将原有的gtid复制功能进行了增强，支持单主模式和多主模式。组复制在数据库层面上做到了只要集群中大多数主机可用，则服务可用，也就是说3台服务器的集群，允许其中1台宕机。

![6a2b9f6d-2abd-41f0-a0fa-d955cd2ac884](6a2b9f6d-2abd-41f0-a0fa-d955cd2ac884.png)

## 2、MGR特点

① 高一致性，基于原生复制及 paxos协议 组复制技术 ，并以插件的方式提供，提供一致数据安全保证；

② 高容错性，只要不是大多数节点坏掉就可以继续工作，有自动检测机制，当不同节点产生资源争用冲突时，不会出现错误，按照先到者优先原则进行处理，并且内置了自动化脑裂防护机制；

③ 高扩展性，节点的新增和移除都是自动的，新节点加入后，会自动从其他节点上同步状态，直到 新节点和其他节点保持一致，如果某节点被移除了，其他节点自动更新组信息，自动维护新的组信息；

④ 高灵活性，有单主模式和多主模式，单主模式下，会自动选主，所有更新操作都在主上进行；多主模式下，所有 server 都可以同时处理更新操作。

MGR 是 MySQL 数据库未来发展的一个重要方向。

小结：高一致性，高容错性，高扩展性，高灵活性

## 3、MGR结构需求

1）引擎必须为 innodb，因为需事务支持在 commit 时对各节点进行冲突检查

2）每个表必须有主键，在进行事务冲突检测时需要利用主键值对比

3）必须开启 binlog 且为 row 格式

4）开启 GTID，且主从状态信息存于表中（--master-info-repository=TABLE 、--relay-log-info-repository=TABLE），--log-slave-updates 打开

5）一致性检测设置--transaction-write-set-extraction=XXHASH64

## 4、MGR使用限制

1）和普通复制 binlog 校验不能共存，需设置--binlog-checksum=none

2）不支持 gap lock（间隙锁），隔离级别需设置为 read_committed

3）不支持对表进行锁操作（lock /unlock table）,不会发送到其他节点执行 ,影响需要对表进行加锁操作的情况，列入 mysqldump 全表备份恢复操作

4）不支持 serializable（序列化）隔离级别

5）DDL 语句不支持原子性，不能检测冲突，执行后需自行校验是否一致；不支持外键：多主不支持，单主模式不存在此问题；最多支持 9 个节点：超过 9 台 server 无法加入组

## 5、准备三台服务器

| 编号 | IP             | 主机名 | 角色    |
| ---- | -------------- | ------ | ------- |
| 1    | 192.168.88.101 | node1  | master  |
| 2    | 192.168.88.102 | node2  | slave01 |
| 3    | 192.168.88.103 | node3  | slave02 |

安装MySQL8.0.40数据库

第一步：关闭防火墙和SELINUX、设置IP与主机映射、时间同步，安装必备工具vim、wget、rsync

```PowerShell
systemctl stop firewalld
systemctl disable firewalld

setenforce 0
vim /etc/selinux/config

vim /etc/hosts
尾部追加如下内容
192.168.88.101 node1 node1.itcast.cn
192.168.88.102 node2 node2.itcast.cn
192.168.88.103 node3 node3.itcast.cn

时间同步参考飞书文档

dnf install vim wget rsync -y
```

第二步：使用MX连接node1、node2、node3

node1（master）

node2（slave01）

node3（slave02）

第三步：

把课件中的master.sh与mysql软件包上传master节点

```PowerShell
source master.sh
```

把课件中的slave.sh与mysql软件包上传slave01/slave02节点

```PowerShell
source slave.sh
```

第四步：

master节点：

```PowerShell
systemctl stop mysqld
rm -rf /export/server/mysql/data/auto.cnf
rsync -av /export/server/mysql/data node2:/export/server/mysql/
rsync -av /export/server/mysql/data node3:/export/server/mysql/
```

## 6、MGR配置实战

第一步：node1主服务器配置

```PowerShell
vim /etc/my.cnf
[mysqld]
basedir=/export/server/mysql
datadir=/export/server/mysql/data
socket=/tmp/mysql.sock
port=3306
log-error=/export/server/mysql/master.err
character_set_server=utf8mb4
collation-server=utf8mb4_unicode_ci

# Group Replication
server_id = 101  # 服务 ID
gtid_mode = ON  # 全局事务
enforce_gtid_consistency = ON # 强制 GTID 的一致性
master_info_repository = TABLE # 将 master.info 元数据保存在系统表中
relay_log_info_repository = TABLE # 将 relay.info 元数据保存在系统表中
binlog_checksum = NONE  # 禁用二进制日志事件校验
log_slave_updates = ON  # 级联复制
log_bin = binlog  # 开启二进制日志记录
binlog_format = ROW  # 以行的格式记录

transaction_write_set_extraction = XXHASH64 # 使用哈希算法将其编码为散列
loose-group_replication_group_name = 'ce9be252-2b71-11e6-b8f4-00212844f856' # 加入的组名
loose-group_replication_start_on_boot = off # 不自动启用组复制集群
loose-group_replication_local_address = '192.168.88.101:33061' # 以本机端口 33061 接受来自组中成员的传入连接
loose-group_replication_group_seeds = '192.168.88.101:33061, 192.168.88.102:33062, 192.168.88.103:33063' # 组中成员访问表
loose-group_replication_bootstrap_group = off # 不启用引导组
```

> 注意：group_replication_group_name 组名称主要是通过 uuidgen 命令生成

重启MySQL服务

```PowerShell
touch /export/server/mysql/master.err
chown -R mysql.mysql /export/server/mysql

systemctl restart mysqld
```

创建复制账号

```PowerShell
mysql -uroot -p
Enter password: 123456

mysql> set SQL_LOG_BIN=0; # 停掉日志记录
mysql> create user repl@'%' identified with 'mysql_native_password' by '123';
mysql> grant replication slave,replication client on *.* to repl@'%';
mysql> flush privileges;
mysql> set SQL_LOG_BIN=1;  # 开启日志记录
mysql> change master to master_user='repl',master_password='123' for channel
'group_replication_recovery';  # 构建 group replication 集群
```

安装 group replication 插件

安装插件

```PowerShell
mysql> install PLUGIN group_replication SONAME 'group_replication.so';
```

查看 group replication 组件

```PowerShell
mysql> show plugins;
```

![cc932b07-da36-46b9-a591-171aa6804ed0](cc932b07-da36-46b9-a591-171aa6804ed0.png)

启动服务器 node1 上 MySQL 的 group replication

```PowerShell
mysql> set global group_replication_bootstrap_group=ON;
mysql> start group_replication;
mysql> set global group_replication_bootstrap_group=OFF;
mysql> select * from performance_schema.replication_group_members;  # 查看状态
```

![9d14110d-b0c3-4ca2-92b8-be6291a6cd14](9d14110d-b0c3-4ca2-92b8-be6291a6cd14.png)

> 注意：在集群没有搭建完成之前，不要往任何数据库中插入数据，否则会导致三端数据不统一，集群容易出现故障！

第二步：node2从服务器配置

修改/etc/my.cnf 配置文件，方法和之前相同vim /etc/my.cnf

```PowerShell
vim /etc/my.cnf
[mysqld]
basedir=/export/server/mysql
datadir=/export/server/mysql/data
socket=/tmp/mysql.sock
port=3306
log-error=/export/server/mysql/slave.err
character_set_server=utf8mb4
collation-server=utf8mb4_unicode_ci

# Group Replication
server_id = 102  # 注意服务 ID 不一样
gtid_mode = ON
enforce_gtid_consistency = ON
master_info_repository = TABLE
relay_log_info_repository = TABLE
binlog_checksum = NONE
log_slave_updates = ON
log_bin = binlog
binlog_format= ROW

transaction_write_set_extraction = XXHASH64
loose-group_replication_group_name = 'ce9be252-2b71-11e6-b8f4-00212844f856'
loose-group_replication_start_on_boot = off
loose-group_replication_local_address = '192.168.88.102:33062'
loose-group_replication_group_seeds = '192.168.88.101:33061,192.168.88.102:33062,192.168.88.103:33063'
loose-group_replication_bootstrap_group = off
```

重启MySQL

```PowerShell
touch /export/server/mysql/slave.err
chown -R mysql.mysql /export/server/mysql
systemctl restart mysqld
```

安装 group replication 插件

```PowerShell
mysql> install PLUGIN group_replication SONAME 'group_replication.so';
```

把实例添加到之前的复制组

```PowerShell
mysql -u root -p
Enter password: 123456

mysql> set SQL_LOG_BIN=0; # 停掉日志记录
mysql> create user repl@'%' identified with 'mysql_native_password' by '123';
mysql> grant replication slave,replication client on *.* to repl@'%';
mysql> flush privileges;
mysql> set SQL_LOG_BIN=1;  # 开启日志记录

mysql> reset master;
mysql> -- 设置想要加入组信息
mysql> change master to master_user='repl',master_password='123' for channel 'group_replication_recovery'; 
mysql> start group_replication;
```

返回node1主服务器，查看复制组状态

```PowerShell
mysql> select * from performance_schema.replication_group_members;
```

注意：在集群没有搭建完成之前，不要往任何数据库中插入数据，否则会导致三端数据不统一，集群容易出现故障！

第三步：node3从服务器配置

修改/etc/my.cnf 配置文件，方法和之前相同vim /etc/my.cnf

```PowerShell
vim /etc/my.cnf
[mysqld]
basedir=/export/server/mysql
datadir=/export/server/mysql/data
socket=/tmp/mysql.sock
port=3306
log-error=/export/server/mysql/slave.err
character_set_server=utf8mb4
collation-server=utf8mb4_unicode_ci

# Group Replication
server_id = 103 #注意服务 ID 不一样
gtid_mode = ON
enforce_gtid_consistency = ON
master_info_repository = TABLE
relay_log_info_repository = TABLE
binlog_checksum = NONE
log_slave_updates = ON
log_bin = binlog
binlog_format= ROW
transaction_write_set_extraction = XXHASH64
loose-group_replication_group_name = 'ce9be252-2b71-11e6-b8f4-00212844f856'
loose-group_replication_start_on_boot = off
loose-group_replication_local_address = '192.168.88.103:33063'
loose-group_replication_group_seeds = '192.168.88.101:33061,192.168.88.102:33062,192.168.88.103:33063'
loose-group_replication_bootstrap_group = off
```

进入mysql，安装 group replication 插件

```PowerShell
touch /export/server/mysql/slave.err
chown -R mysql.mysql /export/server/mysql/slave.err

systemctl start mysqld
mysql -uroot -p
Enter password: 123456

mysql> install PLUGIN group_replication SONAME 'group_replication.so';
```

把实例添加到之前的复制组

```PowerShell
mysql> set SQL_LOG_BIN=0; #停掉日志记录
mysql> create user repl@'%' identified with 'mysql_native_password' by '123';
mysql> grant replication slave,replication client on *.* to repl@'%';
mysql> flush privileges;
mysql> set SQL_LOG_BIN=1;  # 开启日志记录

mysql> reset master;
mysql> -- 设置想要加入组信息
mysql> change master to master_user='repl',master_password='123' for channel 'group_replication_recovery'; 
mysql> start group_replication;
```

在node1上查看复制组状态

```PowerShell
mysql> select * from performance_schema.replication_group_members;
```

如果以上状态为ONLINE，则搭建完毕。如果以上状态为RECORYING，则代表两端数据无法同步到一致状态。解决方案：参考常见问题中问题2进行解决。

建议：当MGR高可用集群搭建完毕后，需要给每一个节点创建快照，因为下面的操作，如果数据不统一等等问题，可能会导致集群出现故障。

第四步：测试MySQL

回到master，登录mysql，执行如下指令：

```PowerShell
mysql> create database test;
mysql> use test;
mysql> create table t1 (id int primary key,name varchar(20)); #注意创建主键
mysql> insert into t1 values (1,'jack');
mysql> select * from t1;
mysql> show binlog events;
```

返回node2或node3，查看数据库发现 test 库和 t1 表已经完成同步操作

```PowerShell
mysql> show databases;
```

第五步：测试高可用效果

查看哪个节点是主节点，然后使用systemctl stop mysqld终止主节点，在查看集群变化

```PowerShell
mysql> select * from performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-----------------+-------------+--------------+-------------+----------------+--------                 --------------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST     | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION | MEMBER_                 COMMUNICATION_STACK |
+---------------------------+--------------------------------------+-----------------+-------------+--------------+-------------+----------------+--------                 --------------------+
| group_replication_applier | 1b6e2247-2ae2-11f0-b1be-000c2961ec25 | node3.itcast.cn |        3306 | ONLINE       | SECONDARY   | 8.0.40         | XCom                                        |
| group_replication_applier | 7792fd47-2998-11f0-81a4-000c29d9e0c0 | node1.itcast.cn |        3306 | ONLINE       | PRIMARY     | 8.0.40         | XCom                                        |
| group_replication_applier | fb783c29-299a-11f0-b046-000c296d8526 | node2.itcast.cn |        3306 | ONLINE       | SECONDARY   | 8.0.40         | XCom                                        |
+---------------------------+--------------------------------------+-----------------+-------------+--------------+-------------+----------------+--------                 --------------------+
3 rows in set (0.01 sec)


# 终止master主节点
systemctl stop mysqld

# 进入node2、node3查看集群变化
select * from performance_schema.replication_group_members;
```

以上单 master 节点的集群就搭建完毕!

## 7、常见问题说明

### 7.1 问题1：MEMBER_STATE列值为RECORYING

某同学错误截图：

![95093175-a608-47c0-95fd-0c61f7700054](95093175-a608-47c0-95fd-0c61f7700054.png)

正常设置完成后，两端数据要求完全一样的，但是如果数据没有同步，然后MEMBER_STATE列值为RECORYING。

以上情况，往往是数据正在同步，需要等待一段时间，如5分钟左右，再次观察，服务状态，如果依然为RECORYING，则代表出现异常。

遇到问题以后，首先要进行问题排查 => 查看错误日志！！！

```PowerShell
查看错误日志，如果有则查看错误文档
log-error=/export/server/mysql/slave.err
如果没有专门的错误日志，我们可以通过/var/log/messages查看解决
tail -100 /var/log/messages
错误日志如下：
Feb 15 17:39:14 node2 mysqld[43267]: 2025-02-15T09:39:14.087853Z 24 [System] [MY-010597] [Repl] 'CHANGE REPLICATION SOURCE TO FOR CHANNEL                              'group_replication_recovery' executed'. Previous state source_host='node1.itcast.cn', source_port= 3306, source_log_file='', source_log_po                             s= 4, source_bind=''. New state source_host='node1.itcast.cn', source_port= 3306, source_log_file='', source_log_pos= 4, source_bind=''.
Feb 15 17:39:14 node2 mysqld[43267]: 2025-02-15T09:39:14.099349Z 80 [Warning] [MY-010897] [Repl] Storing MySQL user name or password infor                             mation in the connection metadata repository is not secure and is therefore not recommended. Please consider using the USER and PASSWORD c                             onnection options for START REPLICA; see the 'START REPLICA Syntax' in the MySQL Manual for more information.
Feb 15 17:39:14 node2 mysqld[43267]: 2025-02-15T09:39:14.127878Z 24 [ERROR] [MY-011582] [Repl] Plugin group_replication reported: 'There w                             as an error when connecting to the donor server. Please check that group_replication_recovery channel credentials and all MEMBER_HOST colu                             mn values of performance_schema.replication_group_members table are correct and DNS resolvable.'
Feb 15 17:39:14 node2 mysqld[43267]: 2025-02-15T09:39:14.127924Z 24 [ERROR] [MY-011583] [Repl] Plugin group_replication reported: 'For det                             ails please check performance_schema.replication_connection_status table and error log messages of Replica I/O for channel group_replicati                             on_recovery.'
Feb 15 17:39:14 node2 mysqld[43267]: 2025-02-15T09:39:14.128465Z 24 [ERROR] [MY-011574] [Repl] Plugin group_replication reported: 'Maximum                              number of retries when trying to connect to a donor reached. Aborting group replication incremental recovery.'
Feb 15 17:39:14 node2 mysqld[43267]: 2025-02-15T09:39:14.128527Z 24 [ERROR] [MY-011620] [Repl] Plugin group_replication reported: 'Fatal e                             rror during the incremental recovery process of Group Replication. The server will leave the group.'
Feb 15 17:39:14 node2 mysqld[43267]: 2025-02-15T09:39:14.128578Z 24 [Warning] [MY-011645] [Repl] Plugin group_replication reported: 'Skipp                             ing leave operation: concurrent attempt to leave the group is on-going.'
Feb 15 17:39:14 node2 mysqld[43267]: 2025-02-15T09:39:14.128593Z 24 [ERROR] [MY-011712] [Repl] Plugin group_replication reported: 'The ser                             ver was automatically set into read only mode after an error was detected.'
```

发现问题，主机名称无法解析

解决方案：

```PowerShell
vim /etc/hosts
192.168.88.101 node1 node1.itcast.cn
192.168.88.102 node2 node2.itcast.cn
192.168.88.103 node3 node3.itcast.cn
```

重启报错的mysql节点，然后重新配置组复制集群

```PowerShell
systemctl restart mysqld
```

重新配置MGR集群：

node1（master）：重新开启引导节点

```PowerShell
mysql> reset master;
mysql> change master to master_user='repl',master_password='123' for channel
'group_replication_recovery';   # 构建 group replication 集群
mysql> set global group_replication_bootstrap_group=ON;
mysql> start group_replication;
mysql> set global group_replication_bootstrap_group=OFF;
mysql> select * from performance_schema.replication_group_members;  # 查看状态

命令说明：
reset master：重置集群连接的master主节点，在下方需要通过change master to重置主节点连接！！！
set global group_replication_bootstrap_group=ON：开启组复制功能，允许其他节点连接，不用一直开启，只要集群中有一个主节点，则这个参数就可以关闭了
set global group_replication_bootstrap_group=OFF：开启组复制功能
```

切换node2（slave01）

```PowerShell
mysql> reset master;
mysql> -- 设置想要加入组信息
mysql> change master to master_user='repl',master_password='123' for channel 'group_replication_recovery'; 
mysql> start group_replication;
```

如果以上方案无法解决，可能是master与slave节点之间数据目录不一致（数据不一致），具体请参考问题2解决方案。

### 7.2 问题2：slave节点运行一段时间自动消失了

![0abde51e-9f6c-4f89-876f-daba1380670d](0abde51e-9f6c-4f89-876f-daba1380670d.png)

运行一段时间后

![07642eb0-1307-4d2b-baa0-e5801bba170a](07642eb0-1307-4d2b-baa0-e5801bba170a.png)

排查故障，查看错误日志 => 主节点报错看主节点日志；从节点报错看从节点日志

主节点日志：/export/server/mysql/master.err

从节点日志：/export/server/mysql/slave.err

```PowerShell
Feb 15 18:43:25 node2 mysqld[49606]: 2025-02-15T10:43:25.184117Z 15 [ERROR] [MY-010584] [Repl] Replica SQL for channel 'group_replication_applier': Worker 1 failed executing transaction 'ce9be252-2b71-11e6-b8f4-00212844f856:4'; Error executing row event: 'Unknown database 'test'', Error_code: MY-001049
```

发现两端数据不一致导致的问题 => master节点相对于slave01多了一个test数据库！！！

解决方案：

master主服务器：mysqldump或者xtabackup导出完整备份（全备），先停止mysqld，还可以通过rsync重新同步数据，保证从节点与主节点数据保持一致即可。

```PowerShell
mysqldump -uroot --all-databases > all.sql -p
rsync -av all.sql node2:/root/
```

slave从服务器：

```PowerShell
mysql> stop group_replication;
mysql> set global super_read_only=0;
mysql> reset master;
mysql> reset slave all;
mysql> set sql_log_bin=0;
mysql> source /root/all.sql
mysql> set sql_log_bin=1;
mysql> -- 设置想要加入组信息
mysql> change master to master_user='repl',master_password='123' for channel 'group_replication_recovery'; 
mysql> start group_replication;
```

切换回主节点，查看同步状态

```SQL
mysql> select * from performance_schema.replication_group_members;  #查看状态
```

到此解决了数据不一致问题！！！

### 7.3 问题3：服务器重启集群失效 或者 从节点无法启动group_replication

slave01:

```PowerShell
mysql> start group_replication;
```

很多同学在尝试启动group_replication的时候，发现无法启动，直接报错。这种情况解决思路，先看从节点错误日志，看看具体什么错误，如果没有明显错误。可以判断master节点可能没有启动或者主节点没有正常工作。

```PowerShell
mysql> select * from performance_schema.replication_group_members;
```

整个MGR集群，必须保证master节点先启动，且状态为ONLINE，从节点才能正常加入主节点。常见错误截图如下：

![39cff387-f6f9-46ef-9a85-05668715a7ba](39cff387-f6f9-46ef-9a85-05668715a7ba.png)

主节点异常，可以考虑重新配置主节点

```PowerShell
mysql> reset master;
mysql> change master to master_user='repl',master_password='123' for channel
'group_replication_recovery';   # 构建 group replication 集群
mysql> set global group_replication_bootstrap_group=ON;
mysql> start group_replication;
mysql> set global group_replication_bootstrap_group=OFF;
mysql> select * from performance_schema.replication_group_members;  # 查看状态
```

然后分别启动从节点 => node2、node3

```PowerShell
mysql> start group_replication;
```

### 7.4 问题4：从节点报错无法启动，然后报数据目录拷贝自其他服务器

![5fb11b20-9909-46b8-bf37-fb57812e1c06](5fb11b20-9909-46b8-bf37-fb57812e1c06.png)

 出现以上问题，往往代表，你的这个节点的数据目录来自于其他服务器，因为redo log文件，往往是放置于master主节点，从节点一般在mysqld启动后才会产生，而不是一开始就有这些文件。

![8e3977e1-cad3-4ba7-9ceb-8964845ba560](8e3977e1-cad3-4ba7-9ceb-8964845ba560.png)

这代表之前大家在同步数据时，可能master并没有停止，或者拷贝后，没有及时清空master节点的一些标记（如relaylog日志），这些都相当于是其他服务器的标记，本机无法使用，所以会报错。

解决方案：删除上方红色框框对应的relaylog日志文件，然后重启mysqld（也可以考虑先重新同步rsync，然后在删除红色框框对应的文件）

![1ba716ff-8b4e-41d9-921a-36825c77816b](1ba716ff-8b4e-41d9-921a-36825c77816b.png)

重点最后4行内容！！！

### 7.5 问题总结：MGR本身很简单，错误就两种情况

第一种情况：MGR群组中master节点没有启动或者主节点异常，导致其他从节点无法加入到MGR群组。常见场景就是查看集群状态，只有一个节点显示。还有一种情况，大家特别喜欢重启mysqld服务。主从、MGR都属于mysql服务，mysqld一旦停止/重启，往往主从、MGR也会随之停止服务，这个时候就查看不到任何节点信息了，需要手工重启服务。

有错误，一定要看错误日志 => 

master：cat /export/server/mysql/master.err

slave：cat /export/server/mysql/slave.err

第二种情况：100%数据不一致，三个节点，有一个节点出现故障。剩余两个节点组成主从架构，如果往主服务器写入数据了，则从服务器会随之同步数据。但是故障节点即使重启了，其数据也会和前面两个节点数据不一致，常见场景：MEMBER_STATE列值为RECORYING、slave节点运行一段时间自动消失了、从节点报错无法启动，大部分都是数据不一致造成的。

找到问题节点，先排查错误 => cat /export/server/mysql/slave.err => 问题号：1236（就是100%数据不一致）

停止从节点的mysqld服务

```PowerShell
systemctl stop mysqld
rm -rf /export/server/mysql/data
```

返回主节点：

```PowerShell
rsync -av /export/server/mysql/data 异常节点号:/export/server/mysql/
记得删除auto.cnf文件、relaylog日志文件
rm -rf /export/server/mysql/data/auto.cnf
rm =rf /export/server/mysql/data/node1-relay-bin-*
重启mysqld
systemctl start mysqld
```

如果从节点异常，重置从节点

```PowerShell
stop group_replication;
reset master;
set sql_log_bin=0;
change master to master_user='repl',master_password='123' for channel 'group_replication_recovery';
start group_replication;
set sql_log_bin=1;
```

![06226b2e-a3ea-4c9d-ad07-3b86ba67d460](06226b2e-a3ea-4c9d-ad07-3b86ba67d460.png)

## 8、主库与从库重置操作

遇到集群构建异常，可以进行重置操作：

master节点故障了，重置主节点

暂时无法在飞书文档外展示此内容

slave节点故障了，重置从节点

暂时无法在飞书文档外展示此内容

说明：以上操作重置完成后可能会产生新PRIMARY主节点，获取到新的主节点以后，注意：因为单主模式，所以只有PRIMARY主节点才能实现DML操作。从节点无法完成！！！

## 9、MGR错误原因排查

从库执行

暂时无法在飞书文档外展示此内容

> 问题：直接看`LAST_ERROR_MESSAGE`，这个虽然也能查看错误信息，但是不如slave.err精准！

# 今日重点

- MySQL8新特性 => 本地克隆、远程克隆
- 难点：MGR组复制（保证node1、node2、node3组成集群即可）