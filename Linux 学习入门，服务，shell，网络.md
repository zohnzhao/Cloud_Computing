[TOC]

# 1 acl访问控制列表（acl策略）

只有Linux有，能够对个别用户，个别组设置独立的权限

| setfacl | -m 修改权限<br>-b 取消acl策略<br>-x 删除指定权限<br>-r 对目录下所有文件与目录都更改 |
| ------- | ------------------------------------------------------------ |
| getfacl | 查看acl权限                                                  |

```shell
setfacl -m u:用户名:权限 文档  # 修改用户的acl策略
setfacl -m g:组名:权限 文档  # 修改组的acl策略	
```



# 2 LDAP网路用户认证

作用：网络用户认证，用户集中管理

集群服务器中，一台LDAP服务器配置用户，其他服务器询问认证

| 客户机连接LDAP服务 | sssd           |
| ------------------ | -------------- |
| 客户机图形配置工具 | authconfig-gtk |
| 默认端口           | 389/TCP        |
| 加密端口           | 636/TCP        |



# 3 NFS 网络文件系统

| 协议         | NFS（2049/TCP） RPC（111/TCP） |
| ------------ | ------------------------------ |
| 文件系统格式 | nfs                            |
| 软件         | nfs-utils                      |
| 服务         | nfs-server                     |
| 配置文件     | /etc/exports                   |

```shell
showmount -e IP	# 查看网络分享
mount ip:目录 挂载点 # 将网络文件系统临时挂载到本地

# nfs配置文件格式
"共享文件夹绝对路径" 客户机IP（ro/rw,no_root_squash） # 客户机IP 可以写成*
# 来自NFS客户端的root用户会被自动降权为普通用户，若要保留其root权限，注意应添加no_root_squash控制参数

# nfs永久挂载
IP：目录 挂载点 nfs defaults,_netdev 0 0
```



# 4 NTP 网络时间同步

| 软件包 | chrony  |
| ------ | ------- |
| 服务名 | chronyd |
| 端口   | 123/UDP |



## 服务器

服务器层级最大为15

```shell
yum -y install chrony

# 修改配置文件
cat /etc/chrony.conf
.. ..
server 0.centos.pool.ntp.org iburst        # server用户客户端指向上层NTP服务器
allow 192.168.4.0/24                        # 允许那个IP或网络访问NTP
#deny  192.168.4.1                        # 拒绝那个IP或网络访问NTP
local stratum 10                            # 设置NTP服务器的层数量
.. ..

# 启动NTP服务
systemctl  restart  chronyd
systemctl  enable  chronyd
```



## 客户端

| 软件包名 | chrony           |
| -------- | ---------------- |
| 服务名   | chronyd          |
| 配置文件 | /etc/chrony.conf |
| 端口     | 123/UDP          |

```shell
# 配置文件写法
server IP iburst # iburst 快速同步

systemctl restart chronyd # 重启chronyd服务
systemctl enable chronyd # 设置chronyd服务开机启动
```



# 5 SELinux 强制访问控制系统

针对用户，进程和文件提供了预设的保护策略，以及管理工具，集成于Linux内核（2.6版本及以上）

| 运行模式 | enforing 强制<br>permissive 宽松<br>disabled 关闭 |
| -------- | ------------------------------------------------- |
| 配置文件 | /etc/selinux/config                               |

```shell
setenforce 1/0 # 临时切换至 强制/宽松 模式。宽松模式记录违规，恢复强制模式后处理。
# 任何模式切换至disabled模式，需要修改配置文件后重启系统，反之亦然。
```

```shell
getsebool [-a] [布尔值条款] # 列出目前系统上面的所有布尔值条款设置

setsebool "条款名" on/off # 设置开关 -P 设置为永久
```



# 6 FTP服务

| 服务端软件包名 | vsftpd                  |
| -------------- | ----------------------- |
| 客户端软件包名 | ftp                     |
| 服务名         | vsftpd                  |
| 配置文件       | /etc/vsftpd/vsftpd.conf |
| 默认共享目录   | /var/ftp                |
| 端口           | 21/tcp                  |

```shell
# 添加防火墙策略，让火墙允许ftp服务； --permanent表示永久添加
firewall-cmd --permanent --add-service="ftp"
```



# 7 Firewall防火墙策略管理

作用：隔离、过滤所有<font color=red>入站</font>的请求

防火墙判断规则：

1. 匹配源IP与规则内IP，匹配成功就进入此区域
2. 匹配失败进入默认区域

| 服务名，默认安装并自启 | firewalld                                                    |
| ---------------------- | ------------------------------------------------------------ |
| 命令行管理工具         | firewall-cmd                                                 |
| 图形化管理工具         | firewall-config                                              |
| 运行规则               | runtime（运行时）<br>permanent（永久）                       |
| 预设保护规则集（区域） | public（默认仅允许访问本地的sshd、ping与dhcp三个服务）<br>trusted（允许任何访问）<br>block（明确拒绝所有访问）<br>drop（直接丢弃，节省资源） |

```shell
firewall-cmd --get-default-zone # 查看当前激活区域
firewall-cmd --set-default-zone="区域名" # 设置激活区域，此操作为永久
firewall-cmd --zone="区域名" --list-all # 查看指定区域的详细信息
firewall-cmd --reload # 重载防火墙规则
firewall-cmd --zone="区域名" --add-service="协议名" # 临时增加协议，添加 --permanent 设置为永久，设置后需要重载防火墙规则
firewall-cmd --zone="区域名" --add-source="IP" # 设置IP进入哪个区域

#端口转发
firewall-cmd --zone="区域名" --add-forward-port=port="对外端口"：proto=tcp:toport="内部端口"
```



# 8 配置高级连接

## 配置IPV6

|            | IPV4地址   | IPV6地址    |
| ---------- | ---------- | ----------- |
| 组成       | 32位二进制 | 128位二进制 |
| 分隔符     | .          | :           |
| 表示方式   | 十进制     | 十六进制    |
| 分几个部分 | 4          | 8           |

一张网卡可同时配置IPV4与IPV6，配置方式一样

ping6 可以测试IPV6网络



## 聚合连接（链路聚合）

由多块网卡组成，实现网卡备份，网卡设备的高可用性。

由虚拟网卡管理真实网卡，5秒询问一次。

两种工作方式：

  - ​	轮询式（roundrobin）：流量负载均衡

  - ​	热备份（activebackup）：连接冗余

```shell
#聚合连接配置
#创建虚拟网卡
nmcli connection add com-name team0 type team ifname team0 config '{"runner":{"name":"activebackup"}}' # com-name 后跟配置文件名称 ifname 后跟查询时显示的网卡名
  
#配置IP
nmcli connection modify team0 ipv*.method manual ipv*.addresses "IP/NETMARK" connection.autoconnect yes
  
#添加成员
nmcli connection add com-name team0-p1 type team-slave ifname eth1 master team0 # ifname后跟成员

#删除网卡
nmcli connection delete "网卡名"

#查询虚拟网卡状态
teamdctl "网卡名" state

#nmcli 未识别的网卡可临时设置IP，设置后可显示
ifconfig eth1 "192.168.1.1" netmask "255.255.255.0" up
```

  

# 9 SMB共享

共享文件夹，Linux与Windows平台之间也可用。

| 协议   | SMB             | cifs                 |
| ------ | --------------- | -------------------- |
| 作用   | 验证协议        | 文件系统格式         |
| 软件包 | samba（服务端） | cifs_utils（客户机） |
| 服务   | smb（服务端）   |                      |

```shell
#配置文件
/etc/samba/smb.conf # ; 代表注释
#内容（尾部追加），以下命令，用哪个写哪个
[自定义共享名]
	path = "文件绝对路径"
	public = no/yes # 默认no
	browseable = yes/no # 默认yes
	readonly = yes/no # 默认yes
	write list = "用户名" # 默认无
	valid users =
	hosts allow =
	hosts deny =
```

| 命令    | 选项                                                         |
| ------- | ------------------------------------------------------------ |
| pdbedit | -a 用户名 # 添加smb用户<br/>-L # 查看smb用户 <br/>-x 用户名 # 删除smb |

客户机查看

```shell
# 安装 samba-client
smbclient -L IP/共享名 # 查看共享的目录
smbclient -U 用户名 # IP/共享名 #连接共享
```

客户机临时、永久挂载

```shell
# 安装 cifs_utils 包后，挂载
#临时挂载
mount -o username= ,password= # IP/共享名 "挂载点" # 格式一
mount -o user= ,pass= # IP/共享名 "挂载点" # 格式二

#永久挂载，追加至/etc/fstab
# IP/共享名 "挂载点" cifs defaults，user=,pass=,_netdev 0 0 # _netdev 声明网络设备，系统完全启动后挂载

```

multiuser 多用户模式，可以临时变成权限较大的用户
	配置文件参数中添加 multiuser,sec=ntlmssp # ntlmssp 提供NT局域网管理安全协议

```shell
cifscreds add -u 用户名 IP # 切换smb登陆用户
```

安装步骤：

- ​	1.安装samba
- ​	2.创建专用用户
- ​	3.创建共享目录
- ​	4.修改配置文件
- ​	5.重启服务
- ​	6.修改SElinux，samba的bool
- ​	7.设置acl  
- ​	8.客户机安装cifs_utils，挂载

```shell
getsebool -a | grep samba # 查看SELinux 管理的所有接口，筛选samba相关
setsebool "接口" on # 开启接口
```



# 10 ISCSI

## 存储技术分类

SCSI 小型计算机系统接口 Smal Computer System Interface

DAS 直连式存储 Direct-Attached Storage

​	不能实现数据与其他主机的共享

​	占用服务器操作系统资源，如CPU、IO等

​	数据梁越大，性能越差

NAS 网络技术存储 Network-Attached Storage

​	一种专用数据存储服务器，与服务器分离，集中管理数据，用户通过TCP/IP协议访问,采用标准的NFS/HTTP/CIFS等

SAN 存储区域网络 Storage Area Network

FC 光纤通道

## ISCSI

共享一个<font color=red>未格式化</font>的分区或硬盘。服务端提供磁盘，客户端连接并本地使用。

| 软件   | targetcli                                                    |
| ------ | ------------------------------------------------------------ |
| 服务名 | target                                                       |
| 端口   | 3260/TCP                                                     |
| 组成   | backstore # 后端存储，对应到服务端的实际设备，需要一个管理名称<br>lun # 逻辑单元，每一个lun对应一个后端设备，在客户端视为一块虚拟硬盘<br>target # 磁盘组，客户端访问目标，作为一个框架，多个lun组成 |



## 服务端操作

```shell
targetcli # 启动配置软件
/backstores/block create name="后端存储名字" # 创建后端存储
/iscsi/ create iqn.2019-04.com.example:server0 # 创建磁盘组
# 磁盘组 IQN 命名规范： iqn.yyyy-mm.倒序域名：自定义标识
/iscsi/磁盘组名/tpg1/luns create /backstores/block/"后端存储名" # lun与后端存储关联
/iscsi/磁盘组名/tpg1/acls create iqn.2019-04.com.example:desktop0 # 创建acls用户，用于远程连接，用户名必须iqn规范
/iscsi/磁盘组名/tpg1/portals create IP # 启动监听IP与默认端口
exit # 退出时自动创建配置文件

systemctl restart target # 启动target服务
systemctl enabled target # 开机自动启动target服务
```



## 客户端操作

| 软件     | iscsi-initiator-utils          |
| -------- | ------------------------------ |
| 配置文件 | /etc/iscsi/initiatorname.iscsi |
| 服务     | iscsi、iscsid                  |

```shell
#修改配置文件
InitiatorName=iqn.2019-04.com.example:desktop0 # 填写acls用户名，不能有任何空格

systemctl restart iscsid # 重启iscsid服务，无法开机自启。刷新acls配置名称

#发现iscsi磁盘方法一
iscsiadm -m discovery -t st -p "IP"
#发现iscsi磁盘方法二，利用 man iscsiadm 查找范例
iscsiadm --mode discoverydb --type sendtargets --portal "IP" --discover

systemctl restart iscsi # 重启iscsi服务，连接iscsi磁盘，并可以使用。

#开机启动iscsi磁盘
systemctl enabled iscsi
#修改配置文件 /var/lib/iscsi/nodes/磁盘组名/IP:端口/default
node.conn[0].startup=automatic # 设置开机启动
```



## 部署Multipath多路径环境

