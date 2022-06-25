# 初涉Nginx ---Nginx 安装与使用

## Nginx 反向代理

### 正向代理

客户端请求目标服务器之间的一个代理服务器，请求会先经过代理服务器，然后转发给目标服务器，获得内容后响应给客户端。**正向代理，其实是"代理服务器"代理了"客户端"，去和"目标服务器"进行交互，目标服务器不知道真正的客户端到底是谁。**

### 反向代理

是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。**其实就是客户端请求目标服务器时，由代理服务器决定访问哪个 IP。反向代理是"代理服务器"代理了"服务端" ，帮助服务器做负载均衡，安全防护等 **

## 使用yum 安装Nginx

### 添加 Nginx  源

Nginx 不在默认的 yum 源中，可以使用 epel 或者官网的 yum 源，本例使用官网的 yum 源。

```shell
sudo rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

- rpm是指Redhat Package Manager，用来管理软件包。
- -i ： install
- -v： 显示指令执行过程
- -h：以"#"号显示安装进度
- 也可以直接用 -Uvh , 这个命令可以用来更新包，-U 表示update

### 安装

```shell
sudo yum install -y nginx
```

- yum：是基于RPM的软件包管理器
- -y：对所有的提问都回答“yes”

### 启动服务

```shell
sudo systemctl start nginx.service
```

- systemctl：是系统服务管理器指令

### 停止服务

```shell
sudo systemctl restart nginx
```

### 设置开机自启

```shell
sudo systemctl enable nginx.service
```

### 重新加载

```shell
sudo systemctl reload nginx
```

Nginx 运行在80端口上，直接访问IP 就可以看到Nginx 的欢迎页，这样就代表安装成功

![nginx-welcome](https://img.zhsong.cn/blog-image/nginx-welcome.png)





