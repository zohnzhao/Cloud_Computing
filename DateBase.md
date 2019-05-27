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

### 数值类型mysql> show slave status;

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

ibd 表空间，数据与索引 ibdata1 ib_logfile0 ib_logfile1 事务日志

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



# Mysql 主从同步

实现数据自动同步

主服务器：接收客户端访问

从服务器：自动从主库同步数据到本机



主服务器配置

- 启用binlog日志
- 添加授权用户
- 查看主binlog信息

```shell
# 启用binlog日志
vim /etc/my.cnf
log_bin=dbsvr1-bin                      # 启用binlog日志，并指定文件名前缀
server_id=1
binlog_format="mixed"
# 添加授权用户
grant replication slave on *.* to cpuser@192.168.4.52 identified by 'Cp123456789**';
flush privileges;
# 查看binlog信息
show master status;
```

从服务器配置

- 添加server_id 不能与主服务器重复
- mysql 内指定主库信息
- mysql 内启动slave进程

```shell
# 设置server_id
vim /etc/my.cnf
log_bin=dbsvr2-bin                      # 启动SQL日志，并指定文件名前缀
server_id=2

mysql> show slave status; # 查看从库信息
# 定主库信息
mysql> CHANGE MASTER TO MASTER_HOST='192.168.4.10',
    -> MASTER_USER='cpuser',
    -> MASTER_PASSWORD='Cp123456789**',
    -> MASTER_LOG_FILE='dbsvr1-bin.000002',      # 对应Master的日志文件
    -> MASTER_LOG_POS=334;                          # 对应Master的日志偏移位置
# 启动slave进程
mysql> START SLAVE; # Slave_IO_Running: Yes,Slave_SQL_Running: Yes 表示成功
# 从服务器配置文件
cat /var/lib/mysql/master.info
# 清除slave配置
rm -rf /var/lib/mysql/master.info
rm -rf /var/lib/mysql/主及名-relay-bin-[编号|index] # 日志只保存两个
rm -rf /var/lib/mysql/relay-log.info
```

主从结构

- 主 --> 从
- 主 --> 从  --> 从
-  主 <--> 主
-  从 <-- 主 --> 从

```shell
# 主 --> 从  --> 从,第一个从添加设置，其他与主从方式一样
vim /etc/my.cnf
[mysqld]
log_slave_updates # 从服务器从主服务器接收的更新是否应记录到从属服务器自己的二进制日志中。
```

## 常用配置选项

主库my.cnf

| 选项                  | 用途         |
| --------------------- | ------------ |
| binlog_do_db=name     | 对库记日志   |
| binlog_ignore_db=name | 不对库记日志 |

从库my.cnf

| 选项                     | 用途             |
| ------------------------ | ---------------- |
| log_slave_updates        | 同步时更新日志   |
| relay_log=name           | 指定中继日志名称 |
| replicate_do_db=name     | 仅复制指定库     |
| replicate_ignore_db=name | 忽略指定库       |

## 复制模式

| 模式                                        | 作用                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| 异步复制（Asynchronous replication）        | 在主库执行完一次事务后，立即将结果返给客户端，不关心从库是否同步(默认) |
| 全同步复制（Fully synchronous replication） | 在主库执行完一次事务后，等待所有从库同步后将结果返给客户端   |
| 半同步复制（Semisynchronous replication）   | 在主库执行完一次事务后，等待至少一个从库同步后将结果返给客户端 |

```mysql
mysql> show variables like 'have_dynamic_loading'; # 查看是否允许动态加载模块

# 主库安装模块
mysql> INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
# 从库安装模块
mysql> INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

# 查看系统库下的表，模块是否安装成功
mysql> SELECT PLUGIN_NAME,PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS WHERE PLUGIN_NAME LIKE '%semi%';

# 启用半同步复制，在安装完插件后，半同步复制默认是关闭的
mysql> SET GLOBAL rpl_semi_sync_master_enabled = 1; # 主启动模块
mysql> SET GLOBAL rpl_semi_sync_slave_enabled = 1; # 从启动模块

# 永久启用半同步复制
 vim /etc/my.cnf
[mysqld]
plugin-load=rpl_semi_sync_master=semisync_master.so
rpl_semi_sync_master_enabled=1

# 在高可用架构下，master和slave需同时启动，以便在切换后能继续使用半同步复制
 vim /etc/my.cnf
[mysqld]
plugin-load="rpl_semi_sync_master=semisync_master.so;rpl_semi_sync_slave=semisync_slave.so"
rpl-semi-sync-master-enabled = 1
rpl-semi-sync-slave-enabled = 1
```



# Mysql 读写分离

三种软件都可以：Mycat mysql-proxy maxscale

```shell
# 配置主从数据库，安装maxscale工具
vim /etc/maxscale.cnf
[maxscale]
threads=auto            # 运行的线程的数量
[server1]               # 定义数据库服务器
type=server
address=192.168.4.10        # 数据库服务器的ip
port=3306
protocol=MySQLBackend        # 后端数据库
[server2]
type=server
address=192.168.4.20
port=3306
protocol=MySQLBackend
[MySQL Monitor]                # 定义监控的数据库服务器
type=monitor
module=mysqlmon
servers=server1, server2        # 监控的数据库列表，不能写ip
user=scalemon                    # 监视数据库服务器时连接mysql的用户名scalemon
passwd=123456                   # 密码123456
monitor_interval=10000        # 监视的频率 单位为秒
#[Read-Only Service]        # 定义只读服务器
#type=service
#router=readconnroute
#servers=server1
#user=myuser
#passwd=mypwd
#router_options=slave
[Read-Write Service]            # 定义读写分离服务
type=service
router=readwritesplit
servers=server1, server2
user=maxscaled            # 用户名 验证连接代理服务时访问数据库服务器的mysql用户scalemon是否存在
passwd=123456               # 密码
max_slave_connections=100%
[MaxAdmin Service]        # 定义管理服务
type=service
router=cli
#[Read-Only Listener]        # 定义只读服务使用的端口号
#type=listener
#service=Read-Only Service
#protocol=MySQLClient
#port=4008
[Read-Write Listener]            # 定义读写服务使用的端口号
type=listener
service=Read-Write Service
protocol=MySQLClient
port=4006
[MaxAdmin Listener]        # 管理服务使用的端口号
type=listener
service=MaxAdmin Service
protocol=maxscaled
socket=default
port=4099     # 手动添加，指定时使用的是默认端口在启动服务以后可以知道默认端口是多少

# 所有数据库添加授权用户
mysql> grant replication slave,replication client on *.* to  scalemon@'%' identified by "123456"; # 监控数据库服务器时，连接数据库服务器的用户
mysql> grant select on mysql.* to maxscaled@"%" identified by "123456"; # 验证访问数据时，连接数据库服务器使用的用户scalemon是否在数据库服务器上存在的连接用户

# 启动服务 maxscale
maxscale -f  /etc/maxscale.cnf
ps -C  maxscale     # 查看进程
netstat  -antup | grep maxscale      # 查看端口

# 测试，在本机访问管理端口查看监控状态
maxadmin -P端口 -u用户名 -p密码
maxadmin -P4099 -uadmin   -pmariadb
MaxScale> list servers # 查看列表

# 客户端访问 4006 端口 进行操作
```



# Mysql 多实例

在一台服务器上，运行多个mysql服务

配置步骤

- 安装支持多实例服务的软件包
- 修改主配置文件，创建数据库目录
- 初始化授权库
- 启动服务

