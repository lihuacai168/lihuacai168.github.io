---
title: FasterRunner(后端)生产模式uWSGI+Nginx+Supervisor+Python虚拟环境部署
date: 2019-11-07 11:35:02
tags:
- FasterRunner
- uWSGI
- Supervisor
- Nginx
categories:

- FasterRunner
---
# 安装`uWSGI`
```
pip install uWSGI
```

# 配置`uWSGI`

## 编辑uWSGI配置文件

打开配置文件:`vim uwsgi.ini`,然后配置相关信息
```
[uwsgi]
# 设置变量
project = FasterRunner
base = /home/faster

# 通过引用变量的方式设置Django项目根路径
chdir = %(base)/%(project)
# Django项目的wsgi配置
module = %(project).wsgi:application

master = true
processes = 4

# socket 通信文件
socket = %(base)/%(project)/%(project).sock
chmod-socket = 666
vacuum = true
```

# 配置`Django`
[Django设置多配置文件(生产和开发)](https://www.jianshu.com/p/90a661a2c9de)
打开`Settings`目录下的`wsgi.py`文件
```
import os

from django.core.wsgi import get_wsgi_application

# 指定Django使用生产配置FasterRunner.settings.pro
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'FasterRunner.settings.pro')

application = get_wsgi_application()

```

# 调试`uWSGI`
```
# 进入Django项目的根路径
`cd /home/faster/FasterRunner`

# 运行uwsgi 
`uwsgi --ini ./uwsgi.ini`
```

# 配置`Nginx`
## 创建`uWSGI`配置文件
- 首先进入`Nginx`的`include`目录`/etc/nginx/conf.d`,这个目录下的`.conf`文件都会被`Nginx`自动导入 

- 创建`uWSGI`的配置文件`uwsgi.conf`,命名是随意的,只要后缀是`.conf`就行
```
server {
   # 监听8000端口
    listen       8000;
    server_name  10.0.3.57;

    location / {
        include         uwsgi_params;
        # 需要和uwsgi.ini 中的 socket 通信文件一样
        uwsgi_pass     unix:/home/faster/FasterRunner/FasterRunner.sock;
    }
}
```

# 测试Nginx

- 启动nginx `systemctl start nginx`

- 测试Nginx配置文件 `nginx -t`

- 重新加载,使Nginx配置文件生效 `nginx -s reload`

```
# 查看当前端口监听,确认8000端口被Nginx监听
(fasterenv) [root@faster_3_57 FasterRunner]# netstat -tunlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name     
tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN      27106/nginx: master 
```

# `Supervisor`管理`uWSGI`进程
[Supervisor守护进程虚拟环境部署Django](https://www.jianshu.com/p/9869cd1cbefe)

## 创建`Supervisor`中的`uWSGI`配置
- 创建`uWSGI`启动的配置文件`uwsgi.conf`
`command`中需要指定`uwsgi`的绝对路径
`Supervisor`配置的注释是`;`符号

```
[program:uwsgi]
command= /home/faster/.virtualenvs/fasterenv/bin/uwsgi --ini ./uwsgi.ini
directory=/home/faster/FasterRunner
stopasgroup=true
stopsignal=QUIT
user=faster
autostart=true
autorestart=true
stderr_logfile=/home/faster/err.log
stdout_logfile=/home/faster/out.log
```
## 通过`Supervisor`启动`uWSGI`
- `Supervisor`启动`supervisord -c  /etc/supervisord.conf`
  
- 重启所有管理的进程`supervisorctl reload`
- 单独启动`uWSGI`进程`supervisorctl start uwsgi`
- 停止`uWSGI`进程`supervisorctl stop uwsgi`


# `uWSGI`日志抛出异常`Permission denied`
```
tailf  /var/log/nginx/error.log

2019/06/15 15:41:01 [crit] 29077#0: *65 connect() to unix:/home/faster/FasterRunner/FasterRunner.sock failed (13: Permission denied) while connecting to upstream, 
client: 192.168.24.131, server: 10.0.3.57, request: "OPTIONS /api/user/login/ HTTP/1.1",
upstream: "uwsgi://unix:/home/faster/FasterRunner/FasterRunner.sock:", host: "10.0.3.57:8000", referrer: "http://10.0.3.57:8080/fastrunner/login"
```
## 修改Nginx启动用户
```
# 找到Nginx当前主配置
[root@faster_3_57 FasterRunner]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```
```
# 编辑Nginx主配置
vim /etc/nginx/nginx.conf
```
```
#user nginx;
user faster;
```
```
# 测试Nginx配置文件
nginx -t

# 重新加载,使Nginx配置文件生效  
nginx -s reload
```

## 异常原因分析
- 修改Nginx启动用户前,默认Nginx用户启动,因为权限问题,无法访问uWSGI的sock文件
```
(fasterenv) [root@faster_3_57 faster]# ps -ef|grep nginx
root     27106     1  0 15:01 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx    29077 27106  0 15:40 ?        00:00:00 nginx: worker process
root     29132 23308  0 15:41 pts/1    00:00:00 grep --color=auto nginx
```
- 查看uWSGI 启动用户,是faster
```
(fasterenv) [root@faster_3_57 faster]# ps -ef|grep uwsgi
faster   27354 28184  0 15:06 ?        00:00:00 /home/faster/.virtualenvs/fasterenv/bin/uwsgi --ini ./uwsgi.ini
faster   27358 27354  0 15:06 ?        00:00:00 /home/faster/.virtualenvs/fasterenv/bin/uwsgi --ini ./uwsgi.ini
faster   27359 27354  0 15:06 ?        00:00:00 /home/faster/.virtualenvs/fasterenv/bin/uwsgi --ini ./uwsgi.ini
faster   27360 27354  0 15:06 ?        00:00:00 /home/faster/.virtualenvs/fasterenv/bin/uwsgi --ini ./uwsgi.ini
faster   27361 27354  0 15:06 ?        00:00:00 /home/faster/.virtualenvs/fasterenv/bin/uwsgi --ini ./uwsgi.ini
root     29325 23308  0 15:45 pts/1    00:00:00 grep --color=auto uwsgi
```
- 修改Nginx启动用户为faster(和启动uWSGI的用户一样),就阔以正常访问啦~~~
```
[root@faster_3_57 FasterRunner]# ps -ef|grep nginx
root     27106     1  0 15:01 ?        00:00:00 nginx: master process /usr/sbin/nginx
faster   29451 27106  0 15:47 ?        00:00:00 nginx: worker process
root     32093 22403  0 16:37 pts/0    00:00:00 grep --color=auto nginx
```
```
[root@faster_3_57 FasterRunner]# tailf  /var/log/nginx/access.log
"http://10.0.3.57:8080/fastrunner/api_record/5" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36" "-"
192.168.24.131 - - [15/Jun/2019:16:40:09 +0800] "GET /api/fastrunner/tree/5/?token=4ee722ce4d8d5fdb9c5894879efb9760&type=1 HTTP/1.1" 200 966 "http://10.0.3.57:8080/fastrunner/api_record/5" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36" "-"
192.168.24.131 - - [15/Jun/2019:16:40:15 +0800] "GET /api/fastrunner/api/?token=4ee722ce4d8d5fdb9c5894879efb9760&node=66&project=5&search= HTTP/1.1" 200 13316 "http://10.0.3.57:8080/fastrunner/api_record/5" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36" "-"
192.168.24.131 - - [15/Jun/2019:16:40:18 +0800] "OPTIONS /api/user/login/ HTTP/1.1" 200 0 "http://10.0.3.57:8080/fastrunner/login" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36" "-"
192.168.24.131 - - [15/Jun/2019:16:40:18 +0800] "POST /api/user/login/ HTTP/1.1" 200 112 "http://10.0.3.57:8080/fastrunner/login" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36" "-"
192.168.24.131 - - [15/Jun/2019:16:40:18 +0800] "GET /api/fastrunner/project/?token=c4cd03206ac7e0deb42d1c2618624b75 HTTP/1.1" 200 593 "http://10.0.3.57:8080/fastrunner/project_list" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36" "-"
```