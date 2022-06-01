使用Centos7 环境安装 `nginx`:

打开 [nginx linux packages](https://nginx.org/en/linux_packages.html) 页面可以查看教程



安装 `yum-utils`:

```bash
yum install utils
```

使用vim添加配置：

```bash
[nginx-stable]
name=nginx stable repo
# 将$releasever 更改为对应的centos版本，我这里用的7，即
# baseurl=http://nginx.org/packages/centos/7/$basearch/
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

然后vim退出保存，然后可以查看一下nginx版本：

```bash
yum list | grep nginx
```

安装Nginx：

```bash
yum install nginx
```

安装完成后，可以查看版本：

```bash
nginx -v

# 打印
nginx version: nginx/1.22.0
```

使用大 `V` 查看Nginx编译参数：

```bash
nginx -V

# 打印
nginx version: nginx/1.22.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017 (running with OpenSSL 1.0.2o-fips  27 Mar 2018)
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'
```





## 1️⃣ 基本参数使用

包含：

1. Nginx安装目录
2. 编译参数
3. Nginx基本配置语法



### 👩🏻‍🏫 安装目录结构

在linux中都是安装的 `rpm` 包，可以使用 `rpm -ql nginx` 查看目录：

```bash
rpm -ql nginx
```

🚀 目录结构：

```bash
# linux中 etc 存放核心配置
# Nginx日志轮转，用于logrotate服务的日志切割
/etc/logrotate.d/nginx        

# 目录，配置文件
/etc/nginx                       
/etc/nginx/conf.d              # Nginx主配置文件   
/etc/nginx/nginx.conf
/etc/nginx/conf.d/default.conf # 安装Nginx后，默认配置文件，server加载的配置文件

# 配置文件 cgi配置相关 fastcgi配置
/etc/nginx/fastcgi_params
/etc/nginx/scgi_params
/etc/nginx/uwsgi_params

# 配置文件，设置http协议的 Content-Type 与扩展名对应关系
/etc/nginx/mime.types

# 目录： Nginx模块目录
/etc/nginx/modules
/usr/lib64/nginx/modules

# 用于配置系统守护进程管理器管理方式
/usr/lib/systemd/system/nginx-debug.service
/usr/lib/systemd/system/nginx.service

/usr/lib64/nginx
/usr/libexec/initscripts/legacy-actions/nginx
/usr/libexec/initscripts/legacy-actions/nginx/check-reload
/usr/libexec/initscripts/legacy-actions/nginx/upgrade

# 命令：Nginx服务的启动管理的终端命令
/usr/sbin/nginx
/usr/sbin/nginx-debug

# 文件、目录：Nginx的手册和帮助文件
/usr/share/doc/nginx-1.22.0
/usr/share/doc/nginx-1.22.0/COPYRIGHT
/usr/share/man/man8/nginx.8.gz

/usr/share/nginx
/usr/share/nginx/html
/usr/share/nginx/html/50x.html
/usr/share/nginx/html/index.html

# 目录：Nginx的缓存目录（Nginx除了HTTP代理外，还可以做缓存）
/var/cache/nginx

# 目录：Nginx的日志目录
/var/log/nginx
```



### 💡 编译参数

通过 `nginx -V` 可以查看编译参数：

```bash
nginx -V

# 打印
nginx version: nginx/1.22.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017 (running with OpenSSL 1.0.2o-fips  27 Mar 2018)
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'
```

下面对这些参数作用做说明：

**📚安装目录或路径**

```bash
--prefix=/etc/nginx                       # 定义Nginx的主目录
--sbin-path=/usr/sbin/nginx               # Nginx命令目录
--module-path=/usr/lib64/nginx/modules    # Nginx模块目录
--conf-path=/etc/nginx/nginx.conf         # 配置文件目录
--error-log-path=/var/log/nginx/error.log # 错误日志目录
--http-log-path=/var/log/nginx/access.log # 访问日志
--pid-path=/var/run/nginx.pid             # 记录Nginx启动的pid（进程id）
--lock-path=/var/run/nginx.lock           # 锁目录
```

**📁 执行对应模块时，临时性保留的文件**：

```bash
--http-client-body-temp-path=/var/cache/nginx/client_temp
--htpp-proxy-temp-path=/var/cache/nginx/proxy_temp
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp
--http-scgi-temp-path=/var/cache/nginx/scgi_temp
```

**👷‍♂️ 设定Nginx进程启动的用户和组用户**：

```bash
--user=nginx
--group=nginx
```

**📻 c语言编译器，优化参数**:

```bash
--with-cc-opt=parameters # 设置额外的参数将被添加到CFLAGS变量
```

**设置附加的参数，链接系统库**：

```bash
--with-ld-otp=parameters
```

 

### 📚 基础配置语法

先进入Nginx主目录：

```bash
cd /etc/nginx

ls
conf.d	fastcgi_params	mime.types  modules  nginx.conf  scgi_params  uwsgi_params
```

Nginx提供了默认2个配置文件：

1. `etc/nginx/nginx.conf`
2. `etc/nginx/conf.d/default.conf`

先查看 `nginx.conf` 配置文件

```bash
cat nginx.conf
```

文件内容：

```nginx
# 设置nginx服务的系统使用用户
user  nginx;
# 工作进程数（一般和cpu保持一致即可）
worker_processes  auto;
# nginx的错误日志
error_log  /var/log/nginx/error.log notice;
# Nginx服务启动时候pid
pid        /var/run/nginx.pid;

# 事件模块
events {
  	# 每个进程允许最大连接数（优化时使用，可以调到10000）
    worker_connections  1024;
  	# 定义使用的内核模型
  	# use
}

# http模块
# 一个http 允许配置多个 server
# 下面的 include /etc/nginx/conf.d/*.conf; 引入包含server
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

  	# 输出日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

  	# 访问日志
    access_log  /var/log/nginx/access.log  main;

  	# 使用零拷贝方式提升性能
    sendfile        on;
    #tcp_nopush     on;

  	# 客户端到Nginx服务端连接超时时间 这里是65s
    keepalive_timeout  65;

    #gzip  on;

  	# 读取  /etc/nginx/conf.d/ 目录下的所有配置文件
    include /etc/nginx/conf.d/*.conf;
}
```

查看 `default.conf`:

```bash
cat /etc/nginx/conf.d/default.conf
```

文件内容被 `nginx.conf` 的 `http` 模块引入：

```nginx
# server 可以存在多个 location 进行路径匹配
server {
  	# 监听端口
    listen       80;
  	# server name 这里写的本地地址，还可以写外网主机ip或者域名
    # 比如 server_name baidu.com;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

  	# location 控制路由
    location / {
        root   /usr/share/nginx/html; # root表示location路径下，首页存放的路径
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    # 错误页面 这里表示错误码为 500 | 502 | 503 | 504 定位到 /50x.html 路径
    error_page   500 502 503 504  /50x.html;
  	# 而这里的 50x.html路径 将返回 /usr/share/nginx/html 这个目录下的 index.html文件
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```



## 2️⃣ 虚拟主机配置

方式：

1. 基于主机多IP的方式：不同的ip访问不同的虚拟主机，比如 `192.168.14.1` 访问 **虚拟主机1** ， `192.168.14.2` 访问 **虚拟主机2** 
2. 基于端口的配置方式：监听不同的端口，比如  `192.168.14.1：80` 访问 **虚拟主机1** ，  `192.168.14.1：81` 访问 **虚拟主机2** 
3. 基于多个hostname方式（多域名方式），比如 `http://1.baidu.com` & `http://2.baidu.com` 都对应的nginx服务器地址是 `192.168.14.1:80`， 然后nginx将不同的域名请求转发给不同的虚拟主机



### 🤔 基于主机多IP的方式

这个也有2种实现方式：

1. 多网卡多IP的方式：即主机存在多个网卡，不同的网卡分别对应一个ip
2. 单网卡多IP的方式：即只有一个网卡，由同一个网卡监听不同的ip



下面以方式2 **单网卡多IP** 做演示，先查看ip，

```bash
ip a
```

打印结果：

```bash
# lo 表示本地回环网卡
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
3: ip6tnl0@NONE: <NOARP> mtu 1452 qdisc noop state DOWN group default qlen 1000
    link/tunnel6 :: brd :: permaddr 1a7a:e11:b1f2::

# eth 开头的表示网卡 可以看出我的电脑有2个网卡
# 172.17.0.3 是ip地址  /16 是掩码地址
23: eth0@if24: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
25: eth1@if26: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:13:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.19.0.2/16 brd 172.19.255.255 scope global eth1
       valid_lft forever preferred_lft forever
```



🚨如果使用docker运行nginx，会出现找不多 `ip` 命令错误提示，[可以使用 `apt-get` 进行安装](https://stackoverflow.com/a/51835279/7185283)：

```bash
apt-get update
apt-get install -y iproute2
```

向网卡中添加ip：(添加ip前先使用ping检查一下ip是否已存在，避免ip冲突):

```bash
# 假设要添加的ip地址是 192.168.14.1
# 发现ping不同 则可以安全的添加ip到网卡
ping 192.168.14.1

# 向eth0网卡添加 ip
# dev eth0 表示设备
ip a add 192.168.14.1/14 dev eth0

# 添加完成后 检查一下是否添加成功
ip a
```

nginx配置 `/etc/nginx/nginx.conf`:

```nginx
# 设置nginx服务的系统使用用户
# ...🖐️其余模块省略
# http模块
# 一个http 允许配置多个 server
# 下面的 include /etc/nginx/conf.d/*.conf; 引入包含server
http {
    # 🖐️其余配置省略

    #gzip  on;

  # 🖥️
  server {
    # 💡 ip1 监听80端口
    listen 192.168.14.1:80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    # 💡 对应的文件地址
  	# location 控制路由
    location / {
        root   /opt/app/code1; # root表示location路径下，首页存放的路径
        index  index.html index.htm;
    }
    
  }
  
  # 🖥️
  server {
    # 💡 ip2 监听80端口
    listen 192.168.14.2:80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    # 💡 对应的文件地址
  	# location 控制路由
    location / {
        root   /opt/app/code2; # root表示location路径下，首页存放的路径
        index  index.html index.htm;
    }
  }
}
```

🚀 配置完后，可以使用下面命令检查一下配置是否正确：

```bash
nginx -tc /etc/nginx/nginx.conf
```

重启nginx：

```bash
# -s 表示 signal 发送信号
nginx -s reload
```

或者先停止后启动：

```bash
nginx -s stop -c /etc/nginx/nginx.conf

nginx -c /etc/nginx/nginx.conf
```



### 🌰 基于端口的配置方式

这个和基于多ip的方式类似，只不过 **ip不变，访问的端口不同**：

```nginx
http {
    # 🖐️其余配置省略

    #gzip  on;

  # 🖥️
  server {
    # 💡 监听81端口
    listen 81;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    # 💡 对应的文件地址
  	# location 控制路由
    location / {
        root   /opt/app/code1; # root表示location路径下，首页存放的路径
        index  index.html index.htm;
    }
    
  }
  
  # 🖥️
  server {
    # 💡 监听82端口
    listen 82;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    # 💡 对应的文件地址
  	# location 控制路由
    location / {
        root   /opt/app/code2; # root表示location路径下，首页存放的路径
        index  index.html index.htm;
    }
  }
}
```

需要注意的是，nginx默认监听的是 `80` 端口，在设置其它端口之前，最好用命令查看一下，端口是否被占用：

```bash
# 查看一下所有端口
# docker环境需要安装lsof
# apt-get install -y lsof
lsof -i -P -n

COMMAND PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx     1 root    7u  IPv4  36636      0t0  TCP *:80 (LISTEN)
nginx     1 root    8u  IPv6  36637      0t0  TCP *:80 (LISTEN)
```

🌰下面使用docker进项模拟，目录结构

```
 .
├──  code1
│   └──  index.html
├──  code2
│   └──  index.html
├──  docker-compose.yml
├──  Dockerfile
└──  nginx
    └──  nginx.conf
```

`Dockerfile`: 

```dockerfile
FROM nginx:latest

RUN mkdir /opt/code1 && mkdir /opt/code2
COPY ./code1/* /opt/code1/
COPY ./code2/* /opt/code2/

EXPOSE 4401
EXPOSE 4402
```

`docker-compose.yml`:

```yaml
version: '3.9'

services:
  nginx:
    build: 
      context: .
    container_name: nginx-ports
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "4401:4401"
      - "4402:4402"
```

`nginx/nginx.conf`:

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    server {
      listen       4401;
      server_name  localhost;

      #access_log  /var/log/nginx/host.access.log  main;

      location / {
          root   /opt/code1;
          index  index.html index.htm;
      }

      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
          root   /usr/share/nginx/html;
      }
    }

    server {
      listen       4402;
      server_name  localhost;

      #access_log  /var/log/nginx/host.access.log  main;

      location / {
          root   /opt/code2;
          index  index.html index.htm;
      }

      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
          root   /usr/share/nginx/html;
      }
    }
}
```

启动docker compose:

```bash
docker-compose up -d
```

然后访问 `localhost:4401` & `localhost:4402` 可以看到不同的页面



### 🔥 基于多hostname进行配置（实际用的比较多）

假设有2台服务器，域名分别为：

1. `1.funny.com`
2. `2.funny.com`

配置 `/etc/nginx/nginx.conf`: 

```nginx
http {
   
    server {
      listen       80;
    	# 💡 域名1
      server_name  1.funny.com;

      #access_log  /var/log/nginx/host.access.log  main;

      location / {
          root   /opt/code1;
          index  index.html index.htm;
      }

      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
          root   /usr/share/nginx/html;
      }
    }

    server {
      listen       4402;
    	# 💡 域名2
      server_name  2.funny.com;

      #access_log  /var/log/nginx/host.access.log  main;

      location / {
          root   /opt/code2;
          index  index.html index.htm;
      }

      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
          root   /usr/share/nginx/html;
      }
    }
}
```