```mysql
# 安装软件包
mysql-5.7.20-linux-glibc2.12-x86_64.tar.gz
mv mysql-5.7.20-linux-glibc2.12-x86_64 /usr/local/mysql
# 停止mysqld 服务，取消开机启动，移走配置文件
systemctl stop mysqld
systemctl disable mysqld
mv /etc/my.cnf /etc/my.cnf.old
# 新建并编辑主配置文件/etc/my.cnf
vim /etc/my.cnf
[mysqld_multi]        # 启用多实例
mysqld = /usr/local/mysql/bin/mysqld_safe       # 指定进程文件路径
mysqladmin = /usr/local/mysql/bin/mysqladmin    # 指定管理命令路径
user = root        # 指定进程用户

[mysqld1]        # 实例进程名称，1 为实例名
port=3307        # 端口号
datadir=/data3307        # 数据库目录 ，要手动创建
socket=/data3307/mysqld.sock        # 指定sock文件的路径和名称。服务停止后，自动消失
pid-file=/data3307/mysql1.pid       # 进程pid号文件位置
log-error=/data3307/mysql1.err       # 错误日志位置 

[mysqld2]
port=3308
datadir=/data3308
socket=/data3308/mysqld.sock
pid-file=/data3308/mysql2.pid
log-error=/data3308/mysql2.err 

# 启动实例 1
/usr/local/mysql/bin/mysqld_multi start 1 # 第一次自动初始化，会显示临时密码
# 本地用 -S 指定sock登录
/usr/local/mysql/bin/mysql -uroot -p'pwd' -S /data3307/mysqld.sock
# 停止实例 1
/usr/local/mysql/bin/mysqld_multi --user=name --password=pwd stop 1

# 调整PATH变量，可以直接使用命令
echo  "export  PATH=/usr/local/mysql/bin:$PATH" >> /etc/profile
source /etc/profile
```



# MySQL 优化

- 升级硬件
- 优化运行参数
- 让程序员优化sql命令
- 带宽

## 优化运行参数

```mysql
mysql> show variables; # 显示所有变量
mysql> show status; # 显示所有状态
mysql> set [global] 变量名 值 # 临时更改变量/状态的值
```

常用设置参数

| 选项              | 作用                                               |
| ----------------- | -------------------------------------------------- |
| max_connections   | 允许的最大并发连接数                               |
| connect_timeout   | 等待连接超时，默认10秒，仅登录时有效               |
| wait_timeout      | 等待关闭连接的不活动超时秒数，默认28800秒（8小时） |
| key_buffer_size   | 用于MyISAM引擎的关键索引缓存大小                   |
| sort_buffer_size  | 为每个要排序的线程分配此大小的缓存空间             |
| read_buffer_size  | 为顺序读取表记录保留的缓存大小                     |
| thread_cache_size | 允许保存在缓存中被重用的线程数量                   |
| table_open_cache  | 为所有线程缓存的打开的表的数量                     |

记录的最大并发连接/允许的最大并发连接数=85%

Max_used_connections/max_connections=0.85

## SQL 查询优化

select 查询过程：先从查询缓存（存储已查询过的数据）查询，没有匹配结果再去硬盘读取，得到结果后，放入查询缓存。

在生产环境下，有单独的缓存服务器，不使用mysql服务器的内存。

```mysql
mysql> show variables like "query_cache%"; # 查询缓存相关
+------------------------------+---------+
| Variable_name                | Value   |
+------------------------------+---------+
| query_cache_limit            | 1048576 |
| query_cache_min_res_unit     | 4096    |
| query_cache_size             | 1048576 |   # 查询缓存 大小
| query_cache_type             | OFF     |   # 查询缓存 是否启动
| query_cache_wlock_invalidate | OFF     |
+------------------------------+---------+
5 rows in set (0.00 sec)

mysql> show global status like "qcache%"; # 查询缓存统计
+-------------------------+---------+
| Variable_name           | Value   |
+-------------------------+---------+
| Qcache_free_blocks      | 1       |
| Qcache_free_memory      | 1031832 |
| Qcache_hits             | 0       |    # 从查询缓存中得到结果的次数
| Qcache_inserts          | 0       |
| Qcache_lowmem_prunes    | 0       |
| Qcache_not_cached       | 11      |
| Qcache_queries_in_cache | 0       |
| Qcache_total_blocks     | 1       |
+-------------------------+---------+
8 rows in set (0.00 sec)
```

mysql 日志类型及选项

| 类型       | 用途                                                         | 配置                                                       |
| ---------- | ------------------------------------------------------------ | ---------------------------------------------------------- |
| 错误日志   | 记录启动/运行/停止过程中的错误消息                           | log-error[=name]                                           |
| 查询日志   | 记录客户端连接和查询操作，文件名称是mysql51.log              | general-log<br>general-log-file=                           |
| 慢查询日志 | 记录耗时较长或不使用索引的查询操作，文件名称是主机名-slow.log | slow-query-log<br>slow-query-log-file=<br>long-query-time= |

| 慢查询选项                   | 含义                 |
| ---------------------------- | -------------------- |
| slow-query-log               | 启用慢查询           |
| slow-query-log-file          | 指定慢查询日志文件   |
| long-query-time              | 超过时间（默认10秒） |
| long-query-not-using-indexes | 记录未使用索引的查询 |

```shell
vim /etc/my.cnf
...
slow_query_log=1
slow_query_log_file=mysql-slow.log
long_query_time=5
log_queries_not_using_indexes=1
# 查看慢查询日志
mysqldumpslow  /var/lib/mysql/mysql-slow.log
# 以下是结果
Reading mysql slow query log from /var/lib/mysql/mysql-slow.log 
Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows=0.0 (0), 0users@0hosts
```



# Mysql 集群

集群：多台服务器提供相同的服务

集群分类：

LB 负载集群

HA 高可用集群（主，备）

HPC 高计算集群

## MHA 高可用mysql集群软件

Master HIgh Availability

数据库故障自动切换操作能做到在0～30秒之内，MHA能确保在故障切换过程中保证数据的一致性。

组成：

MHA Manager（管理节点）

- 可以单独部署在一台独立的机器上，或部署在一台slave节点上

MHA Node（数据节点）

- 运行在每台MySQL服务器上

==manager 可以无密码登录所有node服务器。所有node服务器之间可以无密码登录==

```shell
# 安装node软件
yum -y install perl-*
rpm -ivh mha4mysql-node-0.56-0.el6.noarch.rpm # mysql服务器与管理都安装，先安装
# 安装manager软件
tar -zxf mha4mysql-manager-0.56.tar.gz
cd mha4mysql-manager-0.56/
perl  Makefile.PL # 配置perl程序
make && make install # 安装，成功后会有 masterha_* 命令

# 配置mysql主从同步，主用服务器与备用服务器启用半同步复制模式的 master 与 slave；纯备用启动slave
relay_log_purge=off # 主用服务器与备用服务器 关闭自动删除中继日志
mysql> grant all on *.* to root@"%" identified by "123456"; # 创建远程登录账户

# 配置 manager 服务
mkdir /etc/mha_manager
cp mha4mysql-manager-0.56/samples/conf/app1.cnf  /etc/mha_manager # 建立样板文件 
vim /etc/mha_manager/app1.cnf # 编辑主配置文件app1.cnf
[server default]
manager_workdir=/etc/mha_manager  # 定义工作目录
manager_log=/etc/mha_manager/manager.log
master_ip_failover_script=/etc/mha_manager/master_ip_failover # 指定故障切换脚本
ssh_user=root # 远程登录mysql所在服务器的用户名
ssh_port=22
repl_user=repluser # 远程同步数据mysql数据库的用户名
repl_password=123456
user=root   # 查看mysql数据库的状态的用户名
password=123456
[server1]
hostname=192.168.4.51    
port=3306
candidate_master=1 # 竞选主库
[server2]
hostname=192.168.4.52
port=3306            
candidate_master=1
[server3]
hostname=192.168.4.53
port=3306
candidate_maste masterha_check_ssh  --conf=/etc/mha_manager/app1.cnfr=1
[server4]
hostname=192.168.4.54
no_master=1
[server5]
hostname=192.168.4.55
no_master=1

# 创建故障切换的脚本
cp samples/scripts/master_ip_failover /etc/mha_manager
修改 master_ip_failover 脚本，设置如下内容
34 my $vip = '192.168.4.100/24'; # 指定vip
35 my $key = "1";
36 my $ssh_start_vip = "/sbin/ifconfig eth0:$key $vip";
37 my $ssh_stop_vip = "/sbin/ifconfig eth0:$key down";

# 给主用服务器配置临时vip
ifconfig eth0:1 192.168.4.100/24

#  验证ssh 免密登陆数据节点主机
masterha_check_ssh  --conf=/etc/mha_manager/app1.cnf
# 验证数据节点的主从同步配置
masterha_check_repl --conf=/etc/mha_manager/app1.cnf
# 启动管理服务MHA_Manager,主库故障后，次进程会在重新配置主库后结束自身进程。还会清除原备从的slave配置
masterha_manager --conf=/etc/mha_manager/app1.cnf -remove_dead_master_conf --ignore_last_failover
--remove_dead_master_conf # 删除宕机主库配置，当发生主从切换后，老的主库的ip将会从配置文件中移除。
--ignore_last_failover # 忽略xxx.health文件,方便切换
# 查看状态
masterha_check_status  --conf=/etc/mha_manager/app1.cnf
# 停止服务
masterha_stop  --conf=/etc/mha_manager/app1.cnf
```

