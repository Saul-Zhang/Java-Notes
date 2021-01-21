# 使用crontab将halo博客定时备份到github

## 前言

目前正在使用的博客是halo，https://github.com/halo-dev/halo，博客部署在阿里云上，使用的centos7系统。

halo关于博客的所有资料都在~/.halo目录下，包括用户配置文件，数据库文件，主题相关文件，这个文件非常重要，所以我打算用Linux的定时任务把这个文件上传到github上。

## crontab

> **crontab命令** 被用来提交和管理用户的需要周期性执行的任务，与windows下的计划任务类似，当安装完成操作系统后，默认会安装此服务工具，并且会自动启动crond进程，crond进程每分钟会定期检查是否有要执行的任务，如果有要执行的任务，则自动执行该任务。

### 创建任务文件

```shell
crontab -e
```

使用这个命令后，就会打开一个可编辑的文件，默认使用的编辑器是vi，所以可以按a或者i进入编辑。如果没有写过定时任务，文件是空的，否则也可以看到以前加入的定时任务。

### 创建任务

```shell
00 03 * * *  /root/.halo/auto_commit.sh
```

这个任务表示每天早上3点，执行/root/.halo/auto_commit.sh这个脚本。然后按esc，输入!wq保存。

### 自动提交到github

这里有几个前置步骤要做

- 到github上创建仓库
- 初始化仓库  `cd /root/.halo & git init`
- 去github配置ssh key，目的是push的时候不用输入用户名和密码
- 添加远程仓库到本地  `git remote add origin {你的仓库地址}`
- 向远程仓库推送`git push -u origin master `

然后创建auto_commit.sh脚本，在.halo文件夹下vim atuo_commit.sh

```shell
#! /bin/bash
message=`date +"%Y_%m_%d_%H_%M"`
 
cd /root/.halo
 
git add application.yaml upload/ db/ templates/
 
git commit -m $message
 
git push

```

要注意的是，如果没有在博客后台上传过附件，upload这文件夹是不存在的，所以要把脚本中upload/删掉（或者最好去博客后台上传一个附件）。

脚本写好后

- 给脚本添加可执行权限 `chmod +x atuo_commit.sh`
- 然后直接执行一下脚本看不能正常上传到github上
- 最后执行`service crond start`就可以了

## 关于crontab

`cd /etc/crontab`可以看到配置文件	

``` shell
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

```

从文件中可以看到每个字段的含义，分别表示

minute   hour   day   month   week  user-name command    他的顺序是 分 时 日 月 周

我们写的时间是`00 03 * * *`，第一位是`00`表示分钟，第二位是`03`表示小时，所以两位表示的意思就是03:00这个时间。第三位`*`表示一个月中每一天，第四位`*`表示每月，第五位`*`表示一个星期中每一天

### 特殊字符的含义

- `*`  ：代表所有可能的值，例如第四位month是星号，则表示每月都执行该命令操作
- `-` ：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”
- `,`： 可以用逗号隔开的值指定一个列表范围，例如，如果第二位hour是“1,2,5”则表示，1点，2点，5点
- `/` ：正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，例如*/10，如果用在第一位minute字段，表示每十分钟执行一次。

### 使用实例

每1分钟执行一次command

```shell
* * * * * command
```

在上午8点到11点的第3和第15分钟执行command

```shell
3,15 8-11 * * * command
```

每隔两天的上午8点到11点的第3和第15分钟执行command

```shell
3,15 8-11 */2 * * command
```

## crond服务

查看crontab服务状态：

```shell
service crond status
```

手动启动crontab服务：

```shell
service crond start
```

## 参考

- https://wangchujiang.com/linux-command/c/crontab.html

- [记一次拯救我的博客](https://ryanc.cc/archives/record-how-to-save-my-blog)