# MySQL数据服务管理

# 任务背景

随着**中州智慧养老项目**用户量和数据量的快速增长，系统的MySQL数据库面临着越来越多的查询性能问题，特别是在高并发情况下，查询响应时间显著增加，影响了系统的稳定性和用户体验。运维团队的主要任务是通过SQL查询的监控与优化，确保数据库在大数据量和高并发环境下仍然能够保持良好的性能表现。

# 任务拆解

- 熟悉SQL查询（重点）
  - 学习并掌握基本的SQL查询语句（如`SELECT`、`WHERE`、`GROUP BY`、`LIMIT`等），能够执行数据的筛选、排序、聚合等操作（重点）
  - 熟悉复杂SQL查询，如多表查询、嵌套查询及子查询，确保能够在处理复杂业务逻辑时正确获取数据。（加分项）
- 查询性能监控与分析（重点） =>  优化手段
  - 启用MySQL的慢查询日志，定期分析执行时间较长的SQL查询，识别出性能瓶颈。
  - 使用`EXPLAIN`命令分析复杂查询的执行计划，识别全表扫描等问题。
  - 通过`SHOW PROCESSLIST`工具，实时监控正在运行查询，找出占用系统资源较多的SQL语句，分析其影响。

# 任务目标

- 掌握基础和复杂SQL查询语句的使用，能够高效执行数据操作（增删改查）。
- 通过慢查询日志和性能监控工具，识别并分析查询性能问题。

# 前置知识

# 一、MySQL图形化开发工具DataGrip

前置知识点：在mysql中学会创建远程账号

![1280X1280](1280X1280.PNG)

登录node1的mysql数据库（3306端口）

```PowerShell
mysql -uroot -p
Enter password: 123456

mysql> drop user 'root'@'%';
mysql> create user 'root'@'%' identified by '123456';
mysql> grant all on *.* to 'root'@'%' with grant option;
说明：
grant => 分配权限
all => 所有权限
on => 针对哪个数据库的哪个数据表
*.* => 左边的*代表所有数据库，右边的*代表所有数据表
to => 这个权限分配给哪个用户
```

## 1、DataGrip介绍

DataGrip是JetBrains公司推出的管理数据库的产品，功能非常强大，可以兼容各种数据库，另外,JetBrains公司还有一款知名的IDE开发工具IDEA，用户体验非常不错。

