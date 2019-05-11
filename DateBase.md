RDBMS:Oracle,SQL SERVER,MYSQL,DB2

NoMQL:memcached,Redis,MogoDB

只有MySQL是既开源又跨平台的数据库服务软件

# MySQL

| 服务名           | mysqld，首次启动，数据初始化，较慢                   |
| ---------------- | ---------------------------------------------------- |
| 端口             | 3306                                                 |
| 默认数据库文件夹 | /var/lib/mysql                                       |
| 日志文件         | /var/log/mysqld.log 有初始密码：'temporary password' |

数据存储至数据库步骤：

- 连接
- 建库
- 建表
- 插入记录
- 断开连接

MySQL默认的4个库分别是 information_schema 、performance_schema 、mysql 、test 其中information_schema库不占用物理磁盘空间。



## 初始化

```mysql
mysql-community-client-5.7.17-1.el7.x86_64.rpm # 客户端应用程序
mysql-community-common-5.7.17-1.el7.x86_64.rpm # 数据库和客户端库共享文件
mysql-community-devel-5.7.17-1.el7.x86_64.rpm # 客户端应用程序的库和头文件
mysql-community-embedded-5.7.17-1.el7.x86_64.rpm # 嵌入式函数库
mysql-community-embedded-compat-5.7.17-1.el7.x86_64.rpm # 嵌入式兼容函数库
mysql-community-embedded-devel-5.7.17-1.el7.x86_64.rpm # 头文件和库文件作为Mysql的嵌入式库文件
mysql-community-libs-5.7.17-1.el7.x86_64.rpm # MySql数据库客户端应用程序的共享库
mysql-community-libs-compat-5.7.17-1.el7.x86_64.rpm # 客户端应用程序的共享兼容库
mysql-community-minimal-debuginfo-5.7.17-1.el7.x86_64.rpm
mysql-community-server-5.7.17-1.el7.x86_64.rpm
mysql-community-test-5.7.17-1.el7.x86_64.rpm

systemctl start mysqld                  # 动mysql服务,第一次启动会初始化
systemctl enable mysqld                 # 置开机自启
systemctl status mysqld                 # 看mysql服务状态

mysql -hlocalhost -u root -p'password' [库名]# 非交互登录mysql,极不安全，工作中不能在命令行输入密码。
mysql> \h # 查询帮助
mysql> \c # 结束命令
alter user root@localhost identified by "password"; # 设置root密码
show variables like "%password%"; # 查看变量有password的设置

# validate_password_policy 默认为1，设置的密码必须符合长度，且必须含有数字，小写或大写字母，特殊字符。如果我们不希望密码设置的那么复杂，需要修改两个全局参数：validate_password_policy与validate_password_length。validate_password_length默认值为8,最小值为4，如果你显性指定validate_password_length的值小于4，尽管不会报错，但validate_password_length的值将设为4。
set global validate_password_policy=0;      # 只验证长度
set global validate_password_length=6;     # 修改密码长度,默认值是8个字符

vim /etc/my.cnf # mysql配置文件
validate_password_policy=0
validate_password_length=6
```



## 基本操作

常用SQL操作指令：

- DDL 定义（create,alter,drop）
- DML 操作（insert,update,delete）
- DCL 控制（grant,revoke）
- DTL 事物（commit,rollback,savepoint）

```mysql
# 库指令
show databases; # 显示已有的库
use 库名; # 切换库
select database(); # 显示当前所在的库
select user(); # 显示当前用户及客户端地址
create database 库名; # 创建新库，可以使用字母/数字/下划线，区分大小写，不可使用指令关键字
show tables; # 显示库中所有表
drop database 库名; # 删除库

# 表指令
create table 库名.表名(字段名 类型 约束条件,...字段名 类型 约束条件);
desc 表名; # 查看表结构
select * from 表名; # 查看表记录
drop table 表名; # 删除表
insert into 表名(字段名,字段名,...,字段名) values("对应值","对应值",...,"对应值"),("对应值","对应值",...,"对应值"); # 插入多个数据
update 表名 set 字段=值; # 批量更新记录
delete from 表名; # 清空表记录
drop table 表名; # 删除表

locale # 查看linux系统当前字符集long
show create table 表名; # 查看创建表时的命令
# 在MySQL表内存储中文数据时，需要更改字符集（默认为latin1不支持中文），以便MySQL支持存储中文数据记录；比如，可以在创建库或表的时候，手动添加“DEFAULT CHARSET=utf8”来更改字符集。
mysql> CREATE TABLE mydb.student(
    -> 学号 char(9) NOT NULL,
    -> 姓名 varchar(4) NOT NULL,
    -> 性别 enum('男','女') NOT NULL,
    -> 手机号 char(11) DEFAULT '',
    -> 通信地址 varchar(64),
    -> PRIMARY KEY(学号)
    -> ) DEFAULT CHARSET=utf8; # 手工指定字符集，采用utf8
# 若要修改MySQL服务的默认字符集，可以更改服务器的my.cnf配置文件，添加character_set_server=utf8 配置，然后重启数据库服务
```