| manager命令              | 作用                                  |
| ------------------------ | ------------------------------------- |
| masterha_check_ssh       | 检查MHA的SSH配置状况                  |
| masterha_check_repl      | 检测MySQL复制状况                     |
| masterha_manager         | 启动MHA运行状态                       |
| masterha_check_status    | 检查MHA运行状态                       |
| masterha_manager_monitor | 检测master服务器（主mysql服务器）状态 |



# Mysql 视图

虚拟表，内容与真实的表相似，有字段有记录，视图并不在数据库中以存储的数据形式存在，视图与原表的值互相同步。

视图优点

简单：用户不需要关心视图中的数据如何查询获得，视图中的数据已经是过滤好的符合条件的结果集

安全：用户只能看到视图中的数据

数据独立：一旦视图结构确定，可以屏蔽表结构对用户的影响

视图限制

不能在视图上创建索引

不能使用子查询

```mysql
create view 视图名称 as SQL查询;
create view 视图名称(字段名列表) as SQL查询; # 创建时，若视图已经存在，使用 create or replace 会替换已有的视图
# 对视图的值的增删改查与表一样
drop view 视图名; # 删除视图

# 视图中的字段名不可以重复，需要定义字段别名
create view 视图名 as select 表别名.源字段名 字段别名 from 源表名 表别名 left join 源表名 表别名 on 条件（也要使用别名）;
```

视图算法 ALGORITHM

merge：合并算法，将生成视图的语句和外层的语句合并后在执行。

temptable：临时表算法，先将视图生成一个临时表，再执行外层语句。一次查询会执行两个查询命令。

undefined：未定义，MySQL到底用merge还是用temptable由MySQL决定，这是一个默认的算法，一般视图都会选择merge算法，因为merge效率高。

```mysql
# 在创建视图的时候指定视图的算法
create algorithm=temptable view 视图名 as select 语句
```

限制视图操作

```mysql
CREATE TABLE t1 (a INT);
CREATE VIEW v1 AS SELECT * FROM t1 WHERE a < 2 WITH CHECK OPTION;
CREATE VIEW v2 AS SELECT * FROM v1 WHERE a > 0 WITH LOCAL CHECK OPTION;  
CREATE VIEW v3 AS SELECT * FROM v1 WHERE a > 0 WITH CASCADED CHECK OPTION;
mysql> INSERT INTO v2 VALUES (2);
ERROR 1369 (HY000): CHECK OPTION failed 'test.v2'
mysql> INSERT INTO v3 VALUES (2);
ERROR 1369 (HY000): CHECK OPTION failed 'test.v3'
```

**5.7.6之前：**

- WITH  LOCAL CHECK OPTION

　　会检验视图v4WHERE子句下的条件，但是不会检验底层视图v3的WHERE子句条件

- WITH CASCADED CHECK OPTION

　　会检查视图v4WHERE子句下的条件，然后检查底层视图v3的WHERE条件

- 没有check option

　　均不检查

 

**5.7.6版本：**

- WITH  LOCAL CHECK OPTION

　　会检验视图v4WHERE子句下的条件，然后检验底层视图v3的WHERE子句条件

- WITH CASCADED CHECK OPTION

　　会检查视图v4WHERE子句下的条件，然后检查底层视图v3的WHERE子句条件

- 没有check option

　　不会检查视图v4WHERE子句下的条件，但会检查底层视图三的WHERE子句条件



# 存储过程

存储过程相当于mysql语句组成的脚本，可以使用变量，条件判断，流程控制

优点

- 提高性能
- 可减轻网络负担
- 可以防止对表的直接访问
- 避免重复编写sql操作

```mysql
mysql> delimiter //  # 将命令行结束符号改为// 注意空格
# 创建存储过程
mysql> create procedure 库名.名称（） begin SQL语句 end//
mysql> delimiter ; # 将命令行结束符号改为; 注意空格
# 查看存储过程
mysql> show procedure status;
mysql> select db,name,type，body from mysql.proc where name='存储过程名'; # body SQL语句
# 调用存储过程
mysql> call 库名.存储过程名();
# 删除存储过程
mysql> drop procedure 库名.存储过程名;
```

变量类型

| 名称           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| 会话变量       | 系统变量的一部分，修改仅影响当前的连接                       |
| 全局变量       | 系统变量的一部分，修改影响到整个服务器                       |
| 用户自定义变量 | 在命令行使用，在客户端连接到数据库服务的整个过程中都有效，断开连接后失效。 |
| 局部变量       | 只能在存储过程中使用，语句块执行完后失效。declare专门用来定义局部变量 |

```mysql
mysql> show global variables; # 查看全局变量
mysql> show session variables; # 查看会话变量
mysql> set session sort_buffer_size=40000; # 设置会话变量
mysql> select @@hostname，@@version # 输出系统变量

mysql> set @y=3; # 用户自定义变量，直接赋值
mysql> select max(uid) into @y from user; # 使用sql命令查询结果赋值
mysql> select @y; # 输出用户自定义变量的值

# 定义局部变量并使用
mysql> delimiter //
mysql> create procedure db9.p2() begin declare x int default 9;declare y char(10);set y='jim';select x;select y;end//
mysql> delimiter ;
```

参数类型

| 关键字 | 名称      | 描述                                                         |
| ------ | --------- | ------------------------------------------------------------ |
| in     | 输入参数  | 给存储过程传值，必须在调用存储过程时附值，在存储过程中该参数的值不允许修改；默认类型是in |
| out    | 输出参数  | 该值可在存储过程内部被修改，并可返回。                       |
| inout  | 输入/输出 | 调用时指定，并且可被改变和返回                               |

```mysql
mysql> delimiter //

mysql> CREATE PROCEDURE simpleproc (OUT param1 INT)
    -> BEGIN
    ->   SELECT COUNT(*) INTO param1 FROM t;
    -> END//
Query OK, 0 rows affected (0.00 sec)

mysql> delimiter ;

mysql> CALL simpleproc(@a);
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @a;
+------+
| @a   |
+------+
| 3    |
+------+
1 row in set (0.00 sec)

mysql> delimiter ;
```

四则运算

| 符号 | 描述 | 示例                        |
| ---- | ---- | --------------------------- |
| +    | 加   | set @var1=2+2;  4           |
| -    | 减   | set @var1=2-2;  0           |
| *    | 乘   | set @var1=2*2;  4           |
| /    | 除   | set @var1=10/3; 3.333333333 |
| DIV  | 整除 | set @var1=10 DIV 3; 3       |
| %    | 取模 | set @var1=10%3; 1           |

数值比较

| 类型              | 用途                 |
| ----------------- | -------------------- |
| =                 | 等于                 |
| \>,>=             | 大于，大于或等于     |
| <,<=              | 小于，小于或等于     |
| !=                | 不等于               |
| between .. and .. | 在。。。和。。。之间 |

