# 简介 #
LNMP是Linux、Nginx、MySQL(MariaDB)和PHP的缩写，这个组合是最常见的WEB服务器的运行环境之一。本文将带领大家在CentOS 7操作系统上搭建一套LNMP环境。
   
本教程适用于CentOS 7.x版本。

# Tengine安装 #
1. 下载tengine安装包 
```
wget http://tengine.taobao.org/download/tengine-2.2.0.tar.gz
```
2. 解压tengine
```
tar  zxvf tengine-2.2.0.tar.gz
```
3. 创建nginx用户
```
useradd -s /sbin/nologin  -M nginx
```
4. 安装tenginx
```
yum install pcre pcre-devel  openssl  openssl-devel -y 
```
```
./configure  \
--prefix=/usr/local/nginx \
--sbin-path=/usr/local/nginx/sbin/nginx  \
--conf-path=/etc/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log  \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--user=nginx \
--group=nginx  \
--with-http_ssl_module \
--with-http_stub_status_module \
--with-http_gzip_static_module  \
--http-client-body-temp-path=/var/tmp/nginx/client/ \
--http-proxy-temp-path=/var/tmp/nginx/proxy/ \
--http-fastcgi-temp-path=/var/tmp/nginx/fcgi/  \
--http-uwsgi-temp-path=/var/tmp/nginx/uwsgi
```
```
make  &&  make install 
```
nginx启动
```
/usr/local/nginx/sbin/nginx -c  /etc/nginx/nginx.conf
```
出现如下报错
```
[root@vm172-31-0-13 php-7.1.3]# /usr/local/nginx/sbin/nginx  -c  /etc/nginx/nginx.conf
nginx: [emerg] mkdir() "/var/tmp/nginx/client/" failed (2: No such file or directory)
```
解决方案
```
mkdir -p /var/tmp/nginx/client/
```
安装完毕后，Nginx的配置文件在/etc/nginx目录下。
您可以通过浏览器访问 http://<外网IP地址> 来确定Nginx是否已经启动。


# 安装MySQL(MariaDB) #
MariaDB是MySQL的一个分支，主要由开源社区进行维护和升级，而MySQL被Oracle收购以后，发展较慢。在CentOS 7的软件仓库中，将MySQL更替为了MariaDB。
我们可以选择yum、rpm、二进制、源码包编译方式来安装。
本教程采用二进制包来安装
1.下载二进制安装包
提供Mariadb下载地址列表大家自己选择合适自己的安装方式
```
https://downloads.mariadb.com/MariaDB/
```
2.MariaDB二进制安装包下载
```
wegt https://downloads.mariadb.com/MariaDB/mariadb-10.2.4/bintar-linux-x86_64/mariadb-10.2.4-linux-x86_64.tar.gz
```

