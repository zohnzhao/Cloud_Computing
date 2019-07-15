# Zabbix

实现企业级的开源分布式监控,通过采集监控数据,通过B/S模式实现Web管理

1. 部署LNMP环境
2. 安装Zabbix

```shell
# 监控机
yum -y install  net-snmp-devel curl-devel libevent-devel # 安装相关依赖包
tar -xf zabbix-3.4.4.tar.gz 
cd zabbix-3.4.4/
./configure --enable-server --enable-agent --enable-proxy --with-mysql=/usr/bin/mysql_config --with-net-snmp --with-libcurl
# --enable-server安装部署zabbix服务器端软件
# --enable-agent安装部署zabbix被监控端软件
# --enable-proxy安装部署zabbix代理相关软件
# --with-mysql配置mysql_config路径
# --with-net-snmp允许zabbix通过snmp协议监控其他设备
# --with-libcurl安装相关curl库文件，这样zabbix就可以通过curl连接http等服务，测试被监控主机服务的状态
make install

ls /usr/local/bin/     # 命令
ls /usr/local/sbin/    # 启动服务
ls /usr/local/etc/     # 查看配置文件

# 初始化Zabbix，创建数据库，上线Zabbix的Web页面
mysql
mysql> create database zabbix character set utf8; # 创建数据库，支持中文字符集
mysql> grant all on zabbix.* to zabbix@'localhost' identified by 'zabbix'; # 创建可以访问数据库的账户与密码
cd zabbix-3.4.4/database/mysql/
mysql -uzabbix -pzabbix zabbix < schema.sql
mysql -uzabbix -pzabbix zabbix < images.sql
mysql -uzabbix -pzabbix zabbix < data.sql

cp -a  zabbix-3.4.4/frontends/php/* /usr/local/nginx/html/ # cp -a 在保留原文件属性的前提下复制文件
chmod -R 777 /usr/local/nginx/html/*
useradd -s /sbin/nologin zabbix # 不创建用户无法启动服务


vim /usr/local/etc/zabbix_server.conf
ListenPort=10051 # 默认该行被注释
DBHost=localhost # 数据库主机，默认该行被注释
DBName=zabbix # 设置数据库名称
DBUser=zabbix # 设置数据库账户
DBPassword=zabbix # 设置数据库密码，默认该行被注释
LogFile=/var/log/zabbix/zabbix_server.log    # 设置日志，仅查看以下即可

chown zabbix:zabbix /var/log/zabbix/
zabbix_server        # 启动服务
```

如果是因为配置文件不对，导致服务无法启动时，不要重复执行zabbix_server，

一定要先使用killall zabbix_server关闭服务后，再重新启动一次。

修改Zabbix_agent配置文件，启动Zabbix_agent服务

```shell
# 监控机的监控代理，自身的监控客户端
vim /usr/local/etc/zabbix_agentd.conf
Server=127.0.0.1,192.168.2.5                    # 允许哪些主机监控本机
ServerActive=127.0.0.1,192.168.2.5              # 允许哪些主机通过主动模式监控本机
Hostname=zabbix_server                          # 设置本机主机名
LogFile=/var/log/zabbix/zabbix_server.log       # 设置日志文件
UnsafeUserParameters=1                          # 是否允许自定义监控命令

zabbix_agentd                       # 启动监控agent
ss -ntulp |grep zabbix_agentd       # 查看端口信息为10050

yum -y install  php-gd php-xml php-bcmath php-mbstring

vim /etc/php.ini
date.timezone = Asia/Shanghai               # 设置时区
max_execution_time = 300                    # 最大执行时间，秒
max_input_time = 300                        # 服务器接收数据的时间限制
post_max_size = 32M                         # POST数据最大容量
memory_limit = 128M                         # 内存容量限制，默认值

systemctl  restart php-fpm

firefox http://192.168.2.5/index.php # 进行初始化配置

cat /usr/local/nginx/html/conf/zabbix.conf.php
```

注意：这里有一个PHP LDAP是warning状态是没有问题的！

```shell
# 监控节点
yum -y install gcc pcre-devel
useradd -s /sbin/nologin  zabbix
tar -xf zabbix-3.4.4.tar.gz 
cd zabbix-3.4.4/
./configure --enable-agent
make && make install

vim /usr/local/etc/zabbix_agentd.conf
Server=127.0.0.1,192.168.2.5                # 谁可以监控本机（被动监控模式）
ServerActive=127.0.0.1,192.168.2.5          # 谁可以监控本机（主动监控模式）
Hostname=zabbixclient_web1                  # 被监控端自己的主机名
EnableRemoteCommands=1 # 监控异常后，是否允许服务器远程过来执行命令，如重启某个服务
UnsafeUserParameters=1                      # 是否允许自定义key监控

zabbix_agentd                # 启动agent服务

# 在网站上添加监控节点
```

## 自定义监控项

在客户端编写监控命名给监控服务器调用

配置客户端：

- 启用自定义监控项
- 编写监控命令
- 重启zabbix_agentd服务
- 测试自定义监控项

配置监控服务器：

- 创建新的监控模板
- 创建应用集
- 创建监控项目，并关联命令
- 监控客户端时调用新创建的监控模板
- 查看监控信息

```shell
# 监控节点（客户端）
vim /usr/local/etc/zabbix_agentd.conf
Server=127.0.0.1,192.168.2.5                # 谁可以监控本机（被动监控模式）
ServerActive=127.0.0.1,192.168.2.5          # 谁可以监控本机（主动监控模式）
UnsafeUserParameters=1
Include=/usr/local/etc/zabbix_agentd.conf.d/*.conf         # 加载配置文件目录

vim /usr/local/etc/zabbix_agentd.conf.d/count_user.conf
UserParameter=count_user,wc -l /etc/passwd | awk '{print $1}'
# 自定义key语法格式:
# UserParameter=自定义key名称,命令

killall  zabbix_agentd
zabbix_agentd
# 验证自定义命令
zabbix_get -s 127.0.0.1 -p 10050 -k count_user
```

## 定义报警

自动报警：配置触发器，配置报警命令

邮件报警

触发器表达式的格式：{主机:命令(参数)}<表达式>常数

```shell
# 搭配邮件服务
yum -y install postfix mailx
mail -s 'hello' zabbix  # 发送邮件
asdasd
.

mail -u zabbix  # 查看邮件
```



## 自动发现

创建自动发现规则

创建Action动作，说明发现主机后自动执行什么动作

通过动作，执行添加主机，链接模板到主机等操作



## 主动监控

```shell
# 监控节点
yum -y install gcc pcre-devel
useradd -s /sbin/nologin  zabbix
tar -xf zabbix-3.4.4.tar.gz 
cd zabbix-3.4.4/
./configure --enable-agent
make && make install

vim /usr/local/etc/zabbix_agentd.conf 
#Server=127.0.0.1,192.168.2.5
# 注释该行，允许谁监控本机
StartAgents=0            
# 被动监控时启动多个进程
# 设置为0，则禁止被动监控，不启动zabbix_agentd服务
ServerActive=192.168.2.5
# 允许哪些主机监控本机（主动模式），一定要取消127.0.0.1
Hostname=zabbixclient_web2
# 告诉监控服务器，是谁发的数据信息
# 一定要和zabbix服务器配置的监控主机名称一致（后面设置）
RefreshActiveChecks=120
# 默认120秒检测一次
UnsafeUserParameters=1            
# 允许自定义key
Include=/usr/local/etc/zabbix_agentd.conf.d/
```

克隆模板，更改监控项模式Zabbix Agent（Active 主动模式），修改后有部分不能变为主动模式，将其关闭。

添加主机，ip 0.0.0.0 端口 0



## 拓扑图与聚合图形

拓扑图图表说明

- Icon（图标），添加新的设备后可以点击图标修改属性
- Shape（形状）
- Link（连线），先选择两个图标，再选择连线
- 完成后，点击Update（更新）



## 自定义监控案例

```shell
# UserParameter=key[*],<command> key里的所有参数，都会传递给后面命令的位置变量
vim /usr/local/etc/zabbix_agentd.conf.d/nginx.status.conf
UserParameter=nginx.status[*],/usr/local/bin/nginx_status.sh $1

vim /usr/local/bin/nginx_status.sh
#!/bin/bash
case $1 in
active)
    curl -s http://192.168.2.100/status |awk '/Active/{print $NF}';;
waiting)
    curl -s http://192.168.2.100/status |awk '/Waiting/{print $NF}';;
accepts)
    curl -s http://192.168.2.100/status |awk 'NR==3{print $2}';;
esac

chmod +x  /usr/local/bin/nginx_status.sh

zabbix_get  -s 127.0.0.1 -k 'nginx.status[accepts]'
```

监控mysql并发连接数，监控mysql慢查询结果。

NoSQL数据库状态，php服务状态，toamcat服务器状态。

硬件设备（通过SNMP协议）



第四阶段文档

git clone git://43.254.90.134/nsd1902.git



# KVM

集群n个应用虚拟成一个应用

KVM 是 linux 内核的模块

QEMU 硬件仿真工具

Libvirt 虚拟化管理的接口和工具

必备软件

| qemu-kvm                   | 为kvm提供底层仿真支持          |
| -------------------------- | ------------------------------ |
| Libvirt-daemon             | libvirtd守护进程，管理虚拟机   |
| Libvirt-client             | 用户端软件，提供客户端管理命令 |
| Libvirt-daemon-driver-qemu | libvirtd 连接qemu的驱动        |

功能

| virt-install | 系统安装工具   |
| ------------ | -------------- |
| virt-manager | 图形管理工具   |
| virt-v2v     | 虚拟机迁移工具 |
| virt-p2v     | 物理机迁移工具 |

```shell
yum -y install qemu-kvm libvirt-daemon libvirt-client libvirt-daemon-driver-qemu
systemctl start libvirtd
```

虚拟机组成

- 内核虚拟化模块（KVM）
- 系统设备仿真（QEMU）
- 虚拟机管理程序（LIBVIRT）
- 一个XML文件（虚拟机配置声明文件）
- 位置/etc/libvirt/qemu/
- 一个磁盘镜像文件（虚拟机的硬盘）
- 位置/var/lib/libvirt/images/

virsh命令工具介绍

格式:virsh 控制指令 [虚拟机名称] [参数]

## virsh 虚拟机管理

| list [--all]                                      | 列出虚拟机                     |
| ------------------------------------------------- | ------------------------------ |
| start\|shutdown\|reboot                           | 虚拟机启动,停止,重启           |
| destroy                                           | 强制停止虚拟机                 |
| define\|undefine                                  | 根据 xml 文件 创建/删除 虚拟机 |
| console                                           | 连接虚拟机的 console           |
| edit                                              | 修改虚拟机的配置               |
| autostart [--disable]                             | 设置虚拟机自启动               |
| domiflist                                         | 查看虚拟机网卡信息             |
| domblklist                                        | 查看虚拟机硬盘信息             |
| blockresize --path [绝对路径] --size 50G 虚拟机名 | 虚拟机硬盘扩容                 |

```shell
# dhcp获取ip
DEVICE="eth0"
ONBOOT="yes"
NM_CONTROLLED="no"
TYPE="Ethernet"
BOOTPROTO="dhcp"
PERSISTENT_DHCLIENT="yes"

# 静态ip
# Generated by dracut initrd
DEVICE="eth0"
ONBOOT="yes"
NM_CONTROLLED="no"
TYPE="Ethernet"
BOOTPROTO="static"
IPADDR="192.168.1.130"
NETMASK="255.255.255.0"
GATEWAY="192.168.1.254"
```

### 磁盘镜像文件格式

| 特点/类型  | RAW    | QCOW2 |
| ---------- | ------ | ----- |
| KVM默认    | 否     | 是    |
| I/O效率    | 高     | 较高  |
| 占用空间   | 大     | 小    |
| 压缩       | 不支持 | 支持  |
| 后段盘复用 | 不支持 | 支持  |
| 快照       | 不支持 | 支持  |

qemu-img 命令格式
qemu-img 命令 参数 块文件名称 大小

常用的命令