```shell
 yum install -y device-mapper-multipath
 # 生成配置文件
 cd /usr/share/doc/device-mapper-multipath-0.4.9/
 ls multipath.conf
 cp multipath.conf  /etc/multipath.conf
 
# 获取wwid
# 登陆共享存储后，系统多了两块硬盘，这两块硬盘实际上是同一个存储设备。应用服务器使用哪个都可以，但是如果使用sdb时，sdb对应的链路出现故障，它不会自动切换到sda。
# 为了能够实现系统自动选择使用哪条链路，需要将这两块磁盘绑定为一个名称。
# 通过磁盘的wwid来判定哪些磁盘是相同的。
# 取得一块磁盘wwid的方法如下
/usr/lib/udev/scsi_id --whitelisted --device=/dev/sdb

# 修改配置文件
vim /etc/multipath.conf
# 声明自动发现多路径
defaults {
    user_friendly_names yes
    find_multipaths yes
}

# 在文件的最后加入多路径声明，如果哪个存储设备的wwid和第（3）步获取的wwid一样，那么，为其取一个别名，叫mpatha
multipaths {
	multipath {
		wwid    "360014059e8ba68638854e9093f3ba3a0"
		alias   mpatha
	}
}

# 启用Multipath多路径
systemctl start multipathd
systemctl enable multipathd

# 检查多路径设备文件
# 如果多路径设置成功，那么将在/dev/mapper下面生成名为mpatha的设备文件
ls /dev/mapper/

# 挂载使用 mpatha 盘
```



# 11 Mariadb 基础

常见数据库：oracle、mysql、mariadb、sql-server

| 软件包 | mariadb-server |
| ------ | -------------- |
| 服务   | mariadb        |
| 端口   | 3306/TCP       |

```mysql
mysqladmin -u root password '新密码' # 第一次设置密码
mysqladmin -u root password '旧密码' '新密码'  # 修改密码

# 以下命令进入mysql执行，不能自动补全。
SHOW DATABASES； # 显示所有数据库
CREATE DATABASE "数据库名"; # 创建数据库
DROP DATABASE "数据库名"; # 删除数据库
DESC "表名"; # 查询表结构

# 数据库授权
GRANT "权限列表" on "数据库名"."表名" to "用户名"@"客户地址" identified by "密码";
# 权限列表：all、select、insert、updata、delete
flush privileges; # 刷新配置
```



# 12 httpd 服务

| web框架  | web 基于B/S 架构 Browser/Serve                       |
| -------- | ---------------------------------------------------- |
| 常见软件 | hpptd（Apache）、nginx、tomcat、Tengine（nginx优化） |
| 端口     | 80/TCP                                               |

```shell
# httpd 默认主页 /var/www/html/index.html
# httpd 主配置文件 /etc/httpd/conf/httpd.conf
Listen # 监听端口
ServerName # 本站服务的域名
DocumentRoot # 网页根目录
DirectoryIndex # 默认主页文件

hpptd -t # 自检配置文件
```



## 虚拟主机

一台服务器可以提供多个域名web页面。可基于三种方式区分：IP、域名、端口。

```shell
# 副配置文件：设置权限和虚拟主机 /etc/httpd/conf.d/*.conf
# 虚拟主机格式
Listen "端口"
<VirtualHost "IP":"端口"> # "IP" 可以写 *
	ServerName 域名
	DocunmentRoot 网站根目录
</irtualHost>

# 访问控制
<Directory "目录">
	Require all denied/granted # 全部拒绝或允许
	Require IP
</Directory>
```

若网站根目录不在 /var/www 下，需要处理<font color=red>SELinux</font>和<font color=red>访问控制</font>权限

```shell
# 处理SELinux
semanage fcontext -l # 查看所有上下文
ls -Zd 目录 # 查看目录上下文
chcon -R --reference=模板目录 新目录 # 循环复制模板目录的上下文到新目录
```



## 动态网站（wsgi）

需要软件：mod_wsgi

```shell
<VirtualHost "IP":"端口"> # "IP" 可以写 *
	ServerName 域名
	DocunmentRoot 网站根目录
	WSGIScriptAlias / /var/www/html/wsgi/index.wsgi #打开根目录，自动调转
</irtualHost>
```



## 更改任意端口

```shell
semanage port -l # 查看所有SELinux默认端口
semange port -a -t http_port_t -p tcp 8909 # 开放httpd监听8909端口
```



## 安全的WEB

| 软件 | mod_ssl                       |
| ---- | ----------------------------- |
| 协议 | https（安全的超文本传输协议） |

```shell
/etc/pki/tls/certs/*.crt # 证书存放位置
/etc/pki/tls/private/*.key # 私钥存放位置
/etc/httpd/conf.d/ssl.conf # 配置ssl
Listen 443 https
.. ..
<VirtualHost _default_:443>
DocumentRoot "/var/www/html"                                     # 网页目录
ServerName server0.example.com:443                              # 站点的域名
.. ..
SSLCertificateFile /etc/pki/tls/certs/server0.crt                  # 网站证书
.. ..
SSLCertificateKeyFile /etc/pki/tls/private/server0.key             # 网站私钥
.. .. 
SSLCACertificateFile /etc/pki/tls/certs/example-ca.crt             # 根证书
```



# 13 postfix基础邮件服务

| 软件包 | postfix |
| ------ | ------- |
| 服务   | postfix |
| 端口   | 25/TCP  |

```shell
# vim  /etc/postfix/main.cf
.. ..
inet_interfaces = all                         #监听接口
mydomain = example.com                          #邮件域,默认补全的域名
myhostname = example.com                          #本服务器主机名

mail -s "标题" -r "发件人" "收件人" # 回车后写正文，最后一行以单个 . 结尾
mail -u "用户名" # 查看指定用户邮件
```



# 14 swap分区、parted 分区工具

以分区充当swap分区

parted 分区工具，支持GPT分区模式，最多128个主分区，18EB空间。

自动改变硬盘文件系统，分区空间有误差

```shell
mklable gpt # 指定分区模式
mkpart # 文件系统只记录
unit GB # 设置primt单位
parted  /dev/vdb  mklabel  gpt
parted  /dev/vdb  mkpart primary  1M  50%
```

```shell
# 制作swap分区
mkswap "目录" # 格式化生成文件系统
swapon "目录" # 启用swap分区
swapon  -s # 查看swap分区，权限为优先级，数值越大，优先级越高
swapoff "目录" # 停用swap分区

#开机启动
/dev/* swap swap defaults 0 0
swapon -a # 检测开机启动
```

# 15 DNS

正向解析：域名变IP	反向解析：IP变域名

结构：根域、一级DNS服务器、二级DNS服务器、三级DNS服务器

所有域名必须以点结尾 点称为根域

FQDN 完全合格主机名，格式 站点名.域名.

IANA 互联网数字分配机构

域名解析步骤：

1. ​	查看/etc/hosts
2. ​	查看/etc/resolve.conf（存放DNS服务器IP）
3. ​	给DNS服务器发送请求
4. ​	DNS服务器递归查询
5. ​	DNS服务器与其它服务器迭代查询
6. ​	返回结果

| 软件包             | bind（域名服务）、bind-chroot（虚拟根支持，安全） |
| ------------------ | ------------------------------------------------- |
| 服务名             | named                                             |
| 端口               | 53/UDP                                            |
| 主配置文件         | /etc/named.conf（设置负责解析的域名）             |
| 运行时的虚拟根环境 | /var/named/chroot/                                |
| 地址库文件夹       | /var/named                                        |

```shell
vim  /etc/named.conf                             # 建立新配置
options {
	recursion yes/no;								# 开启/关闭递归查询
    directory  "/var/named";                          # 地址库默认存放位置
    forwarders { IP;IP; }；					#启动DNS缓存，IP为DNS服务器IP
};
zone  "zz.cn" {                                  # 定义正向DNS区域
    type  master;                                     # 主区域
    file  "zz.cn.zone";                             # 自定义地址库文件名
};

cp  -p  named.localhost  zz.cn.zone      #参考范本建地址库文件，需要特别注意 -p

vim  tedu.cn.zone                          # 修订地址库记录
$TTL 1D                                          # 文件开头部分可保持不改
@   IN SOA  @ rname.invalid. (
                    0   ; serial
                    1D  ; refresh
                    1H  ; retry
                    1W  ; expire
                    3H )    ; minimum
@       NS  svr7.zz.cn.                       # 本区域DNS服务器的FQDN
svr7    A   192.168.4.7                       # 为DNS主机提供A记录
pc207   A   192.168.4.207                     # 其他正向地址记录.. ..
www     A   192.168.4.100                     # A 代表IPV4  AAAA 代表IPV6
www     A   192.168.4.110                     # 配置DNS轮询
www     A   192.168.4.120
*       A   119.75.217.56                     # 配置多对一的泛域名解析
```



## 配置DNS子域授权



```shell
# 子域域名.            IN    NS      子DNS的FQDN.
# 子DNS的FQDN.       IN    A        子DNS的IP地址

bj.zz.cn.         NS       pc207.bj.zz.cn.             #子区域及子DNS主机名
pc207.bj.zz.cn.   A       192.168.5.207                  #子DNS的IP地址
```



## 配置Split分离解析

```shell
# 每个区域的个数必须一致，域名必须一致
vim  /etc/named.conf
options {
        directory  "/var/named";
};
acl "mylan" {                                      # 名为mylan的列表
        192.168.4.207; 192.168.7.0/24;
};

view "mylan" {
    match-clients { mylan; };                      # 检查客户机地址是否匹配此列表
    zone "zz.cn" IN {
        type master;
        file "zz.cn.zone.lan";
    };
};

view "other" {
    match-clients { any; };                          # 匹配任意客户机地址
    zone "zz.cn" IN {
        type master;
        file "zz.cn.zone.other";
    };
};
```



# 16 RADIO

| 模式                | 磁盘数    | 优、缺点                         |
| ------------------- | --------- | -------------------------------- |
| 0 条带模式          | 大于等于2 | 没有容错、I/O性能高              |
| 1 镜像模式          | 大于等于2 | 有备份                           |
| 5 高性价比模式      | 大于等于3 | 利用1块磁盘存储校验，可坏1块磁盘 |
| 6 高性价比/可靠模式 | 大于等于4 | 利用2块磁盘存储校验，可坏2块磁盘 |
| 1+0 或 0+1          | 大于等于4 |                                  |



# 17 进程管理

| 命令   | 选项                                                         |
| ------ | ------------------------------------------------------------ |
| pstree | -a # 显示完整的命令行信息<br>-p # 列出对应的PID号，systemd（所有进程的父进程，上帝进程） |
| ps     | aux # a 所有进程（当前终端） u 以用户格式输出 x 当前用户在所有终端下的进程<br>-elf # e 所有进程（系统内）l 长格式显示 f 完整的进程信息（PPID为父进程号） |
| top    | -d 动态进程刷新秒数<br>-U 用户名<br>交互命令<br>? # 查看帮助<br>P、M # 根据%CPU、%MEM 降序排列<br/>T # 根据消耗时间降序排列<br/>K # 杀死指定的进程<br/>q # 退出top |
| pgrep  | -l 进程名 # 检索指定名称进程<br/>-U 用户名 # 检索指定用户进程<br/>-t 终端号 # 检索终端进程<br/>-x # 精确匹配完整的进程名 |

图形命令行终端pts/0 # 0代表第一个，依次排序

## 后台进程

| 命令 &   | 程序运行状态放入后台                           |
| -------- | ---------------------------------------------- |
| Ctrl+z   | 暂停并转入后台                                 |
| jobs     | 列出当前用户当前终端的后台任务<br>-l 显示PID号 |
| fg 编号  | 启动指定编号的后台任务，并恢复至前端           |
| bg  编号 | 启动指定编号的后台任务                         |



## 杀死进程

| kill  [-9]  PID       | 杀死指定PID值的进程          |
| --------------------- | ---------------------------- |
| killall  [-9]  进程名 | 杀死指定名称的所有进程       |
| pkill                 | 根据指定的名称或条件杀死进程 |

# 18 系统日志

| 目录              | 作用                             |
| ----------------- | -------------------------------- |
| /var/log/messages | 记录内核消息、各种服务的公共消息 |
| /var/log/dmesg    | 记录系统启动过程的各种消息       |
| /var/log/cron     | 记录与cron计划任务相关的消息     |
| /var/log/maillog  | 记录邮件收发相关的消息           |
| /var/log/secure   | 记录与访问限制相关的安全消息     |
| /var/log/lastlog  | 最近用户的登陆行为               |
| /var/log/wtmp     | 成功的登陆/注销                  |
| /var/log/btmp     | 失败的登陆                       |
| /var/log/utmp     | 当前登录的用户                   |

```shell
tailf /var/log/messages # 将输出文件的最后10行，然后等待文件增长
# -n 指定显示文件最后的行数(默认显示最后10行)
```



## 日志消息级别

| EMERG（紧急）   | 级别0，系统不可用的情况         |
| --------------- | ------------------------------- |
| ALERT（警报）   | 级别1，必须马上采取措施的情况   |
| CRIT（严重）    | 级别2，严重情形                 |
| ERR（错误）     | 级别3，出现错误                 |
| WARNING（警告） | 级别4，值得警告的情形           |
| NOTICE（注意）  | 级别5，普通但值得引起注意的事件 |
| INFO（信息）    | 级别6，一般信息                 |
| DEBUG（调试）   | 级别7，程序/服务调试消息        |



# 19 用户登陆查询

查询结果简单到详细：users、who、w

last 查看最近登陆成功的信息

lastb  查看最近登陆失败的信息



# 20 PXE批量装机