3.MariaDB安装
3.1 解压mariadb安装包
```
tar  zxvf mariadb-10.2.4-linux-x86_64.tar.gz
```
3.2 创建mysql（mariadb）用户
```
useradd -s /sbin/nologin -M mysql
```
3.3 创建软连接（方便mysql升级）
```
ln -s mariadb-10.2.4-linux-x86_64  mysql
```
3.4 创建数据目录并授权
```
mkdir -p /data/3306/mysql/{binlog,data,logs,undo}
chown -R mysql.mysql /data/3306
chown -R /usr/local/mysql
```
3.5 mariadb配置文件my.cnf
```
[client]
port            = 3306
socket          = /data/3306/mysql/mysql.sock
loose-default-character-set = utf8
[mysqld]
port            = 3306
socket          = /data/3306/mysql/mysql.sock
pid-file        =/data/3306/mysql/mysql.pid
server-id              = 20
character-set-server   = utf8
default-storage-engine = InnoDB

extra_max_connections=3

default-time-zone='+8:00'

lc-messages_dir=/usr/local/mysql/share
lc-messages=en_US
skip-slave-start
skip-name-resolve
skip-external-locking
log-slave-updates
#slave-skip-errors        = 1062,1146
#userstat_running         =1

lock_wait_timeout        = 10

#crash safe options
#relay_log_recovery        =1
#master_info_repository    =TABLE
#relay_log_info_repository =TABLE

back_log                = 500
max_allowed_packet      = 1073741824
table_open_cache             = 8192
max_connections         = 20000
max_connect_errors      = 800

key_buffer_size              = 128M
sort_buffer_size        = 8M
read_buffer_size        = 2M
read_rnd_buffer_size    = 8M
join_buffer_size        = 4M
tmp_table_size          = 512M
max_heap_table_size     = 512M
binlog_cache_size       = 8M
myisam_sort_buffer_size = 64M
thread_cache_size       = 500
query_cache_size        = 0M

slow_query_log          = ON
slow_query_log_file     = /data/3306/mysql/logs/mysql-slow.log
long_query_time         = 3 

log-error               = /data/3306/mysql/logs/mysql-error.log


log-bin                 = /data/3306/mysql/binlog/mysql-bin
#relay_log               = mysql-relay-bin
binlog_format           = row 

tmpdir                  = /tmp
#thread_concurrency      = 16


secure_auth        = 1
local-infile       = 0
event_scheduler    = OFF
explicit_defaults_for_timestamp=on


innodb_file_per_table    = 1
innodb_file_format       = barracuda
#innodb_file_format_check = barracuda
innodb_open_files        = 4096


#undo file location info

innodb_undo_logs                =128
innodb_undo_tablespaces         =6
innodb_undo_directory           =/data/3306/mysql/undo

datadir                         =/data/3306/mysql/data
innodb_data_home_dir            = /data/3306/mysql/data
innodb_data_file_path           = ibdata1:1000M;ibdata2:1000M;ibdata3:1000M;ibdata4:100M:autoextend
innodb_log_group_home_dir       = /data/3306/mysql/data
innodb_table_locks              = 0
innodb_buffer_pool_size         = 6G

#innodb_additional_mem_pool_size = 16M
innodb_log_file_size            = 512M
innodb_log_files_in_group       = 2
innodb_log_buffer_size          = 100M
innodb_flush_method             = O_DIRECT
innodb_flush_log_at_trx_commit  = 0
innodb_old_blocks_time          =1000
innodb_stats_on_metadata        = OFF
#log_slow_verbosity     = microtime,innodb
slave_net_timeout      =60

#set innodb hi partitions
innodb_buffer_pool_populate=1
thread_pool_size        = 50
thread_handling=pool-of-threads
thread_pool_oversubscribe= 40

innodb_max_dirty_pages_pct      = 60      
innodb_write_io_threads         = 16         
innodb_read_io_threads          = 8           
innodb_adaptive_flushing        = 1  
#innodb_adaptive_checkpoint      = changecheckpoint_before_use
innodb_io_capacity              = 200         
#innodb_thread_concurrency       = threadconcurrency_before_use
sync_binlog                     = 100

[mysqldump]
quick
max_allowed_packet = 64M

[mysql]
no-auto-rehash
max_allowed_packet      = 64M
prompt="\\u@\\h> "
no-auto-rehash
show-warnings

[isamchk]
key_buffer       = 256M
sort_buffer_size = 256M
read_buffer      = 2M
write_buffer     = 2M

[myisamchk]
key_buffer       = 256M
sort_buffer_size = 256M
read_buffer      = 2M
write_buffer     = 2M

[mysqlhotcopy]
interactive-timeout
```
4. MariaDB数据库初始化
```
/usr/local/mysql/scripts/mysql_install_db  --user=mysql  --basedir=/usr/local/mysql/  --datadir=/data/3306/mysql/data/  --defaults-file=/etc/my.cnf 
```
5. 设置MariaDB环境变量
```
echo 'export MYSQL="/usr/local/mysql/"' >>.bash_profile 
echo 'export PATH="$MYSQL/bin:$PATH"' >>.bash_profile 
```
6. 配置MariaDB启动
```
cp  /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
chmod +x  /etc/init.d/mysqld 
chkconfig --add  mysqld
chkconfig   mysqld on
/etc/init.d/mysqld start
```
7. MariaDB默认root密码为空，我们需要执行安全脚本
```
/usr/local/mysql/bin/mysql_secure_installation
```
这个脚本会弹出一些交互式问答，来进行MariaDB的安全设置
首先要输入root密码(root密码为空)
```
Enter current password for root (enter for none): 
```
如果出现如下错误需要做软连接
```
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
```
创建软连接
```
ln -s  /data/3306/mysql/mysql.sock  /tmp/mysql.sock
```
回车后继续执行
```
Set root password? [Y/n] 
```
设置root密码，默认选项为Yes，我们直接回车，提示输入密码，在这里设置您的MariaDB的root账户密码。
```
Set root password? [Y/n] y
New password: 
Re-enter new password:
```
是否移除匿名用户，默认选项为Yes，建议按默认设置，回车继续。