| create  | 创建一个磁盘 |
| ------- | ------------ |
| convert | 转换磁盘格式 |
| info    | 查看磁盘信息 |
| resize  | 扩容磁盘空间 |

```shell
# 创建新的镜像盘文件
# qemu-img create -f 格式 磁盘路径 大小
qemu-img create -f qcow2 disk.img 50G

# 查询镜像盘文件的信息
# qemu-img info 磁盘路径
qemu-img info disk.img

# -b 使用后端模板文件
qemu-img create -b disk.img -f qcow2 disk1.img
```

COW技术原理

Copy On Write,写时复制

- 直接映射原始盘的数据内容
- 当数据有修改要求时,在修改之前自动将旧数据拷贝存入前端盘后,对前端盘进行修改
- 原始盘始终是只读的



```shell
# 根分区扩容
LANG=C

# 扩容第一个分区
growpart /dev/vad 1

# 查看文件系统
blkid

# 扩容文件系统
xfs_growfs /

df -h 
```



## 创建虚拟机

```shell
cp /var/lib/libvirt/images/.node_base.xml /etc/libvirt/qemu/002.xml

cd /var/lib/libvirt/images
qemu-img create -f qcow2 -b .node_base.qcow2 002.img 30G

# 配置 名称，内存，cpu，硬盘,网卡
vim /etc/libvirt/qemu/002.xml
...
	<name>002</name>
  <memory unit='KB'>1524000</memory>
  <currentMemory unit='KB'>1524000</currentMemory>
  <vcpu placement='static'>2</vcpu>
...
 <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/var/lib/libvirt/images/002.img'/>
      <target dev='vda' bus='virtio'/>
    </disk>
    <interface type='bridge'>
      <source bridge='vbr'/>
      <model type='virtio'/>
    </interface>
...

# 创建虚拟机
virsh define /etc/libvirt/qemu/002.xml
```

## 网卡及配置文件

```shell
网络配置文件说明
vim /etc/sysconfig/network-scripts/ifcfg-eth0
# Generated by dracut initrd 注释
DEVICE="eth0"     			  # 驱动名称,与ifconfig 看到的名称一致
ONBOOT="yes"  				 	  # 开机启动
NM_CONTROLLED="no"        # 不接受 NetworkManager 控制
TYPE="Ethernet"           # 类型
BOOTPROTO="static"        # 协议(dhcp|static|none)
IPADDR="192.168.1.10"     # IP地址
NETMASK="255.255.255.0"   # 子网掩码
GATEWAY="192.168.1.254"   # 默认网关
```

## 虚拟交换机

```shell
virsh net-edit vbr
<network>
  <name>vbr</name>
  <uuid>bc8c51cf-34d7-4fa4-bde9-f55b7c025421</uuid>
  <forward mode='nat'/>                                # 可转发上网
  <bridge name='vbr' stp='on' delay='0'/>
  <mac address='52:54:00:35:6e:56'/>
  <domain name='localhost' localOnly='no'/>
  <ip address='192.169.1.254' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.169.1.100' end='192.169.1.200'/>
    </dhcp>
  </ip>
</network>
```



# 什么是云计算

- 基于互联网的相关服务的增加、使用和交付模式
- 这种模式提供可用的、便捷的、按需的网络访问,进入可配置的计算资源共享池
- 这些资源能够被快速提供,只需投入很少的管理工作,或与服务供应商进行很少的交互
- 通常涉及通过互联网来提供动态易扩展且经常是虚拟化的资源

## IaaS

IaaS(Infrastructure as a Service),即基础设施即服务

提供给消费者的服务是对所有计算基础设施的利用,包括处理CPU、内存、存储、网络和其它基本的计算资源,用户能够部署和运行任意软件,包括操作系统和应用程序

IaaS通常分为三种用法:公有云、私有云和混合云

## PaaS

PaaS (Platform-as-a-Service),意思是平台即服务

以服务器平台或者开发环境作为服务进行提供就成为了PaaS

PaaS运营商所需提供的服务,不仅仅是单纯的基础平台,还针对该平台的技术支持服务,甚至针对该平台而进行的应用系统开发、优化等服务

简单地说,PaaS平台是指云环境中的应用基础设施服务,也可以说是中间件即服务

## SaaS

SaaS(Software-as-a-Service)软件即服务,是一种通过Internet提供软件的模式,厂商将应用软件统一部署在自己的服务器上,客户可以根据自己实际需求,通过互联网向厂商定购所需的应用软件服务

用户不用再购买软件,而是向提供商租用基于Web的软件,来管理企业经营活动,不用对软件进行维护,提供商会全权管理和维护软件,同时也提供软件的离线操作和本地数据存储



# Openstack

批量管理虚拟机

Openstark是IaaS云的解决方案

## Openstack主要组件

![](openstark.png)

Horizon

- 用于管理Openstack各种服务的、基于web的管理接口
- 通过图形界面实现创建用户、管理网络、启动实例等操作

Keystone

- 为其他服务提供认证和授权的集中身份管理服务
- 也提供了集中的目录服务
- 支持多种身份认证模式,如密码认证、令牌认证、以及AWS(亚马逊Web服务)登陆
- 为用户和其他服务提供了SSO认证服务

Neutron

-  一种软件定义网络服务
- 用于创建网络、子网、路由器、管理浮动IP地址
- 可以实现虚拟交换机、虚拟路由器
- 可用于在项目中创建VPN

Cinder

- 为虚拟机管理存储卷的服务
-  为运行在Nova中的实例提供永久的块存储
- 可以通过快照进行数据备份
- 经常应用在实例存储环境中,如数据库文件

swift

- 对象文件系统

Glance

- 扮演虚拟机镜像注册的角色
- 允许用户为直接存储拷贝服务器镜像
- 这些镜像可以用于新建虚拟机的模板

Nova

- 在节点上用于管理虚拟机的服务
- Nova是一个分布式的服务,能够与Keystone交互实现
- 认证,与Glance交互实现镜像管理
- Nova被设计成在标准硬件上能够进行水平扩展
- 启动实例时,如果有则需要下载镜像



## Openstack部署安装环境

保证节点能互相解析，配置/etc/hosts

保证时间同步，配置 /etc/chrony.conf,chronyc sources -v # 查看同步



所有主机配置yum仓库
CentOS7-1804.iso 系统软件
RHEL7-extras.iso
提供 Python 依赖软件包
RHEL7OSP-10.iso 光盘拥有众多目录,每个目录都是一个软件仓库,我们配置其中2个软件仓库
openstack 主要软件仓库
rhel-7-server-openstack-10-rpms
packstack 软件仓库
rhel-7-server-openstack-10-devtools-rpms



所有主机安装额外软件包
安装openstack期间,有些软件包所依赖的软件包,并没有在安装过程中安装,这些软件包需提前安装

- qemu-kvm
- ibvirt-daemon
- libvirt-daemon-driver-qemu
- libvirt-client
- python-setuptools

 软件包安装

```shell
yum install -y qemu-kvm libvirt-client libvirt-daemon libvirt-daemon-driver-qemu python-setuptools
```



检查cpu指令集

```shell
# vmx 支持虚拟化
grep vmx /proc/cpuinfo
grep ssse3 /proc/cpuinfo

# 如果没有，真机 
vim /etc/modprobe.d/kvm.conf
options kvm-intel nested=1
```



==检查基础环境==
• 是否卸载firewalld 和 NetworkManager
• 检查配置主机网络参数(静态IP)
• 主机名必须能够相互 ping 通
• 检查配置主机yum源(4个,10670)
• 依赖软件包是否安装
• 检查NTP服务器是否可用
• 检查 /etc/resolv.conf 不能有 search 开头的行



## 安装openstack，必须完成基础环境

### packstack配置openstack主机

```shell
# 安装 openstack 需要使用 packstack
# 首先安装 openstack-packstack
yum install -y openstack-packstack
# 使用 packstack 创建通用应答文件，ini 有颜色
packstack --gen-answer-file=answer.ini

# 修改应答文件
vim answer.ini
42: CONFIG_SWIFT_INSTALL=n             # swift 对象存储关闭
45: CONFIG_CEILOMETER_INSTALL=n         # 计费有关
49: CONFIG_AODH_INSTALL=n                # 计费有关
53: CONFIG_GNOCCHI_INSTALL=n               # 计费有关
75: CONFIG_NTP_SERVERS=192.168.1.254       
98: CONFIG_COMPUTE_HOSTS=192.168.1.11        # 在哪台主机上安装nova，所有节点
102: CONFIG_NETWORK_HOSTS=192.168.1.10,192.168.1.11   # 安装nova网络，所有主机
333: CONFIG_KEYSTONE_ADMIN_PW=a                       # openstack的管理员密码
840: CONFIG_NEUTRON_ML2_TYPE_DRIVERS=flat,vxlan        # 支持的网络协议，vxlan 内部通信
910: CONFIG_NEUTRON_OVS_BRIDGE_MAPPINGS=physnet1:br-ex  # 虚拟三层交换机，名字 br-ex
921: CONFIG_NEUTRON_OVS_BRIDGE_IFACES=br-ex:eth0        # 定义交换机 wan口，连接外网
1179: CONFIG_PROVISION_DEMO=n           # 不安装例子

# 如果前期环境准备无误,只要耐心等待安装结束即可
# 根据主机配置不同,安装过程需要20分钟左右或更久
# 如果出现错误,根据屏幕上给出的日志文件进行排错
packstack --answer-file=answer.ini
```

网络配置，自动完成

```shell
# br-ex为外部OVS网桥,查看外部OVS网桥
cat /etc/sysconfig/network-scripts/ifcfg-br-ex
# eth0为外部OVS网桥的端口,查看外部OVS网桥端口
cat /etc/sysconfig/network-scripts/ifcfg-eth0
# 验证OVS配置
ovs-vsctl show
```

安装虽然没有报错,但默认无法打开 Horizon,这是一个软件的配置 BUG

```shell
vim /etc/httpd/conf.d/15-horizon_vhost.conf
WSGIApplicationGroup %{GLOBAL}
ServerAlias localhost
WSGIDaemonProcess apache group=apache processes=3
threads=10 user=apache
WSGIProcessGroup apache
	WSGIApplicationGroup %{GLOBAL} <--- 这里添加
# 重新载入配置文件 apachectl graceful
```



## 项目管理

基本概念

- 项目:一组隔离的资源和对象。由一组关联的用户进行管理
- 在旧版本里,也用租户(tenant)来表示
- 根据配置的需求,项目对应一个组织、一个公司或是一个使用客户等
- 项目中可以有多个用户,项目中的用户可以在该项目创建、管理虚拟资源
- 具有admin角色的用户可以创建项目
- 项目相关信息保存到MariaDB中

缺省情况下,packstack安装的openstack中有两个独立的项目

- admin:为admin账户创建的项目
- services:与安装的各个服务相关联

通过Horizon可以进行项目的创建和删除

```shell
source keystonerc_admin # 认证管理员
# 创建名为myproject项目
openstack project create myproject
# 列出所有项目
openstack project list
# 查看myproject详细信息
openstack project show myproject
# 禁用与激活项目
openstack project set --disable myproject
openstack project set --enable myproject
# 查看项目配额
nova quota-show --tenant myproject
# 更新可用vcpu数目为30
nova quota-update --cores 30 myproject
# 删除myproject
openstack project delete myproject
```

### 管理用户

基本概念
• 用户在openstack中用于身份认证
• 管理员用户admin一般在packstack安装过程中创建
• 其他用户由管理员用户创建,并指定可以访问的项目
• 非管理员用户创建后,保存到MariaDB中

非管理员用户具有以下权限
– 启动实例知
– 创建卷和快照
– 创建镜像
– 分配浮动IP
– 创建网络和路由器
– 创建防火墙以及规则、规则策略
– 查看网络拓扑、项目使用概况等

通过命令行管理用户

