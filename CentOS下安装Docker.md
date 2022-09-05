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
docker run -d -e TZ="Asia/Shanghai" -e MYSQL_ROOT_PASSWORD="-bk,9tI4N5)@ADZ" --name mysql -p 3306:3306 mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

minio

```shell
docker run -p 9000:9000 -p 9001:9001 --name minio -d \                                                                                                 
  -e "MINIO_ACCESS_KEY=ueZSzbbjwfy" \
  -e "MINIO_SECRET_KEY=UHUOpcR~B40cWn_yP" \
  -v /usr/local/docker/minio/data:/data \
  -v /usr/local/docker/minio/config:/root/.minio \
  minio/minio server /data --console-address ":9001"
```

mysql

```shell
docker run -d \                                                                                                                                        
-e TZ="Asia/Shanghai" \
-e MYSQL_ROOT_PASSWORD="-bk,9tI4N5)@ADZ" \
--name mysql \
-p 3306:3306 \
-v /usr/local/docker/mysql/data:/var/lib/mysql \
mysql \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_unicode_ci
```





## 参考

https://docs.docker.com/engine/install/centos/