逻辑条件

| 类型              | 用途                             |
| ----------------- | -------------------------------- |
| OR,AND,!          | 或，与，非                       |
| in .., not in ... | 在。。。范围内，不在。。。范围内 |
| is null           | 字段的值为空                     |
| is not null       | 字段的值不为空                   |
| like              | 模糊匹配                         |
| regexp            | 正则匹配                         |

```mysql
# if syntax
DELIMITER //

CREATE FUNCTION SimpleCompare(n INT, m INT)
  RETURNS VARCHAR(20)

  BEGIN
    DECLARE s VARCHAR(20);

    IF n > m THEN SET s = '>';
    ELSEIF n = m THEN SET s = '=';
    ELSE SET s = '<';
    END IF;

    SET s = CONCAT(n, ' ', s, ' ', m);

    RETURN s;
  END //

DELIMITER ;

# while syntax
CREATE PROCEDURE dowhile()
BEGIN
  DECLARE v1 INT DEFAULT 5;

  WHILE v1 > 0 DO
    ...
    SET v1 = v1 - 1;
  END WHILE;
END;

# loop 死循环
[begin_label:] LOOP
    statement_list
END LOOP [end_label]


# repeat 直到。。。结束循环
mysql> delimiter //

mysql> CREATE PROCEDURE dorepeat(p1 INT)
       BEGIN
         SET @x = 0;
         REPEAT
           SET @x = @x + 1;
         UNTIL @x > p1 END REPEAT;
       END
       //
Query OK, 0 rows affected (0.00 sec)

mysql> CALL dorepeat(1000)//
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @x//
+------+
| @x   |
+------+
| 1001 |
+------+
1 row in set (0.00 sec)
```



# Mysql 数据分片

将存放在一个数据库里的数据按照特定方式拆分，分散存放到多个数据库（主机）中。





分割方式：

垂直分割（纵向切分）

将单个表，拆分成多个表，分散到不同的数据库。

将单个数库局的多个表进行分类，按业务类别分散到不同的数据库上。

水平分割（横向切分）

按照表中某个字段的某种规则，把表中的许多记录按行切分，分散到多个数据库中。



Mycat

适合数据大量写入的存储需求。

分片规则

- 枚举法 sharding-by-intfile
- 固定分片 rule1
- 范围约定 auto-sharding-long
- 求模法 mod-long
- 日期列分区法 sharding-by-date
- 通配取模 sharding-by-pattern
- ASCII码求模通配 sharding-by-prefixpattern
- 编程指定 sharding-by-substring
- 字符串拆分hash解析 sharding-by-stinghash
- 一致性hash sharding-by-murmur

```shell
# 验证openjdk
rpm -qa | grep jdk
# 安装mycat
tar -xf Mycat-server-1.4-beta-20150604171601-linux.tar.gz
mv mycat/ /usr/local/
```

目录结构说明：

- bin mycat命令，如 启动 停止 等

- catlet 扩展功能

- conf 配置文件

- lib mycat使用的jar

- log mycat启动日志和运行日志

- wrapper.log mycat服务启动日志

- mycat.log 记录SQL脚本执行后的报错内容



重要配置文件说明：

- server.xml 设置连mycat的账号信息

- schema.xml 配置mycat的真实库表

- rule.xml 定义mycat分片规则

  

配置标签说明

\<user>.. ..\</user> 定义连mycat用户信息

\<datanode>.. ..\</datanode> 指定数据节点

\<datahost>.. ..\</datahost>	指定数据库地址及用户信息

```shell
# 查看server.xml配置文件
vim server.xml
</system>
<user name="test">        # 连接mycat服务时使用的用户名 test
	<property name="password">test</property> # 使用test用户连接mycat用户时使用的密码
	<property name="schemas">TESTDB</property> # 连接上mycat服务后，可以看到的库名多个时，使用逗号分隔 （是逻辑上的库名,服务器上没有这个库名，随便取，但要记住）
</user>
<user name="user">
	<property name="password">user</property>
	<property name="schemas">TESTDB</property>
	<property name="readOnly">true</property> # 定义只读权限，使用定义的user用户连接mycat服务后只有读记录的权限,不写这一行则是可读可写
</user>

# 修改schema.xml配置文件
vim schema.xml
<schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100">
<table name="travelrecord" dataNode="dn1,dn2" rule="auto-sharding-long" />
# travelrecord（逻辑上的，名字不能随便写，一般不动）表分片到数据节点dn1和dn2，dn1和dn2随便取的名字
<table name="company" primaryKey="ID" type="global" dataNode="dn1,dn2" />
<table name="hotnews" primaryKey="ID" dataNode="dn1,dn2" rule="mod-long" />
</schema>

<dataNode name="dn1" dataHost="c1" database="db1" />
# 数据节点对应的服务器 name="dn1"名称要与上面的对应 dataHost="c1"写本机主机名，database="db1"存在的数据库名,定义分片使用的库，所在的物理主机，真正存储数据的db1库在物理主机mysql55上
<dataNode name="dn2" dataHost="c2" database="db2" />
# 定义分片使用的库，所在的物理主机，真正存储数据的db1库在物理主机mysql55上

# 指定c1名称主机对应的ip地址
<dataHost name="c1" maxCon="1000" minCon="10" balance="0"
                writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                        <heartbeat>select user()</heartbeat>
                <!-- can have multi write hosts -->
                <writeHost host="c1" url="192.168.4.55:3306" user="admin"        
                        password="123456">
# 访问数据库时，mycat服务连接数据库服务器时使用的用户名和密码
                        <!-- can have multi read hosts -->
                </writeHost>
        </dataHost>
# 指定c2名称主机对应的ip地址
        <dataHost name="c2" maxCon="1000" minCon="10" balance="0"    
                 writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                        <heartbeat>select user()</heartbeat>
                <!-- can have multi write hosts -->
                <writeHost host="c2" url="192.168.4.54:3306" user="admin"        
                         password="123456">
# 访问数据库时，mycat服务连接数据库服务器时使用的用户名和密码
                        <!-- can have multi read hosts -->
                </writeHost>
        </dataHost>
        
        
# 启动
/usr/local/mycat/bin/mycat start

ls /usr/local/mycat/logs/
mycat.log  mycat.pid  wrapper.log       # wrapper.log为错误日志

ldconfig  -v        # 更新加载的模块

# 客户端访问
mysql -hmycat主机的IP -P端口号 -u用户 -p密码
```



# Redis 远程字典服务器

redis 支持 字符，list，hash，集合，有序集合 数据类型

| 配置         | 内容                        |
| ------------ | --------------------------- |
| 默认端口     | 6379                        |
| 启动程序     | /etc/init.d/redis_6379      |
| 主配置文件   | etc/redis/6379.conf         |
| 日志文件     | /var/log/redis_6379.log     |
| 数据目录     | /var/lib/redis/6379         |
| 启动程序目录 | /usr/local/bin/redis-server |