```shell
source keystonerc_admin # 认证管理员,必须
# 创建user2用户,指定密码为qlst
openstack user create -- password qlst user2
# 设置user2的email地址
openstack user set --email user2@qlst.top user2
# 列出所有用户
openstack user list
# 查看user2信息
openstack user show user2
# 指定user2可以访问myproject,角色为_member_
openstack role add --user user2 --project myproject _member_
# 查看user2在myproject中的角色
openstack role list --project myproject --user user2
# 禁用用户
openstack user set --disable user2
# 激活用户
openstack user set --enable user2
# 修改user2的密码为redhat
openstack user set --password redhat user2
# 将user2从myproject中移除
openstack role remove -- project myproject --user user2 _member_
# 删除user2用户
openstack user delete user2
```

### 配额基础

• 管理员可以通过配额限制,防止资源的过度使用
• 配额基本项目,限制每个项目可以使用多少资源
• 这些操作上的功能限制,赋予了管理员对每个项目的精准控制

### 资源参数

• 安全组规则:指定每个项目可用的规则数

• 核心:指定每个项可用的VCPU核心数

• 固定IP地址:指定每个项目可用的固定IP数

• 浮动IP地址:指定每个项目可用的浮动IP数

• 注入文件大小 :指定每个项目内容大小

• 注入文件路径:指定每个项目注入的文件路径长度

• 注入文件:指定每个项目允许注入的文件数目

• 实例:指定每个项目可创建的虚拟机实例数目

• 密钥对:指定每个项可创建的密钥数

• 元数据:指定每个项目可用的元数据数目

• 内存:指定每个项目可用的最大内存

• 安全组:指定每个项目可创建的安全组数目

### 云主机类型

基本概念
• 云主机类型就是资源的模板
• 它定义了一台云主机可以使用的资源,如内存大小、磁盘容量和CPU核心数等
• Openstack提供了几个默认的云主机类型
• 管理员还可以自定义云主机类型

云主机类型参数
• Name:云主机类型名称
• ID:云主机类型ID,系统自动生成一个UUID
• VCPUs:虚拟CPU数目
• RAM(MB):内存大小
• Root disk(GB):外围磁盘大小。如果希望使用本地磁盘,设置为0
• 临时磁盘:第二个外围磁盘
• swap磁盘:交换磁盘大小

### 镜像基础

基本概念
• 在红帽Openstack平台中,镜像指的是虚拟磁盘文件,磁盘文件中应该已经安装了可启动的操作系统
• 镜像管理功能由Glance服务提供
• 它形成了创建虚拟机实例最底层的块结构
• 镜像可以由用户上传,也可以通过红帽官方站点下载

Glance磁盘格式
• raw:非结构化磁盘镜像格式
• vhd:VMware、Xen、Microsoft、VirtualBox等均支持的通用磁盘格式
• vmdk:是Vmware的虚拟磁盘格式
• vdi:VirtualBox虚拟机和QEMU支持磁盘格式
• iso:光盘数据内容的归档格式
• qcow2:QEMU支持的磁盘格式。空间自动扩展,并支持写时复制copy-on-write

镜像服务
• 镜像服务提供了服务器镜像的拷贝、快照功能,可以作为模板快速建立、起动服务器
• 镜像服务维护了镜像的一致性
• 当上传镜像时,容器格式必须指定
• 容器格式指示磁盘文件格式是否包含了虚拟机元数据

镜像容器格式
• bare:镜像中没有容器或元数据封装
• ovf:一种开源的文件规范,描述了一个开源、安全、有效、可拓展的便携式虚拟打包以及软件分布格式
• ova:OVA归档文件
• aki:亚马逊内核镜像
• ami:亚马逊主机镜像

### 网络管理

Openstack网络工作原理
• 实例被分配到子网中,以实现网络连通性
• 每个项目可以有一到多个子网
• 在红帽的Openstack平台中,OpenStack网络服务是缺省的网络选项,Nova网络服务作为备用
• 管理员能够配置丰富的网络,将其他Openstack服务连接到这些网络的接口上
• 每个项目都能拥有多个私有网络,各个项目的私有网络互相不受干扰

网络类型
• 项目网络:由Neutron提供的项目内部网络,网络间可用VLAN隔离
• 外部网络:可以让虚拟机接入外部网络,但需要配置浮动IP地址
• 提供商网络:将实例连接到现有网络,实现虚拟机实例与外部系统共享同一二层网络

### 浮动IP地址

浮动IP地址的作用
• 浮动 IP 一般是花钱购买的
• 浮动IP地址用于从外界访问虚拟机实例
• 浮动IP地址只能从现有浮动IP地址池中分配
• 创建外部网络时,浮动IP地址池被定义
• 虚拟机实例起动后,可以为其关联一个浮动IP地址
• 虚拟机实例也可以解除IP地址绑定

### 安全管理

安全组
• 安全组用于控制对虚拟机实例的访问
• 安全组在高层定义了哪些网络及哪些协议是被授权可以访问虚拟机实例的
• 每个项目都可以定义自己的安全组
• 项目成员可以编辑默认的安全规则,也可以添加新的安全规则
• 所有的项目都有一个默认的default安全组

安全组规则
• 安全组规则定义了如何处理网络访问
• 规则基于网络或协议定义
• 每个规则都有出和入两个方向
• 规则也可以指定ip协议版本
• 默认的安全组规则,允许虚拟机实例对外访问,但是阻止所有对虚拟机实例的访问

## 计算节点扩容

基础环境准备
• 扩容 openstack 计算节点
– 参考 nova01 配置过程
– 配置静态 ip:192.168.1.12,及主机名
– 保证与 openstack,和 nova01 能相互 ping 通
– 配置 /etc/resolv.conf ,删除 search 开头行
– 配置时间同步 /etc/chrony.conf
– 配置 yum 源,软仓库一共 4 个( 10670 )
– 安装依赖软件包 qemu-kvm libvirt-client libvirt-daemon libvirt-daemon-driver-qemu python-setuptools

修改应答文件
• openstack 计算节点采用远程安装
– 即 在 openstack 上执行指令,远程安装 nova02
– 远程安装使用 ssh 指令,从 openstack 上必须能够通过 ssh 命令 登录 nova02

在 openstack 上修改配置文件,添加新的ip

```shell
[root@openstack ~] vim answer.ini
98: CONFIG_COMPUTE_HOST=192.168.1.11,192.168.1.12
102:CONFIG_NETWORK_HOSTS=192.168.1.10,192.168.1.11,192.168.1.12
# 在openstack上重新执行安装命令
[root@openstack ~] packstack --answer-file answer.ini
# 本机已安装服务,不会被覆盖,只有改动后的选项才需要重新配置
# 安装后,apache 配置已被还原,需要重新添加
– WSGIApplicationGroup %{GLOBAL}
```

## 迁移云主机

• 有多个 nova 计算节点的时候,我们可以选择性的把某一个云主机从某台机器上迁移到另外一台机器上
• 迁移条件
– nova 计算节点与 openstack 管理节点都能相互 ping通,主机名称也要能 ping 通
– 所有计算节点安装 qemu-img-rhev,qemu-kvm-rhev
– 如未安装,在安装以后需要重启 libvirtd 服务

# Ansible

ansible优点

– 只需要SSH和Python即可使用
– 无客户端
– ansible功能强大,模块丰富
– 上手容易,门槛低
– 基于Python开发,做二次开发更容易
– 使用公司比较多,社区活跃

ansible特性

• 模块化设计,调用特定的模块完成特定任务
• 基于Python语言实现
– paramiko
– PyYAML (半结构化语言)
– Jinja2
• 其模块支持JSON等标准输出格式,可以采用任何编程语言重写

• 部署简单
• 主从模式工作
• 支持自定义模块
• 支持playbook
• 易于使用
• 支持多层部署
• 支持异构IT环境



对管理主机
– 要求Python 2.6 或Python 2.7
• ansible 使用以下模块,都需要安装
– paramiko
– PyYAML
– Jinja2
– httplib2
– six



对于被托管主机
– ansible默认通过SSH协议管理机器
– 被管理主机要开启ssh服务,允许ansible主机登录
– 在托管节点上也需要安装Python2.5或以上的版本
– 如果托管节点上开启了SElinux,需要安装libselinux-python



## 主机管理

==ansible配置文件查找顺序，检测到就停止查找
– 首先检测ANSIBLE_CONFIG变量定义的配置文件
– 其次检查当前目录下的 ./ansible.cfg 文件
– 再次检查当前用户家目录下 ~/ansible.cfg 文件
– 最后检查/etc/ansible/ansible.cfg文件
• /etc/ansible/ansible.cfg是ansible的默认配置文件路径==

```shell
# ansible.cfg 配置文件
# inventory 定义托管主机地址配置文件路径名
# inventory 指定的配置文件,写入远程主机的地址。
14:inventory = /etc/ansible/hosts

# ssh 主机 key 验证配置参数
61:host_key_checking = False
# 如果为 False,不需要输入 yes
# 如果为 True,等待输入 yes

vim /etc/ansible/hosts
[web]
web1
web2

[db]
db[1:2]

[other]
cache
```

ansible 命令

```shell
# 列出要执行的主机
ansible all --list-hosts

# 集合名或主机名
ansible web,cache --list-hosts

# 批量检测主机ssh连接,不是ping
ansible all -m ping -k # -k 交互输入密码，只能输入一次

# 批量获取主机负载
ansible web -a 'uptime'
```

 ansible 主机集合 -m 模块名称 -a 模块参数

– 主机集合 主机名或分组名,多个使用"逗号"分隔
– -m 模块名称,默认 command 模块
– -a or --args 模块参数

– -i inventory文件路径,或可执行脚本
– -k 使用交互式登录密码
– -e 定义变量
– -v 显示详细信息

inventory文件扩展参数

– ansible_ssh_port
– ssh端口号:如果不是默认的端口号,通过此变量设置

– ansible_ssh_user
– 默认的ssh用户名

– ansible_ssh_pass
– ssh密码(这种方式并不安全,我们强烈建议使用--ask-pass或SSH密钥)

– ansible_ssh_private_key_file
– ssh使用的私钥文件,适用于有多个密钥,而你不想使用
SSH代理的情况

• vars 变量定义,用于组名后面
– 例如
– [all:vars]
– ansible_ssh_private_key_file="/root/.ssh/key"

• children 子组定义,用于引用其他组名称
– 例如
– [app:children]
– web
– db

```shell
vim /etc/ansible/ansible.cfg
# 子组定义
[app:children]
web
db
# 变量引用
[other]
cache ansible_ssh_port=222
[all:vars]
ansible_ssh_private_key_file="/root/.ssh/key"
```

## 自定义配置文件

```shell
# 创建文件夹myansible
# 创建配置文件ansible.cfg
vim ansible.cfg
[defaults]
inventory = myhost
host_key_checking = False

# 配置主机文件
vim myhost
[app1]
web1
db1
```

## 批量配置管理

ansible-doc
– 模块的手册相当与shell的man,很重要
– ansible-doc -l 列出所有模块
– ansible-doc modulename 查看模块帮助



ping 模块
– 测试网络连通性, ping模块没有参数
– 注:测试ssh的连通性

```shell
ansible host-pattern -m ping
```



command 模块
– 默认模块,远程执行命令

```shell
# 用法
ansible host-pattern -m command -a '[命令]'
# 查看所有机器负载
ansible all -m command -a 'uptime'
# 查看日期和时间
ansible all -m command -a 'date +%F_%T'
```

– 该模块通过-a跟上要执行的命令可以直接执行,若命令里有如下字符则执行不成功
– "<" , ">" , "|" , "&"
– command 模块不能解析系统变量
– 该模块不启动shell直接在ssh进程中执行,所有使用到shell的命令执行都会失败



shell 模块
– shell 模块用法基本和command一样,区别是shell模
块是通过/bin/sh进行执行命令,可以执行任意命令
– 不能执行交互式的命令,例如 vim top 等

```shell
# 查看所有机器的负载
ansible all -m shell -a 'uptime'
```



script模块

```shell
# 在本地写脚本,然后使用script模块批量执行
ansible t1 -m script -a './us/x.sh'
```

– 注意:该脚本包含但不限于shell脚本,只要指定Shabang解释器的脚本都可运行



yum模块
– 使用yum包管理器来管理软件包
– name:要进行操作的软件包名字
– state: 动作(installed, removed)
– install == installed
– remove == removed