| 所需服务与工具                      | 作用                    |
| ----------------------------------- | ----------------------- |
| DHCP                                | 分配IP、定位引导文件    |
| TFTP                                | 提供引导下载            |
| HTTP                                | 提供yum源、自动应答文件 |
| system-config-kickstart（图形工具） | 生成自动应答文件        |



## DHCP 服务器

| 软件包   | dhcp                 |
| -------- | -------------------- |
| 服务名   | dhcpd                |
| 配置文件 | /etc/dhcp/dhcpd.conf |

整个服务过程以广播进行，先到先得，一个网络中，只能有一个DHCP服务器。

```shell
# vim 末行模式 ：r 文件路径  # 读取文件到当前光标之下
# vim  /etc/dhcp/dhcpd.conf
subnet 192.168.4.0 netmask 255.255.255.0 { # 声明网段
     range  192.168.4.10 192.168.4.200; # IP范围
     option domain-name-servers DNSIP；
     option routers 网关IP；
     next-server  192.168.4.7;  # 引导服务IP
     filename  "pxelinux.0"; # 引导文件、二进制文件。由syslinux软件提供（redhat7独有）
}

# pxelinux.0 位置 /usr/share/syslinux/pxelinux.0
# pxelinux.0 内容 /var/lib/tftpboot/pxelinux.cfg/default
```



## TFTP 服务

| 软件包       | tftp-server       |
| ------------ | ----------------- |
| 服务名       | tftp              |
| 端口         | 69/TCP            |
| 默认共享路径 | /var/lib/tftpboot |

需从光盘内复制

- ​	isolinux.cfg（菜单配置，/var/lib/tftpboot/pxelinux.cfg/default）
- ​	vesamenu.c32（提供图形支持）
- ​	splash.png （背景图片）
- ​	vmlinuz（内核）
- ​	initrd.img（初始化文件）

```shell
vim  /var/lib/tftpboot/pxelinux.cfg/default
default vesamenu.c32                             # 默认交给图形模块处理
timeout 600                                      # 选择限时为60秒（单位1/10秒）
.. ..
menu title  PXE  Installation  Server             # 启动菜单标题信息
.. ..
label  linux                                  # 菜单项标签
    menu  label  ^Install Red Hat Enterprise Linux 7 
    menu  default                              # 默认启动方式
    kernel  vmlinuz                   		   # 内核的位置
    append  initrd=rhel7/initrd.img  ks=http：//IP/应答文件
```

​	

# 21 rsync 同步

- rsync  [选项...]  本地目录1  本地目录2  （把目录1同步到目录2下）
- rsync  [选项...]  本地目录1/  本地目录2 （把目录1下的内容同步到目录2下）
- 可以利用ssh远程同步

| 命令  | 选项                                                         |
| ----- | ------------------------------------------------------------ |
| rsync | -n # 测试同步过程，不做实际修改<br>-a # 归档模式<br/>-v # 显示详细操作信息<br/>-z # 传输过程中启用压缩/解压(超过1G使用)<br/>--delete # 删除目标文件夹内多余的文档 |



# 22 inotifywait 事件监控

| 软件包 | inotify-tools                                                |
| ------ | ------------------------------------------------------------ |
| 程序名 | inotifywait                                                  |
| 格式   | inotifywait  [选项]  目标文件夹                              |
| 选项   | -m # 持续监控（捕获一个事件后不退出）<br>-r # 递归监控、包括子目录及文件<br/>-q # 减少屏幕输出信息<br/>-e # 指定监视的 modify、move、create、delete、attrib 等事件类别  <br/>-qq # 不输出屏幕信息 |



## 实时同步

```shell
ssh-kengen # 生成公钥与私钥
ssh-copy-id root@IP # 将公钥给对方，私钥登陆公钥

while /目录/inotifywait -qqr /目录/
do
	rsync -av 目录 root@IP：目录
done
```



# 23 网络

## 网络拓扑结构

| 点对点 | 广域网                                                       |
| ------ | ------------------------------------------------------------ |
| 星型   | 易实现，易网络扩展，易故障排查。中心节点压力大，组网成本较高 |
| 网状   | 冗余性、容错性和可靠性高。组网成本高                         |



## OSI 七层模型（理论模型）

| 层级       | 作用                                             | 代表                                                         |
| ---------- | ------------------------------------------------ | ------------------------------------------------------------ |
| 应用层     | 网络服务与最终用户的一个接口                     | Telnet、FTP、HTTP、SNMP等                                    |
| 表示层     | 数据的表示、安全、压缩                           | 格式有，JPEG、ASCll、DECOIC、加密格式等                      |
| 会话层     | 建立、管理、终止会话                             | 对应主机进程，指本地主机与远程主机正在进行的会话             |
| 传输层     | 定义传输数据的协议端口号，以及流控和差错校验     | 防火墙、端口。协议有：TCP UDP，数据包一旦离开网卡即进入网络传输层 |
| 网络层     | 进行逻辑地址寻址，实现不同网络之间的路径选择     | 路由器。协议有：CMP IGMP IP（IPV4 IPV6） ARP RARP            |
| 数据链路层 | 建立逻辑连接、进行硬件地址寻址、差错校验等功能。 | 交换机                                                       |
| 物理层     | 建立、维护、断开物理连接。                       | 物理设备                                                     |



## TCP/IP 五层模型（实际应用）

| 应用层 | 数据 | 计算机 |
| ------ | ---------------------------- | ------------------------- |
| 传输层     | 数据段  | 防火墙 |
| 网络层   | 数据包 | 路由器            |
| 数据链路层 | 数据帧 | 交换机                                                       |
| 物理层     | 比特流                   | 物理设备                                                     |

## 568B 线序

白橙 橙 自绿 蓝 白蓝 绿 白棕 棕

## 数据链路层

- 学习
- 广播
- 转发
- 更新

VLAN（Virtual Local Area Network）的中文名为"虚拟局域网"，在计算机网络中，一个二层网络可以被划分为多个不同的广播域，一个广播域对应了一个特定的用户组，默认情况下这些不同的广播域是相互隔离的

VLAN作用：广播控制，增加安全性，提高带宽利用率，降低延迟

VLAN最多4096个：0～4095

TRUNK：跨交换机的同VLAN通信

以太网通道：多个物理接口绑定为一个虚拟接口，增加冗余性



## 网络层

实现两个端系统之间的数据透明传送，具体功能包括寻址和路由选择、连接的建立、保持和终止等

ICMP：控制报文协议。它是TCP/IP协议簇的一个子协议，用于在IP主机、路由器之间传递控制消息

路由器拒绝广播

### OSPF：动态路由协议，开放式最短路径优先



## 传输层

找到应用，端口共有65536个：0～65535

- TCP：传输控制协议，可靠，面向连接，传输效率低
- UDP：用户数据协议，不可靠，无连接，传输效率高

TCP信号：

- SYN 想与对方建立连接
- FIN 想与对方断开连接
- ACK 确认

### ACL 访问控制列表

- 标准访问控制列表：默认不写明的都拒绝，基于源IP地址过滤数据包，列表号1～99
- 扩展访问控制列表：基于源IP、目的IP、指定协议和端口过滤数据包，列表号100～199

反掩码中：

- 0 严格匹配
- 1 不匹配

### NAT 网络地址转换

使用本地地址的主机在和外界通信时，都要在NAT路由器上将其本地地址转换成全球IP地址，才能和因特网连接。

### PAT 端口地址转换 

PAT普遍应用于接入设备中，它可以将中小型的网络隐藏在一个合法的IP地址后面。PAT与动态地址NAT不同，它将内部连接映射到外部网络中的一个单独的IP地址上，同时在该地址上加上一个由NAT设备选定的TCP端口号。

### STP 生成树协议

逻辑上断开环路，防止广播风暴。当线路故障时，阻塞的接口自动激活，恢复通信，起备用线路的作用。

### HSRP 热备份路由器协议

实现HSRP的条件是系统中有多台路由器，它们组成一个“热备份组”，这个组形成一个虚拟路由器。HSRP是cisco平台一种特有的技术，是cisco的私有协议。



# 24 SHELL

/etc/shells 解释器文件

```shell
source	"脚本文件" # 使用用户默认解释器运行脚本
bash "脚本文件" # 使用指定解释器运行脚本，会新开解释器进程
```



## 环境变量

| PWD      | 当前工作目录                                            |
| -------- | ------------------------------------------------------- |
| PATH     | 指定命令的搜索路径                                      |
| USER     | 当前登录的用户名                                        |
| UID      | 当前登录的用户名的ID                                    |
| SHELL    | 当前用户用的是哪种shell                                 |
| HOME     | 用户的主工作目录                                        |
| HOSTNAME | 主机的名称                                              |
| PS1      | 第一级Shell命令提示符，root用户是#，普通用户是$         |
| PS2      | 第二级命令提示符，默认是“>”                             |
| RANDOM   | 随机数变量。每次引用这个变量会得到一个0~32767的随机数。 |



## 内部变量

| $#   | 命令行参数或位置参数的数量                         |
| ---- | -------------------------------------------------- |
| $?   | 最近一次执行的命令或shell脚本的出口状态，0代表成功 |
| $!   | 上一个命令的PID                                    |
| $$   | shell脚本的进程ID                                  |
| $*   | 所有的位置参数，有双引号时，一个整体               |
| $@   | 所有的位置参数，有双引号时，单独个体               |
| $0   | 脚本文件名                                         |
| $n   | $1表示第一个参数,$2表示第二个参数                  |

```shell
stty -echo # 关闭终端显示
stty echo # 打开终端显示
export a=20 # 设置全句变量
unset a # 取消变量
`命令` $(命令) # 两个都是执行命令，获取返回值
```



## 数值运算

整数运算的三种方式

```shell
expr $x + $y
expr $x % $y # 取模，求余

$[x+y]

$((x+y))
```

自增，自减

```shell
$[i++] $[i--] $[i+=2] $[i-=8] $[i/=2]
let i-=7;echo $i # ; 两边各自看作一行命令
```

小数运算

```shell
echo 'scale=4;12.34+5.678' | bc # scale=N 限制小数位的长度
```



## 条件测试

使用 "test 表达式" 或 [表达式]

| 符号      | 作用                                      |
| --------- | ----------------------------------------- |
| ==        | 比较两个字符串是否相等                    |
| !=        | 比较两个字符串是否不相等                  |
| A&&B      | A执行成功，就执行B                        |
| A\|\|B    | A执行失败，就执行B                        |
| A;B       | A执行完，就执行B                          |
| A&&B\|\|C | A执行成功，就执行B，A或B执行失败，就执行C |
| -z        | 检查变量的值是否未设置（空值）            |
| -n        | 测试变量是否不为空                        |
| -eq       | 比较两个整数是否相等                      |
| -ne       | 比较两个整数是否不相等                    |
| -gt       | 比较前面的整数是否大于后面的整数          |
| -ge       | 比较前面的整数是否大于或等于后面的整数    |
| -lt       | 比较前面的整数是否小于后面的整数          |
| -le       | 比较前面的整数是否小于或等于后面的整数    |
| -e        | 判断对象是否存在（不管是目录还是文件）    |
| -d        | 判断对象是否为目录（存在且是目录）        |
| -f        | 判断对象是否为文件（存在且是文件）        |
| -r        | 判断对象是否可读                          |
| -w        | 判断对象是否可写                          |
| -x        | 判断对象是否具有可执行权限                |



## for 循环

```shell
# 格式一
for 变量名 in 值列表
do
	命令
done

# 格式二
for((i=1;i<=10;i+=2))
do
	命令
done
```



## while 循环

```shell
while 条件 # 条件位置使用 : 或 true 表示无限循环 
do
	命令
done
```



## if 条件

```shell
# 单分支
if "条件";then
	命令
else
	命令
fi

# 多分支
if "条件";then
	命令
elif "条件";then
	命令
else
	命令
fi
```



## case 流程控制

```shell
case 变量  in
模式1)
	命令序列1 ;;
模式2)
	命令序列2 ;;
	.. ..
*)
	默认命令序列
esac
```



## 函数

```shell
# 格式一
function  函数名 {
	命令序列
	.. ..
}

# 格式二
函数名() {
	命令序列
	.. ..
}

# 调用函数 函数名 位置参数1 位置参数2

# 生成1到100的两种方式
{1..100}
seq 100
```



## 中断控制

```shell
break # 中断循环体
continue # 直接进入下一次循环
exit # 退出脚步，产生 $? 的值，默认为0
```



## 字符串处理

字符串截取

```shell
# 字符串处理的三种方式
${"变量名":"起始位置":"长度"} # 起始位置从0开始，可省略
expr substr "${变量名}" "起始位置" "长度" # 起始位置从1开始
echo ${变量名} | cut -b "起始位置"-"结束位置" # 起始位置从1开始
```

字符串替换

```shell
${"变量名"/old/new} # 只替换匹配的第一个
${"变量名"//old/new} # 替换匹配的全部
```

字符串掐头

```shell
${"变量名"#*"关键词"} # 从左向右，最短匹配，删除之前所有
${"变量名"##*"关键词"} # 从左向右，最长匹配，删除之前所有
```

字符串去尾

