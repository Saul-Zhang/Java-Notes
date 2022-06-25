# 初涉Nginx ---Nginx配置文件

安装完成后可以通过命令

```shell
whereis nginx
```

- whereis：查找二进制程序、代码等相关文件路径

查看Nginx的相关目录，可以看到如下内容

```shell
nginx: /usr/sbin/nginx /usr/lib64/nginx /etc/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz
```

- /usr/sbin/nginx：这是Nginx 的启动脚本
- /usr/lib64/nginx：Nginx 模块目录
- /etc/nginx：Nginx 配置路径
- /usr/share/nginx：存放了Nginx 首页html
- /usr/share/man/man8/nginx.8.gz：Nginx 的帮助文档

查看Nginx 的默认配置文件，采用不同的安装方式配置文件的位置是不一样的

```shell
cat /etc/nginx/conf.d/default.conf
```

可以看到去掉注释后大概内容如下

```shell
server {
    listen       80;
    server_name  localhost;
    
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

```

- server：配置虚拟主机，server可以有多个
- listen：监听的端口
- server_name：监听的地址，写多个，以空格分隔。比如有一个www.zs.xyz这样的域名就可以写在这里，这样就可以通过这个域名加监听的端口访问这台主机了。
- location：请求的url 过滤  "/" 表示匹配所有请求，因为所有的地址都是以"/"开头
- root：根目录
- index：默认访问的页面，这个页面是在根目录中

conf.d 下存放的只是配置文件的一部分，在nginx.conf 文件中存放了配置文件的公共部分

```shell
cat /etc/nginx/nginx.conf
```

```shell
#可以定义Nginx运行的用户和用户组
user  nginx;
#nginx 进程数，通常设置成和cpu的数量相等
worker_processes  1;

#全局错误日志定义类型，[debug | info | notice | warn | error | crit]
error_log  /var/log/nginx/error.log warn;
#进程pid文件
pid        /var/run/nginx.pid;

events {
	#单个进程最大连接数
    worker_connections  1024;
}

#设定http服务器，利用它的反向代理功能提供负载均衡支持
http {
 	#文件扩展名与文件类型映射表
    include       /etc/nginx/mime.types;
    #默认文件类型
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;
	#开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，
	#如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。
	#注意：如果图片显示不正常把这个改成off。
    sendfile        on;
    
	#长连接超时时间，单位是秒
    keepalive_timeout  65;

	# 包含其他的配置文件
    include /etc/nginx/conf.d/*.conf;
}
```

- Nginx配置文件的结构
  ![nginx-config](https://img.zhsong.cn/blog-image/nginx-config.png)