```shell
# 给所有db 主机安装 mariadb
ansible db -m yum -a 'name="mariadb-server" state=installed'
# cache 主机删除lrzsz
ansible cache -m yum -a 'name="lrzsz" state=removed'
```



service模块
– name:必选项,服务名称
– enabled:是否开机启动 yes|no
– sleep:执行restarted,会在stop和start之间沉睡几秒钟
– state:对当前服务执行启动,停止、重启、重新加载等操作(started,stopped,restarted,reloaded)

```shell
ansible t1 -m service -a 'name="sshd" enabled="yes" state="started"'
```



copy 模块
– 复制文件到远程主机
– src:复制本地文件到远程主机,绝对路径和相对路径都可,路径为目录时会递归复制。若路径以"/"结尾,只复制目录里的内容,若不以"/"结尾,则复制包含目录在内的整个内容,类似于rsync
– dest:必选项。远程主机的绝对路径,如果源文件是一个目录,那该路径必须是目录

backup:覆盖前先是否备份原文件,备份文件包含时间信息。有两个选项:yes|no
force:若目标主机包含该文件,但内容不同,如果设置为yes,则强制覆盖,设为no,则只有当目标主机的目标位置不存在该文件时才复制。默认为yes

```shell
# 复制文件
ansible all -m copy -a 'src=/etc/resolv.conf dest=/etc/resolv.conf'
# 复制目录
ansible all -m copy -a 'src=/etc/yum.repos.d/ dest=/etc/yum.repos.d/'
```



lineinfile模块
– 类似sed的一种行编辑替换模块
– path	目标文件文件
– regexp	正则表达式,要修改的行
– line	最终修改的结果

```shell
# 例如修改 my.cnf,中 bin-log 的格式 mixed --> row
ansible db -m lineinfile -a 'path="/etc/my.cnf" regexp="^binlog-format" line="binlog-format = row"'
```



replace模块
– 类似sed的一种行编辑替换模块
– path 目的文件
– regexp 正则表达式
– replace 替换后的结果

```shell
# 替换指定字符 row --> mixed
ansible db -m replace -a 'path="/etc/my.cnf" regexp="row" replace="mixed"'
```



setup模块
– 主要用于获取主机信息,playbooks里经常会用的另一个参数gather_facts与该模块相关,setup模块下经常用的是filter参数

```shell
# filter过滤所需信息
ansible cache -m setup -a 'filter=ansible_distribution'
```



## ansible七种武器

• 第一种武器
– ansible 命令,用于执行临时性的工作,必须掌握

• 第二种武器
– ansible-doc是ansible模块的文档说明,针对每个模块都有详细的说明及应用案例介绍,功能和Linux系统man命令类似,必须掌握

第三种武器
– ansible-console是ansible为用户提供的交互式工具,用户可以在ansible-console虚拟出来的终端上像Shell一样使用ansible内置的各种命令,这为习惯使用Shell交互方式的用户提供了良好的使用体验

• 第四种武器
– ansible-galaxy从github上下载管理Roles的一款工具,与python的pip类似

• 第五种武器
– ansible-playbook是日常应用中使用频率最高的命令,工作机制:通过读取先编写好的playbook文件实现批量管理,可以理解为按一定条件组成的ansible任务集,必须掌握
• 第六种武器
– ansible-vault主要用于配置文件加密,如编写的playbook文件中包含敏感

• 第七种武器
– ansible-pull

– ansible有两种工作模式pull/push ,默认使用push模式工作,pull和push工作模式机制刚好相反
– 适用场景:有大批量机器需要配置,即便使用高并发线程依旧要花费很多时间
– 通常在配置大批量机器的场景下使用,灵活性稍有欠缺,但效率几乎可以无限提升,对运维人员的技术水平和前瞻性规划有较高要求



# JSON 语法规则

– 数据在名称/值对中
– 数据由逗号分隔
– 大括号保存对象
– 中括号保存数组

JSON语法规则之数组

```json
{ "诗人":
	["李白", "杜甫", "白居易", "李贺"]
}
```

• 复合复杂类型

```json
{ "诗人":
  [ 
    {"李白":"诗仙", "年代":"唐"},
    {"杜甫":"诗圣", "年代":"唐"},
    {"白居易":"诗魔", "年代":"唐"},
    {"李贺":"诗鬼", "年代":"唐"}
  ]
}
```



# YAML简介

• YAML是什么
– 是一个可读性高,用来表达数据序列的格式

YAML基础语法
– YAML的结构通过空格来展示
– 数组使用"- "来表示，有空格
– 键值对使用": "来表示，有空格
– YAML使用一个固定的缩进风格表示数据层级结构关系
– 一般每个缩进级别由两个以上空格组成
– # 表示注释，有空格

高级复合表达式

```yaml
"诗人": 
  - 
    "李白": "诗仙"
    "年代": "唐"
  - 
    "杜甫": "诗圣"
    "年代": "唐“
  - 
    "白居易": "诗魔"
    "年代": "唐"
  - 
    "李贺": "诗鬼"
    "年代": "唐"
```



# Jinja2模版基本语法

– 模板的表达式都是包含在分隔符"{{    }}"内的
– 控制语句都是包含在分隔符"{%    %}"内的
– 模板支持注释,都是包含在分隔符"{#    #}" 内,支持块注释

```jinja2
{# 调用变量 #}
{{varname}}
{# 计算 #}
{{2+3}}
{# 判断 #}
{{1 in [1,2,3]}}

{# Jinja2模版控制语句 #}
{% if name == '诗仙' %}
  李白
{% elif name == '诗圣' %}
  杜甫
{% elif name == '诗魔' %}
  白居易
{% else %}
  李贺
{% endif %}

{# Jinja2模版控制语句 #}
{% if name == ... ... %}
  ... ...
{% elif name == '于谦' %}
  {% for method in [抽烟, 喝酒, 烫头] %}
    {{do method}}
  {% endfor %}
  ... ...
{% endif %}
```

Jinja2过滤器
– 变量可以通过过滤器修改。过滤器与变量用管道符号( | )分割,也可以用圆括号传递可选参数,多个过滤器可以链式调用,前一个过滤器的输出会被作为后一个过滤器的输入

```jinja2
{# 加密一个字符串 #}
{{ 'astr'|password_hash('sha512') }}
{# 在线文档 http://docs.jinkan.org/docs/jinja2/templates.html#builtin-filters #}
```



# playbook，必须运行debug

playbook是什么
– playbook是ansible用于配置,部署和管理托管主机剧本,通过playbook的详细描述,执行其中的一系列tasks,可以让远端主机达到预期状态
– 也可以说,playbook字面意思即剧本,现实中由演员按剧本表演,在ansible中由计算机进行安装,部署应用,提供对外服务,以及组织计算机处理各种各样的事情



为什么要使用playbook
– 执行一些简单的任务,使用ad-hoc命令可以方便的解决问题,但有时一个设施过于复杂时,执行ad-hoc命令是不合适的,最好使用playbook
– playbook可以反复使用编写的代码,可以放到不同的机器上面,像函数一样,最大化的利用代码,在使用ansible的过程中,处理的大部分操作都是在编写playbook



playbook语法格式
– playbook由YAML语言编写,遵循YAML标准
– 在同一行中#之后的内容表示注释
– 同一个列表中的元素应该保持相同的缩进
– playbook由一个或多个play组成
– play中hosts,variables,roles,tasks等对象的表示方法都是键值中间以": "分隔表示，有空格
– YAML还有一个小的怪癖,它的文件开始行都应该是 ---,这是YAML格式的一部分,表明一个文件的开始

– hosts: 定义将要执行playbook的远程主机组
– vars: 定义playbook运行时需要使用的变量
– tasks: 定义将要在远程主机上执行的任务列表
– handlers: 定义task执行完成以后需要调用的任务



playbook执行结果
• 使用ansible-playbook运行playbook文件,输出内容
为JSON格式,由不同颜色组成便于识别
– 绿色代表执行成功
– ***代表系统代表系统状态发生改变
– 红色代表执行失败

```shell
vim mybook.yaml
---
- hosts: all
  remote_user: root
  tasks:
    - ping:

# 执行脚本
ansible-playbook myping.yaml -f 5
# -f 并发进程数量,默认是5
# hosts行 内容是一个(多个)组或主机的patterns,以逗号为分隔符
# remote_user 账户名
# tasks
# 命令的集合
# 每一个play包含了一个task列表(任务列表)
# 一个task在其所对应的所有主机上(通过 host pattern 匹配的所有主机)执行完毕之后,下一个task才会执行
```

```shell
# 给web主机添加用户z3,设置默认密码123
# 使用 user 模块
vim adduser.yaml
---
- hosts: all
  remote_user: root
  tasks: 
    - user: 
        name: zz
    - name: change password # - name 注释起名用,接下来的命令模块不需要"- "
      shell: echo 123 | passwd --stdin zz
    
# 执行脚本
ansible-playbook adduser.yaml -f 5

cat /etc/login.defs  # 查看系统用户与组设置
```

设密码
– 解决密码明文问题
– user模块的password为什么不能设置密码呢
– 经过测试发现,password是把字符串直接写入shadow,并没有改变,而Linux的shadow密码是经过加密的,所以不能使用
• 解决方案
– 变量过滤器password_hash
{{ 'urpassword' | password_hash('sha512')}}

```shell
# 1. 安装Apache
# 2. 修改配置文件的监听端口为8080
# 3. 设置默认主页 hello world
# 4. 启动服务
# 5. 设置开机自启
vim apache.yaml
---
- hosts: all
  remote_user: root
  tasks:
    - yum:
       name: "httpd"
       state: "installed"
    - replace:
       path: "/etc/httpd/conf/httpd.conf"
       regexp: "Listen 80"
       replace: "Listen 8080"
    - shell: echo "hello world" > /var/www/html/index.html
    - service:
       name: "httpd"
       enabled: "yes"
       state: "started"
```



## 变量

```shell
# 添加用户，给 db 主机添加用户l4,设置默认密码123
---
- hosts: web
  remote_user: root
  vars:         # 定义默认变量
    username: zh
    pwd: 123aaa
  tasks:
    - name: create user "{{username}}"
      user:
        name: "{{username}}"
        password: "{{ pwd | password_hash('sha512') }}"
```

```shell
# 传递参数
# -e 参数
# 参数格式必须是 json 或 yaml
# yaml 格式 可以使用参数文件,例如
vim args.yml
---
username: zzz
pwd: 147aaa

# 执行脚本 -e 后使用文件时加@
ansible-playbook adduser.yaml -e @args.yml
```

## error

• ansible-playbook对错误的处理
– 默认情况判断$?,如果值不为0就停止执行
– 但某些情况我们需要忽略错误继续执行

```shell
---
- hosts: cache
  remote_user: root
  tasks:
    - shell: mkdir /tmp/cache
      ignore_errors: True
    - name: ReStart service httpd
      service:
        name: httpd
        state: restarted
```

错误处理方法
– ignore_errors: 对错误的处理方式
– True 表示忽略错误继续执行
– False 表示遇到错误就停止执行
– 默认 False



## tags

给指定的任务定义一个调用标识

```shell
vim apache.yaml
---
- hosts: all
  remote_user: root
  tasks:
    - yum:
         name: "httpd"
         state: "installed"
    - replace:
         path: "/etc/httpd/conf/httpd.conf"
         regexp: "Listen 80"
         replace: "Listen 8080"
    - shell: echo "hello world" > /var/www/html/index.html
      tags: default        # 给shell标记
    - service:
         name: "httpd"
         enabled: "yes"
         state: "started"

# 执行脚本 -e 后使用文件时加@
ansible-playbook apache.yaml -t default # 只执行shell
```



## handlers

• 当关注的资源发生变化时采取的操作
• notify这个action可用于在每个play的最后被触发,这样可以避免有多次改变发生时每次都执行指定的操作,取而代之仅在所有的变化发生完成后一次性地执行指定操作
• 在notify中列出的操作称为handler,即notify调用handler中定义的操作