```
Remove anonymous users? [Y/n] 
```
是否禁止root用户远程登录？如果您只在本机内访问MariaDB，建议按默认设置，回车继续。 如果您还有其他云主机需要使用root账号访问该数据库，则需要选择n。
```
Remove test database and access to it? [Y/n]
```
是否删除测试用的数据库和权限？ 建议按照默认设置，回车继续。

```
Reload privilege tables now? [Y/n]
```
是否重新加载权限表？因为我们上面更新了root的密码，这里需要重新加载，回车。
```
Reload privilege tables now? [Y/n]
```
完成后你会看到success提示，MariaDB安全设置成功，接下来你可以登陆MariaDB了。
```
mysql -uroot -p
```
按提示输入root密码，就会进入MariaDB的交互界面，说明已经安装成功。

# PHP安装 #
1. PHP 7.1.3下载
```
wget http://am1.php.net/distributions/php-7.1.3.tar.gz
```
2. 安装php 7.1.3
2.1 安装编译php所需要的依赖
```
yum install libxml2 libxml2-devel curl  curl-devel libwebp.x86_64  libwebp-devel.x86_64 libwebp-tools.x86_64 libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxslt  libxslt-devel -y
```
2.2 解压php安装包
```
tar zxvf php-7.1.3.tar.gz
```
3. 编译安装php7
```
./configure --prefix=/usr/local/php \
 --with-curl \
 --with-freetype-dir \
 --with-gd \
 --with-gettext \
 --with-iconv-dir \
 --with-kerberos \
 --with-libdir=lib64 \
 --with-libxml-dir \
 --with-mysqli \
 --with-openssl \
 --with-pcre-regex \
 --with-pdo-mysql \
 --with-pdo-sqlite \
 --with-pear \
 --with-png-dir \
 --with-xmlrpc \
 --with-xsl \
 --with-zlib \
 --enable-fpm \
 --enable-bcmath \
 --enable-libxml \
 --enable-inline-optimization \
 --enable-gd-native-ttf \
 --enable-mbregex \
 --enable-mbstring \
 --enable-opcache \
 --enable-pcntl \
 --enable-shmop \
 --enable-soap \
 --enable-sockets \
 --enable-sysvsem \
 --enable-xml \
 --enable-zip
```
```
make &&  make install 
```
4. PHP配置文件
```
cp php.ini-development /usr/local/php/lib/php.ini
```
```
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
```
```
cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
```
```
cp -R ./sapi/fpm/php-fpm  /etc/init.d/php-fpm
```
```
chmod +x  /etc/init.d/php-fpm
```
需要注意的是php7中www.conf这个配置文件配置phpfpm的端口号等信息，如果你修改默认的9000端口号需在这里改，再改nginx的配置
5. 启动
```
/etc/init.d/php-fpm 
```




