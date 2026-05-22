#### 一、备份工具：xtrabackup备份

	三方工具： perconna xtrabackup，支持热备，物理备份（备份数据,而不是sql语句）支持增量备份
	一、完全备份  
		备份的路径 ： xtrabackup --backup --target-dir 
			
	   还原 
		先准备：xtrabackup --prepare  --target-dir 备份路径  
	     xtrabackup --copy-back --target-dir 备份路径
	
	二、完全+增量：
		完全
			xtrabackup --backup --target-dir 备份的路径  
	    第一次增量： 
	    	xtrabackup --backup --target-dir  备份的路径1   --incremental-basedir 完全备份路径
	    第二次增量：
	    	xtrabackup --backup --target-dir  备份的路径2   --incremental-basedir 第一次增量路径 
	   	
	   	还原准备
		 	  准备完全备份
		 		xtrabackup --prepare --apply-log-only  --target-dir 完全备份路径
		 		
		 	  准备第一次增量备份 
		 	  	xtrabackup --prepare --apply-log-only  --target-dir 完全备份路径  --incremental-dir 第一次增量备份  
		 	  	
		 	  准备第二次增量备份
		 	  	xtrabackup --prepare   --target-dir 完全备份路径  --incremental-dir 第二次增量备份
	 
	 	还原 
	 		xtrabackup --copy-back --target-dir 完全备份 
	 	可能需要二进制日志文件 
	 		mysqlbinlog --start-position


​	

#### 二、备份+还原

```
安装percona包：[root@www ~]# yum install percona-xtrabackup-24-2.4.29-1.el7.x86_64.rpm 
systemctl start mysqld
存放完全备份的目录：
[root@www ~]# mkdir /mysqlbackup/fullbackup==
完全备份： xtarbackup --backup --target-dir /mysqlbackup/fullbackup

还原：
删除mysql==：  rm -rf /var/lib/mysql/*
1、准备： xtrabackup --perpare --target-dir /mysqlbackup/fullbackup==准备
2、还原： xtrabackup --copy-back --target-dir /mysqlbackup/fullbackup
3、还原完成后的时候权限不对要修改权限： /var/lib/mysql权限不对（root）修改权限
chown -R mysql：mysql /var/lib/mysql
4、重启数据库： [root@www ~]# systemctl restart mysqld

```



#### 三、增量备份：使用二进制文件备份

每月一号做完全备份，每周做增量备份，保证数据库的完整性

```
还原准备
	 	  准备完全备份
	 		xtrabackup --prepare --apply-log-only  --target-dir 完全备份路径
	 	  准备第一次增量备份 
	 	  	xtrabackup --prepare --apply-log-only  --target-dir 完全备份路径  --incremental-dir 第一次增量备份  
	 	  准备第二次增量备份
	 	  	xtrabackup --prepare   --target-dir 完全备份路径  --incremental-dir 第二次增量备份 
	 	还原 
	 		xtrabackup --copy-back --target-dir 完全备份 
	 	可能需要二进制日志文件 
	 		mysqlbinlog


终端1 ：
先删除完全备份rm -rf /mysqlbackup/fullbackup/*
t1表添加数据：9 10 11并提交
t1表添加数据：12 13 14并提交
t1表添加数据：15 16 17并提交
t1表添加数据：20 21并提交此时还没有备份就掉了
删除数据库： rm -rf /var/lib/mysql/*
加权限： chown -R  mysql：mysql /var/lib/mysql此时只有到17 ，20和21 没有，使用二进制文件还原20和21


终端2：备份==========
完全备份：xtrabackup --backup --target-dir /mysqlbackup/fullbackup
增量备份（对12 13 14增量备份）：xtrabackup --backup --target-dir /mysqlbackup/incremental/ --incremental-basedir /mysqlbackup/fullbackup
增量备份（对15 16 17增量备份）：xtrabackup --backup --target-dir /mysqlbackup/incremental-2/ --incremental-basedir /mysqlbackup/incremental/

from_lsn = 0
to_lsn = 2963124
last_lsn = 2963133  完全备份

from_lsn = 2963124                  
to_lsn = 2965259
last_lsn = 2965268第一次增量备份 

from_lsn = 2965259
to_lsn = 2967394
last_lsn = 2967403 第二次增量备份

还原：===========还原了12-17
准备完全备份：xtrabackup --prepare --apply-log-only --target-dir /mysqlbackup/fullbackup/
第一次还原（完全备份+（12 13 14的备份））：xtrabackup --prepare --apply-log-only --target-dir /mysqlbackup/fullbackup/ --incremental-dir /mysqlbackup/incremental/
第二次还原（前一次+（15 16 17的备份））：trabackup --prepare --apply-log-only --target-dir /mysqlbackup/fullbackup/ --incremental-dir /mysqlbackup/incremental-2/
还原：xtrabackup --copy-back --target-dir /mysqlbackup/fullbackup/
 chown -R mysql:mysql /var/lib/mysql

如果遇到/var/lib/mysql/为empty时：
mv /var/lib/mysql /var/lib/mysql.bak
mkdir /var/lib/mysql
chown mysql:mysql /var/lib/mysql
在进行还原。


还原20-21：
导入二进制日志： [root@www ~]# mysqlbinlog --start-position 1029 /var/log/mysql/mysqlbin.000005  > 20.sql===（mysqlbin.000005在/mysqlbackup/fullbackup/xtrabackup_binlog_info查看）
还原前先关闭二进制日志：  mysql < 20.sql
 

```