下载地址：[https://www.jetbrains.com/datagrip/download/#section=windows](https://www.jetbrains.com/datagrip/download/)

![1280X1280 (1)](1280X1280 (1).PNG)

## 2、DataGrip安装

下载完成后打开安装程序，选择安装路径，一直点击Next即可安装。 直到Finish点击完成安装，并打开应用。 选择 Evaluate for free 免费试用30天。

> 如果DataGrip软件使用比较好，有条件的可以支持一下正版^_^

## 3、创建工程

点击File->New->Project新建DataGrip工程

输入项目名称，点击确定。

选择新项目打开方式：This Windows（在本窗口中打开），New Windows（在新窗口中打开）， Attach（附加模式）

## 4、连接数据库

选择Database下的➕，点击DataSource菜单下的MySQL。 填写对应的参数，连接数据库：连接名，IP，用户名，密码等，点击OK完成连接。 注意：如果第一次使用，需要下载mysql驱动文件。

![976fe7da-c611-47fe-af4d-abe99d3e01cc](976fe7da-c611-47fe-af4d-abe99d3e01cc.png)

设置数据库时区（如果提示需要再创建，不提示则忽略）：

1. 点击**Advanced**按钮；
2. 在**VM options**后面写入`-Duser.timezone=Asia/Shanghai`，就可以啦；

![8001b68b-ae4c-4500-aafc-779a42e7229e](8001b68b-ae4c-4500-aafc-779a42e7229e.png)

设置完成后，单击Apply（应用），单击OK，数据库就连接成功了！

**常见连接异常说明**：

排查1：排查IP地址是否可以ping通

Windows，cmd：

```PowerShell
ping 192.168.88.101
```

排查2：查看MySQL端口是否被占用

```PowerShell
ss -nltp

查看mysqld进程是否占用了3306端口
```

特殊：MySQL软件可以正常使用，但是ss查看没有端口，这就是MySQL启动异常（有进程，没有端口），解决思路

```PowerShell
pkill mysqld
systemctl start mysqld
systemctl enable mysqld
```

排查3：查看服务器防火墙是否开启，如果开启，是否放行3306、33060端口

```PowerShell
systemctl stop firewalld
systemctl disable firewalld
```

排查4：除了以上配置以外，还需要有一个远程账号root@%

```PowerShell
mysql> select user,host from mysql.user;
```

## 5、DataGrip软件设置

### ☆ 设置字体大小

设置文字大小： File--->settings--->Editor---->Font

![d1638373-5758-47d0-bc47-ff23b7250002](d1638373-5758-47d0-bc47-ff23b7250002.png)

### ☆ 设置关键字大小写

设置关键字大写： File--->settings--->Editor---->Code Style--->SQL--->MySql(需要设置的数据库)--->Case

![38bbcf09-e898-4bd1-84fc-44ea1f753dac](38bbcf09-e898-4bd1-84fc-44ea1f753dac.png)

### ☆ 自动排版（了解）

自动排版布局： File--->settings--->Editor---->Code Style--->SQL--->MySql(需要设置的数据库)--->Queries 自动排版快捷键：Ctrl+ Alt + L

![e29a11ba-be79-4b52-a277-63fb10f61b5e](e29a11ba-be79-4b52-a277-63fb10f61b5e.png)

### ☆ 设置SQL代码存储位置

第一步：显示Files菜单

![0999db3e-81ff-4f6b-9b86-8f4fd2c0dca6](0999db3e-81ff-4f6b-9b86-8f4fd2c0dca6.png)

 第二步：找到右侧的Files菜单，展开，鼠标右键

![6ae446b1-004c-4203-95d4-17282c7c9d82](6ae446b1-004c-4203-95d4-17282c7c9d82.png)

![f17a1807-f9d1-466e-a18a-6f16b243ced6](f17a1807-f9d1-466e-a18a-6f16b243ced6.png)

确定，创建SQL代码文件

![1e5f8bab-2e07-4aad-93a7-b22d2d337419](1e5f8bab-2e07-4aad-93a7-b22d2d337419.png)

双击展开，然后选择创建New Session，让这个文件可以连接到我们的MySQL项目中，这样SQL代码就可以正常解析了

![546bf445-807d-449e-ad09-25fb26b5e899](546bf445-807d-449e-ad09-25fb26b5e899.png)

# 二、SQL语句（重点）

## 1、SQL语句概述

结构化查询语言(Structured Query Language)简称SQL，是关系型数据库管理系统都需要遵循的规范，是数据库认识的语句。不同的数据库生产厂商都支持SQL语句，但都有特有内容。

**举例：**

普通话：各数据库厂商都遵循的ISO标准。

方言：数据库特有的关键字。

![189df21a-cfc8-4b0a-985b-060bceb334bb](189df21a-cfc8-4b0a-985b-060bceb334bb.png)

## 2、SQL语句分类（记住）

### ☆ DDL

数据定义语言：简称DDL(Data Definition Language) 用来定义数据库对象：数据库，表，列等。 关键字：create，alter，drop等

> 搭建数据存储的框架，如管理数据库、管理数据表、管理字段

### ☆ DML(增删改)

数据操作语言：简称DML(Data Manipulation Language) 用来对数据库中表的记录进行更新（增加、修改、删除）。 关键字：insert，delete，update等

### ☆ DQL（查）

数据查询语言：简称DQL(Data Query Language) 用来查询数据库中表的记录（查询）。 关键字：select，from，where等

### ☆ DCL

数据控制语言：简称DCL(Data Control Language) 用来定义数据库的访问权限和安全级别及创建用户（账号管理、权限管理）。

## 3、SQL基本语法

① SQL语句可以单行或多行书写，但是最终需要以分号结尾。

```Plaintext
select * from students;
```

② 可使用空格和缩进来增强语句的可读性

```PowerShell
select
   *
from students;
```

③ MySQL数据库的SQL语句默认不区分大小写

```PowerShell
SELECT * FROM user;
等价于
select * from user; 
```

④ 可以使用单行与多行注释

```Plaintext
#  单行注释
-- 单行注释，注意：--后面有一个空格

/*
        ...
        多行注释
        ...
*/
```

## 4、小结

SQL：（结构化查询语言），是关系型数据库管理系统都需要遵循的规范，不同的数据库生产厂商都支持SQL语句，但都有特有内容。

SQL语句分为：数据定义语言（DDL），数据操作语言（DML），数据查询语言（DQL），数据控制语言（DCL）。

SQL通用语法：① 分号结尾 ② 空格或者缩进 ③ 不区分大小写 ④ 注释：单行注释 与 多行注释 

# 任务实施

# 一、DDL数据库操作

## 1、MySQL的组成结构（理解）

![e5bdb4c9-42bd-467a-bbc2-b7a2f0ac6eed](e5bdb4c9-42bd-467a-bbc2-b7a2f0ac6eed.png)

> 注：我们平常说的MySQL，其实主要指的是MySQL数据库管理软件。

一个MySQL DBMS可以同时存放多个数据库，理论上一个项目就对应一个数据库。如博客项目blog数据库、商城项目shop数据库、微信项目wechat数据库。

一个数据库中还可以同时包含多个数据表，而数据表才是真正用于存放数据的位置。（类似我们Office软件中的Excel表格），理论上一个功能就对应一个数据表。如博客系统中的用户管理功能，就需要一个user数据表、博客中的文章就需要一个article数据表、博客中的评论就需要一个message数据表。

一个数据表又可以拆分为多个字段，每个字段就是一个属性。一个用户表，需要保存用户账号、用户的密码等信息

一个数据表除了字段以外，还有很多行，每一行都是一条完整的数据（记录）。

## 2、数据库的基本操作

### ① 创建数据库

普及英语小课堂：

创建 => create

数据库 => database

创建 + 数据库 = create database 数据库名称(字母+数字+下划线组成，以字母开头，不能出现中文以及特殊字符)

基本语法：

```PowerShell
mysql> create database 数据库名称 [设置编码格式];
```

> 特别注意：在MySQL中，当一条SQL语句编写完毕后，一定要使用分号;进行结尾，否则系统认为这条语句还没有结束。

案例：创建数据库的相关案例

```PowerShell
创建db_itheima库
create database db_itheima;

创建db1库并指定默认字符集
create database db_itheima default charset gbk;

如果存在不报错(if not exists)
create database if not exists db_itheima default character set utf8;
说明：不能创建相同名字的数据库！
```

> 扩展：编码格式，常见的gbk（中国的编码格式）与utf8（国际通用编码格式）

latin1：这是早期的字符编码格式，支持256个字符，适用于基本的西方语言字符集。

gb2312/gbk：由于latin1的字符集无法表示汉字，中国开发了自己的编码格式gb2312，后来升级为gbk，以支持更多汉字和扩展字符集。

big5：中国台湾也开发了一套用于表示繁体中文的字符编码格式，称为big5，适用于繁体中文字符集。

utf8：为了支持多语言环境，国际标准化组织开发了utf8编码，能够表示几乎所有已知语言的字符。由于早期版本的utf8不完全支持某些特殊符号和表情符号，MySQL5.6版本以后推出了utf8mb4，用于支持更多字符（如Emoji表情），每个字符最多使用4个字节。

> 编写SQL语句是一个比较细致工作，不建议大家直接在终端中输入SQL语句，可以先把你要写的SQL语句写入一个记事本中，然后拷贝执行。

### ② 查询数据库

英语小课堂：

显示 => show

数据库 => database

显示 + 所有数据库 = show databases;

基本语法：显示所有数据库

```PowerShell
mysql> show databases;
```

### ③ 删除数据库

英语小课堂：

删除 => drop

数据库 => database

删除 + 数据库 = drop database 数据库名称;

基本语法：

```PowerShell
mysql> drop database 数据库名称;
```

案例：删除db_itheima数据库

```PowerShell
mysql> drop database db_itheima;
```

### ④ 选择数据库

从数据库列表中查找需要使用的数据库

格式：

```Plaintext
mysql> use db_itheima;
```

查看正在使用的数据库（8.0以后版本需要基于select查询来获取当前数据库）

```PowerShell
mysql> select database();
```

## 3、小结

创建数据库：CREATE DATABASE 数据库名； 查看数据库：SHOW DATABASES; 删除数据库：DROP DATABASE 数据库名；  -- 慎重 使用数据库：USE 数据库名；

# 二、DDL数据表操作

DDL要学4个关键字：create（创建）、drop（删除）、alter（修改）、show（查询）

特别注意：创建数据表必须有一个前提，首先要明确选择某一个数据库。

## 1、数据表的基本操作

### ☆ 数据表的创建

英语小课堂：

创建 => create

数据表 => table

创建 + 数据表 = create table 数据表名称

基本语法：

```PowerShell
mysql> create table 数据表名称(
        字段1 字段类型 [字段约束],
        字段2 字段类型 [字段约束],
        ...
); 

常用字段类型
整数：int（-21亿 ~ 21亿）
浮点：decimal(M,N)，M代表整数长度 + 小数总长度，N代表小数点后保留几位小数，decimal(5,2) => 999.99
字符串类型：固定长度用char(M)，变化长度用varchar(M) => M代表最大字符数
枚举类型：enum('男','女')，插入值要么为男，要么为女
```

案例：创建一个admin管理员表，拥有3个字段（编号、用户名称、用户密码）

```PowerShell
mysql> create database db_itheima;
mysql> use db_itheima;
```

> use在MySQL中的含义代表选择，use 数据库名称相当于选择指定的数据库。而且use比较特殊，其选择结束后，其尾部可以不加分号；但是强烈建议所有的SQL语句都要加分号，养成一个好习惯。

```PowerShell
mysql> create table tb_admin(
    id tinyint,
    username varchar(20),
    password char(32)
) engine=innodb default charset=utf8;
```

> tinyint ：微整型，范围-128 ~ 127，无符号型，则表示0 ~ 255

> 表示字符串类型可以使用char与varchar，char代表固定长度的字段，varchar代表变化长度的字段。

案例：创建一个article文章表，拥有4个字段（编号、标题、作者、内容）

```PowerShell
mysql> use db_itheima;
mysql> create table tb_article(
        id int,
        title varchar(50),
        author varchar(20),
        content text
) engine=innodb default charset=utf8;
```

> text ：文本类型，一般情况下，用varchar存储不了的字符串信息，都建议使用text文本进行处理。

> varchar存储的最大长度，理论值65535个字符。但是实际上，有几个字符是用于存放内容的长度的，所以真正可以使用的不足65535个字符，另外varchar类型存储的字符长度还和编码格式有关。1个GBK格式的占用2个字节长度，1个UTF8格式的字符占用3个字节长度。GBK = 65532~65533/2，UTF8 = 65532~65533/3

### ☆ 查询已创建数据表

英语小课堂：

显示 => show

数据表 => table

显示所有数据表（当前数据库）

```PowerShell
mysql> use 数据库名称;
mysql> show tables;
```

显示数据表的创建过程（编码格式、字段等信息）

```PowerShell
mysql> desc 数据表名称;
mysql> show create table 数据表名称;
```

### ☆ 修改数据表信息（多写）

#### ① 数据表字段添加

英语小课堂：

修改 => alter

数据表 => table

基本语法：

```PowerShell
mysql> alter table 数据表名称 add 新字段名称 字段类型 first；
mysql> alter table 数据表名称 add 新字段名称 字段类型 after 其他字段名称;
选项说明：
first：把新添加字段放在第一位
after 字段名称：把新添加字段放在指定字段的后面
```

案例：在tb_article文章表中添加一个addtime字段，类型为date(年-月-日)

日期类型有两个：date（年-月-日）、time（小时:分钟:秒）、datetime（年-月-日 小时:分钟:秒）

```PowerShell
mysql> alter table tb_article add addtime date after content;
mysql> desc tb_article;
```

#### ② 修改字段名称或字段类型

修改字段名称与字段类型（也可以只修改名称）

```PowerShell
mysql> alter table tb_admin change username name varchar(40);
mysql> desc tb_admin;

说明：
change比较神奇，既可以修改字段名称，又可以修改字段类型！
change 源字段名称 修改后字段名称 修改后的字段类型（这个也可以不改，但是必须要写）
```

仅修改字段的类型

```PowerShell
mysql> alter table tb_admin modify name varchar(20);
mysql> desc tb_admin;

说明：
modify 字段名称 新字段类型（仅可以修改字段类型）
```

#### ③ 删除某个字段

```PowerShell
mysql> alter table tb_article drop 字段名称;
mysql> desc tb_article;
```

#### ④ 修改数据表名称

```PowerShell
rename table 旧名称 to 新名称;
```

### ☆ 删除数据表

英语小课堂：

删除 => drop

数据表 => table

```PowerShell
mysql> drop table 数据表名称;
```

扩展：查看数据库 与 数据表对应的编码格式

```PowerShell
-- 查看数据库编码格式
show create database 数据库名称;
-- 查看数据表编码格式
show create table 数据表名称;

show create database db_itheima;
show create table tb_article;
```

## 2、字段类型详解（重点）

整数int、浮点数decimal、字符串char/varchar/text、枚举类型enum、日期时间类型date、time、datetime

https://dev.mysql.com/doc/refman/8.0/en/data-types.html

① 整数类型

| **分类**     | **类型名称**   | **说明**                 |
| ------------ | -------------- | ------------------------ |
| tinyint      | 很小的整数     | -128 ~ 127               |
| smallint     | 小的整数       | -32768 ~ 32767           |
| mediumint    | 中等大小的整数 | -8388608 ~ 8388607       |
| int(integer) | 普通大小的整数 | -2147483648 ~ 2147483647 |

选择整数类型，往往存储范围！

age tinyint unsigned => 0 - 255，只需要记住tinyint和int即可！！！

举个例子：

```PowerShell
create table tb_student(
    id int,
    name varchar(20),
    age tinyint unsigned
) default charset=utf8;
```

**SQL优化手段：尽量选择满足这个字段范围区间的数据类型！！！**

② 浮点类型（小数类型）

浮点类型（精度失真情况）和定点类型（推荐使用定点类型）

| 分类         | 类型名称              |
| ------------ | --------------------- |
| float        | 单精度浮点数          |
| double       | 双精度浮点数          |
| decimal(m,d) | 定点数，decimal(10,2) |

> decimal(10,2) ：代表这个数的总长度为10 = 整数长度 + 小数长度，2代表保留2位小数

> decimal(5,2) ：999.99

建议：① 精度要求不高（2-4位小数的情况，影响不大），可以使用float/double

​      ② 如果精度要求比较高的情况，可以使用decimal

参考阅读：https://blog.csdn.net/weixin_45729934/article/details/121389109

案例：

```PowerShell
create table tb_product(
    id int,
    name varchar(100),
    price decimal(11,2)
) default charset=utf8;
```

③ 日期类型

| 份额里    | 类型名称                                                     |
| --------- | ------------------------------------------------------------ |
| year      | YYYY 1901~2155                                               |
| time      | HH:MM:SS -838:59:59~838:59:59                                |
| date      | YYYY-MM-DD 1000-01-01~9999-12-3                              |
| datetime  | YYYY-MM-DD HH:MM:SS 1000-01-01 00:00:00~ 9999-12-31 23:59:59 |
| timestamp | YYYY-MM-DD HH:MM:SS 1970-01-01 00:00:01 UTC~2038-01-19 03:14:07UTC |

选日期类型看格式，用的最多的是 => date、datetime

date：年-月-日

time：小时：分钟：秒数

datetime = date + time合并

案例：

```PowerShell
create table tb_news(
    id int,
    title varchar(100),
    addtime datetime
) default charset=utf8;
```

④ 文本（字符串类型）

| **类型名称**     | **说明**                                                     |
| ---------------- | ------------------------------------------------------------ |
| char(m)          | m为0~255之间的整数定长（固定长度），m代表字符长度，255代表不能超过255个字符 |
| varchar(m)       | m为0~65535之间的整数变长（变化长度），65535代表字节长度，和编码有关，utf8最多2万多个汉字 |
| text             | 允许长度0~65535字节                                          |
| mediumtext       | 允许长度0~167772150字节                                      |
| longtext（扩展） | 允许长度0~4294967295字节                                     |

选择文本，主要长度，固定char、变化varchar、还长的就是text。

varchar(m)：理论值65535个字符，但是理论上底层最大能存储的字节数也是65535。但是如果是汉字，和编码格式有关，gbk模式下，1个汉字2个字节，理论上大概能3万左右个汉字。utf8模式下，1个汉字3个字节，理论值就是2万多个汉字。

普通：enum('值1','值2','值3')，多选一，从多个值中选择其中的某一个！！！

## 3、字段约束详解（重点）

字段约束：在数据类型的基础上，给字段添加约束行为（如不能为空等等）

基本语法：

```PowerShell
create table 数据表名称(
     字段名称  字段类型  [可选，字段约束]
) default charset=utf8mb4;
```

在MySQL中，字段约束一共有五种约束：① 主键约束 ② 唯一约束 ③ 非空约束 ④ 默认值约束 ⑤ 外键约束

### ☆ 主键约束

强调：非空、唯一、一个表中有且仅有1个主键，在MySQL中，往往id字段都是主键（primary key）

article文章表（id主键，title标题，author作者，addtime发布时间）

```PowerShell
use db_itheima;
create table article(
    id int primary key,
    title varchar(80),
    author varchar(20),
    addtime date
) default charset=utf8mb4;

或

create table article(
    id int,
    title varchar(80),
    author varchar(20),
    addtime date,
    primary key(id)
) default charset=utf8mb4;
```

案例：

```PowerShell
use db_itheima;
create table article(
    id int primary key,
    title varchar(80),
    author varchar(20),
    addtime date
) default charset=utf8mb4;
-- 查看数据表
desc article;
-- 向表中插入数据
insert into article values (1, 'MySQL数据库运维', '张三', '2025-02-10');
-- 主键强制为null（错误演示）
insert into article values (null, 'Python运维开发', '李四', '2025-02-10');
-- 主键强制重复（错误演示）
insert into article values (1, 'Python运维开发', '李四', '2025-02-10');
```

### ☆ 唯一约束

强调：唯一，可以为空，一个表中可以同时拥有多个唯一约束

unique

**主键约束 与 唯一约束区别？**

和主键约束相同点：都是唯一的

和主键约束不同点：主键非空、有且仅有1个；唯一可以为空，在一个表中也可以同时拥有多个

案例：创建一个用户表，id主键，name，age，code（身份证号码，要求唯一）

```PowerShell
create table users(
        id int primary key,
        name varchar(20),
        age tinyint unsigned,
        code char(18) unique
) default charset=utf8;

-- 插入数据
insert into users values (1, '张三', 18, '123456789012345678');
-- 插入重复数据（错误演示）
insert into users values (2, '李四', 18, '123456789012345678');
-- 插入数据，但是唯一列为null
insert into users values (2, '王五', 18, null);
```

### ☆ 非空约束

强调：非空，这个列不允许出现null空值

not null

案例：创建一个商品表goods，有id主键，name商品名称，price商品价格，name和price都必须为非空！

```PowerShell
create table goods(
        id int primary key,
        name varchar(80) not null,
        price decimal(11,2) not null
) default charset=utf8mb4;

-- 插入数据
insert into goods values (1, 'Python从入门到放弃', 99.99);
-- 插入数据（错误演示）
insert into goods values (2, 'Java从入门到放弃', null);
```

### ☆ 默认值约束

强调：默认值，插入数据时，如果没有指定对应的值，则其自动使用默认值做为最终结果

default 默认值

案例：创建一个用户表users，id主键、name用户名、age年龄、gender性别，要求gender默认值为male

```PowerShell
drop table users;
create table users(
  id int primary key,
  name varchar(20),
  age tinyint unsigned,
  gender enum('male', 'female') default 'male'
) default charset=utf8mb4;
desc users;
-- 插入数据
insert into users values (1, '张三', 18, 'male');
-- 插入数据（触发默认值）
insert into users(id, name, age) values (2, '李四', 18);
insert into users values (3, '王五', 18, default);
-- 插入数据（不使用默认值）
insert into users values (4, '赵六', 18, 'female');
```

### ☆ 外键约束（了解）

强调：多表，级联操作

![8effe65b-297b-4331-841a-c3ad5f7b6130](8effe65b-297b-4331-841a-c3ad5f7b6130.png)

外键约束前提：① 多表 ② 数据表引擎必须为InnoDB引擎

小结：

MySQL语句中，一共有5种：（主键约束、唯一约束、非空约束、默认值约束、外键约束）

## 4、auto_increment自动增长（重点）

作用：自动增长，往往用于设置主键，可以让其值自动增长，默认从1开始，依次递增。

```PowerShell
drop table users;
create table users(
  id int auto_increment primary key,
  name varchar(20),
  age tinyint unsigned,
  gender enum('male', 'female') default 'male'
) default charset=utf8mb4;

insert into users values (null, '张三', 23, 'male');
insert into users values (null, '李四', 24, 'male');
delete from users where id = 2;
insert into users values (null, '王五', 25, 'male');
```

# 三、DML数据操作语言

## 1、DML包括哪些SQL语句

insert插入、update更新、delete删除

## 2、数据的增删改（重点）

英语小课堂：

增加：insert

删除：delete

修改：update

### ☆ 数据的增加操作

基本语法：

```PowerShell
mysql> insert into 数据表名称([字段1,字段2,字段3...]) values (字段1的值,字段2的值,字段3的值...);
```

> 特别注意：在SQL语句中，除了数字，其他类型的值，都需要使用引号引起来，否则插入时会报错。

第一步：准备一个数据表

```PowerShell
mysql> use db_itheima;
mysql> create table tb_user(
        id int,
        username varchar(20),
        age tinyint unsigned,
        gender enum('男','女','保密'),
        address varchar(255)
) engine=innodb default charset=utf8;
```

> unsigned代表无符号型，只有0到正数。tinyint unsigned无符号型，范围0 ~ 255

> enum枚举类型，多选一。只能从给定的值中选择一个

第二步：使用insert语句插入数据

```PowerShell
mysql> insert into tb_user values (1,'刘备',34,'男','广州市天河区');
mysql> insert into tb_user(id,username,age) values (2,'关羽',33);
```

第三步：批量插入多条数据

```PowerShell
mysql> insert into tb_user values (3,'大乔',19,'女','上海市浦东新区'),(4,'小乔',18,'女','上海市浦东新区'),(5,'马超',26,'男','北京市昌平区');
```

### ☆ 数据的修改操作

基本语法：

```PowerShell
mysql> update 数据表名称 set 字段1=更新后的值,字段2=更新后的值,... [where 更新条件];

where：更新条件，满足条件的才会被更新，如果不写where则默认修改所有记录！
```

> 特别说明：如果在更新数据时，不指定更新条件，则其会把这个数据表的所有记录全部更新一遍。

案例：修改username='赵飞'这条记录，将其性别更新为男，家庭住址更新为广东省深圳市

```PowerShell
mysql> update tb_user set gender='男',address='广东省深圳市' where username='赵飞';
```

案例：今年是2025年，假设到了2026年，现在存储的学员年龄都差1岁，整体进行一次更新

```PowerShell
mysql> update tb_user set age=age+1;
```

### ☆ 数据的删除操作

基本语法：

```PowerShell
mysql> delete from 数据表名称 [where 删除条件];
```

案例：删除tb_user表中，id=1的用户信息

```PowerShell
mysql> delete from tb_user where id=1;
```

delete from与truncate清空数据表操作

```PowerShell
mysql> delete from 数据表;
或
mysql> truncate 数据表;
```

**面试题**：**delete from与truncate区别在哪里？**

- delete：删除数据记录
  - 数据操作语言（DML）
  - 删除大量记录速度慢，只删除数据，**主键自增序列不清零**，1-100 => 新插入 => 101
  - 可以带条件删除
- truncate：删除所有数据记录
  - 数据定义语言（DDL）
  - 清里大量数据速度快，**主键自增序列清零, 1-100 => 新插入 => 1**
  - 不能带条件删除

小结：

DML：数据操作语言，主要涉及增加（INSERT）、修改（UPDATE）、删除（DELETE）

# 四、DQL数据查询语言（核心）

## 1、数据集准备

```SQL
CREATE TABLE product
(
    pid         INT PRIMARY KEY,
    pname       VARCHAR(20),
    price       DOUBLE,
    category_id VARCHAR(32)
) DEFAULT CHARSET=utf8;

pid：p==product，pid==产品id编号
pname：产品名称
price：产品价格
category_id：产品所属分类编号（c001电子产品，c002服装）
```

插入数据：

```SQL
INSERT INTO product VALUES (1,'联想',5000,'c001');
INSERT INTO product VALUES (2,'海尔',3000,'c001');
INSERT INTO product VALUES (3,'雷神',5000,'c001');
INSERT INTO product VALUES (4,'杰克琼斯',800,'c002');
INSERT INTO product VALUES (5,'真维斯',200,'c002');
INSERT INTO product VALUES (6,'花花公子',440,'c002');
INSERT INTO product VALUES (7,'劲霸',2000,'c002');
INSERT INTO product VALUES (8,'香奈儿',800,'c003');
INSERT INTO product VALUES (9,'相宜本草',200,'c003');
INSERT INTO product VALUES (10,'面霸',5,'c003');
INSERT INTO product VALUES (11,'好想你枣',56,'c004');
INSERT INTO product VALUES (12,'香飘飘奶茶',1,'c005');
INSERT INTO product VALUES (13,'海澜之家',1,'c002');
```

> DataGrip软件关键字替换，可以使用Ctrl + R快捷键

## 2、select查询（核心）

```PowerShell
# 根据某些条件从某个表中查询指定字段的内容
格式：select [distinct]*| 列名,列名 from 表 where 条件

select 查询哪些列多个列用逗号隔开 from 数据表 where 查询条件（满足条件的就显示，不满足的就忽略）
```

## 3、简单查询

```Plaintext
# 1.查询所有的商品.  
select * from product;
# 2.查询商品名和商品价格. 
select pname,price from product;
# 3.查询结果是表达式（运算查询）：将所有商品的价格+10元进行显示.
select pname,price+10 from product;
# 4.查询所有商品对应的商品分类信息（不能重复）
select distinct category_id from product;
```

SQL除了简单查询以外，还支持五子句查询（SQL查询五子句）

```PowerShell
select */字段 from 数据表 ① where子句 ② group by子句 ③ having子句 ④ order by子句 ⑤ limit子句

① where：条件查询
② group by：分组查询
③ having：条件查询，只不过发生在分组之后，可以对分组后结果进行筛选
④ order by：排序子句，用于排序操作
⑤ limit：限制查询，用于限制查询数量
五子句可以单独出现，也可以多个关键词一起出现。但是不管多少个关键字都必须严格按照五子句顺序进行书写，否则报错！！！
```

## 4、条件查询（where子句）

作用：五子句的一部分

select * from 数据表 where子句 group by分组子句 having子句 order by子句 limit子句

![5f0e8972-8d17-4031-aaae-8ba3cd783f50](5f0e8972-8d17-4031-aaae-8ba3cd783f50.png)

### ☆ 比较查询

```PowerShell
# 查询商品名称为“花花公子”的商品所有信息：
SELECT * FROM product WHERE pname = '花花公子';
# 查询价格为800商品
SELECT * FROM product WHERE price = 800;
# 查询价格不是800的所有商品
SELECT * FROM product WHERE price != 800;
SELECT * FROM product WHERE price <> 800;
# 查询商品价格大于60元的所有商品信息
SELECT * FROM product WHERE price > 60;
# 查询商品价格小于等于800元的所有商品信息
SELECT * FROM product WHERE price <= 800;
```

### ☆ 范围查询

```PowerShell
# 查询商品价格在200到1000之间所有商品
SELECT * FROM product WHERE price BETWEEN 200 AND 1000;
# 查询商品价格是200或800的所有商品
SELECT * FROM product WHERE price IN (200,800);
```

### ☆ 逻辑查询

```PowerShell
# 查询商品价格在200到1000之间所有商品
SELECT * FROM product WHERE price >= 200 AND price <=1000;
# 查询商品价格是200或800的所有商品
SELECT * FROM product WHERE price = 200 OR price = 800;
# 查询价格不是800的所有商品
SELECT * FROM product WHERE NOT(price = 800);
```

### ☆ 模糊查询

字段 like '匹配条件'

匹配条件有两个符号：%任意个任意字符，_只匹配任意某1个字符

```PowerShell
# 查询以'香'开头的所有商品
SELECT * FROM product WHERE pname LIKE '香%';
# 查询第二个字为'想'的所有商品
SELECT * FROM product WHERE pname LIKE '_想%';
```

### ☆ 空值查询 与 非空查询

where 字段=null（错误）

where 字段 is null（正确）

```PowerShell
# 查询没有分类的商品
SELECT * FROM product WHERE category_id IS NULL;
# 查询有分类的商品
SELECT * FROM product WHERE category_id IS NOT NULL;
```

## 5、聚合查询

前置知识点：as关键词，用于给数据表 或 数据表中的字段定义别名

```PowerShell
# 数据表定义别名
select 别名.A,别名.B from 数据表 [as] 别名;

# 字段定义别名
select name, (chinese+english+math) as 别名 from 数据表;
```

作用：之前我们做的查询都是横向查询，它们都是根据条件一行一行的进行判断，而使用聚合函数查询是纵向查询，它是对一列的值进行计算，然后返回一个单一的值；另外聚合函数会忽略空值。

聚合查询：① 纵向查询（按列查询） ② 默认会忽略空值

今天我们学习如下五个聚合函数：

| **聚合函数** | **作用**                                                     |
| ------------ | ------------------------------------------------------------ |
| count()      | 统计指定列不为NULL的记录行数；                               |
| sum()        | 计算指定列的数值和，如果指定列类型不是数值类型，则计算结果为0 |
| max()        | 计算指定列的最大值，如果指定列是字符串类型，使用字符串排序运算； |
| min()        | 计算指定列的最小值，如果指定列是字符串类型，使用字符串排序运算； |
| avg()        | 计算指定列的平均值，如果指定列类型不是数值类型，则计算结果为0 |

实现原理：

![4b7cccd9-8e33-4ed0-a72e-28f8a5dae872](4b7cccd9-8e33-4ed0-a72e-28f8a5dae872.png)

案例演示：

```PowerShell
# 1、查询商品的总条数
SELECT COUNT(*) FROM product;
# 2、查询价格大于200商品的总条数
SELECT COUNT(*) FROM product WHERE price > 200;
# 3、查询分类为'c001'的所有商品的总和
SELECT SUM(price) FROM product WHERE category_id = 'c001';
# 4、查询分类为'c002'所有商品的平均价格
SELECT AVG(price) FROM product WHERE category_id = 'c002';
# 5、查询商品的最大价格和最小价格
SELECT MAX(price),MIN(price) FROM product;
```

## 6、分组查询（重点）

作用：分组就是为了更好的进行数据的统计，分组 + 聚合。

按性别分组、按学科分组、按部门分组、按年级分组 => group by 分组字段（按这个字段进行划分分组）

光有分组一般没有特别的意义，但是有一个特性：去重

**面试过程中：介绍一下，在SQL语句中，如何实现去重操作？**

答：我了解的一共有两种方式，distinct 或 group by

### ☆ 分组查询介绍

分组查询就是将查询结果按照指定字段进行分组，字段中数据相等的分为一组。

**分组查询基本的语法格式如下：**

GROUP BY 列名 [HAVING 条件表达式]

**说明:**

- 列名: 是指按照指定字段的值进行分组。
- HAVING 条件表达式: 用来过滤分组后的数据。

### ☆ group by的使用

group by可用于单个字段分组，也可用于多个字段分组

```SQL
-- 准备学生表students
create table students(
    id int auto_increment,
    name varchar(20),
    age tinyint unsigned,
    gender enum('male', 'female'),
    score tinyint,
    primary key(id)
) default charset=utf8;
insert into students values (null, 'Tom', 23, 'male', 97);
insert into students values (null, 'Jack', 24, 'male', 88);
insert into students values (null, 'Rose', 26, 'female', 99);
insert into students values (null, 'Eric', 27, 'male', 59);
insert into students values (null, 'Jennifer', 22, 'female', 76);

-- 根据gender字段来分组
select gender from students group by gender;
-- 根据name和gender字段进行分组
select name, gender from students group by name, gender;
```

① group by可以实现去重操作

② group by的作用是为了实现分组统计（group by + 聚合函数）

### ☆ group by + 聚合函数的使用

```SQL
-- 统计不同性别的人的平均年龄
select gender,avg(age) from students group by gender;
-- 统计不同性别的人的个数
select gender,count(*) from students group by gender;
```

![c9a6e500-012c-4080-a73d-aed862ff107a](c9a6e500-012c-4080-a73d-aed862ff107a.png)

group by使用注意事项：

```PowerShell
select 分组字段,聚合函数 from students group by 分组字段;

SQL官方文档：在有group by出现的情况下，select后面的字段要么只能出现在分组中，要么只能出现在聚合函数中
```

## 7、having的使用

having作用和where类似都是过滤数据的，但是两者之间的执行顺序不同

① where子句（发生在分组之前） ② group by子句 ③ having子句（发生在分组之后）

第一种情况：如果只是简单的查询操作（没有group by的情况），大部分时间having是可以直接替代where子句

```PowerShell
select * from product where price > 800;
```

以上语句等价于

```PowerShell
select * from product having price > 800;
```

第二种情况：

```SQL
-- 根据gender字段进行分组，统计分组条数大于2的
select gender,count(*) from students group by gender having count(*)>2;
```

案例演示：

```PowerShell
#1 统计各个分类商品的个数
SELECT category_id ,COUNT(*) FROM product GROUP BY category_id ;

#2 统计各个分类商品的个数,且只显示个数大于1的信息
SELECT category_id ,COUNT(*) FROM product GROUP BY category_id HAVING COUNT(*) > 1;
```

## 8、排序查询（order by子句）

```PowerShell
# 通过order by语句，可以将查询出的结果进行排序。暂时放置在select语句的最后。
格式：SELECT * FROM 表名 ORDER BY 排序字段 ASC|DESC;
ASC  升序 (默认)，默认情况下，ASC关键词可以省略不写
DESC 降序

# 1.使用价格排序(降序)
SELECT * FROM product ORDER BY price DESC;

# 2.在价格排序(降序)的基础上，以分类排序(降序)
SELECT * FROM product ORDER BY price DESC,category_id DESC;
# 首先按照第一个字段排序，如果第一个字段能比较出大小，则不需要进行字段2排序；如果第一个字段值相同，则系统会继续按照第二个字段进行排序
```

## 9、limit子句的使用

应用场景：① 限制查询  ② 分页查询

限制查询：主要限制数据查询的数量（获取数据表中的前3条数据）

```
select * from 数据表 limit ``查询数量``;
select * from 数据表 limit ``偏移量(从哪查默认为0，类似索引下标)``，``查询数量``;
```

偏移量：索引下标，默认从0开始

0  第一条记录

1  第二条记录

2  第三条记录

```PowerShell
-- 基本语法1：select * from 数据表 limit 查询数量;
-- 案例1：查询学生表中，成绩最高的学生信息（只查1个）
select * from students order by score desc limit 0,1;
select * from students order by score desc limit 1;
-- 缺点说明：如果一个学生表有多个同学成绩相同，可能会导致，只能查出1名同学。

-- 基本语法2：select * from 数据表 limit 偏移量(从哪查默认为0，类似索引下标)，查询数量;
-- 案例2：查询学生表中，成绩第2高（第2名）的学生信息
select * from students order by score desc limit 1,1;
```

分页查询：实际特别接近，因为只要有分页的地方，底层100%都是使用limit子句实现的

![2e10c30a-bf3a-41d6-8d89-7975207cd3d9](2e10c30a-bf3a-41d6-8d89-7975207cd3d9.png)

分页查询在项目开发中常见，由于数据量很大，显示屏长度有限，因此对数据需要采取分页显示方式。例如数据共有30条，每页显示5条，第一页显示1-5条，第二页显示6-10条。

![5e005013-7cc8-4586-b0f3-8754cb49c65f](5e005013-7cc8-4586-b0f3-8754cb49c65f.png)

![30bd1f5d-57dd-4865-bcf8-710c44a62497](30bd1f5d-57dd-4865-bcf8-710c44a62497.png)

格式：

```PowerShell
SELECT 字段1，字段2... FROM 表名 LIMIT M,N
M: 整数，表示从第几条索引开始，计算方式 （当前页-1）*   每页显示条数
N: 整数，表示查询多少条数据
SELECT 字段1，字段2... FROM 表明 LIMIT 0,5
SELECT 字段1，字段2... FROM 表明 LIMIT 5,5
```

## 10、小结

```PowerShell
SQL查询五子句分别为：（where子句）、（group by子句）、（having子句）、（order by子句）、（limit子句）

----------------------------------------------------------------------
条件查询：select *|字段名 form 表名 where 条件；
聚合查询函数：count()，sum()，max()，min()，avg()。
分组查询：SELECT 字段1,字段2… FROM 表名 GROUP BY 分组字段 HAVING 分组条件;
排序查询：SELECT * FROM 表名 ORDER BY 排序字段 ASC|DESC;
分页查询：
SELECT 字段1，字段2... FROM 表名 LIMIT M,N
M: 整数，表示从第几条索引开始，计算方式 （当前页-1）*每页显示条数
N: 整数，表示查询多少条数据
```

# 五、多表查询（重点）

## 表与表之间的关系

在SQL语句中，数据表与数据表之间，如果存在关系，一般一共有3种情况：

① 1：1，一对一关系（高级）

比如有A、B两张表，A表中的每一条数据，在B表中有一条唯一的数据与之对应。

用户表user

| user_id（用户编号） | 账号username | 密码password |
| ------------------- | ------------ | ------------ |
| 001                 | admin        | admin888     |
| 002                 | itheima      | 123456       |

用户详情表user_items

| user_id（用户编号） | 真实姓名 | 年龄 | 联系方式 |
| ------------------- | -------- | ---- | -------- |
| 001                 | 张三     | 16   | 10086    |
| 002                 | 李四     | 18   | 10010    |

我们把用户表与用户详情表之间的关系就称之为一对一关系。

② 1：N，一对多关系（重点）

比如有A、B两张表，A表中的每一条数据，在B表中都有多条数据与之对应，我们把这种关系就称之为一对多关系

产品分类表

| 分类id编号 | 分类名称 |
| ---------- | -------- |
| 1          | 手机     |
| 2          | 电脑     |

产品信息表

| 产品id编号 | 产品名称        | 产品价格 | 所属分类id编号 |
| ---------- | --------------- | -------- | -------------- |
| 1          | Apple iPhone 13 | 6799.00  | 1              |
| 2          | Redmi Note 9    | 3499.00  | 1              |

我们把产品分类表与产品表之间的关系就称之为一对多关系。

③ M：N，多对多关系（高级）

用户表

| 用户编号 | 登录账号 | 登录密码 |
| -------- | -------- | -------- |
| 1        | admin    | admin888 |
| 2        | itheima  | 123456   |

权限表

| 权限id编号 | 权限名称 |
| ---------- | -------- |
| 1          | 增加     |
| 2          | 删除     |
| 3          | 修改     |
| 4          | 查询     |

虽然从以上图解来看，两者之间好像没有任何联系，但是两者之间其实是有关系的，这种关系需要通过一张临时表进行呈现。

每个用户，应该有对应的权限，admin账号可以做增删改查，itheima账号可以做查询

反过来

每个权限都应该对应多个用户，查询权限 => admin/itheima

注意：如果两张表之间的关联关系为多对多关系，则必须建立一个中间表，在中间表中体现两者的关系。

中间表 ：用户_权限表

| 用户id编号 | 权限的id编号 |
| ---------- | ------------ |
| 1（admin） | 1（增加）    |
| 1          | 2（删除）    |
| 1          | 3（修改）    |
| 1          | 4（查询）    |
| 2          | 4（查询）    |

## 交叉连接(了解)

准备数据集：

```PowerShell
-- 1. 准备数据集
-- 数据表classes，拥有两个字段，cls_id代表班级编号，cls_name代表班级名称
use db_itheima;
drop table classes;
create table classes(
    cls_id int,
    cls_name varchar(20)
) default charset=utf8mb4;
-- 插入数据，ui、java、python
insert into classes values
    (1, 'UI'),
    (2, 'Java'),
    (3, 'Python');

-- 数据表students，拥有id、name、age、gender值使用male和female、score、cls_id
drop table students;
create table students(
    id int,
    name varchar(20),
    age int,
    gender enum('male', 'female'),
    score decimal(11,2),
    cls_id int
) default charset=utf8mb4;

-- 插入数据，刘备属于Java，貂蝉属于UI，赵云属于Python，关羽属于Python，大乔属于UI
insert into students values
    (1, '刘备', 18, 'male', 100.00, 2),
    (2, '貂蝉', 18, 'female', 99.00, 1),
    (3, '赵云', 18, 'male', 98.00, 3),
    (4, '关羽', 18, 'male', 96.00, 3),
    (5, '大乔', 18, 'female', 97.00, 1);
```

没有意义，但是它是所有连接的基础。其功能就是将表1和表2中的每一条数据进行连接。

结果：

字段数 = 表1字段 + 表2的字段

记录数 = 表1中的总数量 * 表2中的总数量（笛卡尔积）

交叉连接也称之为笛卡尔积连接

```PowerShell
select * from students cross join classes;
或
select * from students, classes;
```

![32073b1b-6b30-46a1-8d79-d371bd00346c](32073b1b-6b30-46a1-8d79-d371bd00346c.png)

## 1、内连接（重点）

### ☆ 连接查询的介绍

连接查询可以实现多个表的查询，当查询的字段数据来自不同的表就可以使用连接查询来完成。

连接查询可以分为:

1. 内连接查询
2. 左外连接查询
3. 右外连接查询
4. 自连接查询（自己查询自己）

### ☆ 内连接查询

查询两个表中符合条件的共有记录

![fa7076cb-9726-4ae5-9265-33b17f03188e](fa7076cb-9726-4ae5-9265-33b17f03188e.png)

**内连接查询语法格式: inner join ... on 关联条件**

```SQL
select 字段 from 表1 inner join 表2 on 表1.字段1 = 表2.字段2
```

**说明:**

- inner join 就是内连接查询关键字
- on 就是连接查询条件

**例1：使用内连接查询学生表与班级表，查询每个学生对应的具体班级信息。**

```SQL
-- 案例：求每个学生所属的班级信息（学生所有字段 + 班级名称）
select * from students inner join classes on students.cls_id = classes.cls_id;
-- 内连接基础上可以筛选想要的字段
select students.*, classes.cls_name from students inner join classes on students.cls_id = classes.cls_id;
-- 引入表别名机制，数据表名称 [as] 别名
select s.name, s.age, s.gender, c.cls_name from students s inner join classes c on s.cls_id = c.cls_id;
-- 通过换行+缩进提升可读性，inner join on还可以简写为join on
select
  s.name,
  s.age,
  s.gender,
  c.cls_name
from students as s
join classes as c
on s.cls_id = c.cls_id;
```

### ☆ 小结

- 内连接使用inner join .. on .., on 表示两个表的连接查询条件
- 内连接根据连接查询条件取出两个表的 “交集”，也就是满足关联条件的结果。

## 2、左外连接

### ☆ 左连接查询

> 有一个主表概念，默认情况下，关联后会保留主表的所有记录。

以左表为主根据条件查询右表数据，如果根据条件查询右表数据不存在使用null值填充

商品分类表

| 编号 | 分类名称 |
| ---- | -------- |
| 1    | 手机     |
| 2    | 电脑     |

商品表

| 编号 | 商品名称   | 商品分类编号 |
| ---- | ---------- | ------------ |
| 1    | Vivo手机   | 1            |
| 2    | Xiaomi手机 | 1            |

左外连接查询：select * from 分类表（主表） left join 商品表 on 分类表.编号 = 商品表.分类编号;

| 编号 | 分类名称 | 编号 | 商品名称   | 商品分类编号 |
| ---- | -------- | ---- | ---------- | ------------ |
| 1    | 手机     | 1    | Vivo手机   | 1            |
| 1    | 手机     | 2    | Xiaomi手机 | 1            |
| 2    | 电脑     | null | null       | null         |

左外连接默认会保留左表，然后与右边的表进行匹配；如果匹配到，则显示右表对应的数据；匹配不到，也要显示。只不过右表的所有字段使用null进行填充！

![f2866b93-a4a0-415e-888e-ab6241e47fa5](f2866b93-a4a0-415e-888e-ab6241e47fa5.png)

**左连接查询语法格式:**

```SQL
select 字段 from 表1 left join 表2 on 表1.字段1 = 表2.字段2
```

**说明:**

- left join 就是左连接查询关键字
- on 就是连接查询条件
- 表1 是左表
- 表2 是右表

**例1：使用左连接查询学生表与班级表，获取每一学生对应的班级信息，要求没有对应的班级也要显示。**

```SQL
-- 使用左连接查询学生表与班级表，获取每一学生对应的班级信息，要求没有对应的班级也要显示。
insert into students values (6, '曹操', 30, 'male', 98, 9);
select s.*,c.cls_name from students s left join classes c on s.cls_id = c.cls_id;
```

### ☆ 小结

- 左连接使用left join .. on .., on 表示两个表的连接查询条件
- 左连接以左表为主根据条件查询右表数据，右表数据不存在使用null值填充。

## 3、右外连接

### ☆ 右连接查询

以右表为主根据条件查询左表数据，如果根据条件查询左表数据不存在则使用null值填充

![0a79c901-1788-44e8-b761-d52f29395367](0a79c901-1788-44e8-b761-d52f29395367.png)

**右连接查询语法格式:**

```SQL
select 字段 from 表1 right join 表2（主表） on 表1.字段1 = 表2.字段2
```

**说明:**

- right join 就是右连接查询关键字
- on 就是连接查询条件
- 表1 是左表
- 表2 是右表

**例1：使用右连接查询学生表与班级表:**

```SQL
-- 查看班级中对应的学生信息，如果某个班级没有对应的学生也要显示。（既可以用左外连接也可以用右外连接）
select s.*,c.cls_name from students s right join classes c on s.cls_id = c.cls_id;
select s.*,c.cls_name from classes c right join students s on c.cls_id = s.cls_id;
```

### ☆ 小结

- 右连接使用right join .. on .., on 表示两个表的连接查询条件
- 右连接以右表为主根据条件查询左表数据，左表数据不存在使用null值填充。

## 4、自连接查询(扩展)

自连接查询：数据表自己连接自己，本质只有1张数据表，表内部数据有层级关系。

前提：连接操作时必须为数据表定义别名！

左表和右表是同一个表，根据连接查询条件查询两个表中的数据。

两个实际的工作场景：求**省市区信息，求分类导航信息**

① 省份表 + 城市表 + 区域表（理论，实际设计表的过程中，通常只有1张数据表）

② 大类 + 中类 + 小类（理论，实际设计表的过程中，通常只有1张数据表）

地域：title

pid 全称 parent id（父级ID编号），如果pid值为null代表本身就是父级，如果pid是一个具体的数值，则代表其属于子级

![9b05425f-a72f-42bc-bd7f-bd42519c7e0f](9b05425f-a72f-42bc-bd7f-bd42519c7e0f.png)

**例1：查询省的名称为“广东省”的所有城市**

**创建areas表:**

```SQL
use db_itheima;
create table areas(
    aid int not null AUTO_INCREMENT,
    atitle varchar(20),
    pid int,
    primary key(aid)
) default charset=utf8;
```

**执行sql文件给areas表导入数据:**

```SQL
insert into areas values (null, '广东省', null),(null, '山西省', null),(null, '深圳市', 1), (null, '广州市', 1), (null, '太原市', 2), (null, '大同市', 2);
```

![6308e3fa-7b4a-49f0-b32b-9f94dc0e3134](6308e3fa-7b4a-49f0-b32b-9f94dc0e3134.png)

**自连接查询的用法:**

```SQL
-- 第一步：把两张表关联查询（内连接）
select * from areas p inner join areas c on p.aid = c.pid;
-- 第二步：在以上查询基础上，获取省份名称为广东省
select * from areas p inner join areas c on p.aid = c.pid where p.atitle = '广东省';
-- 第三步：只显示城市信息
select c.* from areas p inner join areas c on p.aid = c.pid where p.atitle = '广东省';
```

**说明:**

- 自连接查询必须对表起别名

### ☆ 小结

- 自连接查询就是把一张表模拟成左右两张表，然后进行连表查询。
- 自连接就是一种特殊的连接方式，连接的表还是本身这张表

# 六、子查询（了解）

## 1、子查询（嵌套查询）的介绍

在一个 select 语句中,嵌入了另外一个 select 语句, 那么被嵌入的 select 语句称之为子查询语句，外部那个select语句则称为主查询语句。

select * from (select * from xxx) as t;

​                  子查询

主查询

作用：子查询比较适合复杂查询以及多层级查询结构。

**主查询和子查询的关系:**

1. 子查询是嵌入到主查询中
2. 子查询是辅助主查询的,要么充当条件,要么充当数据源(数据表) => 要么出现在where，要么出现在from位置
3. 子查询是可以独立存在的语句,是一条完整的 select 语句

了解：子查询的应用场景

答：在我们需求的基础上，如果这个需求需要通过多条SQL语句分步查询的情况，一般都需要基于子查询。

## 2、子查询的使用（三步走）

数据集：

```PowerShell
-- 1. 准备数据集
drop table student;
-- 创建student学生表，包含id、name、age、gender、score
create table student(
    id int,
    name varchar(20),
    age int,
    gender enum('male', 'female'),
    score decimal(11,2)
) default charset=utf8mb4;
-- 插入7条测试数据
insert into student values
    (1, '张三', 18, 'male', 100.00),
    (2, '李四', 19, 'male', 99.00),
    (3, '王五', 20, 'male', 98.00),
    (4, '赵六', 22, 'male', 97.00),
    (5, '钱七', 24, 'male', 96.00),
    (6, '孙八', 30, 'male', 95.00),
    (7, '周九', 26, 'male', 94.00);
```

**例1. 查询学生表中大于平均年龄的所有学生:**

需求：查询年龄 > 平均年龄的所有学生

前提：① 获取班级的平均年龄值

 ② 查询表中的所有记录，判断哪个同学 > 平均年龄值

第一步：写子查询

```PowerShell
select avg(age) from students;
```

第二步：写主查询

```PowerShell
select * from students where age > (平均值);
```

第三步：第一步和第二步进行合并

```PowerShell
select * from students where age > (select avg(age) from students);
```

**例2. 查询goods产品表中具有分类信息的产品**

数据集：

```PowerShell
create table category(
    cid int,
    cname varchar(20)
) default charset=utf8mb4;
insert into category values
    (1, '手机'),
    (2, '电脑'),
    (3, '图书');

create table goods(
    id int,
    name varchar(20),
    price decimal(11,2),
    cid int
) default charset=utf8mb4;
insert into goods values
    (1, '华为手机', 5000.00, 1),
    (2, '小米手机', 3000.00, 1),
    (3, '苹果手机', 10000.00, 1),
    (4, '华硕电脑', 12000.00, 2),
    (5, '戴尔电脑', 11000.00, 2),
    (6, 'Surface电脑', 13000.00, 2),
    (7, 'Java从入门到放弃',100.00, 3),
    (8, 'HP激光打印机',999.00, 9);
```

需求：查询产品表中具有分类信息的产品（没有与之对应分类信息的产品不显示）

前提：

① 查询分类表中，到底有哪些分类（获取cid编号）

 ② 到产品表中进行判断，判断这个商品的cid编号与①中的是否相等

第一步：编写子查询

```PowerShell
select cid from category;
```

第二步：编写主查询

```PowerShell
select * from goods where cid in (所有分类cid编号)
```

第三步：把主查询和子查询合并

```SQL
select * from goods where cid in (select cid from category);
```

**例3. 查找年龄最小且成绩最高的学生**

注意：同时满足以上条件（年龄**最小且成绩最高**）的数据可能并不存在，此题主要是为了介绍子查询的特殊用法。

第一步：获取年龄最小值和成绩最高值

```PowerShell
select min(age), max(score) from student;
```

第二步：查询所有学员信息（主查询）

```SQL
select * from students where (age, score) = (最小年龄, 最高成绩);
```

第三步：把第一步和第二步合并

```PowerShell
select * from student where (age, score) = (select min(age), max(score) from student);
```

> 注：数据表中必须有这样一条记录，否则可能查询不到结果，重点练习子查询返回多个结果情况。

## 3、小结

子查询是一个完整的SQL语句，子查询被嵌入到一对小括号里面，子查询可以充当条件，也可以充当数据表使用。

掌握子查询编写三步走：① 编写子查询 ② 编写主查询并融合伪代码 ③ 伪代码替换为子查询 

# 七、DCL用户与权限管理（重点）

## 1、创建用户(create user)

注意：MySQL中不能单纯通过用户名来说明用户，必须要加上主机，如jack@10.1.1.1

语法

```PowerShell
创建用户设置密码
create user 'user'@'host' identified by 'password';

说明：用户的信息保存在mysql数据库中的user表中，验证用户是否创建成功如下：
select user,host from mysql.user; 

说明：SQL中的点号
select 数据表.字段 from 数据表;
select * from 数据库.数据表;
```

示例

```PowerShell
create user 'tom'@'localhost' identified by '123456';
create user 'harry'@'localhost' identified by '123456';
create user 'tom'@'192.168.88.100' identified by '123456';
create user  'jack'@'%' identified by '123456';
```

用户主机表示方式

```PowerShell
'user'@'localhost'                     表示user只能在本地通过socket登录数据库
'user'@'192.168.0.1'                   表示user用户只能在192.168.0.1登录数据库
'user'@'192.168.0.0/24'                表示user用户可以在该网络任意的主机登录数据库
'user'@'%'                             表示user用户可以在所有的机器上登录数据库;本机为匿名用户
```

## 2、修改用户(alter user)

使用 `ALTER USER` 语句来修改现有用户的信息，如更改密码、主机等。

前提：必须要保证账号存在。

修改用户密码

```PowerShell
alter user 'username'@'host' identified by 'new_password';
```

示例

```PowerShell
alter user 'tom'@'localhost' identified by '654321';
```

## 3、删除用户(drop user)

语法

```PowerShell
drop user 用户;
```

示例

```PowerShell
删除'user01'@'localhost'用户
mysql> drop user 'user01'@'localhost';

重命名用户名
mysql> rename user 'harry'@'10.1.1.%' to 'harry'@'10.1.1.1';

删除一个匿名用户
mysql> drop user ''@'localhost';

删除mysql中的匿名用户
mysql> delete from mysql.user where user='';
删除root用户从本机::1登录（::1表示IPv6地址）
mysql> delete from mysql.user where user='root' and host='::1';
mysql> flush privileges;

注意：如果tcp/ip登录，服务器端口不是默认3306，则需要加端口号
```

## 4、用户权限管理(grant)

本地连接：

![91e3e5d5-73b0-4f83-aa10-50555ba8172d](91e3e5d5-73b0-4f83-aa10-50555ba8172d.png)

远程连接：

![6afa3fae-81af-40b0-adf6-1ff65571119f](6afa3fae-81af-40b0-adf6-1ff65571119f.png)



所有权限说明https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html

```PowerShell
USAGE        无权限,只有登录数据库,只可以使用test或test_*数据库
ALL          所有权限

以下权限为指定权限
select/update/delete/super/replication slave/reload...

with grant option 选项表示允许把自己的权限授予其它用户或者从其他用户收回自己的权限
```

创建一个admin管理员（要求回到黑窗口执行，因为默认root@%没有with grant option权限）

```PowerShell
create user 'admin'@'%' identified by '123456';
grant all on *.* to 'admin'@'%' with grant option;

注意：分配权限时只能分配<=当前用户的权限，比如当前账号只有insert/delete/update/select，则这个账号只能给其他用户分配insert/delete/update/select。
```

权限保存位置

```PowerShell
mysql.user：记录全局权限（影响所有数据库和表）的用户账号信息。
mysql.db：记录数据库级权限，控制用户对某个数据库（db.*）的操作权限。
mysql.tables_priv：记录表级权限，控制用户对特定表（db.table）的权限。
mysql.columns_priv：记录列级权限，控制用户对特定列（db.table.col）的权限。
mysql.procs_priv：记录存储过程和存储函数的权限，控制用户对存储过程或函数的执行和修改权限。
```

给用户授权

① 语法

```PowerShell
grant 权限1,权限2 on 库.表 to 用户@主机
grant 权限(列1,列2,...) on 库.表 to 用户@主机
```

② 用户授权示例

```PowerShell
给tom@10.1.1.1用户授予查看db01库里所有表权限
mysql> grant select on db_itheima.* to 'tom'@'localhost';
刷新权限表
mysql> flush privileges;

给tom@10.1.1.1用户授予修改db01库的ID字段的权限
mysql> grant update(id) on db_itheima.student to 'tom'@'localhost';
mysql> flush privileges;
查看当前用户权限
mysql> show grants;
查看指定用户权限
mysql> show grants for 'tom'@'localhost';
```

③ 总结

创建用户方法

- `create user...` 需要单独grant授权
- `grant xxx` 直接创建用户并授权，添加with grant option就可以给其他用户分配小于等于自身的权限了。

## 5、回收用户权限

```PowerShell
revoke 权限 on 库.表 from 用户;
撤消指定的权限
mysql> revoke update on db_itheima.student from 'tom'@'localhost';
撤消所有的权限
mysql> revoke all privileges, grant option from 'admin'@'%';
```

工作中使用最多的就是2个功能：① 创建用户 ② grant分配权限

```PowerShell
create user 'xxx'@'允许访问的主机地址' identified by '密码';
grant all on *.* to 'xxx'@'允许访问的主机地址' with grant option;
```

DDL、DML、DQL、DCL（SQL体系讲解完毕了！）

# 八、MySQL体系结构（理解）

面试题：MySQL底层，一条SQL语句的执行流程/原理？

答：经过四层，分别是连接层（连接池）、服务层（查询缓存、分析器、优化器、执行器）、引擎层、存储层。

索引：相当于以前的汉语词典中的目录

数据：相当于以前的汉语词典中的汉字

没有经过索引的操作，叫做全表扫描

走索引（主键索引、唯一索引、普通索引、前缀索引、联合索引...）

![6081d807-e52f-48e7-ad3a-de7f5eeed2ca](6081d807-e52f-48e7-ad3a-de7f5eeed2ca.png)



## 1、客户端(连接者)

- MySQL的客户端可以是某个客户端软件
- MySQL的客户端可以是不同的编程语言(Python/Java等)编写的应用程序（DataGrip）
- MySQL的客户端还可以是一些API的接口

## 2、连接层

 主要作用：管理和缓冲用户连接，为客户端请求做连接处理；身份认证等。

面试过程：进程和线程？

**进程**：一个应用软件，启动后往往会产生1个甚至多个进程，进程需要消耗一定的计算机资源（CPU、内存、磁盘、网络），适合CPU密集型应用（大量的计算程序，需要消耗资源）

**线程**：一个进程可以产生多个线程，线程不能单独存在，必须依赖进程。所有线程共享进程资源，进程启动、停止快，资源开销小。适合IO密集型应用（文件操作、网络爬虫、数据库连接）

![29b6adff-c06e-4c6d-b10b-134f1f042f3a](29b6adff-c06e-4c6d-b10b-134f1f042f3a.png)

连接层中的**缓存池**，就是为了优化数据库连接而设置的机制，专门用来**缓存和复用**已建立的连接。其主要作用是：

1. **减少连接开销**：避免每次新建连接带来的系统负担，提升性能。
2. **提升响应速度**：连接池里有现成的连接，随取随用，减少等待时间。
3. **优化资源利用**：通过控制最大连接数，防止过多连接耗尽系统资源。

工作流程很简单：创建连接→用完放回池中→再次复用，保持高效循环。

总结就是：连接池通过缓存连接，减少开销、加快响应、合理利用资源，让数据库更高效应对高并发。

连接池配置相关参数：

1. **max_connections**：指定MySQL可以同时处理的最大连接数，控制最大连接数目。
2. **wait_timeout**：定义一个连接在闲置状态下最多可以等待的时间，超过这个时间将被关闭。
3. **thread_cache_size**：控制线程缓存池的大小，以缓存空闲线程，避免频繁创建和销毁线程所带来的开销。

> thread_cache_size：根据系统并发情况设置，通常可设置为**CPU核心数的2倍**，以便高效处理并发请求

## 3、服务层（核心层）

主要作用：接受用户的SQL请求，查询分析，权限处理，优化，结果缓存等。

![b4929000-87ba-4242-9784-ca1a838d8348](b4929000-87ba-4242-9784-ca1a838d8348.png)

## 4、引擎层

- 什么是存储引擎？

1）存储引擎说白了就是 如何管理操作数据(存储数据、如何更新、查询数据等)的 一种方法和机制。

2）在MySql数据库中提供了多种存储引擎，各个存储引擎的优势各不一样 => `show engines`

3）用户可以根据不同需求为数据表选择不同的存储引擎，也可以根据自己需要编写自己的存储引擎。

4）甚至一个库中不同的表使用不同的存储引擎，这些都是允许的。

- 常用的存储引擎有哪些？
- 最常用的存储引擎是InnoDB和MyISAM

| 存储引擎                     | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| InnoDB（MySQL5.6版本及以后） | 支持拥有ACID特性事务的存储引擎，并且提供行级的锁定，支持外键、应用广泛。侧重于数据安全，默认引擎 |
| MyISAM（MySQL5.5及之前版本） | 查询速度快，有较好的索引优化和数据压缩技术；但不支持事务、不支持外键约束。 适用于读多写少的应用场景 |
| NDB                          | 用于MySQL Cluster的集群存储引擎，提供数据层面的高可用性      |
| MEMORY                       | 存储数据的位置是内存，因此访问速度最快，但是安全上没有保障。 适合于需要快速的访问或临时表。 |
| BLACKHOLE                    | 黑洞存储引擎，写入的任何数据都会消失，应用于主备复制中的分发主库（中继slave） |

**面试：你用过哪些数据库引擎？各自特点？**

答：早期MySQL5.5版本使用过MyISAM引擎，后期MySQL5.7、MySQL8.0等等都是使用InnoDB引擎，偶尔也了解MEMORY引擎。

① MyISAM引擎，擅长**数据查询**，支持较好的**索引优化**、数据压缩、支持表级锁以及全文索引技术，安全性相对于InnoDB略差一些。

索引优化 => 主键索引（图书目录），有索引，查询速度会更快

数据压缩 => 减少存储空间占用

**表级锁** => 只能进行表级锁，就是锁表时，要锁定整个数据表，在这个过程中，这个表只能进行查询操作，而不能进行增删改等操作，但是粒度太大，对并发有一定的影响。

**全文索引** => 从一篇文章中搜索指定内容，类似模糊查询，更加强大一些。

② InnoDB引擎，擅长数据安全，支持支持**行级锁，支持事务处理，支持外键约束等等，强调安全性**。

行级锁 => 只会对某一行进行锁定，不会全表锁定，粒度更细，并发能力更强。

事务处理 => 一种数据安全策略，保证数据安全

**外键约束**

③ Memory引擎，擅长**数据缓存，加快数据查询，但是由于数据放置于内存，所以安全性没有MyISAM以及InnoDB好**。

扩展：InnoDB事务处理

应用场景：银行转账（最典型）

我的银行卡：0.10

李文凯银行卡：2000.00

发生一系列操作：① 李文凯发起转账，扣款1000，余额-1000 ② 银行接收任务，处理（ATM） ③ 我的银行卡接收到1000，余额+1000

update bank set money=money-1000 where name = "李文凯"

ATM停电了

update bank set money=money+1000 where name = "我"

操作步骤：事务处理配合Python/Java程序一起使用，就是把所有要执行的SQL语句当做一个整体，要么全部成功，要么全部失败。

① 开启事务处理功能 => start transaction;

② 执行一系列的SQL语句（多条）

update bank set money=money-1000 where name = "李文凯"

update bank set money=money+1000 where name = "我"

③ 判断SQL语句是否全部执行成功，如果成功则提交事务 => commit; 失败，则回滚事务 => rollback;。

## 5、存储层（物理层）

核心作用：物理层负责与底层的操作系统交互，将数据存储到磁盘上，并确保数据的物理安全。

默认存储在/export/server/mysql/data数据目录下

**工作方式**：

- 将数据以物理文件的形式存储在磁盘上（如表空间文件、数据文件、日志文件等）。
- 通过文件系统与操作系统进行交互，管理数据的读写、缓存、索引文件等。

## 6、MySQL体系结构总结

- MySQL体系结构分为哪几层？四层
- 每一层是如何工作的？
- MySQL8.0 版本以后默认的存储引擎是哪个？有什么特点？除了这种引擎，你还了解哪些其他引擎？（至少说出2-3种）

扩展：生产环境下，MySQL到底应该如何配置呢？

答：这里所谓的配置主要是针对/etc/my.cnf（MySQL优化、处理等等都是由my.cnf决定的）。

# 九、查询性能监控与分析（重点）

## 1、启动慢查询日志（重点）

SQL语句：执行过程中，有快有慢，找出慢查询SQL。

**作用**：慢查询日志记录了所有执行时间超过设定阈值的查询，帮助你发现需要优化的慢查询SQL语句，辅助优化。

**如何启用**：

```PowerShell
vim /etc/my.cnf
[mysqld]
# 开启慢查询日志
slow_query_log=1
# 指定慢查询日志文件存放路径
slow_query_log_file=/export/server/mysql/logs/mysql-slow.log
# 设置超过 1 秒的查询被记录
long_query_time=1
# 记录未使用索引的查询（可选）
log_queries_not_using_indexes=0
mkdir /export/server/mysql/logs
touch /export/server/mysql/logs/mysql-slow.log
chown -R mysql.mysql /export/server/mysql
```

重启mysql

```PowerShell
systemctl restart mysqld
```

**分析日志**：慢查询日志可以显示查询的执行时间、锁等待时间、以及查询的详细内容。分析日志后，优化这些慢查询是提升性能的首要任务。

添加200万条数据到数据表中，做测试（不要求掌握以下，只是为了做测试）

> SQL高级：类似Python/Shell代码，支持if结构、循环结构以及函数或者存储过程等等。
>
> Prompt提示词：传入表结构，根据simple_table表结构，编写一个存储过程，循环向数据表中插入200万条测试数据。

```SQL
CREATE TABLE simple_table (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    age INT
);

DROP PROCEDURE InsertData;
-- 存储过程
DELIMITER //
CREATE PROCEDURE InsertData()
BEGIN
    DECLARE i INT DEFAULT 1;
    START TRANSACTION;
    WHILE i <= 2000000 DO
        INSERT INTO simple_table (name, age)
        VALUES (CONCAT('User', i), FLOOR(18 + (RAND() * 42)));
        SET i = i + 1;
    END WHILE;
    COMMIT;
END //
DELIMITER ;

-- 调用存储过程
CALL InsertData();

-- 移除自动增长，因为有自动增长的情况，主键无法移除的，因为自动增长所在字段要求必须是一个key索引
alter table simple_table change id id int;
-- 移除主键
alter table simple_table drop primary key;

注意：主键也是索引，可以加快查询
```

做一个查询，然后查看效果

```PowerShell
select * from simple_table where id = 1990000;
```

## 2、使用explain分析执行计划（重点）

简单来说：explain执行计划就是用于分析一个SQL语句如何执行的，核心作用：帮助我们提升SQL查询效率，加快查询速度。

前置知识点：索引优化

学不认识的汉字

传统：一页一页进行查找，如果要查的数据在最后一页，整体要翻一遍 => 全表扫描

索引：给所有的汉字添加一个目录页（拼音、偏旁部首），以后在查找汉字的时候，不是一页一页翻，而是先查找目录页，然后在根据目录页快速定位汉字 => 走索引（索引优化）=> 快速定位我们要查找的内容

![6b0e4ef7-accf-418d-afce-44a5d5163735](6b0e4ef7-accf-418d-afce-44a5d5163735.png)

什么是explain？

简单来说，`EXPLAIN` 是用来 **查看数据库如何执行一个 SQL 查询** 的工具。它能告诉你数据库在执行查询时，选择了什么样的操作步骤（比如是扫描整个表还是使用索引等），并且通过这些信息你可以判断查询是否高效。

作用：假设你写了一个 SQL 查询来查询数据，如果查询效率低下（比如很慢），你可能想知道数据库是怎么执行的，找出瓶颈在哪。`EXPLAIN` 就是用来帮你“拆解”查询，了解数据库是如何处理每一步的。

如何使用：

```PowerShell
EXPLAIN SELECT * FROM orders WHERE user_id = 123;
```

**分析重点**：

当你在查询前加上 `EXPLAIN` 时，它会显示一个执行计划的表格，这个表格给出了执行查询时的每一个细节。每一行代表 SQL 查询执行的一部分，通常包含这些信息：

- **id**: 这个数字表示查询的执行顺序。对于复杂的查询，可能会有多个查询步骤，`id` 就帮助你理解它们的顺序。
- **select_type**: 表示查询的类型，是普通查询还是复杂查询（比如涉及子查询等）。
- **table**: 代表查询是从哪张表中获取数据。
- **type**: 查询是如何连接数据的。这个字段非常重要，它告诉你查询效率的高低，越高效的查询方式类型通常是 `ref`、`range` 等。
- **possible_keys**: 查询可能会使用到哪些索引。索引就像数据的“目录”，使用索引可以加速数据查找。
- **key**: 实际使用了哪个索引。
- **key_len**: 使用的索引的长度（表示数据库选择了多长的索引）。
- **ref**: 显示与哪个列进行匹配来获取数据。
- **rows**: 数据库估算的需要扫描的行数。越少的行数，意味着查询越高效。
- **Extra**: 额外信息，比如是否使用了临时表或者文件排序等。

案例1：

```PowerShell
-- primary key主键索引
-- unique唯一索引
-- key/index普通索引
-- create index 索引名称 on 数据表(字段);
create index student_name_index on student(name);
desc student;
-- 前缀索引：就是一个字段，不需要所有值，只需要把字符值前几个字符取出就能实现数据区分
-- 联合索引：可以针对多个字段简历索引 on 数据表(字段1,字段2)

-- 案例：根据慢查询日志中的SQL使用explain执行计划分析为什么执行缓慢？
-- 没加索引情况
explain select * from simple_table where id = 1990000;
alter table simple_table add primary key(id);
desc simple_table;
-- 添加索引情况
explain select * from simple_table where id = 1990000;
```

![11292d9b-e709-4bf7-8d9a-2410f7b9fe12](11292d9b-e709-4bf7-8d9a-2410f7b9fe12.png)

这个输出告诉我们：

- **id**: `1`，表示这是查询的第一步。
- **select_type**: `SIMPLE`，表示这是一个简单查询，没有复杂的连接操作。
- **table**: `employees`，表示查询的数据表是 `employees`。
- **type**: `ref`，表示数据库用的是“引用”的方式来查找数据，这通常比全表扫描（`ALL`）更高效。
- **possible_keys**: `department`，表示数据库考虑过使用 `department` 列的索引。
- **key**: `department`，表示数据库实际使用了 `department` 索引。
- **key_len**: `50`，表示数据库使用了该索引的 50 字节。
- **ref**: `const`，表示查询使用了常量来查找数据。
- **rows**: `100`，表示数据库预计需要扫描 100 行数据。
- **Extra**: 没有额外的信息。

核心列type说明：

`type` 列非常关键，它反映了查询的效率。这里列出常见的几种类型，效率从高到低排序：

- **const**：最优，表示数据库通过常量查找一个行。
- **ref**：比较高效，表示数据库通过索引查找匹配行。
- **range**：表示数据库通过范围扫描来查找数据。
- **index**：表示数据库扫描整个索引（但不会读取数据表），效率不如 `range`。
- 索引=字段目录，数据=记录
- **ALL**：最差，表示数据库执行了全表扫描，效率最低。

**总结对比**

类型类比速度适用场景const直接翻到指定页码最快主键/唯一索引的等值查询ref按目录查几个名字快普通索引的等值查询range按目录查一个连续区间（如 20~30 岁）较快索引范围查询index扫完整本目录（不读正文）较慢只需索引字段，不查数据ALL一页一页翻完整本书最慢无索引或强制全表扫描

key_ken长度说明：

`INT` 类型通常占 4 字节

`VARCHAR(n)` 类型占用的字节数是 `n`（最大长度）

`DATE` 类型占用 3 字节

`CHAR(n)` 类型占用 `n` 字节

**面试题：在运维环境中，发现MySQL运行缓慢，如何去解决？**

答：

① 从系统层面，检查系统资源使用情况，如CPU负载（top）、检查内存占用（free -h）、检查磁盘空间使用（df -h），因为mysql把数据写入到磁盘，读写都涉及磁盘io（iostat）

② 从日志层面，开启慢查询日志，把查询缓慢的SQL写入到慢查询日志中，进行捕获

③ 从SQL语句层面，使用explain执行计划分析SQL执行过程，是否有走缓慢等等，查看具体慢的原因。如果是全表扫描，可以考虑引入索引（如主键索引、唯一索引、普通索引）等等实现优化操作

## 3、实时监控SQL查询

**作用**：`SHOW PROCESSLIST`可以显示当前正在执行的SQL语句及其状态，帮助你发现正在运行的长时间查询。

注意：适合执行时间较长的SQL语句

```PowerShell
SHOW PROCESSLIST;
```

分析重点：

**Command**：表示查询的状态，比如`Query`表示正在执行查询，`Sleep`表示连接空闲。

**Time**：表示查询的执行时间，时间较长的查询可能是性能瓶颈。

**State**：显示当前查询执行的阶段，帮助定位查询的卡顿点（如`Copying to tmp table`表示查询在使用临时表）。

## 4、使用`SHOW PROFILES`查看查询的详细耗时（重点）

**作用**：`SHOW PROFILES`可以显示最近执行的SQL查询的执行时间，以及每个阶段的耗时，帮助你了解查询在执行时哪个环节耗时最长。

```PowerShell
SET profiling = 1;                                         -- 开启性能分析（开关）
alter table simple_table change id id int;
alter table simple_table drop primary key;
select * from simple_table where id = 1990000;

SHOW PROFILES;                                             -- 查看所有SQL的执行情况
SHOW PROFILE FOR QUERY 1;                                  -- 查看第一个查询的详细耗时
```

小结：

如果我们想捕获SQL语句详细执行以及底层执行过程以及耗时都可以使用(show profiles;)

# 今日重点

- 单表查询五子句，必须掌握！！！
- 多表查询，掌握内连接、外连接以及自连接
- DCL用户+权限管理（重点）
- MySQL四层结构以及每一层作用（面试）
- 慢日志查询（只要会开启会配置即可）
- explain执行计划
- SHOW PROFILES查看查询的详细耗时