```shell
vim apache.yaml
---
- hosts: all
  remote_user: root
  tasks:
    - yum:
         name: "httpd"
         state: "installed"
    - replace:
         path: "/etc/httpd/conf/httpd.conf"
         regexp: "Listen 80"
         replace: "Listen 8080"
    - shell: echo "hello world" > /var/www/html/index.html
      tags: default        # 给shell标记
      notify:    # 设置触发
          - restart_httpd  # handlers的name
  handlers:
    - name: restart_httpd
      service:
         name: "httpd"
         enabled: "yes"
         state: "restarted"

# 执行脚本 -e 后使用文件时加@
ansible-playbook apache.yaml -t default # 只执行shell，自动触发
```

## when

有些时候需要在满足特定的条件后再触发某一项操作,或在特定的条件下终止某个行为,这个时候需要进行条件判断,when正是解决这个问题的最佳选择,远程中的系统变量facts作为when的条件,可以通过setup模块查看。

```shell
---
- name: Install VIM
  hosts: all
  tasks:
    - name: Install VIM via yum
      yum: name=vim-enhanced state=installed
      when: ansible_os_family == "RedHat"
    - name: Install VIM via apt
      apt: name=vim state=installed
      when: ansible_os_family == "Debian"
```

## register

– 有时候我们还需要更复杂的例子,如判断前一个命令的执行结果去处理后面的操作,这时候就需要register模块来保存前一个命令的返回状态,在后面进行调用。

```shell
# 当系统负载超过一定值的时候做特殊处理
---
- hosts: web
  remote_user: root
  tasks:
    - shell: uptime |awk '{printf("%.2f\n",$(NF-2))}'
      register: result
    - shell: touch /tmp/isreboot
      when: result.stdout|float > 0.7
```

## with_items

• with_items是playbook标准循环,可以用于迭代一个列表或字典,通过{{ item }}获取每次迭代的值

```shell
# 例如创建多个用户
---
- hosts: web2
  remote_user: root
  tasks:
    - name: add users
      user: 
          group=: wheel
          password: {{'123456' |password_hash('sha512')}} 
          name: {{item}}
      with_items: 
         - nb
         - dd
         - plj
         - lx
```

```shell
# 为不同用户定义不同组
---
- hosts: web2
  remote_user: root
  tasks:
    - name: add users
      user: 
        group={{item.group}} 
        password={{'123456' | password_hash('sha512')}} 
        name={{item.name}}
      with_items:
        - 
          name: 'nb'
          group: 'root'
        - 
          name: 'dd'
          group: 'root'
        - 
          name: 'plj'
          group: 'wheel'
        - 
          name: 'lx'
          group: 'wheel'
```

## include and roles

在编写playbook的时候随着项目越来越大,playbook越来越复杂,修改也很麻烦。这时可以把一些play、task 或handler放到其他文件中,通过include指令包含进来是一个不错的选择

```shell
tasks:
  - include: tasks/setup.yml
  - include: tasks/users.yml user=plj # users.yml 中可以通过{{ user }}来使用这些变量
handlers:
  - include: handlers/handlers.yml
```

roles像是加强版的include,它可以引入一个项目的文件和目录
• 一般所需的目录层级有
– vars:变量层
– tasks:任务层
– handlers:触发条件
– files:文件
– template:模板
– default:默认,优先级最低

```shell
# 包含了一个叫"x"的role,则
---
- hosts: host_group
  roles:
    -x
# x/tasks/main.yml
# x/vars/main.yml
# x/handler/main.yml
# x/... .../main.yml
# 都会自动添加进这个 playbook
```

## debug

• 对于Python语法不熟悉的同学,playbook书写起来容易出错,且排错困难,这里介绍几种简单的排错调试方法

```shell
# 检测语法,错误行数不准确
ansible-playbook --syntax-check playbook.yaml
# 测试运行
ansible-playbook -C playbook.yaml
```

==– 显示受到影响的主机 --list-hosts
– 显示工作的task --list-tasks
– 显示将要运行的tag --list-tags==

debug模块可以在运行时输出更为详细的信息,帮助排错

```shell
---
- hosts: web
  remote_user: root
  tasks:
    - shell: uptime |awk '{printf("%.2f\n",$(NF-2))}'
      register: result
    - shell: touch /tmp/isreboot
      when: result.stdout|float > 0.7
    - name: Show debug info
      debug: var=result
```



# ELK

ELK分别代表
– Elasticsearch:负责日志检索和储存
– Logstash:负责日志的收集和分析、处理
– Kibana:负责日志的可视化

ELK组件在海量日志系统的运维中,可用于解决
– 分布式日志数据集中式查询和管理
– 系统监控,包含系统硬件和应用各个组件的监控
– 故障排查
– 安全信息和事件管理
– 报表功能

## Elasticsearch

ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎,基于RESTful API的Web接口

Elasticsearch是用Java开发的,并作为Apache许可条款下的开放源码发布,是当前流行的企业级搜索引
擎。设计用于云计算中,能够达到实时搜索,稳定,可靠,快速,安装使用方便

主要特点
– 实时分析
– 分布式实时文件存储,并将每一个字段都编入索引
– 文档导向,所有的对象全部是文档
– 高可用性,易扩展,支持集群(Cluster)、分片和复制(Shards 和 Replicas)
– 接口友好,支持JSON

– Elasticsearch没有典型意义的事务
– Elasticsearch是一种面向文档的数据库
– Elasticsearch没有提供授权和认证特性



相关概念
– Node: 装有一个ES服务器的节点
– Cluster: 有多个Node组成的集群
– Document: 一个可被搜索的基础信息单元
– Index: 拥有相似特征的文档的集合
– Type: 一个索引中可以定义一种或多种类型
– Filed: 是ES的最小单位,相当于数据的某一列
– Shards: 索引的分片,每一个分片就是一个Shard
– Replicas: 索引的拷贝



ES与关系型数据库的对比
– 在ES中,文档归属于一种 类型(type),而这些类型存在于索引(index)中,类比传统关系型数据库
– DB -> Databases -> Tables -> Rows -> Columns

– 关系型       数据库        表             行             列
– ES -> Indices -> Types -> Documents -> Fields
– ES      索引             类型        文档              域(字段)

```shell
# 安装ES
yum -y install elasticsearch

# 修改配置文件
vim /etc/elasticsearch/elasticsearch.yml
network.host: 0.0.0.0

systemctl enable elasticsearch
systemctl start elasticsearch

curl http://192.169.1.50:9200/
```



ES集群配置
– 集群中的所有节点要相互能够ping通,要在所有集群机器上配置/etc/hosts中的主机名与ip对应关系
– 集群中所有机器都要安装Java环境
– cluster.name集群名称配置要求完全一致
– node.name为当前节点标识,应配置本机的主机名
– discovery为集群节点机器,不需要全部配置
– 配置完成以后启动所有节点服务

```shell
# ES集群配置文件
cluster.name: myelk
node.name: es1
network.host: 0.0.0.0
discovery.zen.ping.unicast.hosts: [“es1", “es2", “es3"]

# 验证集群,使用ES内置字段 _cluster/health
curl http://192.168.1.51:9200/_cluster/health?pretty
```



### ES常用插件

• head插件
– 它展现ES集群的拓扑结构,并且可以通过它来进行索引(Index)和节点(Node)级别的操作
– 它提供一组针对集群的查询API,并将结果以json和表格形式返回
– 它提供一些快捷菜单,用以展现集群的各种状态

• kopf插件
– 是一个ElasticSearch的管理工具
– 它提供了对ES集群操作的API

• bigdesk插件
– 是elasticsearch的一个集群监控工具
– 可以通过它来查看es集群的各种状态,如:cpu、内存使用情况,索引数据、搜索情况,http连接数等



ES插件安装、查看

```shell
# 查看安装的插件
/usr/share/elasticsearch/bin/plugin list

# 安装插件 
/usr/share/elasticsearch/bin/plugin install ftp://192.168.1.254/head.zip
/usr/share/elasticsearch/bin/plugin install file:///tmp/kopf.zip
```

– 这里必须使用 url 的方式进行安装,如果文件在本地,我们也需要使用 file:// 的方式指定路径,例如文件在/tmp/xxx下面,我们要写成 file:///tmp/xxx , 删除使用remove 指令



```shell
# 浏览器访问
http://192.169.1.53:9200/_plugin/head/
http://192.169.1.53:9200/_plugin/kopf/#!/cluster
http://192.169.1.53:9200/_plugin/bigdesk/#nodes
```



### RESTful API

ES 常用
– PUT  --- 增
– DELETE --- 删
– POST --- 改
– GET --- 查



curl 常用参数介绍
– -A 修改请求 agent
– -X 设置请求方法
– -i 显示返回头信息

```shell
# 查看查询集群状态,节点信息的命令,在集群插件主机上使用
cat http://192.168.1.51:9200/_cat/

# v参数显示详细信息
cat http://192.168.1.51:9200/_cat/health?v

# help显示帮助信息
cat http://192.168.1.51:9200/_cat/health?help

# nodes 查询节点状态信息
cat http://192.168.1.51:9200/_cat/nodes?v

# 索引信息
cat http://192.168.1.51:9200/_cat/indices?v

# 创建一个索引,并设置分片数量与副本数量
curl -XPUT 'http://192.168.1.55:9200/zz/' -d '{
  "settings":{
      "index":{
          "number_of_shards": 5,
          "number_of_replicas": 1
      }
  }
}'

# RESTful API插入数据
# (增)加数据,使用 PUT 方法
# 调用方式: 数据库地址/索引/类型/id值
curl -XPUT 'http://192.168.1.55:9200/zz/profile/1' -d '{
  "职业": "诗人",
  "名字": "李白",
  "称号": "诗仙",
  "年代": "唐"
}'

# POST修改
# 修(改)数据,使用 POST 方法
# 在修改数据的时候必须调用 _update 关键字
# 调用方式:数据库地址/索引/类型/id值/_update
curl -XPOST 'http://192.168.1.55:9200/zz/profile/1/_update' -d '{
  "doc":{
    "年代": "唐代"
  }
}'

# (查)查询
# 查询使用 GET 方法,GET为默认方法
# 查询显示结果时候可以用 pretty 规范显示格式
# 多条查询需要使用 _mget 关键字配合 json 调用
curl -XGET 'http://192.168.1.55:9200/tedu/teacher/1'

# (删)除
# 删除时候可以是文档,也可以是库,但不能是类型
curl -XDELETE 'http://192.168.1.55:9200/tedu/teacher/1'
curl -XDELETE 'http://192.168.1.55:9200/tedu'
```

#### 批量导入数据

```shell
# 使用_bulk批量导入数据
# 批量导入数据使用POST方式,数据格式为json,url 编码使用data-binary
# 导入含有index配置的json文件
gzip –d logs.jsonl.gz
curl -XPOST 'http://192.168.1.51:9200/_bulk' --data-binary @logs.jsonl
```



## Kibana

```shell
yum -y install kibana
# kibana 默认安装在 /opt/kibana 下面,配置文件在 /opt/kibana/config/kibana.yml
vim /opt/kibana/config/kibana.yml
server.port: 5601
server.host: "0.0.0.0"  # 监听的地址
elasticsearch.url: "http://192.168.1.51:9200"
kibana.index: ".kibana"        # 自动创建索引的名字
kibana.defaultAppId: "discover"   # 默认页面
elasticsearch.pingTimeout: 1500
elasticsearch.requestTimeout: 30000
elasticsearch.startupTimeout: 5000

systemctl enable kibana
systemctl start kibana
```



## Logstash

**Logstash工作结构**
{ 数据源 } ==> input { } ==> filter { } ==> output { } ==> { ES }

**Logstash里面的类型**

| 布尔值类型 | ssl_enable => true               |
| ---------- | -------------------------------- |
| 字节类型   | bytes => "1MiB"                  |
| 字符串类型 | name => "xkops"                  |
| 数值类型   | port => 22                       |
| 数组       | match => ["datetime","UNIX"]     |
| 哈希       | options => {k => "v",k2 => "v2"} |
| 编码解码   | codec => "json"                  |
| 路径       | file_path => "/tmp/filename"     |
| 注释       | #                                |