## 数据类型

### 表结构

字段名 | 字段类型 | 是否为空 | 是否为主键 | 默认值 | 描述信息

### 数值类型

| 类型       | 大小  | 范围（有符号） | 范围（无符号）               | 用途                                                   |
| ---------- | ----- | -------------- | ---------------------------- | ------------------------------------------------------ |
| TINYINT    | 1字节 | -128～127      | 0～255                       | 微小整数                                               |
| SMALLINT   | 2字节 | -32768～32767  | 0～65535                     | 小整数                                                 |
| MEDIUMINT  | 3字节 | -2^23^~2^23^-1 | 0~2^24^-1                    | 中整数                                                 |
| ==INT==    | 4字节 | -2^31^~2^31^-1 | 0~2^32^-1                    | 整数，默认显示11位，空格补全                           |
| BIGINT     | 8字节 | -2^63^~2^63^-1 | unsigned 使用无符号0~2^64^-1 | 极大整数                                               |
| FLOAT      | 4字节 |                |                              | 单精度浮点数，可自定义，默认的float类型都只能存6个数字 |
| ==DOUBLE== | 8字节 |                |                              | 双精度浮点数，可自定义                                 |

对于FLOAT/DOUBLE（M，D），其中M为总位数，D为小数位，M应大于D，占用M+2字节。

unsigned 使用无符号

zerofill 用零补位

数值的括号是至少显示几位

```mysql
create table zz(age tinyint unsigned);
```



### 字符类型

| 类型            | 用途                        | 说明                                                       |
| --------------- | --------------------------- | ---------------------------------------------------------- |
| char(字符数)    | 定长，最长255个字符longlong | 不够的在右侧空格补齐，超过长度无法写入                     |
| varchar(字符数) | 变长                        | 按照实际大小分配存储空间，超过长度无法写入，==处理速度慢== |
| text/blob       | 文本类型                    | 在字符数大于65535字符时使用                                |

==字符与字节不同，字节大小依据编码格式==

```mysql
查看那些用户没有家目录	create teable zz(name char(5),mail varchar(10),address varchar(50));
```



### 日期时间类型

| 类型      | 大小  | 范围                                                    | 说明                                                 |
| --------- | ----- | ------------------------------------------------------- | ---------------------------------------------------- |
| DATETIME  | 8字节 | 1000-01-01 00:00:00.000000 ~ 9999-12-31 23:59:59.999999 | 默认NULL                                             |
| TIMESTAMP | 4字节 | 1970-01-01 00:00:00.000000 ~ 2038-01-19 03:14:07.999999 | 默认系统当前时间                                     |
| DATE      | 4字节 | 0001-01-01 ~ 9999-12-31                                 | 日期                                                 |
| YEAR      | 1字节 | 1901 ~ 2155                                             | 默认使用4位，只用2位时，01～69视为20-，70～99视为19- |
| TIME      | 3字节 | HH:MM:SS                                                | 时刻                                                 |

#### 时间函数

| 类型      | 用途                           |
| --------- | ------------------------------ |
| now()     | 获取系统当前时间（日期和时刻） |
| year()    | 获取指定时间中的年份           |
| day()     | 获取指定时间中的天             |
| sleep(n)  | 休眠n秒                        |
| curdate() | 获取当前系统日期               |
| curtime() | 获取当前系统时刻               |
| month()   | 获取指定时间中的月份           |
| date()    | 获取指定时间中的日期           |
| time()    | 获取指定时间中的时刻           |



### 枚举类型（选择）

| 类型 | 格式          | 作用       |
| ---- | ------------- | ---------- |
| enum | enum(值1,值n) | 单选       |
| set  | set(值1,值n)  | 一个或多个 |



## 约束条件

| Null     | 允许空，默认设置       |
| -------- | ---------------------- |
| NOT NULL | 不允许null             |
| key      | 索引类型               |
| Default  | 设置默认值，默认为null |