```shell
${"变量名"%"关键词"*} # 从右向左，最短匹配，删除之前所有
${"变量名"%%"关键词"*} # 从右向左，最长匹配，删除之前所有
```

字符串初值的处理（默认值）

```shell
${"变量名":-"值"} # 若变量没有值，就写入值
```



## 数组

```shell
# 格式一
数组名=(值1 值2 ... 值n) # 空格分离

# 格式二
数组名[下标]=值 # 下标从0开始

# 输出值
${数组名[下标]} # 单个
${数组名[@]} # 全部
${#数组名[@]} # 数组值个数
${数组名[@]:起始下标:个数} # 输出连续多个
```



# 25 expect 预期交互

安装 expect 包

```shell
# 格式
expect << EOF
spawn ssh 192.168.4.5                               # 创建交互式进程
expect "password:" { send "123456\r" }              # 自动发送密码
expect "#"          { send "touch /tmp.txt\r" }     # 发送命令
expect "#"          { send "exit\r" }
EOF

# 如果不希望ssh时出现yes/no的提示，远程时使用如下选项:
# ssh -o StrictHostKeyChecking=no root@IP
```



# 26 正则表达式

<center>基本列表</center>

| 符号      | 作用                               |
| --------- | ---------------------------------- |
| ^         | 匹配行首                           |
| $         | 匹配行尾                           |
| []        | 集合，匹配集合中任意单个字符       |
| [^]       | 对集合取反，在查找前对规则取反     |
| .         | 任意单个字符                       |
| *         | 匹配前一字符任意次数，不能单独使用 |
| \\{n,m\\} | 匹配前一字符n次到m次               |
| \\{n\\}   | 匹配前一字符n次                    |
| \\{n,\\}  | 匹配前一字符n次以上                |
| \\(\\)    | 保留                               |

<center>扩展列表</center>

| +     | 匹配前一字符至少一次 |
| ----- | -------------------- |
| ?     | 匹配前一字符至多一次 |
| {n,m} | 匹配前一字符n次到m次 |
| ()    | 保留,组合为整体      |
| \\b   | 单词边界             |

- grep 支持基本列表

- grep -E 或 egrep 支持扩展列表

  ```shell
  grep/egrep -i # 忽略大小写
  grep/egrep -v # 取反
  ```

  
# 27 sed文本处理工具

```shell
# 格式一
sed  [选项]  '条件指令'  文件

# 格式二
前置命令 | sed  [选项]  '条件指令'
```



| 选项 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| -n   | 屏蔽默认输出，默认sed会输出读取文档的全部内容                |
| -r   | 让sed支持扩展正则                                            |
| -i   | sed直接修改源文件，默认sed只是通过内存临时修改文件，源文件无影响 |



| 常见指令 | 作用 |
| -------- | ---- |
| p        | 显示 |
| d        | 删除 |
| s        | 替换 |

```shell
# p指令案例集锦
sed -n 'p' /etc/passwd # 打印全部
sed -n '3p' /etc/passwd # 打印第3行
sed  -n '3,6p' /etc/passwd # 输出3到6行
sed -n '3p;5p' /etc/passwd # 打印第3和5行
sed -n '3,+10p' /etc/passwd # 打印第3以及后面的10行
sed -n '1~2p' /etc/passwd # 打印奇数行
sed -n '/root/p' /etc/passwd # 打印包含root的行
sed -n '/bash$/p' /etc/passwd # 打印bash结尾的行

# d指令案例集锦
sed  '3,5d' a.txt             # 删除第3~5行
sed  '/xml/d' a.txt            # 删除所有包含xml的行
sed  '/xml/!d' a.txt         # 删除不包含xml的行，!符号表示取反
sed  '/^install/d' a.txt    # 删除以install开头的行
sed  '$d' a.txt                # 删除文件的最后一行
sed  '/^$/d' a.txt            # 删除所有空行

# s指令案例集锦
sed 's/xml/XML/'  a.txt        # 将每行中第一个xml替换为XML
sed 's/xml/XML/3' a.txt        # 将每行中的第3个xml替换为XML
sed 's/xml/XML/g' a.txt        # 将所有的xml都替换为XML
sed 's/xml//g'     a.txt       # 将所有的xml都删除（替换为空串）
sed 's#/bin/bash#/sbin/sh#' a.txt  # 将/bin/bash替换为/sbin/sh
sed '4,7s/^/#/'   a.txt        # 将第4~7行注释掉（行首加#号）
sed 's/^#an/an/'  a.txt        # 解除以#an开头的行的注释（去除行首的#号）
sed -r 's/^(.)(.*)(.)$/\3\2\1/' a.txt # 将文件中每行的第一个、倒数第1个字符互换
sed -r 's/([A-Z])/[\1]/g' nssw.txt # 为文件中每个大写字母添加括号
```



| 指令 | 作用                   |
| ---- | ---------------------- |
| i    | 在指定的行之前插入文本 |
| a    | 在指定的行之后追加文本 |
| c    | 替换指定的行           |
| r    | 读入文档，有-i才会保存 |
| w    | 保存到文档             |

```shell
sed  '2a XX'   a.txt            # 在第二行后一行，追加XX
sed  '2i XX'   a.txt            # 在第二行前一行，插入XX
sed  '2c XX'   a.txt            # 将第二行替换为XX
# 可以用\n换行
sed '2r m.txt' a.txt            # 在第2行下方插入m.txt
sed '1,4r m.txt' a.txt          # 1到4行保存到m.txt
```

  

复制、剪切

模式空间

- 存放当前处理的行，将处理结果输出
- 若当前行不符合处理条件，原样输出
- 处理完当前行再读入下一行来处理

保持空间

- 类似与“剪贴板”
- 默认存放一个换行符

| 复制 | H：模式空间--[追加]--保持空间<br>h：模式空间--[覆盖]--保持空间 |
| :--- | ------------------------------------------------------------ |
| 粘贴 | G：保持空间--[追加]--模式空间<br/>g：保持空间--[覆盖]--保持空间 |

```shell
sed '1h;1d;$G' a.txt # 把第1行剪切到文末
sed '1h;2,3H;$G' a.txt # 1～3行复制到文末
```



# 28 awk 数据处理引擎

格式：awk [选项] '[条件]{指令}' 文件

```shell
awk '{print $1,$3}' test.txt        # 打印文档第1列和第3列

# 选项 -F 可指定分隔符
# 输出passwd文件中以分号分隔的第1、7个字段
awk -F: '{print $1,$7}' /etc/passwd

# 以“:”或“/”分隔，输出第1、10个字段
awk -F [:/] '{print $1,$10}' /etc/passwd

awk -F: '{print $1,"的解释器:",$7}' /etc/passwd
root 的解释器: /bin/bash
bin 的解释器: /sbin/nologin
… …
```

<center>awk常用内置变量</content>

| $0     | 文本当前行的全部内容       |
| ------ | -------------------------- |
| $n(>0) | 文本的第n列                |
| NR     | 文件当前行的行号           |
| NF     | 文件当前行的列数（有几列） |



- BEGIN{ }		行前处理，读取文件内容前执行，指令执行1次 
- { }			逐行处理，读取文件过程中执行，指令执行n次 
- END{ }		行后处理，读取文件结束后执行，指令执行1次 

```shell
# 统计系统中使用bash作为登录Shell的用户总个数
awk 'BEGIN{x=0}/bash$/{x++} END{print x}' /etc/passwd
```

awk 处理条件：正则表达式、数值/字符串比较、逻辑比较、运算符

正则： ~ 代表匹配、!~ 代表不匹配		

```shell
awk -F: '/bash$/{print}' /etc/passwd # 输出其中以bash结尾的完整记录
awk -F: '/root/' /etc/passwd # 输出包含root的行数据
awk -F: '/^(root|adm)/{print $1,$3}' /etc/passwd # 输出root或adm账户的用户名和UID信息
awk -F: '$1~/root/' /etc/passwd # 输出账户名称包含root的基本信息
awk -F: '$7!~/nologin$/{print $1,$7}' /etc/passwd # 输出其中登录Shell不以nologin结尾（对第7个字段做!~反向匹配）的用户名、登录Shell信息
awk -F: 'NR==3{print}' /etc/passwd # 输出第3行（行号NR等于3）的用户记录
awk -F: '$3>=1000{print $1,$3}' /etc/passwd # 输出账户UID大于等于1000的账户名称和UID信息
awk -F: '$3<10{print $1,$3}' /etc/passwd # 输出账户UID小于10的账户名称和UID信息
awk -F: '$3>10 && $3<20' /etc/passwd # 输出账户UID大于10并且小于20的账户信息
awk -F: '$3>1000 || $3<10' /etc/passwd # 输出账户UID大于1000或者账户UID小于10的账户信息
```

​		

## awk流程控制

```shell
# 单分支
awk -F: '{if($7~/bash$/){i++}}END{print i}'  /etc/passwd # 统计/etc/passwd文件中登录Shell是“/bin/bash”的用户个数
 
# 双分支
awk -F: '{if($3<=1000){i++}else{j++}}END{print i,j}' /etc/passwd # 分别统计/etc/passwd文件中UID小于或等于1000、UID大于1000的用户个数
```



## awk数组

定义数组的格式：数组名[下标]=元素值

调用数组的格式：数组名[下标]，下标是任意字符串

遍历数组的用法：for(变量 in 数组名){print  数组名[变量]}。

```shell
 awk 'BEGIN{a[0]=0;a[1]=11;a[2]=22; for(i in a){print i,a[i]}}'
 awk  '{ip[$1]++} END{for(i in ip) {print i,ip[i]}}' /var/log/httpd/access_log | sort -nr
```



# 29 Nginx服务器

<center>web服务器软件</center>

| Unix 与 Linux | Apache、Nginx、Tenginx、Lighttpd、Tomcat、IBMWebSphere、Jboss |
| ------------- | ------------------------------------------------------------ |
| Windows       | IIS                                                          |

```shell
# 安装Nginx
yum -y install gcc pcre-devel openssl-devel      # 安装依赖包
useradd -s /sbin/nologin nginx 
./configure   \
> --prefix=/usr/local/nginx   \               # 指定安装路径
> --user=nginx   \                            # 指定用户
> --group=nginx  \                            # 指定组
> --with-http_ssl_module                      # 开启SSL加密功能

# ./auto/options 有安装包选项

curl IP # 命令行浏览器

/usr/local/nginx/sbin/nginx                    # 启动服务
/usr/local/nginx/sbin/nginx -s stop            # 关闭服务
/usr/local/nginx/sbin/nginx -s reload          # 重新加载配置文件
/usr/local/nginx/sbin/nginx -V                 # 查看软件信息
ln -s /usr/local/nginx/sbin/nginx /sbin/       # 方便后期使用

netstat  -anptl # 查看全部端口 
```



## 升级Nginx服务器

```shell
./configure   \ # 将需要的功能源码放入objs/src内
> --prefix=/usr/local/nginx   \ 
> --user=nginx   \ 
> --group=nginx  \ 
> --with-http_ssl_module

make # 将objs/src内源码编译成程序，如objs/nginx

mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginxold # 备份旧程序
cp objs/nginx  /usr/local/nginx/sbin/         # 拷贝新版本
make upgrade                            # 升级
```



## 用户认证

```shell
vim  /usr/local/nginx/conf/nginx.conf # 打开配置文件
user nginx; # 进程所有者
worker_processes  1;  #  启动进程数量，与CPU一致
error_log  logs/error.log; # 错误日志文件
pid        logs/nginx.pid; # PID号文件

events {
    worker_connections  1024; # 单个进程最大并发量
}

http{
    server { # 定义虚拟主机 可基于IP 端口 域名
            listen       80;	# 监听端口，可以写成 IP：端口
            server_name  localhost; # 监听的域名
            auth_basic "Input Password:";                        # 认证提示符
            auth_basic_user_file "/usr/local/nginx/pass";        # 认证密码文件
            location / { # 匹配url，支持正则(前加~)，可以多个
                root   html; # 网站根目录
                index  index.html index.htm; # 默认主页左边优先级高
            }
     }
     server{
        listen 80;
        server_name www.xyz.com;
        location / { 
            root   www;                                
            index  index.html index.htm;
        }
     }
}

# 生成密码文件，创建用户及密码
yum -y install  httpd-tools
htpasswd -c /usr/local/nginx/pass   tom        # 创建密码文件
htpasswd  /usr/local/nginx/pass   jerry        # 追加用户，不使用-c选项

/usr/local/nginx/sbin/nginx -s reload   # 重新加载配置文件
```



## SSL虚拟主机

```shell
# 安装Nginx时必须使用--with-http_ssl_module参数，启用加密模块
vim  /usr/local/nginx/conf/nginx.conf
… …    
server {
        listen       443 ssl;
        server_name            www.c.com;
        ssl_certificate      cert.pem;         # 这里是证书文件 /usr/local/nginx/conf
        ssl_certificate_key  cert.key;         # 这里是私钥文件
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
        location / {
            root   html;
            index  index.html index.htm;
        }
    }
```



## 部署LNMP环境