```shell
# 安装redis
yum -y install gcc
tar -zxf redis.tar.gz
cd redis/
make && make install
# 初始化配置 端口，主配置文件，数据库目录，pid文件，启动程序
./redis/utils/install_server.sh # 运行次程序，自动初始化
Welcome to the redis service installer
This script will help you easily set up a running redis server
Please select the redis port for this instance: [6379] 
Selecting default: 6379
Please select the redis config file name [/etc/redis/6379.conf] 
Selected default - /etc/redis/6379.conf
Please select the redis log file name [/var/log/redis_6379.log] 
Selected default - /var/log/redis_6379.log
Please select the data directory for this instance [/var/lib/redis/6379] 
Selected default - /var/lib/redis/6379
Please select the redis executable path [/usr/local/bin/redis-server] 
Selected config:
Port           : 6379                   # 端口号
Config file    : /etc/redis/6379.conf         # 配置文件目录
Log file       : /var/log/redis_6379.log      # 日志目录
Data dir       : /var/lib/redis/6379          # 数据库目录
Executable     : /usr/local/bin/redis-server  # 启动程序的目录
Cli Executable : /usr/local/bin/redis-cli     # 命令行的连接工具
Is this ok? Then press ENTER to go on or Ctrl-C to abort.  # 回车完成配置
Copied /tmp/6379.conf => /etc/init.d/redis_6379    # 服务启动脚本
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!        //安装成功

# 查看状态
/etc/init.d/redis_6379 status
# 连接本地redis服务
redis-cli
# 常用指令
set keyname value ex 2 px 1000 # 储存 ex 秒 px 毫秒
get keyname # 获取
select 数据库编号0-15 # 切换库
kyes * # 打印所有变量
kyes a? # 打印指定变量 ? 代表一个字符
exists keyname # 是否存在变量 1 代表存在
ttl keyname # 查看生存时间，-1 永久 -2 过期
type keyname # 查看类型
move keyname dbname # 移动变量
expire keyname 10 # 设置有效时间
del keyname # 删除指定变量
flushall # 删除所有变量
save # 保存所有变量
shutdown # 关闭redis服务

# 修改Redis服务运行参数
cp /etc/redis/6379.conf  /root/6379.conf     # 可以先备份一份，防止修改错误没法还原
/etc/init.d/redis_6379 stop
vim /etc/redis/6379.conf
...
bind  192.168.4.51                # 设置服务使用的ip，由此IP接受的请求才会处理
port 6351                         # 更改端口号
requirepass 123456                # 设置密码
# 以上任意一项修改后，不能使用脚本停止服务
/etc/init.d/redis_6379 start
Starting Redis server...

ss -antul | grep 6351        # 查看有端口6351
tcp    LISTEN     0      128    192.168.4.51:6351                  *:*

# 由于修改了配置文件所以在连接的时候需要加上ip和端口
redis-cli  -h 192.168.4.51 -p 6351
127.0.0.1:6379> auth 123456  # 输入密码连接
# 还可以直接在命令行输入密码连接
redis-cli  -h 192.168.4.51 -p 6351  -a 123456
# 停止服务
redis-cli  -h 192.168.4.51 -p 6351   -a 123456 shutdown
```



## redis和memcached的区别

观点一：

1、Redis和Memcache都是将数据存放在内存中，都是内存数据库。不过memcache还可用于缓存其他东西，例如图片、视频等等；

2、Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，hash等数据结构的存储；

3、虚拟内存-Redis当物理内存用完时，可以将一些很久没用到的value 交换到磁盘；

4、过期策略--memcache在set时就指定，例如set key1 0 0 8,即永不过期。Redis可以通过例如expire 设定，例如expire name 10；

5、分布式--设定memcache集群，利用magent做一主多从;redis可以做一主多从。都可以一主一从；

6、存储数据安全--memcache挂掉后，数据没了；redis可以定期保存到磁盘（持久化）；

7、灾难恢复--memcache挂掉后，数据不可恢复; redis数据丢失后可以通过aof恢复；

8、Redis支持数据的备份，即master-slave模式的数据备份；



redis 配置文件

| 常用项                          | 作用                           |
| ------------------------------- | ------------------------------ |
| port 6379                       | 端口                           |
| bind 127.0.0.1                  | IP地址                         |
| tcp-backlog 511                 | tcp连接总数                    |
| timeout 0                       | 连接超时时间，0代表一直连接    |
| tcp-keepalive 300               | 长连接时间，交互时间           |
| daemmonize yes                  | 守护进程方式运行，no是休眠方式 |
| databases 16                    | 数据库数量                     |
| logfile /var/log/redis_6379.log | 日志目录                       |
| maxclients 10000                | 并发连接数量                   |
| dir /var/lib/redis/6379         | 数据库目录                     |

| 内存管理        | 作用                            |
| --------------- | ------------------------------- |
| volatile-lru    | 最近最少使用（设置了ttl的变量） |
| allkeys-lru     | 删除最少使用的变量              |
| volatile-random | 随机删除（设置了ttl的变量）     |
| allkeys-random  | 随机删除变量                    |
| volatile-ttl    | 删除最近过期的变量              |
| noeviction      | 不删除，写满时报错              |

```shell
# 选项默认设置
maxmemory <bytes> # 最大内存
maxmemory-policy noeviction # 定义使用的策略
maxmemory-samples 5 # ttl 与 lru 删除时对比例子个数
#maxmemory-samples在redis-3.0.0中的默认配置为5，如果增加，会提高LRU或TTL的精准度，redis作者测试的结果是当这个配置为10时已经非常接近全量LRU的精准度了，并且增加maxmemory-samples会导致在主动清理时消耗更多的CPU时间。
```



## Redis 高可用集群

```shell
vim /etc/redis/6379.conf
...
bind 192.168.4.51        # 修改ip
port 6351        # 不允许相同，只指定物理接口的ip
daemonize yes         # 以守护进程方式运行
pidfile /var/run/redis_6351.pid 
cluster-enabled yes     # 是否启用集群，前提是以守护进程方式运行
cluster-config-file nodes-6351.conf   # 存储集群信息的配置文件，自动生成，不允许相同
cluster-node-timeout 5000        # 集群节点通信超时时间

netstat -tanulp | grep redis
tcp        0      0 192.168.4.53:6353       0.0.0.0:*    LISTEN    1426/redis-server 1 
tcp        0      0 192.168.4.53:16353      0.0.0.0:*    LISTEN    1426/redis-server 1  # 集群端口

127.0.0.1:6379> cluster info # 查看集群信息
127.0.0.1:6379> cluster nodes # 查看集群节点信息

# cluster语言是ruby语言
yum -y install  ruby rubygems ruby-devel
gem install redis-3.2.1.gem
cp /root/redis/redis-4.0.8/src/redis-trib.rb /usr/local/bin/
chmod +x /usr/local/bin/redis-trib.rb

# 创建集群，必须有三台机器是主库服务器
redis-trib.rb  create --replicas 1 \ 
192.168.4.51:6351  192.168.4.52:6352 \ 
192.168.4.53:6353 192.168.4.54:6354  \ 
192.168.4.55:6355 192.168.4.56:6356
# --replicas 1 给每一个主库配置一个从库。以上会生成3个主库，3个从库

# 5.0.5 创建集群命令
redis-cli -h 192.168.4.51 -p 6351 --cluster create --cluster-replicas 1 192.168.4.51:6351  192.168.4.52:6352 192.168.4.53:6353 192.168.4.54:6354  192.168.4.55:6355 192.168.4.56:6356

ls /var/lib/redis/6379/ # 集群目录

redis-cli --cluster check 192.168.4.54:6354 # 查看集群信息

# 测试集群
redis-cli -c -h ip地址 -p 端口 # -c 使用集群
```

集群不能用的情况：

有半数或者半数以上的主库机器挂掉或者一组主从都坏掉，集群就不能用了

把一个从库升级成主，没有从库，集群不能用（前提是：有半数或者半数以上的主库机器挂掉）

一个主库挂掉，它的从库自动顶替为主库，正常使用（前提是：有半数或者半数以上的主库机器能用），挂掉的主库修复好后，会成为从库，不会抢占为主

6）集群节点选举策略（三主，三从）

停止某个主库的redis服务，对应的从库会自动升级为主库

先查看节点信息的主从情况



redis 集群管理

格式 redis-trib.rb 选项 参数

| 选项             | 作用           |
| ---------------- | -------------- |
| add-node         | 添加master主机 |
| check            | 检测集群       |
| reshard          | 重新分片       |
| add-node --slave | 添加slave主机  |
| del-node         | 删除主机       |

添加master主机步骤

- 添加master主机
- 检查主机
- 重新分片