**Logstash条件判断**

| 等于       | ==     |
| ---------- | ------ |
| 不等于     | !=     |
| 小于       | <      |
| 大于       | >      |
| 小于等于   | <=     |
| 大于等于   | >=     |
| 匹配正则   | =~     |
| 不匹配正则 | !~     |
| 包含       | in     |
| 不包含     | not in |
| 与         | and    |
| 或         | or     |
| 非与       | nand   |
| 非或       | xor    |
| 复合表达式 | ()     |
| 取反符合   | !()    |

安装

```shell
# Logstash依赖Java环境,需要安装java-1.8.0-openjdk
# Logstash没有默认的配置文件,需要手动配置
# Logstash安装在/opt/logstash目录下
yum –y install logstash

vim /etc/logstash/logstash.conf
input{
	stdin{ codec => "json" }
}
filter{ }
output{
	stdout{ codec => "rubydebug" }
}
# 启动并验证
/opt/logstash/bin/logstash -f /etc/logstash/logstash.conf
```

Logstash插件

– 上页的配置文件使用了logstash-input-stdin和logstash-output-stdout两个插件, Logstash还有filter和codec类插件,查看插件的方式是

```
/opt/logstash/bin/logstash-plugin list
```

– 插件及文档地址 https://github.com/logstash-plugins

codec类插件
– 常用的插件:plain、json、json_lines、rubydebug、multiline等
– 使用刚刚的例子,这次输入json数据
– 设置输入源的codec是json,在输入的时候选择rubydebug

```shell
input{
  file{
    start_position => "beginning"
    sincedb_path => "/var/lib/logstash/sincedb"
    path => ['/tmp/a.log','/tmp/b.log']
    type => 'filelog' # 标签
  }
}

filter{ }

output{
  stdout{ codec => "rubydebug" }
}
# sincedb_path记录读取文件的位置
# start_position配置第一次读取文件从什么地方开始
```

filter grok插件
– 解析各种非结构化的日志数据插件
– grok使用正则表达式把飞结构化的数据结构化
– 在分组匹配,正则表达式需要根据具体数据结构编写
– 虽然编写困难,但适用性极广
– 几乎可以应用于各类数据

```shell
input{
  file{
    start_position => "beginning"
    sincedb_path => "/var/lib/logstash/sincedb"
    path => ['/tmp/a.log','/tmp/b.log']
    type => 'filelog' # 标签
  }
  beats{
    port => 5044
  } 
}

filter{
if [type] == "weblog"{
	grok { # 自写(?<name>reg),宏 %{hongname:name}
    match => { "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" }
  }
 }
}

output{
  # stdout{ codec => "rubydebug" }
  if [type] == "weblog"{
    elasticsearch {
        hosts => ["es1:9200","es2:9200","es3:9200"]
        index => "weblog"
        flush_size => 2000
        idle_flush_time => 10
    }
  }
}

# 查看可用宏
cat /opt/logstash/vendor/bundle/jruby/1.9/gems/logstash-patterns-core-2.0.5/patterns/grok-patterns
```

```shell
# 客户端安装
yum –y install filebeat
systemctl enable filebeat
systemctl start filebeat

# 修改配置文件
vim /etc/filebeat/filebeat.yml
paths:
  - /root/logs.jsonl
		document_type: weblog
... ...
paths:
	- /root/accounts.json
		document_type: account
output:
	logstash:
		hosts: ["192.168.1.57:5044"]
```

# 大数据与Hadoop

大数据的5V特性是什么?
– (V)olume (大体量)可从数百TB到数十数百PB、甚至EB的规模
– (V)ariety(多样性)大数据包括各种格式和形态的数据
– (V)elocity(时效性)很多大数据需要在一定的时间限度下得到及时处理
– (V)eracity(准确性)处理的结果要保证一定的准确性
– (V)alue(大价值)大数据包含很多深度的价值,大数据分析挖掘和利用将带来巨大的商业价值

## Hadoop

Hadoop是什么
– Hadoop是一种分析和处理海量数据的软件平台
– Hadoop是一款开源软件,使用JAVA开发
– Hadoop可以提供一个分布式基础架构

Hadoop特点
– 高可靠性、高扩展性、高效性、高容错性、低成本

2003年开始Google陆续发表了3篇论文
– GFS,MapReduce,BigTable

GFS

– GFS是一个可扩展的分布式文件系统,用于大型的、分布式的、对大量数据进行访问的应用
– 可以运行于廉价的普通硬件上,提供容错功能

MapReduce

– MapReduce是针对分布式并行计算的一套编程模型,由Map和Reduce组成,Map是映射,把指令分发到多个worker上,Reduce是规约,把worker计算出的结果合并

BigTable
– BigTable是存储结构化数据
– BigTable建立在GFS,Scheduler,Lock Service和MapReduce之上
– 每个Table都是一个多维的稀疏图

– GFS - - -> HDFS
– MapReduce - - -> MapReduce
– BigTable - - -> Hbase



## Hadoop常用组件

• HDFS:Hadoop分布式文件系统(核心组件)
• MapReduce:分布式计算框架(核心组件)
• Yarn:集群资源管理系统(核心组件)
• Zookeeper:分布式协作服务
• Hbase:分布式列存数据库
• Hive:基于Hadoop的数据仓库
• Sqoop:数据同步工具
• Pig:基于Hadoop的数据流系统
• Mahout:数据挖掘算法库
• Flume:日志收集工具

## HDFS角色及概念

hdfs java;ceph python

• Hadoop体系中数据存储管理的基础,是一个高度容错的系统,用于在低成本的通用硬件上运行
• 角色和概念
– Client
– Namenode
– Secondarynode
– Datanode

NameNode
– Master节点,管理HDFS的名称空间和数据块映射信息,配置副本策略,处理所有客户端请求

 Secondary NameNode
– 定期合并fsimage 和fsedits,推送给NameNode
– 紧急情况下,可辅助恢复NameNode

• 但Secondary NameNode并非NameNode的热备

DataNode
– 数据存储节点,存储实际的数据
– 汇报存储信息给NameNode

Client
– 切分文件
– 访问HDFS
– 与NameNode交互,获取文件位置信息
– 与DataNode交互,读取和写入数据

Block
– 每块缺省128MB大小
– 每块可以多个副本



## MapReduce角色及概念

源自于Google的MapReduce论文,JAVA实现的分布式计算框架

角色和概念
– JobTracker
– TaskTracker
– Map Task
– Reducer Task

JobTracker
– Master节点只有一个
– 管理所有作业/任务的监控、错误处理等
– 将任务分解成一系列任务,并分派给TaskTracker

TaskTracker
– Slave节点,一般是多台
– 运行Map Task和Reduce Task
– 并与JobTracker交互,汇报任务状态

Map Task

– 解析每条数据记录,传递给用户编写的map()并执行,将输出结果写入本地磁盘
– 如果为map-only作业,直接写入HDFS

Reducer Task

–从Map Task的执行结果中,远程读取输入数据,对数据进行排序,将数据按照分组传递给用户编写的reduce函数执行

## Yarn角色及概念

Yarn是Hadoop的一个通用的资源管理系统
• Yarn角色
– Resourcemanager
– Nodemanager
– ApplicationMaster
– Container
– Client

ResourceManager
– 处理客户端请求
– 启动/监控ApplicationMaster
– 监控NodeManager
– 资源分配与调度

NodeManager
– 单个节点上的资源管理
– 处理来自ResourceManager的命令
– 处理来自ApplicationMaster的命令

Container
– 对任务运行行环境的抽象,封装了CPU 、内存等
– 多维资源以及环境变量、启动命令等任务运行相关的信息资源分配与调度

ApplicationMaster
– 数据切分
– 为应用程序申请资源,并分配给内部任务
– 任务监控与容错

Client
– 用户与Yarn交互的客户端程序
– 提交应用程序、监控应用程序状态,杀死应用程序等

Yarn的核心思想
将JobTracker和TaskTacker进行分离,它由下面几大构成组件
– ResourceManager一个全局的资源管理器
– NodeManager每个节点(RM)代理
– ApplicationMaster表示每个应用
– 每一个ApplicationMaster有多个Container在NodeManager上运行

## Hadoop的三种部署模式

– 单机
– 伪分布式
– 完全分布式

单机模式
• Hadoop的单机模式安装非常简单
– 获取软件http://hadoop.apache.org
– 安装配置Java环境,安装jps工具
安装Openjdk和Openjdk-devel
– 设置环境变量,启动运行
– hadoop-env.sh
	JAVA_HOME="JAVA安装路径"
	HADOOP_CONF_DIR="hadoop配置文件路径"



单机

```shell
# 安装hadoop,解压就行
# 配置JAVA_HOME
vim hadoop/etc/hadoop/hadoop-env.sh
export JAVA_HOME="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64/jre/"
export HADOOP_CONF_DIR="/usr/local/hadoop/etc/hadoop/"

# 统计词频
./bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.7.jar wordcount input output
# jar 运行jar文件
```

伪分布式

– 伪分布式的安装和完全分布式类似,区别是所有角色安装在一台机器上,使用本地磁盘,一般生产环境都会使用完全分布式,伪分布式一般是用来学习和测试Hadoop的功能
– 伪分布式的配置和完全分布式配置类似

Hadoop配置文件及格式

```shell
# 文件格式
Hadoop-env.sh
JAVA_HOME
HADOOP_CONF_DIR
# xml文件配置格式
<property>
    <name>关键字</name>
    <value>变量值</value>
    <description> 描述 </description>
</property>
```

### HDFS部署

完全分布式

基础环境准备
– 在3台机器上配置/etc/hosts
– 注意:所有主机都能ping通namenode的主机名,namenode能ping通所有节点
– java -version 验证java安装
– jps 验证角色

配置SSH信任关系(NameNode)

```shell
# 注意:不能出现要求输入yes的情况,每台机器都要能登录成功,包括本机!!!
vim /etc/ssh/ssh_config
StrictHostKeyChecking no
```

HDFS完全分布式系统配置
– 环境配置文件:hadoop-env.sh
– 核心配置文件:core-site.xml
– HDFS配置文件:hdfs-site.xml
– 节点配置文件:slaves

环境配置文件hadoop-env.sh
– OpenJDK的安装目录:JAVA_HOME
– Hadoop配置文件的存放目录:HADOOP_CONF_DIR

```shell
# 核心配置文件 core-site.xml
# fs.defaultFS:文件系统的配置参数
# hadoop.tmp.dir:数据目录的配置参数,存储所有数据
vim hadoop/etc/hadoop/core-site.xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/var/hadoop</value>
    </property>
</configuration>

mkdir /var/hadoop

# HDFS配置文件hdfs-site.xml
# Namenode:地址声明
# dfs.namenode.http-address
# Secondarynamenode:地址声明
# dfs.namenode.secondary.http-address
# 文件冗余份数
# dfs.replication
vim hadoop/etc/hadoop/hdfs-site.xml
<configuration>
    <property>
        <name>dfs.namenode.http-address</name>
        <value>hadoop:50070</value>
    </property>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop:50090</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>  # 数据存几份
    </property>
</configuration>

# 节点配置文件slaves
# 只写DataNode节点的主机名称
vim hadoop/etc/hadoop/slaves
node1
node2
node3
# 同步配置
# Hadoop所有节点的配置参数完全一样,在一台配置好后,把配置文件同步到其它所有主机上

# namenode 格式化
./hadoop/bin/hdfs namenode -format
# 启动集群
./hadoop/sbin/start-dfs.sh

# NameNode和DataNode,JPS验证角色
jps
# 集群节点验证,NameNode上
./hadoop/bin/hdfs dfsadmin -report
```

### MapReduce部署

```shell
cp ./hadoop/etc/hadoop/mapred-site.xml.template ./hadoop/etc/hadoop/mapred-site.xml

# 配置yarn管理计算集群
vim ./hadoop/etc/hadoop/mapred-site.xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

分布式计算框架mapred-site.xml
– 只支持local和yarn两种
– 单机使用local
– 集群使用yarn

### yarn部署

资源管理yarn-site.xml
– resourcemanager 地址
		yarn.resourcemanager.hostname
– nodemanager 使用哪个计算框架
		yarn.nodemanager.aux-services
– mapreduce_shuffle 计算框架的名称
		mapreduce_shuffle

```shell
vim ./hadoop/etc/hadoop/yarn-site.xml
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop</value>  # namanode
  </property>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
