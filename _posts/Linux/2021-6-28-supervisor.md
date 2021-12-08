---
title: Supervisor
category: Linux
typora-root-url: ../../
---

## supervisor介绍

首先，介绍一下supervisor。Supervisor（http://supervisord.org/）是用Python开发的一个client/server服务，是Linux/Unix系统下的一个进程管理工具，不支持Windows系统。它可以很方便的监听、启动、停止、重启一个或多个进程。用Supervisor管理的进程，当一个进程意外被杀死，supervisort监听到进程死后，会自动将它重新拉起，很方便的做到进程自动恢复的功能，不再需要自己写shell脚本来控制

环境：centos



> 超级难用，还是用golang的吧
>
> https://github.com/ochinchina/supervisord



## 安装supervisor

```
sudo yum install -y supervisor
```


supervisor安装完成后会生成三个执行程序：supervisortd、supervisorctl、echo_supervisord_conf，分别是supervisor的守护进程服务（用于接收进程管理命令）、客户端（用于和守护进程通信，发送管理进程的指令）、生成初始配置文件程序。

 

## 配置supervisor

配置文件目录

```
/etc/supervisord.d
```

配置文件

```
/etc/supervisord.conf
```





## 主配置文件参数

mkdir /var/run/supervisor/

```
[root@game-server etc]# vi /etc/supervisord.conf 
; Sample supervisor config file.

[unix_http_server]
file=/var/run/supervisor/supervisor.sock   ; (the path to the socket file)
;chmod=0700                 ; sockef file mode (default 0700)
;chown=nobody:nogroup       ; socket file uid:gid owner
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

;[inet_http_server]         ; inet (TCP) server disabled by default
;port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

[supervisord]
logfile=/var/log/supervisor/supervisord.log  ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB       ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10          ; (num of main logfile rotation backups;default 10)
loglevel=info               ; (log level;default info; others: debug,warn,trace)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
nodaemon=false              ; (start in foreground if true;default false)
minfds=1024                 ; (min. avail startup file descriptors;default 1024)
minprocs=200                ; (min. avail process descriptors;default 200)
;umask=022                  ; (process file creation umask;default 022)
;user=chrism                 ; (default is current user, required if root)
;identifier=supervisor       ; (supervisord identifier, default is 'supervisor')
;directory=/tmp              ; (default is not to cd during start)
;nocleanup=true              ; (don't clean up tempfiles at start;default false)
;childlogdir=/tmp            ; ('AUTO' child log dir, default $TEMP)
;environment=KEY=value       ; (key value pairs to add to environment)
;strip_ansi=false            ; (strip ansi escape codes in logs; def. false)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor/supervisor.sock ; use a unix:// URL  for a unix socket
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
;username=chris              ; should be same as http_username if set
;password=123                ; should be same as http_password if set
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")
;history_file=~/.sc_history  ; use readline history if available

; The below sample program section shows all possible program subsection values,
; create one or more 'real' program: sections to be able to control them under
; supervisor.

;[program:theprogramname]
;command=/bin/cat              ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=999                  ; the relative start priority (default 999)
;autostart=true                ; start at supervisord start (default: true)
;autorestart=true              ; retstart at unexpected quit (default: true)
;startsecs=10                  ; number of secs prog must stay running (def. 1)
;startretries=3                ; max # of serial start failures (default 3)
;exitcodes=0,2                 ; 'expected' exit codes for process (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=true          ; redirect proc stderr to stdout (default false)
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A=1,B=2           ; process environment additions (def no adds)
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample eventlistener section shows all possible
; eventlistener subsection values, create one or more 'real'
; eventlistener: sections to be able to handle event notifications
; sent by supervisor.

;[eventlistener:theeventlistenername]
;command=/bin/eventlistener    ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;events=EVENT                  ; event notif. types to subscribe to (req'd)
;buffer_size=10                ; event buffer queue size (default 10)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=-1                   ; the relative start priority (default -1)
;autostart=true                ; start at supervisord start (default: true)
;autorestart=unexpected        ; restart at unexpected quit (default: unexpected)
;startsecs=10                  ; number of secs prog must stay running (def. 1)
;startretries=3                ; max # of serial start failures (default 3)
;exitcodes=0,2                 ; 'expected' exit codes for process (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=true          ; redirect proc stderr to stdout (default false)
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups        ; # of stderr logfile backups (default 10)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A=1,B=2           ; process environment additions
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample group section shows all possible group values,
; create one or more 'real' group: sections to create "heterogeneous"
; process groups.

;[group:thegroupname]
;programs=progname1,progname2  ; each refers to 'x' in [program:x] definitions
;priority=999                  ; the relative start priority (default 999)

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[include]
files = supervisord.d/*.ini
```

　　