```shell
yum -y install gcc openssl-devel pcre-devel # pcre-devel 正则依赖
yum -y install   mariadb   mariadb-server   mariadb-devel # 安装数据库
yum -y  install  php   php-mysql # php 为解释器 php-mysql 连接mysql插件
yum -y  install php-fpm-5.4.16-42.el7.x86_64.rpm # php-fpm php服务

# 查看php-fpm配置文件
vim /etc/php-fpm.d/www.conf
[www]
listen = 127.0.0.1:9000            # PHP端口号
pm.max_children = 32                # 最大进程数量
pm.start_servers = 15                # 最小进程数量
pm.min_spare_servers = 5            # 最少需要几个空闲着的进程
pm.max_spare_servers = 32           # 最多允许几个进程处于空闲状态

# 修改Nginx配置文件并启动服务
vim /usr/local/nginx/conf/nginx.conf
location / {
            root   html;
            index  index.php  index.html   index.htm;
#设置默认首页为index.php，当用户在浏览器地址栏中只写域名或IP，不说访问什么页面时，服务器会把默认首页index.php返回给用户
        }
 location  ~  \.php$  { # 正则匹配
            root           html;
            fastcgi_pass   127.0.0.1:9000;    #将请求转发给本机9000端口，PHP解释器
            fastcgi_index  index.php;
            #fastcgi_param   SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi.conf;
        }
```



## 地址重写

```shell
# 修改配置文件(访问a.html重定向到b.html)
 vim /usr/local/nginx/conf/nginx.conf
.. ..
server {
        listen       80;
        server_name  localhost;
		rewrite /a.html  /b.html (redirect); # 最后加redirect,url显示新地址
        location / {
            root   html;
        index  index.html index.htm;
        }
}

# 访问192.168.4.5的请求重定向至www.zz.cn
vim /usr/local/nginx/conf/nginx.conf
.. ..
server {
        listen       80;
        server_name  localhost;
        rewrite ^/ http://www.zz.cn/;
        location / {
            root   html;
        	index  index.html index.htm;
        }
}

# 访问192.168.4.5/下面子页面，重定向至www.zz.cn/下相同的页面
vim /usr/local/nginx/conf/nginx.conf
.. ..
server {
        listen       80;
        server_name  localhost;
        rewrite ^/(.*)$ http://www.zz.cn/$1;
        location / {
            root   html;
        	index  index.html index.htm;
        }
        # 这里，~符号代表正则匹配，*符号代表不区分大小写
        if ($http_user_agent ~* firefox) {            # 识别客户端firefox浏览器
        	rewrite ^(.*)$ /firefox/$1;
        }
}
```

地址重写格式 rewrite 旧地址  新地址 [选项];

- last        不再读其他rewrite，只在同location有用，
- break       不再读其他语句，结束判断
- redirect    临时重定向
- permament   永久重定向



## Nginx反向代理

Nginx采用轮询的方式调用后端Web服务器

```shell
vim /usr/local/nginx/conf/nginx.conf
.. ..
http {
.. ..
# 使用upstream定义后端服务器集群，集群名称任意(如webserver)
# 使用server定义集群中的具体服务器和端口
upstream webserver {
				# 通过ip_hash设置调度规则为：相同客户端访问相同服务器
                ip_hash;
                server 192.168.2.100 weight=1 max_fails=1 fail_timeout=30;
                server 192.168.2.200 weight=2 max_fails=2 fail_timeout=30;
                server 192.168.2.101 down;
                # weight设置服务器权重值，默认值为1
                # max_fails设置最大失败次数
                # fail_timeout设置失败超时时间，单位为秒
                # down标记服务器已关机，不参与集群调度
        }
.. ..
server {
        listen        80;
        server_name  localhost;
        location / {
			# 通过proxy_pass将用户的请求转发给webserver集群
            proxy_pass http:// webserver;
        }
}
```

### Nginx的TCP/UDP调度器

```shell
yum –y install gcc pcre-devel openssl-devel        # 安装依赖包
tar  -xf   nginx-1.12.2.tar.gz
cd  nginx-1.12.2
./configure   \
> --with-http_ssl_module   \                             # 开启SSL加密功能
> --with-stream                                       # 开启4层反向代理功能
make && make install           # 编译并安装

vim /usr/local/nginx/conf/nginx.conf
stream {
            upstream backend {
               server 192.168.2.100:22;            # 后端SSH服务器的IP和端口
               server 192.168.2.200:22;
			}
            server {
                listen 12345;                    # Nginx监听的端口
                proxy_connect_timeout 10s;
                proxy_timeout 30s;
                 proxy_pass backend;
            }
}
http {
.. ..
}
```



## Nginx常见问题处理

### 自定义报错页面

```shell
vim /usr/local/nginx/conf/nginx.conf
.. ..
error_page   404  /40x.html;    # 自定义错误页面
.. ..
```



### 查看服务器状态信息

```shell
./configure   \
> --with-http_ssl_module                        # 开启SSL加密功能
> --with-stream                                 # 开启TCP/UDP代理模块
> --with-http_stub_status_module                # 开启status状态页面

# 修改Nginx配置文件，定义状态页面
vim /usr/local/nginx/conf/nginx.conf
… …
location /status {
                stub_status on;
                 #allow IP地址;
                 #deny IP地址;
        }
… …

# 查看状态页面信息
curl  http:// 192.168.4.5/status
Active connections: 1 
server accepts handled requests
 10 10 3 
Reading: 0 Writing: 1 Waiting: 0
# Active connections：当前活动的连接数量。
# Accepts：已经接受客户端的连接总数量。
# Handled：已经处理客户端的连接总数量。（一般与accepts一致，除非服务器限制了连接数量）。
# Requests：客户端发送的请求数量。
# Reading：当前服务器正在读取客户端请求头的数量。
# Writing：当前服务器正在写响应信息的数量。
# Waiting：当前多少客户端在等待服务器的响应。
```



### 优化Nginx并发量

```shell
# 优化Nginx并发量
vim /usr/local/nginx/conf/nginx.conf
.. ..
worker_processes  2;                    # 与CPU核心数量一致
events {
	worker_connections 65535;        # 每个worker最大并发连接数
use epoll;
}
.. ..

# 优化Linux内核参数（最大文件数量）
ulimit -a                        # 查看所有属性值
ulimit -Hn 100000                # 设置硬限制（临时规则）
ulimit -Sn 100000                # 设置软限制（临时规则）
vim /etc/security/limits.conf # 手动写入
    .. ..
*               soft    nofile            100000
*               hard    nofile            100000
# 该配置文件分4列，分别如下：
# 用户或组    硬限制或软限制    需要限制的项目   限制的值

# 测试并发量
ab -n 2000 -c 2000 http://192.168.4.5/
```



### 优化Nginx数据包头缓存

```shell
vim /usr/local/nginx/conf/nginx.conf
.. ..
http {
    client_header_buffer_size    1k;        # 默认请求包头信息的缓存    
    large_client_header_buffers  4 4k;      # 大请求包头部信息的缓存个数与容量 一个请求16k
    .. ..
}
```



### 浏览器本地缓存静态数据

```shell
vim /usr/local/nginx/conf/nginx.conf
server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        location ~* \.(jpg|jpeg|gif|png|css|js|ico|xml)$ {
            expires        30d;            # 定义客户端缓存时间为30天
        }
}
```



### 日志切割

```shell
# 每周5的03点03分自动执行脚本完成日志切割工作
vim /usr/local/nginx/logbak.sh
#!/bin/bash
date=`date +%Y%m%d`
logpath=/usr/local/nginx/logs
mv $logpath/access.log $logpath/access-$date.log
mv $logpath/error.log $logpath/error-$date.log
kill -USR1 $(cat $logpath/nginx.pid)

crontab -e
03 03 * * 5  /usr/local/nginx/logbak.sh
```

<center>KILL 信号</center>

| 字符型号 | 对应数字 | 作用                             |
| -------- | -------- | -------------------------------- |
| HUP      | 1        | 终端断线，重新加载配置，平滑升级 |
| INT      | 2        | 中断（同Ctrl+c）                 |
| QUIT     | 3        | 退出（同Ctrl+\）                 |
| TERM     | 15       | 终止（无信号选项时的默认值）     |
| KILL     | 9        | 强制终止                         |
| CONT     | 18       | 继续（与STOP相反，fg/bg命令）    |
| STOP     | 19       | 暂停（同Ctrl+z）                 |

以上除了KILL信号，程序都可以忽略。



### 对页面进行压缩处理

```shell
vim /usr/local/nginx/conf/nginx.conf
http {
.. ..
gzip on;                            # 开启压缩
gzip_min_length 1000;               # 小文件不压缩
gzip_comp_level 4;                  # 压缩比率
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
                                    # 对特定文件压缩，类型参考mime.types
.. ..
}
```



### 服务器内存缓存

```shell
http { 
    open_file_cache          max=2000  inactive=20s;
            open_file_cache_valid    60s;
            open_file_cache_min_uses 5;
            open_file_cache_errors   off;
    # 设置服务器最大缓存2000个文件句柄，关闭20秒内无请求的文件句柄
    # 文件句柄的有效时间是60秒，60秒后过期
    # 只有访问次数超过5次会被缓存
    # 缓存报错不报警
} 
```



# 30 memcached服务

memcached是高性能的分布式缓存服务器，用来集中缓存数据库查询结果，减少数据库访问次数，以提高动态Web应用的响应速度。

## 构建memcached服务

```shell
yum -y  install   memcached

vim /usr/lib/systemd/system/memcached.service
ExecStart=/usr/bin/memcached -u $USER -p $PORT -m $CACHESIZE -c $MAXCONN $OPTIONS

vim /etc/sysconfig/memcached
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS=""
```

验证时需要客户端主机安装telnet，远程memcached来验证服务器的功能：

```shell
telnet  192.168.4.5  11211
Trying 192.168.4.5...
……
## 提示：0表示不压缩，180为数据缓存时间，3为需要存储的数据字节数量。
set name 0 180 3                # 定义变量，变量名称为name
plj                            # 输入变量的值，值为plj                
STORED
get name                        # 获取变量的值
VALUE name 0 3                 # 输出结果
plj
END
## 提示：0表示不压缩，180为数据缓存时间，3为需要存储的数据字节数量。
add myname 0 180 10            # 新建，myname不存在则添加，存在则报错
set myname 0 180 10            # 添加或替换变量
replace myname 0 180 10        # 替换，如果myname不存在则报错
get myname                    # 读取变量
append myname 0 180 10        # 向变量中追加数据
delete myname                    # 删除变量
stats                        # 查看状态
flush_all                        # 清空所有
quit                            # 退出登录         
```

| add       | 新建变量         |
| --------- | ---------------- |
| set       | 新建或替换变量   |
| replace   | 替换变量         |
| get       | 读取变量         |
| append    | 向变量中追加数据 |
| delete    | 删除变量         |
| status    | 查看状态         |
| flush_all | 清空所有         |



## LNMP+memcached

使用PHP来操作memcached，注意必须要为PHP安装memcache扩展（php-pecl-memcache），否则PHP无法解析连接memcached的指令。



## PHP Session 共享

```shell
/var/lib/php/session/            # 查看服务器本地的Session信息

vim  /etc/php-fpm.d/www.conf            # 修改该配置文件的两个参数
# 文件的最后2行
修改前效果如下:
php_value[session.save_handler] = files
php_value[session.save_path] = /var/lib/php/session
# 原始文件，默认定义Sessoin会话信息本地计算机（默认在/var/lib/php/session）
修改后效果如下:
php_value[session.save_handler] = memcache
php_value[session.save_path] = "tcp:// 192.168.2.5:11211"
# 定义Session信息存储在公共的memcached服务器上，主机参数中为memcache（没有d）
# 通过path参数定义公共的memcached服务器在哪（服务器的IP和端口）
```



## PHP -FPM 错误日志文件

```
/var/log/php-fpm/www-error.log
```



# 31 Tomcat

| 软件环境     | openjdk                            |
| ------------ | ---------------------------------- |
| 软件包       | tomcat                             |
| 端口         | 8080                               |
| 主配置文件   | /usr/local/tomcat/conf/server.xml  |
| Java SE      | 标准版                             |
| Java EE      | 企业版                             |
| Java ME      | 移动版                             |
| 支持Java网页 | tomcat、websphere、weblogic、Jboss |

部署Tomcat服务器软件

```shell
yum -y install  java-1.8.0-openjdk                # 安装JDK
yum -y install java-1.8.0-openjdk-headless        # 安装JDK
java -version                                    # 查看JAVA版本

ls /usr/local/tomcat
bin/                                            # 主程序目录
lib/                                            # 库文件目录
logs/                                          # 日志目录  
temp/                                         # 临时目录
work/                                        # 自动编译目录jsp代码转换servlet
conf/                                        # 配置文件目录
webapps/                                        # 页面目录

# 启动服务
/usr/local/tomcat/bin/startup.sh

netstat -nutlp |grep java        # 查看java监听的端口
tcp        0      0 :::8080              :::*                LISTEN      2778/java           
tcp        0      0 ::ffff:127.0.0.1:8005     :::*         LISTEN       2778/java  
```



