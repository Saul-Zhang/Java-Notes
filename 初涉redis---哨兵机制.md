# 初涉redis---哨兵机制

在主从复制模式下，__主节点可以读写数据，但是从节点只能读数据而不能写__，如果在从节点写数据就会报`(error) READONLY You can't write against a read only replica`这种错误。所以在这种模式下，如果主节点宕机了，那么整个redis就没法写数据了，这个时候可以引入哨兵机制。

## 配置

主要是配置`sentinel.conf`问价，按照前面的配置，我把这个文件从源码包`/home/software/redis-6.0.9`中复制到`/usr/local/redis/`这个文件夹下

- 去掉`protected-mode no`的注释
- `daemonize yes`让服务在后台运行
- 修改一下日志文件的位置`logfile "/usr/local/redis/sentinel/redis-sentinel.log"`，如果服务没有启动到时候可以去看日志文件
- sentinel auth-pass {master节点名称} {master节点的密码}
- 配置工作目录`dir /usr/local/redis/sentinel`，要记得创建sentinel文件夹`mkdir sentinel`
- sentinel monitor {自定义的master节点的名称} {master节点ip} {master节点的端口} {需要几个哨兵投票才能切换master节点}，比如我配置成 `sentinel monitor zs-master 192.168.56.101 6379 2`，master节点名称默认是`mymaster`，用默认的比较简单，否则还有一些配置使用的默认的名是`mymaster`，这些也要改
- `sentinel down-after-milliseconds zs-master 30000`节点名改成自己设置的
- `sentinel parallel-syncs zs-master 1`节点名改成自己设置的
- `sentinel failover-timeout zs-master 180000`节点名改成自己设置的，含义是如果180s后主节点仍没有响应，就从剩下的slave中选一个升级为master

把配置文件分别拷贝给另外两个主机。

```shell
scp sentinel.conf root@192.168.56.102:/usr/local/redis
scp sentinel.conf root@192.168.56.103:/usr/local/redis
```

接下来就可以启动哨兵了，直接使用

```shell
redis-sentinel sentinel.conf #要指定配置文件的位置
```

这里一定要注意**哨兵开启之后配置文件是会发生变化的**，并且在配置文件的最后还会生成哨兵的相关信息。所以哨兵启动之后就不要把哨兵使用的配置文件再copy给其他哨兵使用了，这个时候应该从源码包`/home/software/redis-6.0.9`中去复制

使用`ps -ef | grep redis`

![redis-ps](https://img.zhsong.cn/blog-image/redis-ps.png)

这样哨兵就启动成功了，如果不成功去看一下前面配的日志`/usr/local/redis/sentinel/redis-sentinel.log`。

`tailf sentinel/redis-sentinel.log `，这样就是正常的

```shell
1789:X 22 Dec 2020 21:38:19.120 # Sentinel ID is efc2c6130df753b1151da8ae4c1018c9f4d31eec
1789:X 22 Dec 2020 21:38:19.120 # +monitor master zs-master 192.168.56.101 6379 quorum 2
1789:X 22 Dec 2020 21:38:19.121 * +slave slave 192.168.56.103:6379 192.168.56.103 6379 @ zs-master 192.168.56.101 6379
1789:X 22 Dec 2020 21:38:19.135 * +slave slave 192.168.56.102:6379 192.168.56.102 6379 @ zs-master 192.168.56.101 6379
```

日志就这样放着，然后再分别去开启另外两个节点的哨兵，开启之后发现日志有了变化，有另外两个哨兵接入了

```shell
1789:X 22 Dec 2020 22:20:21.999 * +sentinel sentinel 8bd9c4d4df0b069375e1ed2d5ef2467c7cdb25f2 192.168.56.102 26379 @ zs-master 192.168.56.101 6379
1789:X 22 Dec 2020 22:20:47.072 * +sentinel sentinel 0dff81c1c7b4a4317cd1add24430db92193773d4 192.168.56.103 26379 @ zs-master 192.168.56.101 6379
```

## 查看哨兵状态

接下来可以检查一下哨兵是否正常

可以在任意一台主机使用`redis-cli `指定端口连接sentinel

```shell
redis-cli -p 26379
```

然后可以查看哨兵的状态

```shell
info sentinel
```

```shell
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=zs-master,status=ok,address=192.168.56.101:6379,slaves=2,sentinels=3
```

最后一行`sentinels=3`表示目前有三个哨兵在工作，如果哨兵数量不对的话，检查一下配置是否正确，再查看一几个哨兵的`sentinel myid `是否不同，这个是打开哨兵之后配置文件自动生成的，**这个id必须是不同的**，如果id相同的话删掉这个配置再重启哨兵就可以了。

## 查看哨兵的工作效果

一切正常后来查看一下如果主节点宕机了，是否能正常切换节点

在主节点上关掉redis

```shell
/etc/init.d/redis_init_script stop 
```

3分钟后日志会发生变化

```shell
1789:X 22 Dec 2020 22:55:00.682 # +new-epoch 6
# 主节点宕机了
1789:X 22 Dec 2020 22:55:00.878 # +sdown master zs-master 192.168.56.101 6379
# 3个节点中有两个检测到主机点宕机了
1789:X 22 Dec 2020 22:55:00.980 # +odown master zs-master 192.168.56.101 6379 #quorum 3/2
1789:X 22 Dec 2020 22:55:00.980 # +new-epoch 7
1789:X 22 Dec 2020 22:55:00.980 # +try-failover master zs-master 192.168.56.101 6379
1789:X 22 Dec 2020 22:55:01.150 # +new-epoch 8
# 投票8bd9c4d4df0b069375e1ed2d5ef2467c7cdb25f2（这是102节点哨兵的id）成为哨兵中的leader
1789:X 22 Dec 2020 22:55:01.153 # +vote-for-leader 8bd9c4d4df0b069375e1ed2d5ef2467c7cdb25f2 8
# 更新102节点的配置文件
1789:X 22 Dec 2020 22:55:01.873 # +config-update-from sentinel 8bd9c4d4df0b069375e1ed2d5ef2467c7cdb25f2 192.168.56.102 26379 @ zs-master 192.168.56.101 6379
# 102成为主节点
1789:X 22 Dec 2020 22:55:01.873 # +switch-master zs-master 192.168.56.101 6379 192.168.56.102 6379
# 以下是从节点的信息
1789:X 22 Dec 2020 22:55:01.873 * +slave slave 192.168.56.103:6379 192.168.56.103 6379 @ zs-master 192.168.56.102 6379
1789:X 22 Dec 2020 22:55:01.873 * +slave slave 192.168.56.101:6379 192.168.56.101 6379 @ zs-master 192.168.56.102 6379
1789:X 22 Dec 2020 22:55:31.941 # +sdown slave 192.168.56.101:6379 192.168.56.101 6379 @ zs-master 192.168.56.102 6379
```

此时表明主节点从101切换到了102。再打开102的redis客户端看一下主从节点的信息

```shell
#连接客户端redis-cli -a {密码}，执行 info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.56.103,port=6379,state=online,offset=1015963,lag=1
...
```

可以看出102是主节点，从节点只有103。

接着再打开101节点的redis

日志会发生变化

```shell
1789:X 22 Dec 2020 23:05:55.431 * +convert-to-slave slave 192.168.56.101:6379 192.168.56.101 6379 @ zs-master 192.168.56.102 6379
```

101节点变成了从节点。

再打开102的redis客户端看一下主从节点的信息