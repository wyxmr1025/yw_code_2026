# Redis集群

目标：掌握Redis集群配置 + 区分Redis哨兵与集群区别？

相同点：都是基于主从模式

哨兵：针对主从高可用架构，发现master主节点故障，哨兵实现自动切换（10-30s）=> 1主1从或1主多从

集群：针对主从高可用架构，发现master主节点故障，不需要切换，因为在整个集群中，多主多从，某个节点出现故障，不影响整个集群使用，所以切换速度特别快。

# 一、Redis集群概述

## 1、什么是Redis集群

2018年十月 Redis 发布了稳定版本的 5.0 版本，推出了各种新特性，其中一点是放弃 Ruby的集群方式，改为 使用 C语言编写的 redis-cli的方式，使集群的构建方式复杂度大大降低。关于集群的更新可以在 Redis5 的版本说明中看到，如下：

> The cluster manager was ported from Ruby (redis-trib.rb) to C code inside redis-cli. check `redis-cli --cluster help ` for more info.

可以查看Redis官网查看集群搭建方式，连接如下

https://redis.io/topics/cluster-tutorial

以下步骤是在一台 Linux 服务器上搭建有6个节点的 Redis集群。

注：实际运维工作中，大概需要3台机器，每台机器2个节点。但是详细规划主从关系，尽量不要把一组主从放在同一台服务器中。

Redis01  Redis02  Redis03

1主          1从

​                2主           2从

3从						   3主



## 2、创建目录

IP地址：192.168.88.111 ~ 88.113

主机名称：redis01.itcast.cn/redis02.itcast.cn/redis03.itcast.cn

## 3、下载源码并解压编译

```
dnf install wget -y
wget https://download.redis.io/releases/redis-7.4.0.tar.gz
tar xzf redis-7.4.0.tar.gz
cd redis-7.4.0
make
make PREFIX=/usr/local/redis install
```

## 4、清理redis集群环境

master和slave都要配置

```
systemctl stop redis
cd /usr/local/redis
rm -rf dump.rdb
rm -rf appendonlydir
```



# 二、Redis集群配置（单节点，了解）

## 1、创建配置文件

注：6个配置文件不能在同一个目录，此处我们定义如下：

```
# mkdir -p /redis/conf/700{1..6}
/redis/conf/7001/redis.conf
/redis/conf/7002/redis.conf
/redis/conf/7003/redis.conf
/redis/conf/7004/redis.conf
/redis/conf/7005/redis.conf
/redis/conf/7006/redis.conf
```

## 2、配置文件内容

```powershell
port 7001 #端口
cluster-enabled yes #启用集群模式
cluster-config-file nodes_7001.conf
cluster-node-timeout 5000 #超时时间
appendonly yes
daemonize yes #后台运行
protected-mode no #非保护模式
pidfile /var/run/redis_7001.pid
```

注：其中 port 和 pidfile 需要随着文件夹的不同递增，一键配合脚本

```powershell
#!/bin/bash
for i in $(seq 7001 7006)
do
cat > /redis/conf/$i/redis.conf <<EOF
port $i
cluster-enabled yes
cluster-config-file nodes_$i.conf
cluster-node-timeout 5000
appendonly yes
daemonize yes
protected-mode no
pidfile /var/run/redis_$i.pid
EOF
done
```

## 3、启动节点

```powershell
#!/bin/bash
for i in $(seq 7001 7006)
do
/usr/local/redis/bin/redis-server /redis/conf/$i/redis.conf
done
```

## 4、启动集群

```powershell
/usr/local/redis/bin/redis-cli --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 --cluster-replicas 1
```

至此，Reids7 集群搭建完成。

测试：

```powershell
# /usr/local/redis/bin/redis-cli -c -p 7001
注：一定要添加-c选项，否则redis-cli默认启动是不以集群方式启动
```

## 5、关闭集群

方法一：

```powershell
/usr/local/redis/bin/redis-cli -p 7001 shutdown
/usr/local/redis/bin/redis-cli -p 7002 shutdown
/usr/local/redis/bin/redis-cli -p 7003 shutdown
/usr/local/redis/bin/redis-cli -p 7004 shutdown
/usr/local/redis/bin/redis-cli -p 7005 shutdown
/usr/local/redis/bin/redis-cli -p 7006 shutdown
```

方法二：