## 虚拟主机

```shell
cat /usr/local/tomcat/conf/server.xml
<Server>
   <Service>
     <Connector port=8080 /> # 监听端口 虚拟主机通用
     <Connector port=8009 />
     <Engine name="Catalina" defaultHost="localhost"> # defaultHost 定义访问ip时 默认打开的Host
    <Host name="www.a.com" appBase="webapps/a" unpackWARS="true" autoDeploy="true"> # 每个Host是一个虚拟主机
    	<Context path="" docBase="base"/>
    	<Context path="/test" docBase="/var/www/html/" /> # 当用户访问http:// www.a.com/test打开/var/www/html目录下的页面
    </Host>
    <Host name="www.b.com" appBase="webapps/b" unpackWARS="true" autoDeploy="true">
</Host>
… …

# appBase 定义基础目录，默认项目文件夹是ROOT
# docBase 定义首页目录，默认项目文件夹是ROOT
# path 匹配url
```



## SSL 加密站点

```shell
vim /usr/local/tomcat/conf/server.xml
… …
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
keystoreFile="/usr/local/tomcat/keystore" keystorePass="123456" clientAuth="false" sslProtocol="TLS" />
# 端口 8443
# 备注，默认这段Connector被注释掉了，打开注释，添加密钥信息即可
```



## 配置Tomcat日志

```shell
 vim /usr/local/tomcat/conf/server.xml
.. ..
<Host name="www.a.com" appBase="a" unpackWARS="true" autoDeploy="true">
<Context path="/test" docBase="/var/www/html/" />
# 从默认localhost虚拟主机中把Valve这段复制过来，适当修改下即可
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix=" a_access" suffix=".txt" # prefix 文件名 suffix 扩展名
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>
<Host name="www.b.com" appBase="b" unpackWARS="true" autoDeploy="true">
<Context path="" docBase="base" />
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix=" b_access" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>
.. ..
```



# 32 Varnish缓存服务器

- 使用Varnish加速后端Web服务 
- 代理服务器可以将远程的Web服务器页面缓存在本地 
- 远程Web服务器对客户端用户是透明的 
- 利用缓存机制提高网站的响应速度 
- Varnish支持将缓存数据存储在硬盘和内存

## 部署Varnish缓存服务器

```shell
yum -y install gcc readline-devel    # 安装软件依赖包
yum -y install ncurses-devel         # 安装软件依赖包
yum -y install pcre-devel            # 安装软件依赖包
yum -y install python-docutils-0.11-0.2.20130715svn7687.el7.noarch.rpm # 安装软件依赖包
useradd -s /sbin/nologin varnish                # 创建账户
tar -xf varnish-5.2.1.tar.gz
cd varnish-5.2.1
./configure
make && make install

# 复制启动脚本及配置文件
cp  etc/example.vcl   /usr/local/etc/default.vcl # 可以复制到任意地点

# 修改代理配置文件
vim  /usr/local/etc/default.vcl
backend default {
     .host = "192.168.2.100";
     .port = "80";
 }
 
# 启动服务
varnishd  -f /usr/local/etc/default.vcl
# varnishd命令的其他选项说明如下：
# varnishd –s malloc,128M        定义varnish使用内存作为缓存，空间为128M
# varnishd –s file,/var/lib/varnish_storage.bin,1G 定义varnish使用文件作为缓存

# 查看varnish日志
varnishlog                        # varnish日志
varnishncsa                    # 访问日志

# 更新缓存数据，在后台web服务器更新页面内容后，用户访问代理服务器看到的还是之前的数据，说明缓存中的数据过期了需要更新（默认也会自动更新，但非实时更新）。
varnishadm  
varnish> ban req.url ~ .*
# 清空缓存数据，支持正则表达式
```



# 33 Subversion 版本控制

自动管理一个文件的多个版本

Subversion 允许数据恢复到早期版本，可以检查数据修改的历史，允许多人协作文档并跟踪所做的修改

Subversion 通信方式：本地访问、SVN服务、web服务

```shell
# 安装Subversion服务
yum -y install subversion

# 创建版本库
mkdir /var/svn/ # 内部放置本地仓库
svnadmin create /var/svn/project # 创建一个仓库文件夹

# 本地数据导入仓库
svn import . file:///var/svn/project/ -m "Init Data" # -m 注释 数据库方式存储 . 代表当前目录
 
# 修改配置文件 三个 必须顶头写
# 第一个
vim /var/svn/project/conf/svnserve.conf
[general]
### These options control access to the repository for unauthenticated
### and authenticated users.  Valid values are "write", "read",
### and "none".  The sample settings below are the defaults.
anon-access = none
# 19行，匿名无任何权限
auth-access = write
# 20行，有效账户可写
password-db = passwd
# 27行，密码文件
authz-db = authz
# 34行，ACL访问控制列表文件

# 第二个
vim /var/svn/project/conf/passwd 
… …
[users]
harry = 123456 # 用户名和密码
tom = 123456 # 用户名和密码

# 第三个
cat /var/svn/project/conf/authz
[/test]
tom = rw						   # 只有tom能读写/test的内容

[/hello]
harry = r						   # 只有harry能读/hello的内容

[/]                                # 定义ACL访问控制
harry = rw                         # 用户对项目根路径可读可写
tom = rw
* =								   # 其他人没有权限

# 启动服务
svnserve -d  -r /var/svn/project # -d 后台运行 -r 指定目录

netstat -nutlp |grep svnserve
tcp        0      0 0.0.0.0:3690    0.0.0.0:*    LISTEN      4043/svnserve  
```

客户端

```shell
# 安装Subversion服务
yum -y install subversion
svn --username harry --password 123456 co svn://192.168.2.100/ code        
# 建立本地副本,从服务器192.168.2.100上co checkout 下载代码到本地code目录
# 用户名harry,密码123456

svn ci -m "modify user"        # ci commit 将本地修改的数据同步到服务器
svn update                    # 将服务器上新的数据同步到本地
svn info     svn://192.168.2.100    # 查看版本仓库基本信息
svn log     svn://192.168.2.100     # 查看版本仓库的日志
svn add test.sh                # 将新的文件或目录加入版本控制
svn mkdir subdir                # 创建目录

svn rm timers.target            # 使用svn删除本地文件
svn ci -m "xxx"                # 提交svn删除文件操作

svn diff                     # 查看所有文件的差异
svn diff umount.target       # 仅查看某一个文件的差异
svn cat svn://192.168.2.100/reboot.target    # 查看服务器文件的内容

svn revert tmp.mount            # 还原tmp.mount文件
svn merge -r7:2    tuned.service    # 将文件从版本7还原到版本2
```



## 使用dump指令备份版本库数据

```shell
svnadmin dump /var/svn/project > project.bak  # 备份
 
制作nginx的RPM包svnadmin create /var/svn/project2               # 新建空仓库
svnadmin load /var/svn/project2 < project.bak      # 还原
```



## 目前常用的版本控制工具git和svn，各有各的优缺点，该如何选择呢？

SVN是Subversion的简称，目前是Apache项目底下的一个开放源代码的版本控制系统，它的设计目标就是取代CVS。

SVN是集中式管理。

优点

- 集中式管理，管理方式在服务端配置好，客户端只需要同步提交即可，使用方便，操作简单，很容易就可以上手。
- 在服务端统一控制好访问权限，利用代码的安全管理。
- 所有的代码已服务端为准，代码一致性高。

缺点

- 所有操作都需要通过服务端进行同步，这会导致服务器性能要求比较高。如果服务器宕机了就无法提交代码了。
- 分支管理不灵活，svn分支是一个完整的目录，且这个目录拥有完整的实际文件，这些操作都是在服务端进行同步的，不是本地化操作，如果要删除分之，也是需要将远程的分支进行删除，这会导致大家都得同步。
- 需要联网。如果无法连接到SVN服务器，就无法提交自己的代码，更别说还原、对比等操作了。如果在内网还好，网速比较稳定，同步相对比较快，如果是通过外网同步，有可能就需要同步很久。



git是Linus Trovalds大神的作品，是一个开放源码的版本控制软件。与SVN最大的区别，就是分布式的管理。

优点

- 分布式开发时，可以git clone克隆一个本地版本，然后在本地进行操作提交，本地可以完成一个完整的版本控制。在发布的时候，使用git push来推送到远程即可。
- git分支的本质是一个指向提交快照的指针，速度快、灵活，分支之间可以任意切换。都可以在本地进行操作可以不同步到远程。
- 冲突解决，多人开发很容易就会出现冲突，可以先pull远程到本地，然后在本地合并一下分支，解决好冲突，在push到远程即可。
- 离线工作，如果git服务器出现问题，也可以在本地进行切换分支的操作，等联网后再提交、合并等操作。

缺点

- git没有严格的权限控制，一般是通过系统设置文件的读写权限来做权限控制。
- 工作目录只能是整个目录，而svn可以单独checkout某个有权限的目录。
- git上手可能没有svn那边顺手，需要经过学习一下。





总结

如果对访问控制、权限分配和代码安全性等要求比较高的，建议使用svn。

如果是分布式，多人开发，版本迭代比较快的项目，建议使用git。

# 34 制作RPM包

```shell
# 制作nginxRPM包
# 安装rpm-build软件
yum -y install  rpm-build

rpmbuild -ba x.spec               # 会报错，没有文件或目录 自动创建目录
ls /root/rpmbuild                    # 自动生成的目录结构
BUILD  BUILDROOT  RPMS  SOURCES  SPECS  SRPMS

cp nginx-1.12.2.tar.gz /root/rpmbuild/SOURCES/ # 将源码软件复制到SOURCES目录

# 创建并修改SPEC配置文件
vim /root/rpmbuild/SPECS/nginx.spec  # .spec 自动生成格式
Name:nginx        # 源码包名字，不能写错
Version:1.12.2	 # 源码包版本号，不能写错
Release:    10   # 第几次制作RPM包
Summary: Nginx is a web server software.    # 简介，任意
Group： # 属于哪个组包 yum groupinstall 组包名 # 安装组包
License:GPL    # 协议，GPL 发布无限制
URL:    www.test.com    
Source0:nginx-1.12.2.tar.gz # 源码包全名，不能写错
#BuildRequires:    
#Requires:    # 提示依赖关系
%description # 详细介绍信息
nginx [engine x] is an HTTP and reverse proxy server.

%post  #后跟脚本,自己添加
useradd nginx                       # 非必需操作：安装后脚本(创建账户)

%prep
%setup –q                           # 自动解压源码包，并cd进入目录

%build
./configure --with-http_ssl_module  # 修改
make %{?_smp_mflags}

%install
make install DESTDIR=%{buildroot}

%files
%doc

/usr/local/nginx/*            # 对哪些文件与目录打包

%changelog


# 创建RPM包
rpmbuild -ba /root/rpmbuild/SPECS/nginx.spec
```



# 35 VPN Virtual Private Network

虚拟专用网络

在公用网络上建立专用私有网络，<font color=red>进行加密通讯</font>

多用于为集团公司的各地子公司建立连接

可用软件：GRE，PPTP，L2TP+IPSec，SSL

安全度，左到右越来越安全

左到右越来越难



## GRE

windows不支持，linux默认支持GRE模块(不自动激活)，思科，华为设备默认支持,适合局域网

只是封装数据，不加密数据。

本案例要求搭建一个GRE VPN环境，并测试该VPN网络是否能够正常通讯，要求如下：

- 启用内核模块ip_gre 
- 创建一个虚拟VPN隧道(10.10.10.0/24) 
- 实现两台主机点到点的隧道通讯 

```shell
# 激活ip_gre模块
lsmod                            # 显示模块列表
lsmod  | grep ip_gre             # 确定是否加载了gre模块
modprobe  ip_gre                 # 加载模块ip_gre
modinfo ip_gre                   # 查看模块信息
rmmod ip_gre 					 # 关闭模块

# 创建VPN隧道
ip tunnel add tun0  mode gre remote 201.1.2.5 local 201.1.2.10
# ip tunnel add创建隧道（隧道名称为tun0），ip tunnel help可以查看帮助
# mode设置隧道使用gre模式
# local后面跟本机的IP地址，remote后面是与其他主机建立隧道的对方IP地址

# 启用该隧道
ip link show                # 查看网卡信息
ip link set tun0 up         # 设置UP
ip link show

# 为VPN配置隧道IP地址
ip addr add 10.10.10.10/24 peer 10.10.10.5/24 dev tun0
# 为隧道tun0设置本地IP地址（10.10.10.10.10/24）
# 隧道对面的主机IP的隧道IP为10.10.10.5/24
ip a s                      # 查看IP地址
```



## PPTP VPN

PPTP 部分数据加密

本案例要求搭建一个PPTP VPN环境，并测试该VPN网络是否能够正常通讯，要求如下:

- 使用PPTP协议创建一个支持身份验证的隧道连接 

- 使用MPPE对数据进行加密 

- 为客户端分配192.168.3.0/24的地址池 

- 客户端连接的用户名为jacob，密码为123456 

