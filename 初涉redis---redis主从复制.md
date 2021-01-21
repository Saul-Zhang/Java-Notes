# 初涉redis---redis主从复制

## 配置

我是先准备了三台虚拟机，ip地址分别为192.168.56.101，192.168.56.102，192.168.56.103，搭建一主两从的结构。redis的安装与配置在前一篇[“初涉redis---redis安装配置”](http://zsly.xyz/archives/%E5%88%9D%E6%B6%89redis---redis%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE)。虚拟机新建了个虚拟机101，另外两台是根据这台复制的，所以环境都一样，都安装好了redis。

连接客户端后使用`info replication`命令可以查看当前节点的信息

```shell
# Replication
role:master
connected_slaves:0
...
```

因为没有配置主从关系，可以看出当前节点就是master节点，没有从节点。

首先要去修改两个从节点的配置文件`redis.conf`

- 用`/replicaof`搜索，这个配置默认是注释掉的，把他改成`replicaof 192.168.56.101 6379`，配置的内容是主节点的ip和端口
- 找到`masterauth` 放开注释，这是填主节点密码的

配置好后重启redis，再用`info replication`命令可以看到当前节点已经变成从节点了

```shell
# Replication
role:slave
master_host:192.168.56.101
master_port:6379
master_link_status:up
```

`master_link_status:up`表示连接上了主节点，如果是`dowm`的话看一下是不是主节点开了防火墙

```shell
# 查看防火墙状态
firewall-cmd --state # running表示开了防火墙
#关闭防火墙
systemctl stop firewalld.service
#设置开机自启动
systemctl enable firewalld.service
```

然后主节点上操作数据就会看到从节点上也会同样发生变化。