## 管理一个进程

下面创建一个配置文件

```
mkdir /home/zesen/game_server/stdlog

vi /etc//supervisord.d/slots.ini
```



```
#项目名
[program:slots]
#脚本目录
directory=/home/zesen/game_server
#脚本执行命令
command=/home/zesen/game_server/slots_server --config=/home/zesen/game_server/config_online.json --srvName=A_node1_v1

#supervisor启动的时候是否随着同时启动，默认True
autostart=true
#当程序exit的时候，这个program不会自动重启,默认unexpected，设置子进程挂掉后自动重启的情况，有三个选项，false,unexpected和true。如果为false的时候，无论什么情况下，都不会被重新启动，如果为unexpected，只有当进程的退出码不在下面的exitcodes里面定义的
autorestart=true
#这个选项是子进程启动多少秒之后，此时状态如果是running，则我们认为启动成功了。默认值为1
startsecs=10

#脚本运行的用户身份 
user = zesen

#关闭时候发送信号
stopsignal=TERM

#日志输出 
stderr_logfile=/home/zesen/game_server/stdlog/blog_stderr.log 
stdout_logfile=/home/zesen/game_server/stdlog/blog_stdout.log 
#把stderr重定向到stdout，默认 false
redirect_stderr = true
#stdout日志文件大小，默认 50MB
stdout_logfile_maxbytes = 50MB
#stdout日志文件备份数
stdout_logfile_backups = 10
```



## 启动Supervisor

```
supervisord -c /etc/supervisord.conf 
```

> 注意要启动的进程不能后台启动， 否则supervisord监控不到，会一直重启

 

## 查看状态

```
supervisorctl status
slots                            RUNNING   pid 28467, uptime 0:00:43
```



## 常用命令

```
查看当期进程状态
supervisorctl status
停止一个进程
supervisorctl stop <name>
supervisorctl stop all
启动
supervisorctl start <name>
supervisorctl start all
重启
supervisorctl restart <name>
supervisorctl restart all
重启supervisord主进程
supervisorctl reload
```

 

## web界面管理

开启web访问

 

```
vim /etc/supervisor/supervisord.conf
  [inet_http_server]        
  port=0.0.0.0:9001       
  username=user            
  password=123   
```

 