```shell
# 部署VPN服务器
yum -y install pptpd-1.4.0-2.el7.x86_64.rpm

# 修改配置文件
vim /etc/pptpd.conf
.. ..
localip 201.1.2.5                                   # 服务器本地IP
remoteip 192.168.3.1-50                             # 分配给客户端的IP池

vim /etc/ppp/options.pptpd
named pptpd											# 服务器标记（默认有）
require-mppe-128                                    # 使用MPPE加密数据（默认有）
ms-dns 8.8.8.8                                      # DNS服务器

vim /etc/ppp/chap-secrets            # 修改账户配置文件
jacob           *               123456      *
# 用户名    服务器标记    密码    客户端

# 启动pptpd
systemctl start pptpd

# 翻墙设置
 iptables -t nat -A POSTROUTING -s 192.168.3.0/24 -j SNAT --to-source 201.1.2.5
 
# 日志
/var/log/message
```

  

## L2TP+IPSec VPN

L2TP 建立主机之间的VPN通道，压缩、验证

IPSec 提供数据加密、数据校验、访问控制

```shell
# 部署IPSec服务
yum -y install libreswan

cat /etc/ipsec.conf                # 仅查看一下该主配置文件
.. ..
include /etc/ipsec.d/*.conf                    # 加载该目录下的所有配置文件

vim /etc/ipsec.d/myipsec.conf            
# 新建该文件，参考lnmp_soft/vpn/myipsec.conf    
conn IDC-PSK-NAT
    rightsubnet=vhost:%priv                        # 允许建立的VPN虚拟网络
    also=IDC-PSK-noNAT
conn IDC-PSK-noNAT
    authby=secret                                    # 加密认证
        ike=3des-sha1;modp1024                       # 算法
        phase2alg=aes256-sha1;modp2048               # 算法
    pfs=no
    auto=add
    keyingtries=3
    rekey=no
    ikelifetime=8h
    keylife=3h
    type=transport
    left=201.1.2.10                                # 重要，服务器本机的外网IP
    leftprotoport=17/1701
    right=%any                                    # 允许任何客户端连接
    rightprotoport=17/%any

# 创建IPSec预定义共享密钥
cat /etc/ipsec.secrets                # 仅查看，不要修改该文件
include /etc/ipsec.d/*.secrets

vim /etc/ipsec.d/mypass.secrets       # 新建该文件
201.1.2.10   %any:    PSK    "randpass"       # randpass为预共享密钥 201.1.2.10是VPN服务器的IP

# 启动IPSec服务
systemctl start ipsec

# 部署XL2TP服务
yum -y install xl2tpd-1.3.8-2.el7.x86_64.rpm

# 修改xl2tp配置文件（修改3个配置文件的内容）
vim  /etc/xl2tpd/xl2tpd.conf                # 修改主配置文件
[global]
.. ..    
[lns default]
.. ..
ip range = 192.168.3.128-192.168.3.254                    # 分配给客户端的IP池
local ip = 201.1.2.10                                    # VPN服务器的IP地址

vim /etc/ppp/options.xl2tpd            # 认证配置
require-mschap-v2                                         # 取消注释一行，强制要求认证
#crtscts                                               # 注释或删除该行
#lock                                                # 注释或删除该行

vim /etc/ppp/chap-secrets                    # 修改密码文件
jacob   *       123456  *                # 账户名称   服务器标记   密码   客户端IP

# 启动服务
systemctl start xl2tpd
```

分布式开发时，可以git clone克隆一个本地版本，然后在本地进行操作提交，本地可以完成一个完整的版本控制。在发布的时候，使用git push来推送到远程即可。

git分支的本质是一个指向提交快照的指针，速度快、灵活，分支之间可以任意切换。都可以在本地进行操作可以不同步到远程。

冲突解决，多人开发很容易就会出现冲突，可以先pull远程到本地，然后在本地合并一下分支，解决好冲突，在push到远程即可。

离线工作，如果git服务器出现问题，也可以在本地进行切换分支的操作，等联网后再提交、合并等操作。

## linux 路由器功能

```shell
# 临时生效：
echo "1" > /proc/sys/net/ipv4/ip_forward

# 永久生效的话，需要修改/etc/sysctl.conf：
net.ipv4.ip_forward = 1
sysctl -p # 立即生效
```



# 36 PSSH

可以同时远程连接多个主机，可以批量操作

格式：pssh  [ ]  执行命令

<center>PSSH 选项</center>

| -A     | 使用密码远程其他主机（默认使用密钥） |
| ------ | ------------------------------------ |
| -i     | 显示结果                             |
| -H     | 需要连接的主机名                     |
| -h     | 连接的主机列表文件                   |
| -p     | 并发连接数                           |
| -t     | 超时时间                             |
| -o dir | 标准输出信息保存的目录               |
| -e dir | 错误输出信息保存的目录               |
| -x     | 传递参数给ssh                        |

```shell
pssh -i  -A -H  'host1 host2 host3' -x '-o StrictHostKeyChecking=no' echo hello

ssh-keygen -N  ''   -f /root/.ssh/id_rsa     # 非交互生成密钥文件
ssh-copy-id  IP

# pscp.pssh提供并发拷贝文件功能
# -r    递归拷贝目录
# 其他选项基本与pssh一致
 pscp.pssh -r -h host.txt   /etc   /tmp 
 
 
# pslurp提供远程下载功能
# 选项与pscp.pssh基本一致
pslurp  -h host.txt  /etc/passwd  /pass # 将远程主机的/etc/passwd，拷贝到当前目录下，存放在对应IP下的pass文件中
```



# 37 udev规则

实现如下功能：

- 处理设备命名 
- 决定要创建哪些设备文件或链接 
- 决定如何设置属性 
- 决定触发哪些事件 

udev默认规则存放在/etc/udev/rules.d目录下，通过修改此目录下的规则实现设备的命名、属性、链接文件等。

| 选项               | 值                                 |
| ------------------ | ---------------------------------- |
| ==                 | 表示匹配                           |
| ！=                | 表示不匹配                         |
| =                  | 指定赋予的值                       |
| +=                 | 添加新值                           |
| :=                 | 指定值，不允许被替换               |
| NAME=              | 定义设备名称                       |
| SYMLINK+=          | 的定义设备的别名                   |
| OWNER=             | 定义设备的所有者                   |
| GROUP=             | 定义设备的所属组                   |
| MODE=              | 定义设备的权限                     |
| ACTION==           | 判断设备的操作动作（添加、删除等） |
| KERNEL==“sd[a-z]1” | 判断设备的内核名称                 |
| RUN+=“”            | 执行脚本                           |

udev常用替代变量：

- %k：内核所识别出来的设备名，如sdb1 
- %n：设备的内核编号，如sda3中的3 
- %p：设备路径，如/sys/block/sdb/sdb1 

```shell
# 加载USB设备的同时实时查看设备的相关属性，可以使用monitor指令。
udevadm monitor --property

# 如果设备已经加载则无法使用monitor查看相关属性。可以使用下面的命令查看设备路径等属性。
udevadm info --query=path --name=/dev/sda
# 利用路径单独查看某个磁盘分区的属性信息
udevadm info --query=property --path=/block/sdada1

# 编写udev规则文件
vim  /etc/udev/rules.d/70-usb.rules
ACTION=="add",ENV{ID_VENDOR}=="TOSHIBA",ENV{DEVTYPE}=="partition",ENV{ID_SERIAL_SHORT}=="60A44CB4665EEE4133500001",SYMLINK="usb%n",OWNER="root",GROUP="root",MODE="0644",RUN+="/usr/bin/systemctl start httpd"
```



# 38 集群-LVS

## 集群

一组通过网络互联的计算机组，核心技术是任务调度

目的：

- 提高性能

- 降低成本

- 提高可扩展性

- 增强可靠性

分类：

- 高性能计算集群HPC：通过集群开发的并行应用程序，解决复杂的科学问题。
- 负载均衡（LB）集群：客户端负载在计算机集群中尽可能平均分摊。
- 高可用（HA）集群：避免单点故障，当一个系统发生故障时，可以快速迁移。

主流集群 nginx、LVS、Haprxy

性能：LVS > Haprxy > nginx

功能：nginx > Haprxy > LVS

协议：LVS 仅支持4层、不能正则，ngingx 支持4层与7层，Haprxy 支持4层与7层。



## LVS

镶嵌在内核上。

组成：

- 前端：负载均衡层，由一台或多台负载调度器构成
- 中间：服务器群组层，一组实际运行服务的服务器
- 底端：数据共享存储层

Director Server 调度服务器

Real Server 真实服务器

VIP 虚拟IP地址，面对客户

RIP 真实IP地址，集群使用

DIP 调度器连接节点服务器的IP

工作模式：

VS/NAT

- 通过网络地址转换实现的虚拟服务器
- 大并访问时，调度器的性能成为瓶颈

VS/DR

- 直接使用路由器技术实现虚拟服务器
- 节点服务器需要配置VIP，注意MAC地址广播

VS/TUN

- 通过隧道方式实现虚拟服务器



LVS目前实现了10种调度算法，常用4种：

轮询（Round Robin）

加权轮询（Weighted Round Robin）

最少连接（Least Connections）

加权最少连接（Weighted Least Connections）

IP_hash (sh)

## ipvsadm 调用LVS

| ipvsadm -A            | 添加集群                               |
| --------------------- | -------------------------------------- |
| ipvsadm -E            | 修改集群                               |
| ipvsadm -C            | 删除集群                               |
| ipvsadm -a            | 集群添加真实服务器                     |
| ipvsadm -e            | 集群修改真实服务器                     |
| ipvsadm -d            | 集群删除真实服务器                     |
| ipvsadm -L            | 查看LVS规则表                          |
| ipvsadm -Ln           | n 显示端口                             |
| -s [rr\|wrr\|lc\|wlc] | 在指定集群算法                         |
| -m                    | 使用NAT模式（所有流量走调度服务器）    |
| -g                    | 使用DR模式（所有流量直接返回给客户端） |
| -i                    | 使用Tunnel模式                         |

```shell
ipvsadm -A -t|u 192.168.4.5:80 -s [算法] # -t tcp -u udp
ipvsadm -a -t|u 192.168.4.5:80 -r 192.168.2.100 [-g|i|m] [-w 权重] 
ipvsadm -a -t 192.168.4.5:80 -r 192.168.2.100 -m -w 1
# -g DR模式 -i 隧道模式 -m NAT模式 -r 指定后端服务器
ipvsadm-save -n > /etc/sysconfig/ipvsadm # 永久保存所有规则，可以手动写

# 开启linux路由转发
echo 1 > /proc/sys/net/ipv4/ip_forward
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf # 修改配置文件，设置永久规则
```



## 部署LVS-NAT

```shell
# 开启linux路由转发
# 创建集群服务器
yum -y install ipvsadm
ipvsadm -A -t 192.168.4.5:80 -s wrr

# 添加真实服务器
ipvsadm -a -t 192.168.4.5:80 -r 192.168.2.100 -w 1 -m
ipvsadm -a -t 192.168.4.5:80 -r 192.168.2.200 -w 1 -m

# 查看规则列表，并保存规则
ipvsadm -Ln
ipvsadm-save -n > /etc/sysconfig/ipvsadm
```



## 部署LVS-DR集群

```shell
# 为了防止冲突，VIP必须要配置在网卡的虚拟接口！！！虚拟网卡用户可以访问到。
cd /etc/sysconfig/network-scripts/
cp ifcfg-eth0{,:0}

vim ifcfg-eth0
TYPE=Ethernet
BOOTPROTO=none
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.4.5
PREFIX=24

vim ifcfg-eth0:0
TYPE=Ethernet
BOOTPROTO=none
DEFROUTE=yes
NAME=eth0:0
DEVICE=eth0:0
ONBOOT=yes
IPADDR=192.168.4.15
PREFIX=24

systemctl restart network
 
# 给web1配置VIP地址,lo用于自身伪装。
cd /etc/sysconfig/network-scripts/
cp ifcfg-lo{,:0}
vim ifcfg-lo:0
DEVICE=lo
linux查看日志： 
# cd /var/log 
# less secure 
或者 
# less messages 
最近登录的日志： 
# last :0
IPADDR=192.168.4.15
NETMASK=255.255.255.255 # 必须32位
NETWORK=192.168.4.15
BROADCAST=192.168.4.15
ONBOOT=yes
NAME=lo:0

# 防止地址冲突的问题：
# 这里因为web1也配置与代理一样的VIP地址，默认肯定会出现地址冲突；
# sysctl.conf文件写入这下面四行的主要目的就是访问192.168.4.15的数据包，只有调度器会响应，其他主机都不做任何响应，这样防止地址冲突的问题。

vim /etc/sysctl.conf
#手动写入如下4行内容
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.lo.arp_announce = 2
net.ipv4.conf.all.arp_announce = 2
#当有arp广播问谁是192.168.4.15时，本机忽略该ARP广播，不做任何回应
#本机不要向外宣告自己的lo回环地址是192.168.4.15

sysctl -p # 配置立即生效
```



# 39 Keepalived