```powershell
echo 7001 7002 7003 7004 7005 7006 | xargs -n1 -I{} /usr/local/redis/bin/redis-cli -p {} shutdown

-n  ：表示每次执行命令时传递的参数个数
-n1：表示每次只传递 1 个参数 给后续命令
-I   ：用于指定一个占位符（通常用 {}），后续命令中可以通过占位符引用输入参数
```

**示例：**
输入数据为 `1 2 3 4`，执行 `echo` 命令：

```powershell
echo "1 2 3 4" | xargs -n1 echo "Number:"
Number: 1
Number: 2
Number: 3
Number: 4
```

不加 `-n1` 时，默认所有参数一次性传递：

```powershell
echo "1 2 3 4" | xargs echo "Numbers:"
Numbers: 1 2 3 4
```

# 三、Redis集群配置（多节点）

## 1、Redis集群实现

第一步：环境规划（3主3从，对外提供服务的一共是3个节点）

前提：Redis集群往往在搭建环境时必须要提前设计，而且Redis集群要求，在创建集群之前，任何都不能有数据！！！

| 编号 | 主机名称          | IP地址         | 角色      |
| ---- | ----------------- | -------------- | --------- |
| 1    | redis01.itcast.cn | 192.168.88.111 | redis7001 |
| 2    | redis01.itcast.cn | 192.168.88.111 | redis7002 |
| 3    | redis02.itcast.cn | 192.168.88.112 | redis7003 |
| 4    | redis02.itcast.cn | 192.168.88.112 | redis7004 |
| 5    | redis03.itcast.cn | 192.168.88.113 | redis7005 |
| 6    | redis03.itcast.cn | 192.168.88.113 | redis7006 |

> 设置之前，把Redis01、Redis02、Redis03机器上的所有Redis全部停止，删除所有配置文件redis.conf

第二步：在/usr/local/redis/conf目录中创建redis7001.conf...redis7006.conf

Redis01 =>  redis7001.conf/redis7002.conf

Redis02 =>  redis7003.conf/redis7004.conf

Redis03 =>  redis7005.conf/redis7006.conf

节点配置类似：

```powershell
port 7001
cluster-enabled yes
cluster-config-file nodes_7001.conf
cluster-node-timeout 5000
appendonly yes
daemonize yes
protected-mode no
pidfile /var/run/redis_7001.pid
```

```powershell
port 7002
cluster-enabled yes
cluster-config-file nodes_7002.conf
cluster-node-timeout 5000
appendonly yes
daemonize yes
protected-mode no
pidfile /var/run/redis_7002.pid
```

```powershell
port 7003
cluster-enabled yes
cluster-config-file nodes_7003.conf
cluster-node-timeout 5000
appendonly yes
daemonize yes
protected-mode no
pidfile /var/run/redis_7003.pid
```
参考以上配置，准备7004 ~ 7006！！！

> 注意更改：port、cluster-config-file、pidfile

第三步：配置完成后，启动Reids6个节点

```powershell
# cd /usr/local/redis
# bin/redis-server conf/redis7001.conf
# bin/redis-server conf/redis7002.conf
# bin/redis-server conf/redis7003.conf
# bin/redis-server conf/redis7004.conf
# bin/redis-server conf/redis7005.conf
# bin/redis-server conf/redis7006.conf
```

第四步：创建集群（集群最少需要3个主节点）

| Redis01   | Redis02   | Redis03   |
| --------- | --------- | --------- |
| 1主(7001) | 1从(7003) |           |
|           | 2主(7004) | 2从(7005) |
| 3从(7002) |           | 3主(7006) |

​	                    

```powershell
任意一台服务器上：
# bin/redis-cli --cluster create 192.168.88.111:7001 192.168.88.112:7004 192.168.88.113:7006 192.168.88.112:7003 192.168.88.113:7005 192.168.88.111:7002 --cluster-replicas 1

参数说明：
--cluster create创建集群，后面跟Redis及端口号
--cluster-replicas 1，每个主节点默认都有1个从节点
集群模式不需要提前搭建主从架构，而是在创建集群时，系统会自动配置主从，不需要人工干预
bin/redis-cli --cluster create  ①主节点 ②主节点  ③主节点  ①从节点 ②从节点 ③从节点
```

常见问题说明：

常见问题：Redis集群要求Redis中不能有数据，包括appendonlydir、dump.rdb、nodes_700X.conf

解决方案：哪个节点报错，就清除哪个节点（以7001为例）