![img](https://images2018.cnblogs.com/blog/1334074/201806/1334074-20180630131258681-1838599973.png)

 

 

 

------

 

下面开始说报警的事，有些时候，进程莫名其妙的退出了，然后又立刻被supervisor给拉起来了，导致了一些问题出现，想立刻知道这个进程已经被重启过了怎么办？这时候 就可以用superlance来了

 

## superlance介绍

superlance就是基于supervisor的事件机制实现的一系列命令行的工具集，它实现了许多supervisor本身没有实现的实用的进程监控和管理的特性，包括内存监控，http接口监控，邮件和短信通知机制等。同样的，superlance本身也是使用python编写的

> 超级难用，还是用golang的吧
>
> https://github.com/ouqiang/supervisor-event-listener



## superlance命令

superlance是一系列命令行工具的集合，其包括以下这些命令：

- - httpok
    通过定时对一个HTTP接口进行GET请求，根据请求是否成功来判定一个进程是否处于正常状态，如果不正常则对进程进行重启。
  - crashmail
    当一个进程意外退出时，发送邮件告警。
  - memmon
    当一个进程的内存占用超过了设定阈值时，发送邮件告警。
  - crashmailbatch
    类似于crashmail的告警，但是一段时间内的邮件将会被合成起来发送，以避免邮件轰炸。
  - fatalmailbatch
    当一个进程没有成功启动多次后会进入FATAL状态，此时发送邮件告警。与crashmailbatch一样会进行合成报警。
  - crashsms
    当一个进程意外退出时发送短信告警，这个短信也是通过email网关来发送的



```
1.当supervisord启动的时候，如果我们的listener配置为autostart=true的话，listener就会作为supervisor的子进程被启动。

2.listener被启动之后，会向自己的stdout写一个"READY"的消息,此时父进程也就是supervisord读取到这条消息后，会认为listener处于就绪状态。

3.listener处于就绪状态后，当supervisord产生的event在listener的配置的可接受的events中时，supervisord就会把该event发送给该listener。

4.listener接收到event后，我们就可以根据event的head，body里面的数据，做一系列的处理了。我们根据event的内容，判断，提取，报警等等操作。

5.该干的活都干完之后，listener需要向自己的stdout写一个消息"RESULTnOK"，supervisord接受到这条消息后。就知道listener处理event完毕了。
```



## Supervisord支持的Event

 

```
PROCESS_STATE    进程状态发生改变
PROCESS_STATE_STARTING  进程状态从其他状态转换为正在启动(Supervisord的配置项中有startsecs配置项， 是指程序启动时需要程序至少稳定运行x秒才认为程序运行正常，在这x秒中程序状态为正在启动)
PROCESS_STATE_RUNNING   进程状态由正在启动转换为正在运行
PROCESS_STATE_BACKOFF   进程状态由正在启动转换为失败
PROCESS_STATE_STOPPING   进程状态由正在运行转换为正在停止
PROCESS_STATE_EXITED   进程状态由正在运行转换为退出
PROCESS_STATE_STOPPED   进程状态由正在停止转换为已经停止(exited和stopped的区别是exited是程序自行退出，而stopped为人为控制其退出)
PROCESS_STATE_FATAL   进程状态由正在运行转换为失败
PROCESS_STATE_UNKNOWN   未知的进程状态
REMOTE_COMMUNICATION   使用Supervisord的RPC接口与Supervisord进行通信
PROCESS_LOG   进程产生日志输出，包括标准输出和标准错误输出
PROCESS_LOG_STDOUT   进程产生标准输出
PROCESS_LOG_STDERR   进程产生标准错误输出
PROCESS_COMMUNICATION   进程的日志输出包含 和
PROCESS_COMMUNICATION_STDOUT   进程的标准输出包含 和
PROCESS_COMMUNICATION_STDERR   进程的标准错误输出包含 和
SUPERVISOR_STATE_CHANGE_RUNNING Supervisord  启动
SUPERVISOR_STATE_CHANGE_STOPPING Supervisord  停止
TICK_5   每隔5秒触发
TICK_60   每隔60秒触发
TICK_3600   每隔3600触发
PROCESS_GROUP   Supervisord的进程组发生变化
PROCESS_GROUP_ADDED   新增了Supervisord的进程组
PROCESS_GROUP_REMOVED   删除了Supervisord的进程组
```

　　

## 安装监听

既然有了上面的event特性，下面就配置一个发邮件报警，当nginx莫名其妙的重启后 就立刻发邮件通知。



## 配置监听

```
vi /etc/supervisord.d/listener.ini

[eventlistener:supervisor-event-listener]
command=/home/zesen/listener/supervisor-event-listener -c /home/zesen/listener/supervisor-event-listener.ini
events=PROCESS_STATE_EXITED,PROCESS_STATE_FATAL,TICK_5
stdout_logfile=/home/zesen/listener/listener_stdout.log
stderr_logfile=/home/zesen/listener/listener_stderr.log
user=root
```

　　

添加好一个进程配置文件后，supervisorctl reload 重启一下

 

![img](https://images2018.cnblogs.com/blog/1334074/201806/1334074-20180630131635752-825548858.png)

 

 

已经是两个进程在running了

下面测试一下 kill 掉nginx进程

 

```
 ps aux | grep nginx
 kill -9 17659 17660 
 
```

![img](https://images2018.cnblogs.com/blog/1334074/201806/1334074-20180630131655693-2032899871.png)

 

 

然后看一下supervisor

 

![img](https://images2018.cnblogs.com/blog/1334074/201806/1334074-20180630131708946-1344241569.png)

 

 

此时 nginx pid已经变化，说明kill之后 又被拉起来了。

 

![img](https://images2018.cnblogs.com/blog/1334074/201806/1334074-20180630131719074-158315056.png)

 

 

也很快 就收到邮件报警了。嘿嘿。。