```shell
redis-trib.rb  reshard   192.168.4.58:6358  
How many slots do you want to move (from 1 to 16384)?4096 # 拿出多少个hash 槽给主机192.168.4.58
What is the receiving node ID?  c5e0da48f335c46a2ec199faa99b830f537dd8a0 # 主机192.168.4.58的id值
Source node #1:all      # 从当前所有的主里面获取hash槽
Do you want to proceed with the proposed reshard plan (yes/no)?yes

# 删除master角色的主机 先删除主机占用的hash槽
redis-trib.rb  reshard 192.168.4.58:6358
How many slots do you want to move (from 1 to 16384)?4096  # 移除hash 槽的个数
What is the receiving node ID?  bc5c4e082a5a3391b634cf433a6486c867cfc44b # 要移动给谁的id即目标主机（这里可以随机写一个master的ID）  
Source node #1: c5e0da48f335c46a2ec199faa99b830f537dd8a0
//从谁那移动即源主机（这里写4.58的ID）  
Source node #2:done           # 设置完毕
redis-trib.rb del-node 192.168.4.58:6358 c5e0da48f335c46a2ec199faa99b830f537dd8a0    # 删除谁+删除的id

# 添加从节点主机
redis-trib.rb  add-node  --slave 192.168.4.57:6357  192.168.4.51:6351
```

移除从节点，从节点主机没有槽位范围，直接移除即可
		命令格式：
		redis-trib.rb del-node 192.168.4.57:6357 主机id值



## redis 主从同步

### redis 主从同步工作原项目相关信息理

- slave 向 master 发送 sync 命令
- master 启动后台存盘进程，并收集所有修改数据命令
- master 完成后台存盘后，传送整个数据文件到slave
- slave 接收数据文件，加载到内存中完成首次完全同步
- 后续有新数据产生时，master 继续将新的数据收集到的修改命令依次传给slave，完成同步

缺点

网络繁忙，系统繁忙，造成数据同步延时。

```shell
# 配置主从关系，在从库上输入命令
redis-cli -h 192.168.4.52 -p 6352
192.168.4.52:6352> slaveof 192.168.4.51 6351 # 作为51的从库

# 关闭主从关系
192.168.4.52:6352> slaveof no one

# 永久配置主从
vim /etc/redis/6379.conf
replicaof <masterip> <masterport>
```

配置带验证的主从复制

```shell
# 主库配置密码，然后重启服务
vim /etc/redis/6379.conf
requirepass 123456
# 从库配置
vim /etc/redis/6379.conf
replicaof <masterip> <masterport>
masterauth 123456 # 指定主库的密码
```

配置哨兵，从库在主库宕机后自动成为主库。

```shell

vim /etc/redis/sentinel.conf
bind 0.0.0.0 # 监听所有网卡
port 26379
sentinel monitor redis51 192.168.4.51 6351 1
#				  主机名   ip地址	   端口  票数
# 票数：有几台哨兵主机连接不上主库时，切换主库。
sentinel auth-pass redis51 password

# 启动哨兵服务
redis-sentinel /etc/redis/sentinel.conf
```



## Redis 数据持久化

RDB 模式

按照指定时间间隔，将内存中的数据集快照写入硬盘，恢复时，将快照文件直接读入内存。

==只要停止服务，就会把内存数据存入硬盘==

```shell
vim /etc/redis/6379.conf
# 相关参数
dbfilename "dump.rdb" # 文件名称
save "" # 禁用rdb
save 900 1   # 900秒修改1次就保存
save 300 10    # 300秒修改10次就保存
save 60 10000 # 60秒修改10000次就保存

# 手动立刻存盘
save     # 阻塞写存盘
bgsave    # 不阻塞写存盘

# 压缩
rdbcompression yes|no
# 在存储快照后，使用crc16算法做数据校验
rdbchecksum yes|no
# bgsave出错时停止写操作
stop-writes-on-bgsave-error yes|no
```

RDB 优点

- 高性能的持久化实现——创建一个子进程来执行持久化，先将数据写入临时文件，持久化过程结束后，再用这个临时文件替代替换上次持久化产生的文件；过程中主进程不做任何I/O操作。
- 比较适合大规模数据恢复，且对数据完整性要求不是非常高的场合。

RDB 缺点

- 意外宕机时，最后一次持久化的数据会丢失。



AOF 模式

类似mysql的binlog日志文件，可以使用cat查看

==如果同时有RDB，AOF。只读AOF==

```shell
# 相关配置
appendfilename "appendonly.aof"    # 文件名
appendonly no|yes

appendfsync always               # 有新修改数据操作立即记录操作与数据
appendfsync everysec             # 每秒记录一次操作与数据
appendfsync no                   # 先记录操作，等系统闲置时再写数据

# 日志重写配置，默认配置当aof文件是上次rewrite后大小的1倍且文件大于64M时触发
auto-aof-rewrite-percentage 100 
auto-aof-rewrite-min-size 64mb

# 修复AOF文件
redis-check-aof -fix appendonly.aof
```

AOF 优点

- 可以灵活设置持久化方式
- 意外宕机时，仅可能丢失1秒的数据

AOF 缺点

- 持久化文件体积通常比RDB大
- 速度比RDB慢



## 数据类型

### String字符串

set key value [ex seconds] [px milliseconds] [nx|xx]

设置key及值，过期时间可以使用秒或毫秒为单位

setrange key offset value

从偏移量开始复写key的特定位的值

```shell
redis-cli -h 192.168.4.51 -a 123456
192.168.4.51:6379> set  first  "hello world"
OK
192.168.4.51:6379> setrange  first  6  "Redis"    # 改写为hello Redis
(integer) 11
192.168.4.51:6379> get first
"hello Redis"
```

strlen key，统计字串长度

```shell
192.168.4.51:6379> strlen first
(integer) 11
```

append key value 存在则追加，不存在则创建key及value，返回key长度

```shell
192.168.4.51:6379> append myname jacob
(integer) 5
```

setbit key offset value 对key所存储字串，设置或清除特定偏移量上的位(bit)，value值可以为1或0，offset为0~2^32之间，key不存在，则创建新key

```shell
192.168.4.51:6379> setbit  bit  0  1          # 设置bit第0位为1
(integer) 0
192.168.4.51:6379> setbit  bit  1  0          # 设置bit第1位为0 
(integer) 0
```

bitcount key 统计字串中被设置为1的比特位数量

```shell
192.168.4.51:6379> setbit  bits 0 1        # 0001
(integer) 0
192.168.4.51:6379> setbit  bits 3 1        # 1001
(integer) 0
192.168.4.51:6379> bitcount  bits           # 结果为2
(integer) 2
```

记录网站用户上线频率，如用户A上线了多少天等类似的数据，如用户在某天上线，则使用setbit，以用户名为key，将网站上线日为offset，并在该offset上设置1，最后计算用户总上线次数时，使用bitcount用户名即可，这样即使网站运行10年，每个用户仅占用10*365比特位即456字节

```shell
192.168.4.51:6379> setbit  peter  100  1        # 网站上线100天用户登录了一次
(integer) 0
192.168.4.51:6379> setbit  peter  105  1        # 网站上线105天用户登录了一次
(integer) 0
192.168.4.51:6379> bitcount  peter
(integer) 2
```

decr key 将key中的值减1，key不存在则先初始化为0，再减1

```shell
192.168.4.51:6379> set z 10
OK
192.168.4.51:6379> decr z
(integer) 9
192.168.4.51:6379> decr z
(integer) 8
192.168.4.51:6379> decr bb
(integer) -1
192.168.4.51:6379> decr bb
(integer) -2
```

decrby key decrement 将key中的值，减去decrement

```shell
192.168.4.51:6379> set count 100
OK
192.168.4.51:6379> DECRBY cc 20    //定义每次减少20（步长）
(integer) -20
192.168.4.51:6379> DECRBY cc 20
(integer) -40
```

get key 返回key存储的字符串值，若key不存在则返回nil，若key的值不是字串，则返回错误，get只能处理字串

```shell
192.168.4.51:6379> get a
(nil)
```

getrange key start end 返回字串值中的子字串，截取范围为start和end，负数偏移量表示从末尾开始计数，-1表示最后一个字符，-2表示倒数第二个字符

