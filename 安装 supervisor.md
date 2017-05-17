# 在centos7 安装 supervisor

## systemctl介绍
> System是Linux系统工具，用来启动守护进程，已经成为大多数发行版本的标准配置。可以通过`systemctl --version`命令来查看使用的版本。
### 常用命令
```sql
# 立即启动一个服务
$ systemctl start apache.service
# 立即停止一个服务
$ systemctl stop apache.service
# 重启一个服务
$ systemctl restart apache.service
# 杀死一个服务的所有子进程
$ systemctl kill apache.service
# 重新加载一个服务的配置文件
$ systemctl reload apache.service
# 重载所有修改过的配置文件
$ systemctl daemon-reload
# 显示某个 Unit 的所有底层参数
$ systemctl show httpd.servic
```
## Supervisor
>Supervisor是是一个用python写的进程管理程序，不仅仅可以用来管理进程，还可以用来做开机启动。它有但不限于以下一些功能：

>1.重启机器后，能够自启动。
>2.平时有个方便的进程查看方式。
>3.能够有个方便的方式重启进程。

>配置方法这里就不做记录了，不过要注意，默认的配置文件里面会把一些supervisor生成的重要文件放到 /tmp 目录下面，操作系统可能会把这些文件进行删除，导致 `supervisorctl` 命令由于找不到这些以前放到 /tmp 的文件而操作不了已经启动的supervisor进程。
### 使用方法
>为了能够在机器启动之后自动启动supervisor，需要把supervisor进程配置进systemd，

步骤:
>进入目录 `/usr/lib/systemd/system/`，增加文件 `supervisord.service`，来使得机器启动的时候启动supervisor，文件内容
```sql
# supervisord service for systemd (CentOS 7.0+)
# by ET-CS (https://github.com/ET-CS)
[Unit]
Description=Supervisor daemon

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf
ExecStop=/usr/bin/supervisorctl $OPTIONS shutdown
ExecReload=/usr/bin/supervisorctl $OPTIONS reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```
>1.激活开机启动命令
`systemctl enable supervisord.service`
>2.启动supervisor进程
`systemctl start supervisord.service`
>3.关闭supervisor进程
`systemctl stop supervisord.service`
>4.如果修改了`supervisor.service`文件，可以通过reload命令来重新加载配置文件
`systemctl reload supervisord.service`
## Supervisor 配置
配置文件`/etc/supervisord.conf`
简单配置只需修改服务文件路径(文件后缀名ini或conf)
```
[include]
files = supervisord.d/*.ini
```
```sql
[program:kemi] #kemi进程名
command=/usr/java/bin/java  -jar /data/kemi/kemi-0.0.1-SNAPSHOT.jar --port=8068
stdout_logfile=/data/kemi/kemi.log
autostart=true
autorestart=true
startsecs=5
priority=1
stopasgroup=true
killasgroup=true
```
### 配置文件参数说明
>supervisor的配置参数较多，下面介绍一下常用的参数配置，详细的配置及说明，请参考官方文档介绍。 
注：分号（;）开头的配置表示注释
```sql
[unix_http_server]
file=/tmp/supervisor.sock   ;UNIX socket 文件，supervisorctl 会使用
;chmod=0700                 ;socket文件的mode，默认是0700
;chown=nobody:nogroup       ;socket文件的owner，格式：uid:gid

;[inet_http_server]         ;HTTP服务器，提供web管理界面
;port=127.0.0.1:9001        ;Web管理后台运行的IP和端口，如果开放到公网，需要注意安全性
;username=user              ;登录管理后台的用户名
;password=123               ;登录管理后台的密码

[supervisord]
logfile=/tmp/supervisord.log ;日志文件，默认是 $CWD/supervisord.log
logfile_maxbytes=50MB        ;日志文件大小，超出会rotate，默认 50MB，如果设成0，表示不限制大小
logfile_backups=10           ;日志文件保留备份数量默认10，设为0表示不备份
loglevel=info                ;日志级别，默认info，其它: debug,warn,trace
pidfile=/tmp/supervisord.pid ;pid 文件
nodaemon=false               ;是否在前台启动，默认是false，即以 daemon 的方式启动
minfds=1024                  ;可以打开的文件描述符的最小值，默认 1024
minprocs=200                 ;可以打开的进程数的最小值，默认 200

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ;通过UNIX socket连接supervisord，路径与unix_http_server部分的file一致
;serverurl=http://127.0.0.1:9001 ; 通过HTTP的方式连接supervisord

; [program:xx]是被管理的进程配置参数，xx是进程的名称
[program:xx]
command=/opt/apache-tomcat-8.0.35/bin/catalina.sh run  ; 程序启动命令
autostart=true       ; 在supervisord启动的时候也自动启动
startsecs=10         ; 启动10秒后没有异常退出，就表示进程正常启动了，默认为1秒
autorestart=true     ; 程序退出后自动重启,可选值：[unexpected,true,false]，默认为unexpected，表示进程意外杀死后才重启
startretries=3       ; 启动失败自动重试次数，默认是3
user=tomcat          ; 用哪个用户启动进程，默认是root
priority=999         ; 进程启动优先级，默认999，值小的优先启动
redirect_stderr=true ; 把stderr重定向到stdout，默认false
stdout_logfile_maxbytes=20MB  ; stdout 日志文件大小，默认50MB
stdout_logfile_backups = 20   ; stdout 日志文件备份数，默认是10
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile=/opt/apache-tomcat-8.0.35/logs/catalina.out
stopasgroup=false     ;默认为false,进程被杀死时，是否向这个进程组发送stop信号，包括子进程
killasgroup=false     ;默认为false，向进程组发送kill信号，包括子进程

;包含其它配置文件
[include]
files = relative/directory/*.ini    ;可以指定一个或多个以.ini结束的配置文件

include示例：
[include]
files = /opt/absolute/filename.ini /opt/absolute/*.ini foo.conf 
```