自动配置LVS，用于监控LVS集群，防止单点故障。照抄路由器VRRP功能（VRRP 热备份）

配优先级，VIP

## Keepalived高可用服务器

```shell
# 安装Keepalived
yum install -y keepalived
# 配置文件
vim /etc/keepalived/keepalived.conf
global_defs {
  notification_email {
    admin@zz.com.cn                # 设置报警收件人邮箱
  }
  notification_email_from ka@localhost    # 设置发件人
  smtp_server 127.0.0.1                # 定义邮件服务器
  smtp_connect_timeout 30
  router_id  web1                        # 设置路由ID号（实验需要修改）
}
vrrp_instance VI_1 {
  state MASTER                         # 主服务器为MASTER（备服务器需要修改为BACKUP）
  interface eth0                    # 定义网络接口,不会配置在主接口上
  virtual_router_id 50                # 主备服务器VRID号必须一致
  priority 100                     # 服务器优先级,优先级高优先获取VIP（实验需要修改）
  advert_int 1                     # 间隔确认时间
  authentication {
    auth_type pass
    auth_pass 1111                       # 主备服务器密码必须一致
  }
  virtual_ipaddress {        # 谁是主服务器谁获得该VIP（实验需要修改
        192.168.200.16
        192.168.200.17
        192.168.200.18
   }     
}
# 启动keepalived会自动添加一个drop的防火墙规则，需要清空！
 iptables -F # 清空防火墙规则

# 启动服务
systemctl start keepalived

# 查看ip 
ip a s eth0
```



## Keepalived+LVS服务器

```shell
vim /etc/keepalived/keepalived.conf
global_defs {
  notification_email {
    admin@tarena.com.cn                # 设置报警收件人邮箱
  }
  notification_email_from ka@localhost    # 设置发件人
  smtp_server 127.0.0.1                # 定义邮件服务器
  smtp_connect_timeout 30
  router_id  lvs1                       # 设置路由ID号(实验需要修改)
}
vrrp_instance VI_1 {
  state MASTER                            # 主服务器为MASTER
  interface eth0                        # 定义网络接口
  virtual_router_id 50                    # 主辅VRID号必须一致
  priority 100                         # 服务器优先级
  advert_int 1
  authentication {
    auth_type pass
    auth_pass 1111                      # 主辅服务器密码必须一致
  }
  virtual_ipaddress {  # 配置VIP（实验需要修改）
  	192.168.4.15  
  }   
}

virtual_server 192.168.4.15 80 {           # 设置ipvsadm的VIP规则（实验需要修改）
  delay_loop 6
  lb_algo wrr                          # 设置LVS调度算法为WRR
  lb_kind DR                              # 设置LVS的模式为DR
  #persistence_timeout 50    
  #注意以上这条的作用是保持连接，开启后，客户端在一定时间内始终访问相同服务器
  
  protocol TCP
  real_server 192.168.4.100 80 {         # 设置后端web服务器真实IP（实验需要修改）
    weight 1                            # 设置权重为1
    TCP_CHECK {                           # 对后台real_server做健康检查
    connect_timeout 3	
    nb_get_retry 3   # 连接失败 重试3次
    delay_before_retry 3   # 每隔3秒 检查一次
    }
  }
 real_server 192.168.4.200 80 {       # 设置后端web服务器真实IP（实验需要修改）
    weight 2                           # 设置权重为2
    TCP_CHECK {
    connect_timeout 3
    nb_get_retry 3
    delay_before_retry 3
    }
  }
  real_server 192.168.200.2 1358 { # 另一种检查方式
        weight 1
        HTTP_GET {
            url {
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d # md5
            }
            url {
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}
systemctl start keepalived
ipvsadm -Ln                     # 查看LVS规则
ip a  s                          # 查看VIP配置
```



# 40 HAProxy负载平衡集群

```shell
# 部署HAProxy服务器
yum -y install haproxy

vim /etc/haproxy/haproxy.cfg
global
 log 127.0.0.1 local2   ###[err warning info debug]
 chroot /usr/local/haproxy
 pidfile /var/run/haproxy.pid ###haproxy的pid存放路径
 maxconn 40000     ###全部集群最大连接数，默认4000
 user haproxy
 group haproxy
 daemon       ###创建1个进程进入deamon模式（后台）运行
defaults
 mode http    ###默认的模式mode { tcp|http|health } 
 log global   ###采用全局定义的日志
 option dontlognull  ###不记录健康检查的日志信息
 option httpclose  ###每次请求完毕后主动关闭http通道
 option httplog   ###日志类别http日志格式
 option forwardfor  ###后端服务器可以从Http Header中获得客户端ip
 option redispatch  ###web服务器挂掉后强制定向到其他健康服务器
 timeout connect 10000 #如果backend没有指定，默认为10s
 timeout client 300000 ###客户端连接超时
 timeout server 300000 ###服务器连接超时
 maxconn  6000  ### 集群默认最大连接数
 retries  3   ###3次连接失败就认为服务不可用，也可以通过后面设置
 # 以下集群，可以用案例的格式
listen stats # stats 集群名
    bind 0.0.0.0:1080   #监听端口
    stats refresh 30s   #统计页面自动刷新时间
    stats uri /stats   #统计页面url
    stats realm Haproxy Manager #统计页面密码框上提示文本
    stats auth admin:admin  #统计页面用户名和密码设置
  #stats hide-version   #隐藏统计页面上HAProxy的版本信息
listen  websrv-rewrite 0.0.0.0:80 # websrv-rewrite 集群名
   balance roundrobin # rr = roundrobin
   server  web1 192.168.2.100:80 check inter 2000 rise 2 fall 5 
   # check inter 检查 毫秒 rise 成功次数 fall 失败次数
   server  web2 192.168.2.200:80 check inter 2000 rise 2 fall 5
   
systemctl start haproxy

# /etc/haproxy/configure	帮助文档

```



# 41 ceph

分布式文件系统：Distributed File System 文件系统管理的物理存储资源不一定直接连接在本地节点上，而是通过计算机网络与节点相连，基于B/S模式

常用分布式文件系统：Lustre Hadoop FastDFS Ceph GlusterFS

ceph 默认3个副本。提供对象存储，块存储，文件系统存储

每个node 需要一个固态盘，两个普通盘。固态盘分为两份，分别作为两个普通盘的缓存。

<font color=red>ceph只能做一个文件系统共享</font>

| 存储软件包（至少三台） | OSDs Object Storage Device，它的主要功能是存储数据、复制数据、平衡数据、恢复数据等，与其它OSD间进行心跳检查等 |
| ---------------------- | ------------------------------------------------------------ |
| 调度（至少三台）       | Monitors 监视器，负责监视Ceph集群，维护Ceph集群的健康状态，同时维护着Ceph集群中的各种Map图 |
| 文件系统服务支持       | MDSs Ceph MetaData Server，做文件系统。主要保存的文件系统服务的元数据，但对象存储和块存储设备是不需要使用该服务的。 |
| 对象存储               | raddosgw                                                     |
| 客户端                 | Ceph-coomon                                                  |

```shell
# 部署软件 需要执行的主机对其他主机有ssh的权限
yum -y install ceph-deploy
# 部署Ceph集群
mkdir ceph-cluster # ceph-deploy 必须在此目录中
cd ceph-cluster/
ceph-deploy new node1 node2 node3 # 创建三个节点 /etc/hosts 配置node对应的ip
ceph-deploy install node1 node2 node3 # 给所有节点安装软件包,节点已经配置好yum
ceph-deploy mon create-initial # 初始化所有节点的mon服务

ceph  -s # 查看集群状态

# 三个节点 创建OSD
parted  /dev/vdb  mklabel  gpt # 指定分区模式
parted  /dev/vdb  mkpart primary  1M  50% # 1M 到50% 一个区
parted  /dev/vdb  mkpart primary  50%  100% # 50% 到100% 一个区
chown  ceph.ceph  /dev/vdb1 # 临时设置
chown  ceph.ceph  /dev/vdb2

vim /etc/udev/rules.d/70-vdb.rules # 永久设置
ENV{DEVNAME}=="/dev/vdb1",OWNER="ceph",GROUP="ceph"
ENV{DEVNAME}=="/dev/vdb2",OWNER="ceph",GROUP="ceph"

# 初始化清空磁盘数据（仅在node1操作即可）
ceph-deploy disk  zap  node1:vdc   node1:vdd
ceph-deploy disk  zap  node2:vdc   node2:vdd
ceph-deploy disk  zap  node3:vdc   node3:vdd
# 创建OSD存储空间（仅在node1操作即可）
ceph-deploy osd create node1:vdc:/dev/vdb1 node1:vdd:/dev/vdb2  
# 创建osd存储设备，vdc为集群提供存储空间，vdb1提供JOURNAL缓存，
# 一个存储设备对应一个缓存设备，缓存需要SSD，不需要很大
ceph-deploy osd create node2:vdc:/dev/vdb1 node2:vdd:/dev/vdb2
ceph-deploy osd create node3:vdc:/dev/vdb1 node3:vdd:/dev/vdb2
```

## 创建Ceph块存储

块共享：需要挂载磁盘，分区，格式化

多个存储池，每个存储池多个镜像，镜像提供空间

```shell
# 查看存储池
ceph osd lspools # 默认有rbd 存储池

rbd create demo-image --image-feature  layering --size 10G # 创建镜像demo-image --image-feature layering 分层快照
rbd create rbd/image --image-feature  layering --size 10G # 指定池 rbd 创建 image

rbd list # 查看rbd池的镜像
rbd info demo-image 

# 缩小容量
rbd resize --size 7G image --allow-shrink

# 扩容容量
rbd resize --size 15G image
```

```shell
#集群内将镜像映射为本地磁盘
rbd map demo-image # 挂载 默认/dev/rdb
rbd showmapped # 查询挂载
rbd unmap demo-image # 卸载
#客户端需要安装ceph-common软件包
#拷贝配置文件（否则不知道集群在哪）
#拷贝连接密钥（否则无连接权限）
yum -y  install ceph-common
scp 192.168.4.11:/etc/ceph/ceph.conf  /etc/ceph/
scp 192.168.4.11:/etc/ceph/ceph.client.admin.keyring /etc/ceph/
# 然后查看镜像池、挂载
```

### 创建镜像快照

```shell
# 查看镜像快照 snapshot
rbd snap ls image
# 创建镜像快照，快照是对某一刻数据备份
rbd snap create image --snap image-snap1 # 对 image 镜像创建快照 名字是 image-snap1
# 还原快照
rbd snap rollback image --snap image-snap1
```

### 创建快照克隆

```shell
rbd snap protect image --snap image-snap1 # 保护镜像的快照 unprotect 取消保护
rbd clone image --snap image-snap1 image-clone --image-feature layering # 使用image的快照image-snap1克隆一个新的image-clone镜像
#克隆镜像很多数据都来自于快照链
#如果希望克隆镜像可以独立工作，就需要将父快照中的数据，全部拷贝一份，但比较耗时！！！
rbd flatten image-clone
```



## Ceph文件系统

文件系统例如：NFS，ISCSI，Samba等

```shell
# 部署元数据服务器
yum -y install ceph-mds
# 登陆node1部署节点操作
cd  /root/ceph-cluster
ceph-deploy mds create node4
# 同步配置文件和key
ceph-deploy admin node4
#node3 创建存储池，两个 一个用于inode，一个用于block
ceph osd pool create cephfs_data 128 # 创建存储池，分128组，必须是2的幂次
ceph osd pool create cephfs_metadata 128
ceph mds stat   # 查看mds状态
# 创建Ceph文件系统
ceph fs new myfs1 cephfs_metadata cephfs_data
# myfs1 文件系统名 cephfs_metadata 存inode cephfs_data 存block
ceph fs ls # 查看
# 客户端挂载
mount -t ceph 192.168.4.11:6789:/  /mnt/cephfs/ -o name=admin,secret=AQBTsdRapUxBKRAANXtteNUyoEmQHveb75bISg==
# 注意:文件系统类型为ceph
# 192.168.4.11为MON节点的IP（是管理包所在，不是MDS节点）
# admin是用户名,secret是密钥
# 密钥可以在/etc/ceph/ceph.client.admin.keyring中找到
```



## Ceph对象存储服务器



```shell
# 部署RGW软件包
ceph-deploy install --rgw node5
# 同步配置文件与密钥到node5
cd /root/ceph-cluster
ceph-deploy admin node5
# 启动一个rgw服务
ceph-deploy rgw create node5
# 修改服务端口 登陆node5，RGW默认服务端口为7480，修改为8000或80更方便客户端记忆和使用
vim  /etc/ceph/ceph.conf
[client.rgw.node5]
host = node5
rgw_frontends = "civetweb port=8000"
# node5为主机名
# civetweb是RGW内置的一个web服务

```



# 50 KVM 使用

虚拟机构成：

- 磁盘镜像文件 /var/lib/libvirt/images/文件.qcow2
- 硬件描述文件 /etc/libvirt/qemu/文件.xml

## 附件 

[从单机到2000万QPS并发的Redis高性能缓存实践之路](https://www.douban.com/group/topic/124428411/)