</configuration>

# 启动服务
./hadoop/sbin/start-yarn.sh

# 验证服务 jps 和 ./bin/yarn node -list
./hadoop/bin/yarn node -list
```

使用Web访问Hadoop

– namenode web页面

​	192.169.1.60:50070

– secondory namenode web 页面(nn01)

​	192.168.1.60:50090

– datanode web 页面(node1,node2,node3)

​	192.168.1.62:50075
– resourcemanager web页面(nn01)

​	192.168.1.60:8088

– nodemanager web页面(node1,node2,node3)

​	192.168.1.62:8042

## HDFS基本使用

HDFS基本命令

```shell
# 对应shell命令 ls /
./bin/hadoop fs -ls /

# 对应shell命令 mkdir /abc
./bin/hadoop fs -mkdir /abc

# 对应shell命令 touch /urfile
./bin/hadoop fs -touchz /urfile

# 上传文件
./bin/hadoop fs -put localfile /remotefile

# 下载文件
./bin/hadoop fs -get /remotefile
```

## 节点管理

### HDFS节点管理

HDFS增加结点
– 启动一个新的系统,设置SSH免密码登录
– 在所有节点修改 /etc/hosts,增加新节点的主机信息
– 安装java运行环境(java-1.8.0-openjdk-devel)
– 修改NameNode的slaves文件增加该节点
– 拷贝NamNode的/usr/local/hadoop到本机
– 在该节点启动DataNode

```shell
# 在新节点启动DataNode
./hadoop/sbin/hadoop-daemon.sh start datanode

# 设置同步带宽,并同步数据
./bin/hdfs dfsadmin -setBalancerBandwidth 60000000
./sbin/start-balancer.sh
```

### HDFS修复节点

– 修复节点比较简单,与增加节点基本一致
– 注意:新节点的ip和主机名要与损坏节点的一致
– 启动服务

```shell
./sbin/hadoop-daemon.sh start datanode
```

– 数据恢复是自动的
– 上线以后会自动恢复数据,如果数据量非常巨大,可能需要一定的时间

### HDFS删除节点

```shell
# 配置NameNode的hdfs-site.xml
# 增加dfs.hosts.exclude配置
vim hadoop/etc/hadoop/hdfs-site.xml
<property>
    <name>dfs.hosts.exclude</name>
    <value>/usr/local/hadoop/etc/hadoop/exclude</value>  # 文件路径
</property
# 增加exclude配置文件,写入要删除的节点主机名
vim /usr/local/hadoop/etc/hadoop/exclude
nn4 # 下线的主机名

# 更新数据
./hadoop/bin/hdfs dfsadmin -refreshNodes
```

HDFS删除节点状态
– 查看状态

```shell
./bin/hdfs dfsadmin -report
```

– Normal:正常状态
– Decommissioned in Program:数据正在迁移
– Decommissioned:数据迁移完成

==注意:仅当状态变成Decommissioned才能down机下线==

### yarn节点管理

yarn的相关操作,由于Hadoop在2.x引入了yarn框架,对于计算节点的操作已经变得非常简单

```shell
#　增加节点，在节点上操作
sbin/yarn-daemon.sh start nodemanager

#　删除节点，在节点上操作
sbin/yarn-daemon.sh stop nodemanager

# 查看节点 (ResourceManager)
./bin/yarn node -list
```

### NFS配置

• NFS 网关用途
– 用户可以通过操作系统兼容的本地NFSv3客户端来浏览HDFS文件系统
– 用户可以从HDFS文件系统下载文档到本地文件系统
– 用户可以通过挂载点直接流化数据,支持文件附加,但是不支持随机写
– NFS网关支持NFSv3和允许HDFS作为客户端文件系统的一部分被挂载

• 特性
– HDFS超级用户是与NameNode进程本身具有相同标识的用户,超级用户可以执行任何操作,因为权限检查永远不会认为超级用户失败

• 注意事项
– 在非安全模式下,运行网关进程的用户是代理用户
– 在安全模式下,Kerberos keytab中的用户是代理用户

• 配置代理用户
– 在NameNode和NFSGW上添加代理用户
– ==代理用户的UID,GID,用户名必须完全相同==
– 如果因特殊原因客户端的用户和NFS网关的用户UID不能保持一致,需要我们配置nfs.map的静态映射关系
– nfs.map
	uid 10 100 # Map the remote UID 10 the local UID 100
	gid 11 101 # Map the remote GID 11 to the local GID 101

配置用户

```shell
# 在 namenode(nn01)上添加用户和组
groupadd -g 800 nfsuser
useradd -u 800 -g 800 -r -d /var/hadoop nfsuser
# 在 nfs 网关服务器也同样执行以上两条命令
```

配置core-site.xml
hadoop.proxyuser.{代理用户}.groups
hadoop.proxyuser.{代理用户}.hosts
– 这里的{代理用户}是主机上真实运行的nfs3的用户
– 在非安全模式下,运行nfs网关的用户为代理用户
– groups为挂载点用户所使用的组
– hosts为挂载点主机地址

```shell
vim ./hadoop/etc/hadoop/core-site.xml
    <property>
        <name>hadoop.proxyuser.nfsuser.groups</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.nfsuser.hosts</name>
        <value>*</value>
    </property>
    
# 关闭Hadoop集群
./hadoop/sbin/stop-all.sh 

# 同步配置文件到所有主机

# 启动 hdfs
./sbin/start-dfs.sh
```

#### NFSGW配置

配置步骤
– 启动一个新的系统,卸载rpcbind 、nfs-utils
– 配置/etc/hosts,添加所有NameNode和DataNode的主机名与ip对应关系
– 安装JAVA运行环境(java-1.8.0-openjdk-devel)
– 同步NameNode的/usr/local/hadoop到本机
– 配置hdfs-site.xml
– 启动服务

配置文件hdfs-site.xml
• nfs.exports.allowed.hosts
– 默认情况下,export可以被任何客户端挂载。为了更好的控制访问,可以设置属性。值和字符串对应机器名和访问策略,通过空格来分割。机器名的格式可以是单一的主机、Java的正则表达式或者IPv4地址
– 使用rw或ro可以指定导出目录的读写或只读权限。如果访问策略没被提供,默认为只读。每个条目使用";"来分割

• nfs.dump.dir
– 用户需要更新文件转储目录参数。NFS客户端经常重新安排写操作,顺序的写操作会随机到达NFS网关。这个目录常用于临时存储无序的写操作。对于每个文件,无序的写操作会在他们积累在内存中超过一定阈值(如,1M)时被转储。需要确保有足够的空间的目录
– 如:应用上传10个100M,那么这个转储目录推荐1GB左右的空间,以便每个文件都发生最坏的情况。只有NFS网关需要在设置该属性后重启

```shell
vim hadoop/etc/hadoop/hdfs-site.xml
...
    <property>
        <name>nfs.exports.allowed.hosts</name>
        <value>* rw</value>
    </property>
    <property>
        <name>nfs.dump.dir</name>
        <value>/var/nfstmp</value>
    </property>
...
#　创建/var/nfstmp文件夹
mkdir /var/nfstmp

# 并且把该文件夹的属组改成代理用户
chown 800.800 /var/nfstmp/ # nfsuser:nfsuser
```

nfs启动

```shell
# 设置/usr/local/hadoop/logs权限,为代理用户赋予读写执行的权限
setfacl -m user:nfsuser:rwx /usr/local/hadoop/logs

# 使用root用户启动portmap服务，先启动，必须是root
./sbin/hadoop-daemon.sh --script ./bin/hdfs start portmap

# 使用代理用户启动nfs3，必须是nfsuser
sudo - nfsuser ./sbin/hadoop-daemon.sh --script ./bin/hdfs start nfs3
```

• 警告
– 启动portmap需要使用root用户
– 启动nfs3需要使用core-site里面设置的代理用户
– 必须先启动portmap之后再启动nfs3
– 如果portmap重启了,在重启之后nfs3也需要重启

客户机挂载
– 目前NFS只能使用v3版本
vers=3
– 仅使用TCP作为传输协议
proto=tcp
– 不支持NLM
nolock
– 禁用access time的时间更新
noatime

```shell
yum install nfs-utils
mount -t nfs -o vers=3,proto=tcp,noatime,nolock,sync,noacl 192.168.1.65:/ /mnt/ # nfs服务器ip
```



# Zookeeper

**• Zookeeper是什么**
– Zookeeper是一个开源的分布式应用程序协调服务

**• Zookeeper能做什么**
– Zookeeper是用来保证数据在集群间的事务一致性

**• Zookeeper应用场景**
– 集群分布式锁
– 集群统一命名服务
– 分布式协调服务

**• Zookeeper角色与特性**
– Leader:接受所有Follower的提案请求并统一协调发起提案的投票,负责与所有的Follower进行内部数据交换
– Follower:直接为客户端服务并参与提案的投票,同时与Leader进行数据交换
– Observer:直接为客户端服务但并不参与提案的投票,同时也与Leader进行数据交换

**• Zookeeper角色与选举**
– 服务在启动的时候是没有角色的(LOOKING)
– 角色是通过选举产生的
– 选举产生一个Leader,剩下的是Follower

**• 选举Leader原则**
– 集群中超过半数机器投票选择Leader
– 假如集群中拥有n台服务器,那么Leader必须得到n/2+1台服务器的投票

**• Zookeeper角色与选举**
– 如果Leader死亡,重新选举Leader
– 如果死亡的机器数量达到一半,则集群挂掉
– 如果无法得到足够的投票数量,就重新发起投票,如果参与投票的机器不足n/2+1,则集群停止工作
– 如果Follower死亡过多,剩余机器不足n/2+1,则集群也会停止工作
– Observer不计算在投票总设备数量里面

**• Zookeeper可伸缩扩展性原理与设计**
– Leader所有写相关操作
– Follower读操作与响应Leader提议
– 在Observer出现以前,Zookeeper的伸缩性由Follower来实现,我们可以通过添加Follower节点的数量来保证Zookeeper服务的读性能,但是随着Follower节点数量的增加,Zookeeper服务的写性能受到了影响

– 客户端提交一个请求,若是读请求,则由每台Server的本地副本数据库直接响应。若是写请求,需要通过一致性协议(Zab)来处理
– Zab协议规定:来自Client的所有写请求都要转发给ZK服务中唯一的Leader,由Leader根据该请求发起一个Proposal。然后其他的Server对该Proposal进行Vote。之后Leader对Vote进行收集,当Vote数量过半时Leader会向所有的Server发送一个通知消息。最后当Client所连接的Server收到该消息时,会把该操作更新到内存中并对Client的写请求做出回应

– ZooKeeper在上述协议中实际扮演了两个职能。一方面从客户端接受连接与操作请求,另一方面对操作结果进行投票。这两个职能在Zookeeper集群扩展的时候彼此制约
– 从Zab协议对写请求的处理过程中可以发现,增加Follower的数量,则增加了协议投票过程的压力。因为Leader节点必须等待集群中过半Server响应投票,是节点的增加使得部分计算机运行较慢,从而拖慢整个投票过程的可能性也随之提高,随着集群变大,写操作也会随之下降

– 所以,我们不得不在增加Client数量的期望和我们希望保持较好吞吐性能的期望间进行权衡。要打破这一耦合关系,我们引入了不参与投票的服务器Observer。Observer可以接受客户端的连接,并将写请求转发给Leader节点。但Leader节点不会要求Observer参加投票,仅仅在上述第3歩那样,和其他服务节点一起得到投票结果

– Observer的扩展,给Zookeeper的可伸缩性带来了全新的景象。加入很多Observer节点,无须担心严重影响写吞吐量。但并非是无懈可击,因为协议中的通知阶段,仍然与服务器的数量呈线性关系。但是这里的串行开销非常低。因此,我们可以认为在通知服务器阶段的开销不会成为瓶颈
– Observer提升读性能的可伸缩性
– Observer提供了广域网能力

```shell
# 安装zookeeper
tar -zxvf /opt/zookeeper-3.4.13.tar.gz -C /usr/local/
cd /usr/local/zookeeper-3.4.13/
cp conf/zoo_sample.cfg conf/zoo.cfg
vim conf/zoo.cfgcp
server.1=nn1:2888:3888
server.2=nn2:2888:3888
server.3=nn3:2888:3888
server.4=hadoop:2888:3888:observer

