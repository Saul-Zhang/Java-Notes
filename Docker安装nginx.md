# Docker 安装Nginx

```shell
docker run --name nginx -p 80:80 -d nginx
```

将容器里的配置文件拷贝到宿主机

```
docker cp nginx:/etc/nginx/nginx.conf /docker/nginx/nginx.conf
docker cp nginx:/etc/nginx/conf.d/ /docker/nginx/conf.d/
```

然后删除容器

```
docker rm -f nginx
```

## 启动 nginx

```
docker run \
    -p 80:80 \
    -p 443:443 \
    -v /docker/nginx/nginx.conf:/etc/nginx/nginx.conf/:rw \
    -v /docker/nginx/conf.d:/etc/nginx/conf.d:rw \
    -v /docker/nginx/logs:/var/log/nginx/:rw \
    -v /docker/nginx/ssl:/ssl/:rw \
    --restart=always \
    --name=nginx \
    -d nginx
```

**参数：**

- **-p 80:80** 将宿主机的80端口映射到容器的80端口，用于http请求

- **-p 443:443** 将宿主机的443端口映射到容器的443端口，用于https请求

- **–restart=always** 保证docker重启的时候，这个容器跟着启动

- **-v /root/nginx/nginx.conf:/etc/nginx/nginx.conf/:rw** 把宿主机的`/root/nginx/nginx.conf`文件挂载到容器的`/etc/nginx/nginx.conf`文件，赋予读写权限

- **-v /root/nginx/conf.d:/etc/nginx/conf.d:rw** 把宿主机的`/root/nginx/conf.d`目录挂载到容器的`/etc/nginx/conf.d`目录，赋予读写权限

- **-v /root/nginx/logs:/var/log/nginx/:rw** 把宿主机的`/root/nginx/logs`目录挂载到容器的`/var/log/nginx/`目录，赋予读写权限

- **-v /root/ssl:/ssl/:rw** 把宿主机的`/root/ssl`目录挂载到容器的`/ssl/`目录，赋予读写权限

- **–name=nginx** 容器名

- **-d** 后台运行

至此，nginx就在docker中部署好了。

日常使用，就在`/docker/nginx/conf.d`目录下添加或者修改相应的conf文件就OK了

## 常见问题

### 部署好后只能访问80和443端口，其他端口访问不了

这是因为我们只映射了这两个端口，如果还要nginx监听其他端口，比如我要监听8081端口，需要重新配置一下nginx的端口映射

1.先停止nginx容器

```
docker stop {容器名}
如 docker stop nginx
```

2.重新生成一个镜像文件，这一步是不会丢失文件的

```
docker commit {nginx容器id或者名称} {新的容器名称}
如 docker commit nginx mynginx
```

3.启动新的容器

```
docker run \
    -p 80:80 \
    -p 443:443 \
    -p 8081:8081 \ 
    -v /docker/nginx/nginx.conf:/etc/nginx/nginx.conf/:rw \
    -v /docker/nginx/conf.d:/etc/nginx/conf.d:rw \
    -v /docker/nginx/logs:/var/log/nginx/:rw \
    -v /docker/nginx/ssl:/ssl/:rw \
    --restart=always \
    --name=mynginx \
    -d  mynginx
```

这里我新映射了8081端口

4.移除旧的容器，这步要却道新容器已经运行成功了再执行

```
　docker rm 旧容器名称
　如: docker rm nginx　
```

