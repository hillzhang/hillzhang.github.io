---
layout: post
title: Sample Post
date: 2016-02-15 15:32:24.000000000 +09:00
---

## supervisor　安装　配置


### 安装 supervisor

```
easy_install supervisor
```

>supervisor安装完成后会生成三个执行程序：supervisortd、supervisorctl、echo_supervisord_conf，分别是supervisor的守护进程服务（用于接收进程管理命令）、客户端（用于和守护进程通信，发送管理进程的指令）、生成初始配置文件程序。


### 配置

>运行supervisord服务的时候，需要指定supervisor配置文件，如果没有显示指定，默认在以下目录查找：

```
$CWD/supervisord.conf
$CWD/etc/supervisord.conf
/etc/supervisord.conf
/etc/supervisor/supervisord.conf (since Supervisor 3.3.0)
../etc/supervisord.conf (Relative to the executable)
../supervisord.conf (Relative to the executabl

```
> $CWD表示运行supervisord程序的目录。

可以通过运行echo_supervisord_conf程序生成supervisor的初始化配置文件，如下所示：

```
mkdir /etc/supervisor
echo_supervisord_conf > /etc/supervisor/supervisord.conf
```
### 配置文件参数说明
>supervisor的配置参数较多，下面介绍一下常用的参数配置，详细的配置及说明，请参考官方文档介绍。 
注：分号（;）开头的配置表示注释

```bash
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

```

>include示例：

```
[include]
files = /opt/absolute/filename.ini /opt/absolute/*.ini foo.conf config??.ini
```
>例子　配置elasticsarch

```bash
[program:elasticsearch]
directory = /home/zhangls/elasticsearch-5.6.2
command = /home/zhangls/elasticsearch-5.6.2/bin/elasticsearch
autostart = true
startsecs = 5
user = zhangls
redirect_stderr = true
stdout_logfile = /var/log/supervisord/elasticsearch.log
```

### 启动　supervisor 服务

`supervisord -c /etc/supervisor/supervisord.conf`

### bash 终端

```
supervisorctl status
supervisorctl stop tomcat
supervisorctl start tomcat
supervisorctl restart tomcat
supervisorctl reread
supervisorctl update
```

### 开机启动supervisor 服务

#### 配置

1. 进入/lib/systemd/system目录，并创建supervisor.service文件

```bash

[root@nas-flow-01-0002 logstash-5.6.3]# cat /lib/systemd/system/supervisor.service
[Unit]
Description=supervisor
After=network.target
[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf
ExecStop=/usr/bin/supervisorctl shutdown
ExecReload=/usr/bin/supervisorctl reload
KillMode=process
Restart=on-failure
RestartSec=42s
[Install]
WantedBy=multi-user.target

```
2. 设置开机启动

```
systemctl enable supervisor.service
systemctl daemon-reload
```


> ==注意==
>> 1. supervisorctl reload 会重启所有的服务
>> 2. 用 supervisorcel reread 重新加载配置文件　然后通过 supervisorctl update　更新
