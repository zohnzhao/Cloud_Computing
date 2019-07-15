# php 源码编译

获取并解压 PHP 源代码:

```shell
tar -zxf php-x.x.x

cd ./php-x.x.x
./configure --enable-fpm --enable-mysqlnd
make
sudo make install

# 创建配置文件，并将其复制到正确的位置。
cp php.ini-development /usr/local/php/php.ini
cp /usr/local/etc/php-fpm.conf.default /usr/local/etc/php-fpm.conf
cp sapi/fpm/php-fpm /usr/local/bin

# 需要着重提醒的是，如果文件不存在，则阻止 Nginx 将请求发送到后端的 PHP-FPM 模块， 以避免遭受恶意脚本注入的攻击。

# 将 php.ini 文件中的配置项 cgi.fix_pathinfo 设置为 0 。

#　打开 php.ini:

vim /usr/local/php/php.ini
#　定位到 cgi.fix_pathinfo= 并将其修改为如下所示：
cgi.fix_pathinfo=0

#　在启动服务之前，需要修改 php-fpm.conf 配置文件，确保 php-fpm 模块使用 www-data 用户和 www-data 用户组的身份运行。

vim /usr/local/etc/php-fpm.conf
#　找到以下内容并修改：
include=/usr/local/php/etc/php-fpm.d/*.conf

cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
vim /usr/local/php/etc/php-fpm.d/www.conf

; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;       will be used.
user = www-data
group = www-data


#　然后启动 php-fpm 服务：
useradd -s /sbin/nologin www-data

/usr/local/bin/php-fpm
```



# FPM制作rpm包

```shell
# 安装依赖包
yum -y install ruby rubygems ruby-devel gcc make

# 添加仓库
gem sources -a http://mirrors.aliyun.com/rubygems/

# 移除原有的仓库
gem sources --remove https://rubygems.org/
gem sources --remove http://rubygems.org/

# 查看仓库是不是只有自己添加的那个仓库地址
gem sources -l

# 安装fpm
gem install fpm
```

```shell
`fpm --help　　`
```

**常用参数：**

-s 指定源类型
-t 指定目标类型
-n 指定包的名字
-v 指定包的版本号
-C 指定打包的相对路径 change directory to here before searching for files
-d 指定依赖于哪些包
-f 第二次打包时目录下如果有同名安装包存在，则覆盖它
-p 输出的安装包的目录，不想放在当前目录下就需要指定
--post-install 软件包安装完成之后所要运行的脚本，同--after-install
--pre-install 软件包安装完成之前所要运行的脚本，同--before-install
--post-uninstall 软件包卸载之后所要运行的脚本，同--after-install
--pre-uninstall 软件包卸载之前所要运行的脚本，同--before-install



## 打包php

```bash
cp /usr/local/etc/php-fpm.conf /usr/local/php/tmp/
cp /usr/local/binphp-fpm /usr/local/php/tmp/

vim php_post_init.sh  # 软件包安装完成之后所要运行的脚本
#!/bin/bash
useradd -s /sbin/nologin www-data
cp '/usr/local/php/tmp/php-fpm' '/usr/local/bin/'
cp '/usr/local/php/tmp/php-fpm.conf' '/usr/local/etc/php-fpm.conf'
mkdir -p /usr/local/var/log/                       
touch /usr/local/var/log/php-fpm.log

# 开始打包
fpm -s dir -t rpm -n php -v 7.3.6 --description 'author: zohn' --post-install php_post_init.sh /usr/local/php/
```



## 打包MySQL

事先安装好MySQL，MySQL安装过程这里不在详述。命令行终端输入以下命令，然后等待rpm包制作完成。

```shell
`# fpm -s dir -t rpm -n mysql -v 5.6.27 --description 'author: jkzhao' -d 'libaio' -d 'libaio-devel' --pre-install /usr/local/mysql/mysql_pre_init.sh --post-install /usr/local/mysql/mysql_post_init.sh  /usr/local/mysql /usr/local/mysql-5.6.27-linux-glibc2.5-x86_64 /data`
```

**【注意】：**默认打好的包是在当前目录下。

**命令说明：**
-s dir：指定源文件是目录的形式
-t rpm：指定打包的格式
-n：指定打包后名称
-v：版本号
--description：描述信息
-d：指定需要依赖的包。安装MySQL前需要在系统上安装libaio、libaio-devel。当你安装fpm打包成的rpm包时，它会先去检测系统上是否安装了这两个包，如果没有安装会给出提示，并终止rpm的安装。
--pre-install：安装rpm包前需要执行的脚本
--post-install：安装rpm包后需要执行的脚本 

- mysql_pre_init.sh的内容如下：

```shell
`#!/bin/bash` `user=mysql``group=mysql` `# create group if not exists.``egrep` `"^$group"` `/etc/group` `>& ``/dev/null``if` `[ $? -``ne` `0 ]``then``    ``groupadd -r -g 300 $group``fi` `# create user if not exists. ``egrep` `"^$user"` `/etc/passwd` `>& ``/dev/null``if` `[ $? -``ne` `0 ]``then``    ``useradd` `-g $group -r -s ``/sbin/nologin` `-u 300 $user``fi`
```

- mysql_post_init.sh的内容如下：

```shell
`#!/bin/bash` `# cp my.cnf force.``\``cp` `/usr/local/mysql/my``.cnf ``/etc/` `# start/stop/restart script.``\``cp` `/usr/local/mysql/support-files/mysql``.server ``/etc/init``.d``/mysqld``chkconfig --add mysqld` `# MySQL Client PATH.``\``cp` `/usr/local/mysql/mysql``.sh ``/etc/profile``.d/` `cd` `/usr/local/mysql``chown` `-R root.mysql .``chown` `-R mysql.mysql ``/data`
```

打包完成后正常安装，如：　　

```
`# rpm -ivh mysql-5.6.27-1.x86_64.rpm`
```