==""是空白符 不是null==



## 修改表结构

基本格式 ALTER TABLE 表名 执行动作;

添加新字段

ADD 字段名 类型（宽度）约束条件; 默认在最后追加，可加 AFTER 字段名 或 FIRST

修改字段类型

MODIFY 字段名 类型（宽度）约束条件; 不能与源数据冲突，可加 AFTER 字段名 或 FIRST

修改字段名

CHANGE 原字段名 新字段名 类型（宽度）约束条件; 

删除字段

DROP 字段名1,DROP 字段名2;                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             

修改表名

RENAME 新表名;

```mysql
 alter table zz.user add id int primary key auto_increment first;
 alter table zz.user drop id;
```



## 键值

索引

对记录集的多个字段进行排序的方法。类似目录，类型包括：Btree，B+tree，hash

默认使用Btree排序（二叉树）

优点：通过创建唯一性索引，可以保证数据库表中每一行数据的唯一性。可以加快数据的检索速度

缺点：当对表中的数据进行增删改的时候，索引也要动态的维护，降低数据的维护速度。占用物理空间

键值类型

index 普通索引

primary key 主键

foreign key 外键

unique 唯一索引

fulltext 全文索引



index 使用说明

一个表中可以有多个index字段，允许字段的值重复或null，经常把做条件查询的字段设置为index，标志是MUL。

```mysql
create table zz(id char(6) not null,name varchar(4),index(id),index(name)); # 新建表时指定索引字段
create index 索引名 on 表名(字段名); # 已有表指定索引字段,默认索引名与字段名相同
show index from 表名\G; # 显示表的索引信息 \G 列显示
drop index 索引名 on 表名(字段名); # 删除索引字段
```



primary key 使用说明

只能有一个primary key，不允许null。如果有多个字段都要作为primary key，称为复合主键，必须一起创建，一起删除。标志是PRI。通常与 AUTO_INCREMENT 连用。

```mysql
create table zz(id char(6) not null,name varchar(4),primary key(id)); # 新建表时指定索引字段
alter table 表名 add primary key(字段名); # 已有表指定索引字段
alter table 表名 drop primary key; # 删除索引字段
create table zz(id char(6) not null,name varchar(4),primary key(id，name)); # 复合主键
```



foreign key 使用说明

|        | 主键                                       | 外键                                                 | 索引                               |
| ------ | ------------------------------------------ | ---------------------------------------------------- | ---------------------------------- |
| 定义： | 唯一标识一条记录，不能有重复的，不允许为空 | 表的外键是另一表的主键, 外键可以有重复的, 可以是空值 | 该字段没有重复值，但可以有一个空值 |
| 作用： | 用来保证数据完整性                         | 用来和其他表建立联系用的                             | 提高查询排序的速度                 |
| 个数： | 主键只能有一个                             | 一个表可以有多个外键                                 | 一个表可以有多个索引               |

使用外键的条件

- 表的存储引擎必须是innodb
- 字段类型，宽度一致
- 被参照字段必须是主键

```mysql
# 基本用法
FOREIGN KEY(本表的字段名) References 参照表(字段名)
on update cascade # 同步更新
on delete cascade # 同步删除
# 删除外键字段
alter table 表名 drop foreign key 约束名;

# e.g.
create table yg(yg_id int primary key auto_increment,name char(15))engine=innodb; # 建立参照表
create table gz(gz_id int,gz float(7,2) default 20000,foreign key(gz_id) references yg(yg_id) on update cascade on delete cascade)engine=innodb;

show create table gz\G; # 查询创建表信息 可以看到约束名，在CONSTRAINT后
alter table gz add foreign key(gz_id) references yg(yg_id) on update cascade on delete cascade)engine=innodb;
```



## 存储引擎

MySQL的功能程序，提供不同的功能和数据存储方式。

5.5 之前默认是MyISAM 5.5开始默认是 InnoDB

```mysql
show engines; # 查看可用的存储引擎
```

```shell
vim /etc/my.cnf # 设置默认存储引擎
[mysqld]
default-storage-engine=innodb

systemctl restart mysqld
```

### 存储引擎特点

| 存储引擎 | 主要特点                                 | 相关的表文件                                                 |
| -------- | ---------------------------------------- | ------------------------------------------------------------ |
| myisam   | 支持表级锁<br>不支持事务，事务回滚，外键 | 表名.frm<br>表名.MYI<br>表名.MYD                             |
| innodb   | 支持行级锁<br/>支持事务，事务回滚，外键  | 表名.frm<br>表名.ibd<br>ibdata1<br>ib_logfile0<br>ib_logfile1 |