#### 四、**mysql 主从复制**

mysql主库用来写，从库用来读，为保证数据一致采用主从复制，主库将二进制文件复制到从库的中继日志中，再将中继日志文件的数据同步到从库数据库中，从而实现数据的一致。从库可以分担主库读的压力，此时调度器就要分辨客户的请求（是读还是写的操作），当主库挂掉了，调度器就要选择其他从库扮演主库，此时就要对主库实现高可用。

 	mysql主从复制：保证主库和从库数据的一致性，只是对主库的备份并没有分担主库的压力。
 	msyql读写分离：分担主库读的压力，对读进行负载均衡，一旦主库挂了就没有写的操作。
 	mysql高可用：在从库中挑选一个成为主库
 	
 	mysql主从复制：
 		主库将数据变化记录到二进制文件中
 		从库通过主库的二进制日志文件到从库的中继日志中再将中继日志文件重放到从数据库中


 		

#### 五、主从复制：

 	二进制日志文件：记录主数据库数据变化
 	中继日志文件：用于从服务器同步主库二进制日志文件
 	
 	主服务器：负责写
 	 	1： 开启二进制日志文件
 	 			log_bin=/var/log/mysql/logbin
 	 	2： 设定server-id 
 	 			server-id=1
 	 	3:  添加用于主从复制的用户
 	 			GRANT REPLICATION SLAVE  ON *.* TO 'zhang'@'%' IDENTIFIED BY 'Aa@123456';
 	
 	从服务器：负责读，分担主库读的压力
 			vi /etc/my.cnf
 			1： 开启中继日志
 					relay_log=/var/log/mysql/relaylog
 			2： 设定server-id
 					server-id=2
 			3:  设定从服务器为只读[选配]
 					read_only=1
 			修改中继日志的权限： chown -R mysql:mysql /var/log/mysql
 			4:  连接主服务器 
 	     CHANGE MASTER TO MASTER_HOST='192.168.18.13', 		     				    MASTER_USER='zhang', 
 		   	   		MASTER_PASSWORD='Aa@123456', 
 		   	   		MASTER_LOG_FILE='mysqlbin.000013', ===这个在fullback.sql里面查看在那个位置。
 			   		   MASTER_LOG_POS=1604;
 			5： 启动从服务器 
 					start slave 
 			6： 查看从服务器 
 					 show slave status \G 
 			7:  关闭从服务器 
 					 stop slave ;
 			8:  清空从服务器配置 
 					 reset slave all  



#### 六、主从复制例：

```
13（主服务器）：
[root@www ~]# scp /etc/yum.repos.d/mysql-community.repo 192.168.18.200:/etc/yum.repos.d/ ====》yum install mysql-server -y --nogpgcheck
1、先完全备份dbtest库（使用mysqldump）：
[root@www ~]# mysqldump --databases dbtest --flush-logs --master-data=2 --single-transaction > fullbackup.sql====先做完全备份，将fullbackup.sql提交到从库
2、授权：GRANT REPLICATION SLAVE  ON *.* TO 'zhang'@'%' IDENTIFIED BY 'Aa@123456';
mysql> flush privileges ;
[root@www ~]# scp fullbackup.sql 192.168.18.200:/root/

mysql> insert into t1 values(22) ;



200（从服务器）： yum install mysql-server -y --nogpgcheck
修改密码：alter user 'root'@'localhost' identified by 'Aa@123456'
写在配置文件中：vi .my.cnf 
[client]
user=root
password=Aa@123456

[root@localhost ~]# vi /etc/my.cnf=修改配置开启中继日志为只读
relay_log=/var/log/mysql/relaylog
server-id=2
read_only=1

[root@localhost ~]# mkdir /var/log/mysql
[root@localhost ~]# chown -R mysql:mysql /var/log/mysql
[root@localhost ~]# systemctl restart mysqld
13主服务器中： scp fullbackup.sqp 192.168.18.200:/root/
[root@localhost ~]# mysql < fullback.sql 

和主服务器进行同步：查看vi fullback.sql得知从哪个文件开始复制logbin.00000
mysql>   CHANGE MASTER TO MASTER_HOST='192.168.18.13',MASTER_USER='zhang',MASTER_PASSWORD='Aa@123456',MASTER_LOG_FILE='logbin.000011',MASTER_LOG_POS=1004;

mysql> start slave ;

```

 mysql集群代理服务器实现读写分离
 		通过mycat 实现读写分离
 		安装mycat  
 			1： 准备java环境 
 			2：

schema name=testdb --> dbnode--->dn1--->xxx
					   datahost=localhost1 xxx	
					   writehost	
					   readhost


 mysql集群高可用