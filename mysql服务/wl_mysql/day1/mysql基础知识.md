	1、关系型数据库  业务数据  强一致型=====查询修改效率低
		mysql  
		mariadb
		sql server
		oracle ===性能好
		postgsql
	表： 字段固定
	表之间存在一定参考关系
	强一致型
	mysql ---》 orcale 
	mariadb
		sql语句  增 
				 删
				 改
				 查
	
	tidb 数据可以做成分布式（多台）
	负载均衡： 当前服务器压力大，一台web服务器不能满足需求，再加一台web服务器，在前面再加一个调度器。web服务器是一个无状态的服务，每一次请求都是没有关联的，也没有数据的持久化，会加一个session回话服务器，从而保证负载均衡。关系型数据库没办法来做负载均衡
	
	2、非关系型数据库 最终一致型
		redis：  k-v键值对
		 per1:{ name： 张三,
			  age： 20,
			  major: 云计算
			 }
		 per2: { name:
				age
				major
				hobby: []
		}
		mongodb：文档数据库
		hbase：列式数据库




```
1、找到仓库
2、修改配置文件
[root@localhost ~]# vi /etc/yum.repos.d/mysql-community.repo ---关闭8.0的sql
3、更新仓库
[root@localhost ~]# yum repolist 
4、安装mysql
yum install mysql-server(一路Y)
5、开启mysql
[root@localhost ~]# systemctl start mysqld
[root@localhost ~]# ss -taunp | grep 3306
tcp    LISTEN     0      80       :::3306                 :::*                   users:(("mysqld",pid=5710,fd=13));vULrUrjl4KK
6、查看随机密码：vi /var/log/mysqld.log
7、修改密码：
alter user 'root'@'localhost' identified by 'Aa@123456';
8、刷新：flush privileges ;

9、将密码写在码客户端配置文件中[client]     ====服务端配置文件： [mysqld]/etc/my.cnf
[root@localhost ~]# vi .my.cnf
[client]
user=root
password=Aa@123456

/var/log/mysqld.log   临时密码存放

alter user 'root'@'localhost' identified by '密码'

flush privileges ；刷新
```



```
客户端： mysql_secure_installation 删除 不必要用户和库
 	\c  清空 
 	\g 和; 提交 
 	\G   提交/竖向显示

```

navicat：商业,破解版https://www.cnblogs.com/linshengqian/p/16809472.html
dbeaver: 免费

```
1、允许第三方连接mysql（授权远程登录root）
mysql> grant all privileges on *.* to 'root'@'%' identified by 'Aa@123456' ;
'root'@'%'%代表任意的主机
2、刷新
mysql> flush privileges ;
3、关闭虚拟机里面的防火墙和setenforce 0
4、连接navicat
测试名：y2312
主机：192.168.66.110
用户：root
密码：0000（授权的密码）
```



	库：  
		创建  create database   
		删除  drop database  
		查询  show databases 
	表：  
		字段组成  类型
					1、数值  
					 整型 
					 浮点型
	
				    2、字符型 
						定长
						不定长 
					3、文本     text 
	
					4、布尔型
	
					5、枚举 （男、女） 
		键： 
			 主键： 确定这一行数据（唯一，不为null）
			 唯一键 ：  唯一 为null
			 外键 ：

```
CREATE TABLE `y2312`.`student`  (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(100) NOT NULL,
  `age` int NOT NULL,
  `height` float NOT NULL,
  `gender` enum('男','女') NOT NULL DEFAULT '女',
  `birthday` datetime(255) NULL,
  PRIMARY KEY (`id`)
);

show tables  查看表
desc tb_name  查看字段

drop table 表名  删除表

ALTER TABLE `y2312`.`student` 
ADD COLUMN `tid` int NULL AFTER `name`;
```



	增  
		insert into tb_name values()
		insert into student (name,age,height,gender,birthday) values("lisi",30,2.00,"male",'1997-2-10 00:00:00') ;
		 
	删  
			delete from student where id=2 ;
	改  
	 		update student set age=23  where name="lisi" ;
	查   
	 		select * from  tb_name  
	
	导入库
	[root@www ~]# mysql < test.sql   <输入重定向
	
	 where 条件  = != < > >= <=
	 				 			AND
	 				 			OR
	 				 			NOT !
	 				 			BETWEEN   AND
	 				 			IN
	 				 			LIKE  ''  
	 				 					%  
	 				 					?
	concat 
	CONCAT_WS  字符串拼接
	SELECT 执行什么操作
		CEIL( DATEDIFF( NOW(), birth_date )/ 365 ) AS age,
		first_name 
		FROM
		employees  
		WHERE
		CEIL( DATEDIFF( NOW(), birth_date )/ 365 ) > 65 


 	 				

 		

    聚合函数   
    	max 
    	min
    	avg
    	count 次数

SELECT
	AVG( salary ) AS pay,
	emp_no -- 4
	
FROM
	salaries -- 1
	
WHERE
	emp_no > 10001 -- 2
	

	GROUP BY-- 3
	emp_no 
HAVING
	pay > 10000 -- 5
	
ORDER BY
	pay DESC -- 6
	
	LIMIT 0,2 --7 

