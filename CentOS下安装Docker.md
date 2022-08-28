# CentOS 下使用 Docker搭建基础设施

## 安装 Docker

### 先决条件

只支持CentOS 7，8，9，之前的版本不支持

```shell
# 查看 CentOS 版本
cat /etc/redhat-release
```



## 删除旧版本

```shell
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                 
```

```shell
sudo yum install docker-ce-20.10.9 docker-ce-cli-20.10.9 containerd.io docker-compose-plugin
```

```shell
docker run -d -e TZ="Asia/Shanghai" -e MYSQL_USER="jmlx" -e MYSQL_ROOT_PASSWORD="-bk,9tI4N5)@ADZ" --name mysql -p 3306:3306 mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```



## 参考

https://docs.docker.com/engine/install/centos/