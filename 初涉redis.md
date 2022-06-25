# 初涉redis---redis安装配置

## 安装

进入redis的官网https://redis.io/

![redis-website](https://img.zhsong.cn/blog-image/redis-website.png)

点击直接可以下载，在windows和linux都能使用。本文主要介绍在linux上的使用

我在linux上使用了wget下载

```shell
wget https://download.redis.io/releases/redis-6.0.9.tar.gz
```

如果没有wget的话可以先用yum安装wget

```shell
yum -y install wget
```

redis下载好之后解压

```shell
tar -zxvf redis-6.0.9.tar.gz
```

进入redis文件夹

```shell
cd redis-6.0.9
```

要先安装gcc-c++

```shell
yum -y install gcc-c++
```

使用make编译

```shell
make
```

如果有这样的错误的话

```shell
server.c:5335:19: 错误：‘struct redisServer’没有名为‘supervised_mode’的成员
         if (server.supervised_mode == SUPERVISED_SYSTEMD) {
                   ^
server.c:5342:15: 错误：‘struct redisServer’没有名为‘maxmemory’的成员
     if (server.maxmemory > 0 && server.maxmemory < 1024*1024) {
               ^
server.c:5342:39: 错误：‘struct redisServer’没有名为‘maxmemory’的成员
     if (server.maxmemory > 0 && server.maxmemory < 1024*1024) {
                                       ^
server.c:5343:176: 错误：‘struct redisServer’没有名为‘maxmemory’的成员
         serverLog(LL_WARNING,"WARNING: You specified a maxmemory value that is less than 1MB (current value is %llu bytes). Are you sure this is what you really want?", server.maxmemory);

```

可能是gcc的版本太低了，升级一下gcc

```shell
gcc -v                             # 查看gcc版本
yum -y install centos-release-scl  # 升级到9.1版本
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
scl enable devtoolset-9 bash
以上为临时启用，如果要长期使用gcc 9.1的话：
echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
```

然后清理上次编译残留文件，重新编译，然后安装

```shell
make distclean  && make
make install
```

此时redis已经安装好了，并且可以用了

先打开服务端，客户端和服务端的打开脚本都在src文件中

```shell
src/resdis-server
```

![redis-start](https://img.zhsong.cn/blog-image/redis-start.png)

然后再开一个窗口打开客户端

```shell
src/redis-cli
```

![redis-cli](https://img.zhsong.cn/blog-image/redis-cli.png)

这样就OK了

## 配置

### 修改配置文件

创建一个目录将redis的配置文件复制过去

```shell
mkdir /usr/local/redis
cp redis.conf  /usr/local/redis
cd /usr/local/redis
vim redis.config
```

在命令模式下可以输入`/{你要查找的内容}`来搜索，回车后可以看到已经高亮了匹配到的字符，按`n`向下查找，按`N`向上查找

- 将`daemonize no`改为`daemonize yes`，让redis可以在后台运行

- 找到`dir ./`这个配置了redis的工作目录，持久化数据会存在这里，可以改为`dir /usr/local/redis/working`，这里配置的是一个目录，所以我们要记得去创建这个目录

- 将`bind 127.0.0.1`改为`bind 0.0.0.0`否则只有本机能访问redis，这样设置是外部机器都能访问redis服务端。bind设置的是本机的网卡对应的IP地址

- `requirepass foobared`这个是设置redis的密码的，默认是关闭的，可以去掉注释，`foobared`换成新的密码

### 修改启动脚本

将启动脚本复制到/etc/init.d文件夹下

```shell
cp utils/redis_init_script /etc/init.d
```

然后到`init.d`文件夹下打开`redis_init_script `

- 将`CONF="/etc/redis/${REDISPORT}.conf"`改为`CONF="/usr/local/redis/redis.conf"`这是配置文件的位置
- 配置开机自启动。加上`#chkconfig: 22345 10 90 ` `#description: Start and Stop redis` 

- 保存之后执行`chkconfig redis_init_script on`

  ```shell
  #!/bin/sh
  
  #chkconfig: 22345 10 90
  #description: Start and Stop redis
  
  REDISPORT=6379
  #服务端所处位置，在make install后默认存放与`/usr/local/bin/redis-server`，如果未make install则需要修改该路径，下同。
  EXEC=/usr/local/bin/redis-server
  CLIEXEC=/usr/local/bin/redis-cli
  
  PIDFILE=/var/run/redis_${REDISPORT}.pid
  CONF="/usr/local/redis/redis.conf"
  
  ```

  下面还有一行`$CLIEXEC -p $REDISPORT shutdown` 这里是停止redis使用的命令，如果我们设置了密码的话要更改这里`$CLIEXEC -a {你的密码} -p $REDISPORT shutdown`

  然后就可以启动redis了，这是redis启动的另一种方式

  ```shell
  ./redis_init_script start
  ```

  打开客户端

  ![redis-cli1](https://img.zhsong.cn/blog-image/redis-cli1.png)

  

  因为我们设置过密码，所以接下来在输入`auth {你的密码}`就可以了。也可以打开客户端的时候输入`redis-cli -a {你的密码}`

  

## redis的数据类型

redis有string，hash，list，set，zset五种数据类型。这里简单记一下常用命令，详细的可以查看http://redisdoc.com/

### string

```
set name zs
```

可以添加一条key为name，value为zs的数据

在输入set的时候，客户端会给出提示

![redis-hint](https://img.zhsong.cn/blog-image/redis-hint.png)

```
set key value [ex] [px] [nx|xx]
ex为键值设置秒级过期时间 
px为键值设置毫秒级过期时间 
nx是指key必须不存在,才可以设置成功,用于添加 
xx与nx相反,key必须存在,才可以设置成功,用于更新 
setnx setex 与上面的nx ex作用相同
```

比如`set name zs ex 10 nx`就表示设置的这条记录的过期时间为10s，并且原本不存在key为name的记录才能set成功。也可以写成

`setnx name zs `然后再`expire name 10 `。使用`ttl name`可以看到name这条数据当前还剩的过期时间

```shell
#可以在value值后拼接上112
append name 112
```

```shell
# 如果value为数字可以累加
incr age 
# 如果value为数字可以累减
decr age 
#加上指定值
incrby age 10 #age的值会被加上10
#截取字符串
getrange name 0 -1 #后边两个数表示截取的下标，-1表示最后一位
#字符串替换
setrange name 1 aa #会将name的第一位替换为aa
#设置多条记录
mset name1 zs1 name2 zs2
#取多条记录
mget name1 name2
```

### hash

```shell
#设置值
hset user name zs #相当于设置"user"：{"name":"zs"}
# 设置多个值，目前使用的redis6.0.9是可以直接用hset设置多个值的，redis5貌似是不可以的
hmset usr name zs age 18
# 列出当前对象所有key
hkeys user
#获取对象中某一个值
hget user name
#获得对象中所有值
hgetall user
```

### list

```shell
#从左插入
lpush list1 aa bb cc #数据顺序为 cc bb aa
#从右插入
rpush list2 aa bb cc #数据顺序为aa bb cc
#查看list中的值
lrange list1 0 -1 ## cc bb aa
# 从最左侧取一个值
lpop list1 #cc
# 从最右侧取一个值
rpop list1  # aa
# 查看list长度
llen list1 # 1 因为执行了两次pop
#查看你某一位的值
lindex list2 0 #第0位是aa

```

### set

set会自动去掉重复数据

```shell
#添加数据
sadd set1 aa bb cc aa
#查看set中的值
smembers set1
#从set中随机获取几个值
srandmember set1 2 #从set1中随机获取2个值
#把值从一个set移到领一个set
smove set2 set1 ff #把set2中的ff移动到set1中（不是复制是移动）
# 差集
sdiff set1 set2 #set1中有但是set2中没有
# 交集
sinter set1 set2
# 并集
sunion set1 set2
```

### zset

zset是有序的set，与set的区别每个值都有一个分数，并且会按照这个分数排序

```shell
#添加数据
zadd zset1 10 aa  20 cc 30 dd 40 ee #aa cc dd ee的分数分别是10 20 30 40
#查看所有值及分数
zrange zset1 0 -1 withscores
#查看某个值的位置
zrank zset1 aa
#查看某个值的分数
zscore zset1 aa
#查看值的个数
zcard zset1 #4
#查看某个分数直接的值
zrangebyscore zset1  20 30 # cc dd
```