# 同步配置到其他节点
for i in {61..63}; do rsync -aP /usr/local/zookeeper-3.4.13/ 192.169.1.${i}:/usr/local/zookeeper-3.4.13/ ; done

# 根据配置文件 dataDir=/tmp/zookeeper,创建目录
mkdir /tmp/zookeeper

# 根据配置文件 创建myid,输入对应id
echo 4 > /tmp/zookeeper/myid # hadoop主机上写4,其他主机写自己对应的id号

# 启动集群,查看验证(在所有集群节点执行)
for i in {60..63}; do ssh 192.169.1.${i} "/usr/local/zookeeper-3.4.13/bin/zkServer.sh start" ; done
# 查看角色
/usr/local/zookeeper-3.4.13/bin/zkServer.sh status
```

关于myid文件
– myid文件中只有一个数字
– 注意:请确保每个server的myid文件中id数字不同
– server.id中的id与myid中的id必须一致
– id的范围是1~255

Zookeeper管理文档
http://zookeeper.apache.org/doc/r3.4.10/zookeeperAdmin.html



# Kafka

• Kafka是什么
– Kafka是由LinkedIn开发的一个分布式的消息系统
– Kafka是使用Scala编写
– Kafka是一种消息中间件

• 为什么要使用Kafka
– 解耦、冗余、提高扩展性、缓冲
– 保证顺序,灵活,削峰填谷
– 异步通信

• Kafka角色与集群结构
– producer:生产者,负责发布消息
– consumer:消费者,负责读取处理消息
– topic:消息的类别
– Parition:每个Topic包含一个或多个Partition
– Broker:Kafka集群包含一个或多个服务器

• Kafka通过Zookeeper管理集群配置,选举Leader

• Kafka集群的安装配置
– Kafka集群的安装配置依赖Zookeeper,搭建Kafka集群之前,请先创建好一个可用的Zookeeper集群
– 安装OpenJDK运行环境
– 同步Kafka拷贝到所有集群主机
– 修改配置文件
– 启动与验证

```shell
tar -zxvf /opt/kafka_2.12-2.1.0.tgz -C /usr/local/
cd /usr/local/kafka_2.12-2.1.0/
vim config/server.properties
# 每台服务器的broker.id都不能相同
broker.id
zookeeper.connect=nn1:2181,nn2:2181,nn3:2181

# 同步配置
# 在所有主机启动服务
./bin/kafka-server-start.sh -daemon config/server.properties
```

验证

​	jps命令应该能看到Kafka模块

​	netstat应该能看到9092在监听

集群验证与消息发布

```shell
# 创建一个 topic
./bin/kafka-topics.sh --create --partitions 2 --replication-factor 2 --zookeeper localhost:2181 --topic mymsg
# 生产者,发布消息
./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic mymsg
# 消费者,读取消息
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mymsg
```



# Hadoop高可用

官方提供了两种解决方案
– HDFS with NFS
– HDFS with QJM

HA方案对比
– 都能实现热备
– 都是一个Active NN和一个Standby NN
– 都使用Zookeeper和ZKFC来实现自动失效恢复
– 失效切换都使用Fencin配置的方法来Active NN
– NFS数据共享变更方案把数据存储在共享存储里,我们还需要考虑NFS的高可用设计
– QJM不需要共享存储,但需要让每一个DN都知道两个NN的位置,并把块信息和心跳包发送给Active和Standby这两个NN

• 使用原因(QJM)
– 解决NameNode单点故障问题
– Hadoop给出了HDFS的高可用HA方案:HDFS通常由两个NameNode组成,一个处于Active状态,另一个处于Standby状态。Active NameNode对外提供服务,比如处理来自客户端的RPC请求,而Standby NameNode则不对外提供服务,仅同步Active NameNode的状态,以便能够在它失败时进行切换

• 典型的HA集群
– NameNode会被配置在两台独立的机器上,在任何时候 , 一 个 NameNode 处 于 活 动 状 态 , 而 另 一 个NameNode则处于备份状态
– 活动状态的NameNode会响应集群中所有的客户端,备份状态的NameNode只是作为一个副本,保证在必要的时候提供一个快速的转移

• NameNode高可用架构
– 为了让Standby Node与Active Node保持同步,这两个Node 都 与 一 组 称 为 JNS 的 互 相 独 立 的 进 程 保 持 通 信(Journal Nodes)。当Active Node更新了namespace,它将记录修改日志发送给JNS的多数派。Standby Node将会从JNS中读取这些edits,并持续关注它们对日志的变更
– Standby Node将日志变更应用在自己的namespace中,当Failover发生时,Standby将会在提升自己为Active之前,确保能够从JNS中读取所有的edits,即在Failover发生之前Standy持有的namespace与Active保持完全同步

– NameNode更新很频繁,为了保持主备数据的一致性,为了支持快速Failover,Standby Node持有集群中blocks的最新位置是非常必要的。为了达到这一目的,DataNodes上需要同时配置这两个Namenode的地址,同时和它们都建立心跳连接,并把block位置发送给它们

– 任何时刻,只能有一个Active NameNode,否则会导致集群操作混乱,两个NameNode将会有两种不同的数据状态,可能会导致数据丢失或状态异常,这种情况通常称为"split-brain"(脑裂,三节点通讯阻断,即集群中不同的DataNode看到了不同的Active NameNodes)
– 对于JNS而言,任何时候只允许一个NameNode作为writer;在Failover期间,原来的Standby Node将会接管Active的所有职能,并负责向JNS写入日志记录,这种机制阻止了其他NameNode处于Active状态的问题

| 主机                   | 角色                                   | 软件               |
| ---------------------- | -------------------------------------- | ------------------ |
| 192.168.1.60           | NameNode1                              | Hadoop             |
| 192.168.1.66           | NameNode2                              | Hadoop             |
| 192.168.1.61<br>node1  | DataNode<br/>journalNode<br/>Zookeeper | HDFS<br/>Zookeeper |
| 192.168.1.62<br/>node2 | DataNode<br/>journalNode<br/>Zookeeper | HDFS<br/>Zookeeper |
| 192.168.1.63<br/>node3 | DataNode<br/>journalNode<br/>Zookeeper | HDFS<br/>Zookeeper |

同步所有/etc/hosts,两个namenode可以无密码登录所有主机.

```shell
# 配置hadoop集群
vim hadoop/etc/hadoop/hadoop-env.sh
export JAVA_HOME="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64/jre/"
export HADOOP_CONF_DIR="/usr/local/hadoop/etc/hadoop/"

vim hadoop/etc/hadoop/slaves
node1
node2
node3

vim hadoop/etc/hadoop/core-site.xml
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://mycluster</value> # mycluster,集群name名,随意
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/var/hadoop</value>
    </property>
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>node1:2181,node2:2181,node3:2181</value>
    </property>

vim hadoop/etc/hadoop/hdfs-site.xml
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <!-- 指定hdfs的nameservices名称为mycluster -->
    <property>
        <name>dfs.nameservices</name>
        <value>mycluster</value>  # 与之前的集群name名一致
    </property>
    <!-- 指定集群的两个NaneNode的角色名称分别为nn1,nn2,角色名称不能写别的 -->
    <property>
        <name>dfs.ha.namenodes.mycluster</name> # 与之前的集群name名一致
        <value>nn1,nn2</value>
    </property>
    <!-- 配置nn1,nn2的rpc通信端口 -->
    <property>
        <name>dfs.namenode.rpc-address.mycluster.nn1</name> # 与之前的集群name名一致
        <value>nn01:8020</value>
    </property>
    <property>
        <name>dfs.namenode.rpc-address.mycluster.nn2</name> # 与之前的集群name名一致
        <value>nn02:8020</value>
    </property>
    <!-- 配置nn1,nn2的http通信端口 -->
    <property>
        <name>dfs.namenode.http-address.mycluster.nn1</name>
        <value>nn01:50070</value>
    </property>
    <property>
        <name>dfs.namenode.http-address.mycluster.nn2</name>
        <value>nn02:50070</value>
    </property>
    <!-- 指定NameNode元数据存储在journalnode中的路径 -->
    <property>
        <name>dfs.namenode.shared.edits.dir</name>
        <value>qjournal://node1:8485;node2:8485;node3:8485/mycluster</value>
    </property>
    <!-- 指定journalnode日志文件存储的路径 -->
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>/var/hadoop/journal</value>
    </property>
    <!-- 指定HDFS客户端连接Active NameNode的java类 -->
    <property>
        <name>dfs.client.failover.proxy.provider.mycluster</name>
  	<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>
    <!-- 配置隔离机制为SSH -->
    <property>
        <name>dfs.ha.fencing.methods</name>
        <value>sshfence</value>
    </property>
    <!-- 指定密钥的位置 -->
    <property>
        <name>dfs.ha.fencing.ssh.private-key-files</name>
        <value>/root/.ssh/id_rsa</value>
    </property>
    <!-- 开启自动故障转移 -->
    <property>
        <name>dfs.ha.automatic-failover.enabled</name>
        <value>true</value>
    </property>
```

```shell
# 配置yarn管理计算集群
vim ./hadoop/etc/hadoop/mapred-site.xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

yarn高可用
• ResourceManager高可用
– RM的高可用原理与NN一样,需要依赖ZK来实现,这里配置文件的关键部分,感兴趣的同学可以自己学习和测试
– yarn.resourcemanager.hostname
– 同理因为使用集群模式,该选项应该关闭

```shell
# 配置yarn高可用
vim ./hadoop/etc/hadoop/yarn-site.xml
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
    </property>
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
    </property>
    <property>
        <name>yarn.resourcemanager.recovery.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>yarn.resourcemanager.store.class</name>
        <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
    </property>
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>node1:2181,node2:2181,node3:2181</value>
    </property>
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>yarn-ha</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>nn01</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>nn02</value>
    </property>
```



高可用启动验证

```shell
# 初始化
# ALL: 同步配置到所有集群机器
# NN1: 初始化ZK集群
./hadoop/bin/hdfs zkfc -formatZK

# nodeX: 启动journalnode服务
./hadoop/sbin/hadoop-daemon.sh start journalnode

# NN1: 格式化
./hadoop/bin/hdfs namenode -format

# NN2: 数据同步到本地/var/hadoop/dfs
rsync -aSH nn01:/var/hadoop/dfs /var/hadoop/

# NN1: 初始化JNS
./hadoop/bin/hdfs namenode -initializeSharedEdits

# nodeX: 停止journalnode服务
./hadoop/sbin/hadoop-daemon.sh stop journalnode

# 启动集群
# NN1: 启动hdfs
./hadoop/sbin/start-dfs.sh

# NN1: 启动yarn
./hadoop/sbin/start-yarn.sh

# NN2: 启动热备ResourceManager
./hadoop/sbin/yarn-daemon.sh start resourcemanager

# 集群验证
# 查看集群状态
# 获取NameNode状态
./hadoop/bin/hdfs haadmin -getServiceState nn1
./hadoop/bin/hdfs haadmin -getServiceState nn2
# 获取ResourceManager状态
./hadoop/bin/yarn rmadmin -getServiceState rm1
./hadoop/bin/yarn rmadmin -getServiceState rm2

# 查看集群状态
# 获取节点信息
./hadoop/bin/hdfs dfsadmin -report
./hadoop/bin/yarn node -list
# 访问集群文件
./hadoop/bin/hadoop fs -mkdir /input
./hadoop/bin/hadoop fs -ls hdfs://mycluster/

# 主从切换Activate
./hadoop/sbin/hadoop-daemon.sh stop namenode
```