frm 表结构 myd 数据 myi 索引

ibd 数据与索引 ibdata1 ib_logfile0 ib_logfile1 事务日志

锁机制

表级锁：一次直接对整张表进行加锁

行级锁：只锁定某一行

页级锁：对整个页面（MySQL管理数据的基本存储单位）进行加锁

锁类型

读锁（共享锁）：支持并发读

写锁（互斥锁，排他锁）：上锁期间其他线程不能读表或写表

事务：一次访问从建立连接到断开连接的过程。

事务回滚：在事务过程中，任意一步失败，会恢复所有操作。

事务特性

Atomic 原子性

事务的整个操作是整体，不可分割，要么全部成功，要么全部失败。

Consistency 一致性

保证在一个事务中的多次操作的数据中间状态对其他事务不可见。一致性关注数据的可见性，中间状态的数据对外部不可见，只有最初状态和最终状态的数据对外可见

Isolation 隔离性

事务操作是相互隔离不受影响的。

Durability 持久性

数据一旦提交，不可改变，永久改变表数据。

```mysql
show variables like "autocommit"; # 查看提交状态
set autocommit=off; # 关闭自动提交事务操作功能，对单个连接有效
rollback; # 数据回滚
commit; # 手动提交操作
```

查询访问多的表使用myisam引擎，节省系统资源。

写操作多的表使用innodb引擎，并发访问量大。



## 数据导入导出

```mysql
# 默认数据目录
show variables like "secure_file_priv"; # /var/lib/mysql-files/
# 修改默认目录
vim /etc/my.conf
[mysqld]
secure_file_priv="dir"

# 数据导入
load data infile "文件" into table 库.表 fields terminated by "列分隔符" lines terminated by "\n";
# 数据到出 导出文件不能存在
SQL查询 into outfile "文件" fields terminated by "列分隔符" lines terminated by "\n";

# e.g.
load data infile "/var/lib/mysql-files/passwd" into table zz.user fields terminated by ":" lines terminated by "\n";
select * from zz.user where uid < 100 into outfile '/var/lib/mysql-files/re';
```



## 管理表记录

添加记录，使用""保持特殊字符

```mysql
insert into 表名(字段名) values(值),(值);
select */字段名 from 表名 where 条件; # 字段名之间不加","相当于给前一个字段起别名
update 表名 set 字段名=值,字段名=值 where 条件;
delete from 表名 where 条件;
```

普通条件

| 类型                    | 用途               |
| ----------------------- | ------------------ |
| =                       | 数值等于           |
| \> ,>=                  | 数值大于，大于等于 |
| <,<=                    | 数值小于，小于等于 |
| !=                      | 数值不等于         |
| =                       | 字符等于           |
| !=                      | 字符不等于         |
| is null                 | 空                 |
| is not null             | 非空               |
| or                      | 或                 |
| and                     | 与                 |
| !                       | 非                 |
| ()                      | 提高优先级         |
| in (值列表)             | 在值列表内         |
| not in (值列表)         | 不在值列表内       |
| between 数字1 and 数字2 | 在。。。之间       |
| distinct 字段名         | 去重显示           |

 模糊查询

Where 字段名 like '通配符'

_ 匹配单个字符  % 匹配0～N个字符

正则表达式

Where 字段名 regexp '正则表达式'

^ $ . [] * |

四则运算

| 类型 | 用途           |
| ---- | -------------- |
| +    | 加法           |
| -    | 减法           |
| *    | 乘法           |
| /    | 除法           |
| %    | 取余数（求模） |



## 操作查询结果函数

| avg(字段名)   | 统计字段平均值 |
| ------------- | -------------- |
| sum(字段名)   | 统计字段之和   |
| min(字段名)   | 统计字段最小值 |
| max(字段名)   | 统计字段最大值 |
| count(字段名) | 统计字段值个数 |

```mysql
# 排序
SQL查询 order by 字段名 [asc|desc]; # desc 从大到小，字段是数值，默认升序

# 分组
SQL查询 group by 字段名;

# 对查询结果过滤，此方法节省资源
SQL查询 having 条件;
SQL查询 group by 字段名 having 条件;

# 限制查询结果显示行数
SQL查询 limit N;  # 显示前N条记录
SQL查询 limit N,M; # 显示指定范围内的结果，从N+1开始，共M条。
```



## 多表查询

### 复制表