```shell
192.168.4.51:6379> set x 123456789
OK
192.168.4.51:6379> getrange x -5 -1
"56789"
192.168.4.51:6379> getrange x 0 4
"12345"
```

incr key 将key的值加1，如果key不存在，则初始为0后再加1，主要应用为计数器

```shell
192.168.4.51:6379> set page 20
OK
192.168.4.51:6379> incr page
(integer) 21
```

incrby key increment 将key的值增加increment

```shell
192.168.4.51:6379> set x 10
OK
192.168.4.51:6379> incr x
(integer) 11
192.168.4.51:6379> incr x
(integer) 12
```

incrbyfloat key increment 为key中所储存的值加上浮点数增量 increment

```shell
192.168.4.51:6379> set num 16.1
OK
192.168.4.51:6379> incrbyfloat num 1.1
"17.2"
```

mset key value [key value …] 设置多个key及值，空格分隔，具有原子性

```shell
192.168.4.51:6379> mset j 9  k 29
OK
```

mget key [key…] 获取一个或多个key的值，空格分隔，具有原子性

```shell
192.168.4.51:6379> mget j k
1) "9"
2) "29"
```



### list列表

Redis的list是一个字符队列，先进后出，一个key可以有多个值

lpush key value [value…] 将一个或多个值value插入到列表key的表头，Key不存在，则创建key

```shell
192.168.4.51:6379> lpush list a b c        # list值依次为c b a
(integer) 3
```

lrange key start stop 从开始位置读取key的值到stop结束

```shell
192.168.4.51:6379> lrange list 0 2        # 从0位开始，读到2位为止
1) "c"
2) "b"
3) "a"
192.168.4.51:6379> lrange list 0 -1     # 从开始读到结束为止
1) "c"
2) "b"
3) "a"
192.168.4.51:6379> lrange list 0 -2       # 从开始读到倒数第2位值
1) "c"
2) "b"
```

lpop key 移除并返回列表头元素数据，key不存在则返回nil

```shell
192.168.4.51:6379> lpop list        //删除表头元素，可以多次执行
"c"
192.168.4.51:6379>  LPOP list
"b"
```

llen key 返回列表key的长度

```shell
192.168.4.51:6379>  llen list
(integer) 1
```

lindex key index 返回列表中第index个值

```shell
192.168.4.51:6379> lindex  list  1
"c"
```

lset key index value 将key中index位置的值修改为value

```shell
192.168.4.51:6379> lpush list a b c d 
(integer) 5
192.168.4.51:6379> lset list 3 test        # 将list中第3个值修改为test
OK
```

rpush key value [value…] 将value插入到key的末尾

```shell
192.168.4.51:6379> rpush list3  a b c  # list3值为a b c
(integer) 3
192.168.4.51:6379> rpush list3 d    # 末尾插入d
(integer) 4
```

rpop key 删除并返回key末尾的值

```shell
192.168.4.51:6379> RPOP list3 
"d"
```



### hash表

hset key field value 将hash表中field值设置为value

```shell
192.168.4.51:6379> hset site google 'www.g.cn'
(integer) 1
192.168.4.51:6379> hset site baidu 'www.baidu.com'
(integer) 1
```

hget key filed 获取hash表中field的值

```shell
192.168.4.51:6379> hget site google
"www.g.cn"
```

hmset key field value [field value…] 同时给hash表中的多个field赋值

```shell
192.168.4.51:6379> hmset site google www.g.cn  baidu www.baidu.com
OK
```

hmget key field [field…] 返回hash表中多个field的值

```shell
192.168.4.51:6379> hmget site google baidu
1) "www.g.cn"
2) "www.baidu.com"
```

hkeys key 返回hash表中所有field名称

```shell
192.168.4.51:6379> hmset site google www.g.cn baidu www.baidu.com
OK
192.168.4.51:6379> hkeys  site
1) "google"
2) "baidu"
```

hgetall key 返回hash表中所有key名和对应的值列表

```shell
192.168.4.51:6379> hgetall site
1) "google"
2) "www.g.cn"
3) "baidu"
4) "www.baidu.com"
```

hvals key 返回hash表中所有key的值

```shell
192.168.4.51:6379> hvals site
1) "www.g.cn"
2) "www.baidu.com"
```

hdel key field [field…] 删除hash表中多个field的值，不存在则忽略

```shell
192.168.4.51:6379> hdel  site  google  baidu
(integer) 2
```



## Redis 性能测试

```shell
# 主机为 127.0.0.1，端口号为 6379，执行的命令为 set,lpush，请求数为 10000，通过 -q 参数让结果只显示每秒执行的请求数。
redis-benchmark -h 127.0.0.1 -p 6379 -t set,lpush -n 10000 -q

SET: 146198.83 requests per second
LPUSH: 145560.41 requests per second
```



# MongoDB服务

介于关系数据库和非关系数据库之间的产品

一款基于分布式文件存储的数据库，目的在于为web应用提供可扩展的高性能数据存储解决方案。

将数据存储为一个文档（类似JSON），数据结构由键值对组成

支持丰富的查询表达式，可以设置索引。

支持副本集（高可用），分片。

```shell
# 安装
tar -xf /opt/mongodb-linux-x86_64-rhel70-3.6.3.tgz
cp -r mongodb-linux-x86_64-rhel70-3.6.3/bin /usr/local/mongod
cd /usr/local/mongod
mkdir -p etc log data/db
vim etc/mongodb.conf
dbpath=/usr/local/mongodb/data/db/    # 指定数据库目录
logpath=/usr/local/mongodb/log/mongodb.log    # 指定日志文件
logappend=true      # 以追加的方式记录日志信息
fork=true       # 服务以守护进程的方式运行

# 启动
./bin/mongod -f /usr/local/mongodb/etc/mongodb.conf

netstat -taunlp | grep mongod
tcp        0      0 127.0.0.1:27017         0.0.0.0:*               LISTEN      2900/./bin/mongod

# 连接服务
/usr/local/mongodb/bin/mongo # 本地连接，默认没有密码

vim etc/mongodb.conf
bind_ip=192.168.4.51    # 在原先的基础上面加上这两个，指定ip
port=27077              # 指定端口号

# 连接服务
mongo --host 192.168.4.51 --port 27077
> show dbs # 显示所有数据库
> db # 显示当前数据库
> use 库名 # 创建或切换
> show collections 或 show tables # 查看库下已有集合
> db.dropDatabase()    # 删除当前所在的库

# 集合管理（表管理）
> db.集合名.drop() # 删除集合
> db.集合名.save({","}) # 创建集合，集合不存在时，创建添加文档（行）

# 文档管理 
db.集合名.find()
db.集合名.count()
db.集合名.insert({“name”:”jim”})
db.集合名.find(条件)
db.集合名.findOne() # 返回查询一条文档
db.集合名.remove({}) # 删除所有文档
db.集合名.remove({条件}) # 删除与条件匹配的所有文档(行)
```



## 数据类型

### 字符串string

UTF-8字符串都可以表示为字符串类型的数据

{name:"张三"} 或 {name:"zz"}

### 布尔bool

ture和false {x:true}

### 空null

空值 {x:null}

### 数字

shell 默认使用64位浮点型数值。 {x:3.14} 或 {x:3}

NumberInt（4字节整数）{x:NumberInt(3)}

NumberLong（8字节整数）{x:NumberLong(3)}

### 数组array

{x:["a","b","c"]}

### 代码

查询和文档中可以包括任何javaScript代码

{x:function(){/* code */}}

### 日期

日期被存储为自新纪元以来经过的毫秒数，不含时区

{x:new Date()}

### 对象

对象id是一个12字节的字符串，是文档的唯一标识，自动会有

{x:ObjectID()}

### 内嵌

文档可以嵌套其他文档，被嵌套的文档作为值来处理

{tarena:{address:"Bei",tel:"88888888",person:"hansy"}}

### 正则表达式

查询时，使用正则表达式作为限定条件