```powershell
ps -ef |grep redis-server

找到7001对应的进程
kill -9 进程号

rm -rf appendonlydir
rm -rf dump.rdb
rm -rf node_7001.conf

清除完成后，重启Redis
bin/redis-server conf/redis7001.conf
```

## 2、测试集群

```powershell
# bin/redis-cli -c -h 主节点IP地址 -p 7001
选项说明：
-c代表已集群的方式进行连接
```

## 3、关闭集群

```powershell
# bin/redis-cli -c -h 192.168.88.111 -p 7001 shutdown
# bin/redis-cli -c -h 192.168.88.111 -p 7002 shutdown
# bin/redis-cli -c -h 192.168.88.112 -p 7003 shutdown
# bin/redis-cli -c -h 192.168.88.112 -p 7004 shutdown
# bin/redis-cli -c -h 192.168.88.113 -p 7005 shutdown
# bin/redis-cli -c -h 192.168.88.113 -p 7006 shutdown
```

## 4、集群重启

重启步骤非常简单，只需要把每台服务器的各个节点依次启动即可。

```powershell
# cd /usr/local/redis
# bin/redis-server conf/7001.conf
# bin/redis-server conf/7002.conf
# bin/redis-server conf/7003.conf
# bin/redis-server conf/7004.conf
# bin/redis-server conf/7005.conf
# bin/redis-server conf/7006.conf
```

## 5、代码绑定redis集群

配置好了redis集群。java/pathon/php代码。如何实现绑定redis集群？

java

![Snipaste_2025-09-24_17-05-34](media\Snipaste_2025-09-24_17-05-34.png)

spring boot框架，通过yaml进行配置 application.yaml

![Snipaste_2025-09-24_17-08-19](media\Snipaste_2025-09-24_17-08-19.png)

python

![Snipaste_2025-09-24_17-10-51](media\Snipaste_2025-09-24_17-10-51.png)

php

![Snipaste_2025-09-24_17-11-57](media\Snipaste_2025-09-24_17-11-57.png)



## 四、集群原理

## 1、16384个哈希槽

redis cluster在设计的时候，就考虑到了去**中心化**，去中间件，也就是说，集群中的每个节点都是平等的关系，都是对等的，每个节点都保存各自的数据和整个集群的状态。每个节点都和其他所有节点连接，而且这些连接保持活跃，这样就保证了我们只需要连接集群中的任意一个节点，就可以获取到其他节点的数据。

Redis 集群没有并使用传统的一致性哈希来分配数据，而是采用另外一种叫做`哈希槽 (hash slot)`的方式来分配的。redis cluster 默认分配了 16384 个slot，当我们set一个key 时，会用`CRC16`算法来取模得到所属的`slot`，然后将这个key 分到哈希槽区间的节点上，具体算法就是：`CRC16(key) % 16384。所以我们在测试的时候看到set 和 get 的时候，直接跳转到了7000端口的节点。`

redis兼具了高可用和故障切换能力：Redis 集群会把数据存在一个 master 节点，然后在这个 master 和其对应的salve 之间进行数据同步。当读取数据时，也根据一致性哈希算法到对应的 master 节点获取数据。只有当一个master 挂掉之后，才会启动一个对应的 salve 节点，充当 master 。

需要注意的是：必须要`3个或以上`的主节点，否则在创建集群时会失败，并且当存活的主节点数小于总节点数的一半时，整个集群就无法提供服务了。

3主  3从 = 正常

1主  3从 = 1 < (1+3)/2 = 2 = 集群失效

---

简单理解：Redis集群就是一个大仓库，这个仓库中为了方便数据存储，拆分为16384 slot哈希槽。

因为写只和master主节点相关，所以16384要被3个master拆分

Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383

问题：为什么要分槽？写入一条记录如name:itheima，写入到哪个槽中？

答：分槽目的是为了实现数据最大程度使用，也可以避免数据写入混乱。

写入一条记录如name:itheima，写入到哪个槽中？

在Redis设计过程中，引入了一个crc16函数，用于针对key求解，结果返回一个数字 => crc16(name) = 5000

具体数据写入到哪里 => 哈希求余 => crc16(name) % 3 = 5000 % 3 = 1666（存放槽位置）==%取模==