```mysql
create table zzz SQL 查询语句; # 创建zzz表
create table vvv select * from xxx where false;	# 复制表xxx的结构到新表vvv
```

多表查询

```mysql
select 字段名列表 from 表a,表b where 条件;
select 字段名列表 from 表名 where 条件 (select 字段名列表 from 表名 where 条件);
# e.g.
select name,age from student where age < (select avg(age) from student);

select 字段名列表 from 表a left join 表b on 条件; # 以左表为主显示查询结果
select 字段名列表 from 表a right join 表b on 条件; # 以右表为主显示查询结果
# e.g.
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo FROM Persons LEFT JOIN Orders ON Persons.Id_P=Orders.Id_P
```



## 管理工具

访问mysql数据库服务的方式

- 命令行
- 网站
- 图形界面

```shell
yum  -y   install   httpd     php    php-mysql  # 装包
systemctl  start  httpd       # 启动服务
systemctl  enable httpd       # 设置开机自启
chown  -R  apache:apache  phpmyadmin/ # 改变phpmyadmin目录权限
cp   config.sample.inc.php   config.inc.php  # 备份主配置文件

vim   config.inc.php  # 编辑主配置文件
17 $cfg['blowfish_secret'] = 'plj123';     # 给cookie做认证的值，可以随便填写
31 $cfg['Servers'][$i]['host'] = 'localhost';  # 指定主机名，定义连接哪台服务器
```



## 恢复MySQL管理密码

```mysql
vim /etc/my.cnf
[mysqld]
skip-grant-tables

# 重启Mysqld服务
systemctl restart mysqld

# 修改密码
mysql>update mysql.user set authentication_string=password("pwd") where user = "root" and host="localhost";
mysql>flush privileges;

# 命令行修改密码
mysqladmin -uroot -p password "新密码"
Enter password: # 输入旧密码
```



## 用户授权及撤销

授权库 mysql

| user         | 存储授权用户对所有数据库访问权限       |
| ------------ | -------------------------------------- |
| db           | 单独存储授权用户对数据库的特殊访问权限 |
| tables_priv  | 单独存储授权用户对表的特殊访问权限     |
| columns_priv | 单独存储授权用户对字段的特殊访问权限   |

```mysql
# 授权格式
grant 权限列表 on 库名.表名 to 用户@'客户端地址' identified by '密码' [with grant option];
# 撤销格式
revoke 权限列表 on 库名.表名 from 用户@'客户端地址';
# 删除用户格式
drop user 用户@'客户端地址';

# 权限列表
all # 所有权限
select,update,insert....
select,update(字段1,字段2...)...

# 客户端地址
% # 匹配所有主机
192.168.1.% # 网段
192.168.1.1 # 单个ip

# 查看自己的授权
show grants;

# 管理员查看其他用户的权限
show grants for 用户@'客户端地址';

# 授权用户修改自己的密码
set password=password("新密码");

# 管理员修改授权用户的密码
set password for 用户@'客户端地址'=password("新密码");

show full processlist; # 查看全部连接
kill ID; # 断开指定用户
```



## 数据库备份

备份的三种方式

- 物理备份：cp,tar...。必须备份整个目录
- mysql命令
- 安装软件备份

备份策略

- 完全备份：备份所有数据
- 增量备份：备份==上次==备份后，所有新产生的数据
- 差异备份：备份==完全==备份后，所有新产生的数据

```shell
# 完全备份操作
mysqldump -uroot -p"pwd" 库名 > dir/x.sql
# 恢复完全备份,需要先创建库
mysql -uroot -p 库名 < dir/x.sql
```

库名表示方式

| --all-databases \| -A  | 所有库                            |
| ---------------------- | --------------------------------- |
| 库名                   | 单个库                            |
| 库名 表名              | 单张表                            |
| 库名 [表名1 表名2 ...] | 多张表                            |
| -B 库名1 库名2         | 多个库;==多库恢复时不需要写库名== |



### binlog

| 类型       | 用途                                                         | 配置                                                         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 二进制日志 | 记录所有==成功更改数据==的操作<br>可用于数据恢复<br>配置mysql主从同步的必要条件 | log_bin[=dir/name]<br>server_id=number(1-255)<br>max_binlog_size=numberm |

```shell
mysql>show master status; # 显示binlog信息
# 启用binlog
vim /etc/my.cnf
[mysqld]
...
log_bin # 启用binlog
server_id=100 # 指定id
```

binlog 相关文件