{x:/正则表达式/}

### 数据导出

```shell
# 语法格式1
mongoexport  --host  192.168.4.51 --port 27077  -d  ddsdb  -c t1 -f name --type=csv    -o /root/lig1.csv
# 导出csv格式，必须要指定导出的字段名 ，导出name字段
mongoexport  --host  192.168.4.51 --port 27077  -d  ddsdb  -c t1 -q '{name:"bob"}' -f name,age --type=csv    -o /root/lig2.csv
# 从ddsdb的它1里导出名字为bob的name字段和age字段，-q 条件

# 语法格式2
mongoexport  --host  192.168.4.51 --port 27077 -d ddsdb -c t1 --type=json    -o /root/lig3.json
# 导出json格式
mongoexport  --host  192.168.4.51 --port 27077 -d ddsdb -c t1 -f name --type=json    -o /root/lig4.json
# 指定列名导出，导出name字段
```

### 数据导入

用json的格式导入：==表里要没有数据==，不然导入不成功

```shell
mongoimport --host  192.168.4.51 --port 27077 -d ddsdb -c t1 --type=json       /root/lig3.json
```

用csv的格式导入：表里可以有数据

```shell
mongoimport --host  192.168.4.51 --port 27077  -d ddsdb -c t1   --headerline  --type=csv /root/lig1.csv
# 必须指定文件的列名，不然不成功 -f和--headerline不能一起用  --headerline：把第一行的字段隐藏即去掉文件列的标题name，不然标题也会导进去，导入时t1表可以不存在
```

## 数据备份与恢复

```shell
mongodump --host  192.168.4.51 --port 27077
# 不指定备份哪个库，默认备份所有，不指定目录，自动生成dump目录，备份的数据在这个里面

# 查看文件内容
bsondump dump/ddsdb/t1.bson       # 查看bson文件内容
 
# 备份时指定备份的库和备份目录
mongodump --host  192.168.4.51 --port 27077  -d  ddsdb -o /root/bbsdb
# -d备哪个库，-o指定备份的目录，备份bbsdb库里的所有到/root/bbsdb

# 只备份ddsdb库里的集合t1
mongodump --host  192.168.4.51 --port 27077  -d  ddsdb -c t1 -o /root/bbsdb.t 

# 恢复数据
mongorestore --host  192.168.4.51 --port 27077  -d  ddsdb  /root/bbsdb.t/ddsdb/
# -d  ddsdb恢复到数据库的目录，从/root/bbsdb.t1/ddsdb/目录恢复
```



## 副本集

Replica Sets 工作原理

至少需要两个节点。一个是主节点，负责处理客户端请求，其余是从节点，负责复制主节点数据

主节点记录所有操作oplog，从节点定期轮询主节点获取这些操作，然后对自己的数据副本执行这些操作。

自动修复成员节点，故障自动切换。

发现主节点宕机后，从节点对比出数据最接近主节点。数据一样时，先发现宕机的从节点变为主节点。整体结构会暂停30秒。

```shell
# 修改配置文件，所有的副本集成员使用相同的副本集名称
vim etc/mongodb.conf
replSet=rs1  # 加入到副本集，rs1名字随便起。

# 配置Replica Sets集群信息，任意一台都可以
> rs1_config = {      # rs1_config随便起变量名,要记住
 _id:"rs1",  # 必须为rs1这个，三台主机集群名，配置文件里面写的是这个
 members:[
 {_id:0,host:"192.168.4.51:27077"},      # _id值随意，host值固定，可以利用 priority:10 ，设置权重
 {_id:1,host:"192.168.4.52:27078"},
 {_id:2,host:"192.168.4.53:27079"}
 ]
 };        # 回车，出现下面情况为成功
{
    "_id" : "rs1",
    "members" : [
        {
            "_id" : 0,
            "host" : "192.168.4.51:27077"
        },
        {
            "_id" : 1,
            "host" : "192.168.4.52:27078"
        },
        {
            "_id" : 2,
            "host" : "192.168.4.53:27079"
        }
    ]
}
# 初始化 Replica Sets
> rs.initiate(rs1_config)
# 查看状态信息
rs.status()
# 查看是否是master库
rs1:PRIMARY>  rs .isMaster()

# 从库验证
rs1:SECONDARY> db.getMongo().setSlaveOk() # 同步数据验证，允许从库查看数据
```



## 文档管理

### 插入文档

```shell
> db.集合名.save({key:"值",key:"值"})
```

注意：

集合不存在时自动创建集合，然后再插入记录

_id 字段值已存在时，修改文档字段值

_id 字段值不存在时，插入文档

```shell
> db.集合名.insert({key:"值",key:"值"})
> db.集合名.insert({key:"值",key:"值"},{key:"值",key:"值"})
```

注意：

集合不存在时自动创建集合，然后再插入记录

_id 字段值已存在时，==放弃==插入

_id 字段值不存在时，插入文档

### 查询文档

```shell
> db.集合名.find() # 显示所有行
> db.集合名.findOne() # 显示第一行
> db.集合名.find({条件},{定义显示的字段}) # 0 不显示，1 显示
> db.user.find({},{_id:0,name:1,shell:1})
> db.集合名.find().limit(数字) # 显示前几行
> db.集合名.find().skip(数字) # 跳过前几行
> db.集合名.find().sort(字段名) # 1 升序，-1 降序
> db.user.find({shell:"/sbin/nologin"},{_id:0,name:1,uid:1,shell:1}).skip(2).limit(2)

# 条件
# 范围比较
> db.user.find({uid:{$in:[1,6,9]}}) # 在。。。里
> db.user.find({uid:{$nin:[1,6,9]}}) # 不在。。。里
> db.user.find({$or:[{name:"root"},{uid:1}]}) # or

# 正则
> db.user.find({name:/^a/})

# 数值比较
$lt $lte $gt $gte $ne
 <  <=    >   >=   !=
> db.user.find({uid:{$gte:10,$lte:40}},{_id:0,name:1,uid:1})

# 匹配null：可以匹配没有的字段，也可以检查这个字段有没有
> db.user.save({name:null,uid:null})

# 更新文档
> db.user.update({条件},{修改的字段})
> db.user.update({name:"root"},{password:"XXX"})  
# 如果这一列不写完整，这一行除了password这一行，这一列的其他值都没有了相当于删除（要写完整）

# 多文档更新，默认只更新与条件匹配的第1行
> db.user.update({条件},{$set:{修改的字段}},flase,true)
$set # 修改指定字段的值
$unset # 删除与条件匹配文档的字段
$inc # 加
rs1:PRIMARY> db.user.update({uid:{$lte:10}},{$inc:{uid:-1}})    
# 负数时是自减1，默认改第一行
rs1:PRIMARY> db.user.update({uid:{$lte:10}},{$inc:{uid:2}},false,true)    
# 设置字段值自加2，false,true改全部

$push # 向数组中添加新元素
db.user.update({name:"bob"},{$push:{like:"Z"}})       # 默认添加到最后
$addToSet # 避免重复添加
db.user.update({name:"bob"},{$addToSet:{like:"W"}})
$pop # 删除数组末尾一个元素，1删除最后一个，-1删除第一个
rs1:PRIMARY> db.user.update({name:"bob"},{$pop:{like:1}}) # 删除匹配的第一条的最后一个
rs1:PRIMARY> db.user.update({name:"bob"},{$pop:{like:-1}}) # 删除匹配的第一条的第一个
$pull # 删除数组里的指定元素，若有两个bob可以用_id值定义把name:"bob"换成id值
db.user.update({name:"bob"},{$pull:{like:"c"}})

# 删除文档
remove()与drop()的区别
remove()删除文档时不删除索引
drop()删除集合的时候同时删除索引
rs1:PRIMARY> db.t1.remove({})
rs1:PRIMARY> db.user.remove({name:"/^a/"})       # 删除以a开头的记录
rs1:PRIMARY> db.t1.drop()                        # 删除集合t1
```