>
>
>总结：
>
>1、redis cluster在设计的时候，就考虑到了去**中心化**，去中间件，也就是说，集群中的每个节点都是平等的关系，都是对等的，每个节点都保存各自的数据和整个集群的状态。
>
>2、redis cluster 默认分配了 16384 个slot，当我们set一个key 时，采用`CRC16`算法来算出key的取值（CRC16(key) % 16384）得到值后就会落在某个区间，然后就转到那个节点上
>
>如何实现数据的存和取：总共有16384个哈希槽，然后为主节点进行分配。存或取数据时通过CRC16的算法对16384取模，计算落在那个区间（槽）从而实现存或者取数据。

## 2、扩展：添加新节点

第一步：添加新的主节点

```powershell
redis-cli --cluster add-node 192.168.88.114:7007(新节点) 192.168.88.111:7001(代表任意一个主节点)
```

可能遇到的问题

```powershell
# bin/redis-cli --cluster add-node 192.168.88.114:7007 192.168.88.111:7001
>>> Adding node 192.168.88.114:7007 to cluster 192.168.88.111:7001
>>> Performing Cluster Check (using node 192.168.88.111:7001)
S: f40ee549fc670dcbd0b73832617b737485851a7a 192.168.88.111:7001
   slots: (0 slots) slave
   replicates 048cd723f0c6c298d514829d03c7238215d479ad
M: 6ad928aab9065dc384b38812a8f539cadc2b09aa 192.168.88.113:7006
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: f1fe7f56fb43faa0c5fa6c1e23e226de48e89472 192.168.88.113:7005
   slots: (0 slots) slave
   replicates d567f02dc622fa288364f6b69240d4b562f480fe
M: 048cd723f0c6c298d514829d03c7238215d479ad 192.168.88.112:7003
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: d567f02dc622fa288364f6b69240d4b562f480fe 192.168.88.112:7004
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: f66e9c3326b1204f21afc96cf4925eee2563c0f0 192.168.88.101:7002
   slots: (0 slots) slave
   replicates 6ad928aab9065dc384b38812a8f539cadc2b09aa
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
[WARNING] Node 192.168.88.112:7003 has slots in importing state 5798.
[WARNING] Node 192.168.88.112:7004 has slots in importing state 741,3680.
[WARNING] The following slots are open: 3680,741,5798.
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

第二步：修复未完成的槽迁移（若存在）

```powershell
# 修复 192.168.88.112:7003 的槽 5798
redis-cli -h 192.168.88.112 -p 7003 cluster setslot 5798 stable

# 修复 192.168.88.112:7004 的槽 741 和 3680
redis-cli -h 192.168.88.112 -p 7004 cluster setslot 741 stable
redis-cli -h 192.168.88.112 -p 7004 cluster setslot 3680 stable
```

第三步：迁移槽到新主节点（自动平衡）

```powershell
redis-cli --cluster rebalance 192.168.88.111:7001 --cluster-threshold 1
```

> Redis 7 的 `rebalance` 命令更智能，可能减少人工干预。

第四步：添加从节点 `192.168.88.114:7008`

```powershell
# 获取新主节点 ID
NODE_ID_7007=$(redis-cli -h 192.168.88.114 -p 7007 cluster nodes | grep myself | awk '{print $1}')

# 添加从节点并绑定到主节点
redis-cli --cluster add-node 192.168.88.114:7008 192.168.88.111:7001 \
--cluster-slave \
--cluster-master-id $NODE_ID_7007
```

第五步：验证集群状态

```powershell
redis-cli --cluster check 192.168.88.111:7001
```

>redis为高内存型应用，要求较高，一般为64G或者128G

# 今日重点

Redis集群最少有3个主节点，有3个主节点必须有对应的3个从节点 =》3主3从

Redis主服务器负担读写操作，Redis从服务器只负责同步数据，可以承担读的任务！

  物流公司

M1   M2  M3

S1    S2   S3

一共16384个哈希槽，存储数据格子。Redis仓库，哈希槽就是里面每一个小格子。如果是写操作，首先通过哈希求余算法，求解数据落在哪个范围。

set name itheima

---------------------------------------------------------------

CRC16（name）=  131,072,000 % 16384 = 0

bin/redis-cli -h 192.168.88.113 -p 7005

127.0.0.1:7001> move 192.168.88.111:7001

192.168.88.111> set name itheima

---

get name

CRC16（name）% 16384 = 0

127.0.0.1:7001> move M1/S1

M1/S1> itheima

双主 => MySQL数据库，在一个集群中有2个主节点

M1   S1

M2   S2

如果M1和M2互为主从，就是所谓的MySQL双主架构。如果某个主节点出现故障对整个集群没有任何影响