- 主机名-bin.index 记录已有日志文件名
- 主机名-bin.000001 第一个二进制日志
- 主机名-bin.000002 第一个二进制日志

手动生成新的日志文件的四种方法

- 重启mysqld服务
- 执行sql操作 flush log;
- mysqldump --flush-logs;
- mysql -uroot -p密码 -e 'flush log'

```mysql
# 清理早于指定版本的binlog日志
mysql>purge master logs to "binlog文件名";
# 清理所有binlog日志
mysql>reset master;
```

binlog 三种记录格式：

- statement 每一条修改数据的sql命令都记录
- row 仅保存哪条记录被修改
- mixed 以上两种混合使用

```mysql
# 查看当前记录格式
mysql>show variables like "binlog_format";
# 修改记录格式
vim /etc/my.cnf
[mysqld]
binlog_format=mixed
```

分析binlog

```shell
# 格式
mysqlbinlog [选项] binlog文件
# 常用选项
--start-datetime="yyyy-mm-dd hh:mm:ss"
--stop-datetime="yyyy-mm-dd hh:mm:ss"
--start-position=number # 之后的执行
--stop-position=number # 之前的执行
```

恢复binlog

```shell
mysqlbinlog mysql-bin.000001 | mysql -uroot -p
```



# innobackupex备份/恢复

物理备份缺点

- 跨平台差
- 备份时间长，浪费存储空间

mysqldump备份缺点

- 效率较低，备份和还原速度慢
- 备份过程中锁表，数据插入和更新操作会被挂起

## XtraBackup 工具

一款强大的在线热备份工具，备份过程中不锁库表，适合生产环境

主要包含两个组件

- XtraBackup：C程序，支持InnoDB/XtraDB
- innobackupex：以Perl脚本封装XtraBackup，额外支持MyISAM

innobackupex 五种方式：

- 完全备份
- 完全恢复
- 增量备份
- 增量恢复
- 恢复完全备份中的单张表

```shell
# 安装 XtraBackup 工具
yum install percona-xtrabackup-24-2.4.6-2.el7.x86_64.rpm

# innobackupex完整备份、增量备份操作
--host 主机名 # 默认本机
--port 3306 # 默认3306
--user 用户名
--password 密码
--databases="库名" # 不写默认全部
--databases="库1 库2"
--databases="库.表"
--no-timestamp # 不用日期命名备份文件存储的子目录，使用备份的数据库名做备份目录名
--no-timestmap # 不使用日期命名备份目录名
--redo-only # 日志合并
--apply-log # 恢复日志
--copy-back # 恢复数据
--incremental 目录 # 增量备份
--incremental-basedir=目录 # 增量备份，指定上一次备份数据存储的目录名
--incremental-dir=目录 # 恢复数据时，指定增量备份的目录
--export # 导出表信息
import # 导入表空间

# 完全备份
innobackupex --user root --password "pwd" "备份目录（自动创建，不能有数据）" --no-timestamp

#  查看信息
cat /mybak/xtrabackup_checkpoints

# 完全恢复
innobackupex --apply-log "目录" # 准备恢复日志
rm -rf /var/lib/mysql/* # 清空原数据库
innobackupex --copy-back "目录" # 恢复数据
chown -R mysql:mysql /var/lib/mysql/ # 更改权限

# 增量备份，必须先过一次完全备份。
innobackupex --user root --password "pwd" --incremental "增备目录" --incremental-basedir="已有备份目录" --no-timestamp

# 增量恢复
rm -rf /var/lib/mysql/* # 清空原数据库
innobackupex --apply-log --redo-only "目录" # 准备恢复日志
innobackupex --apply-log --redo-only "完备目录" --incremental-dir="增备目录"  # 把增备目录合并至完备目录
innobackupex --copy-back "完备目录" # 恢复数据
chown -R mysql:mysql /var/lib/mysql/

# 恢复一张表
# 已经存在完全备份
innobackupex --user root --password "pwd" --databases="库名" --apply-log --export "完备目录"; # 导出库中表信息
# 创建需要恢复的表，结构与原来的一致
mysql>alter table 库名.表名 discard tablespace; # 删除.ibd文件（数据文件）
cp "完备目录/x.{ibd,cfg,exp}" /var/lib/mysql/xxx # 复制表信息与ibd到数据库
chown -R mysql:mysql /var/lib/mysql/xxx
mysql> alter table 库.表 import tablespace; # 导出表空间
rm /var/lib/mysql/xxx/x.{cfg,exp} # 清理表信息，避免日后错误
```