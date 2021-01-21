# 初涉Nginx ---Nginx 进程模型

使用命令

```shell
ps -ef | grep nginx
```

- ps：报告当前系统的进程状态
- -e：显示所有进程，与'-A' 一样
- -f：显示UID(程序被该 UID 所拥有),PID(进程ID),PPID(其上级父程序的ID),C(CPU 使用的资源百分比)与STIME(开始时间)栏位等

```shell
#UID      PID   PPID %CPU STIME  TTY  TIME(此进程运行的总时间)  CMD(命令名)
root      1452     1  0   15:46    ?   00:00:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
nginx     2017  1452  0   16:26    ?   00:00:00 nginx: worker process
```

可以看到这里有一个master进程和一个worker进程，在前面nginx.conf配置文件中worker_processes设置的就是wroker进程的数量，所以这里只有一个worker进程

