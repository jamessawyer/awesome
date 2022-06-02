Nginx日志默认存放位置：

- `/var/log/nginx/access.log`
- `/var/log/nginx/error.log`

除了这些，一般会给每个server自定义日志，假设我们网站是 `www.funny.com` ，可以添加 `/var/log/nginx/www.funny.com.*.log`



1️⃣ 日志在配置中定义的位置：

```nginx
# /etc/nginx/nginx.conf

# error_log 表示错误日志
# /var/log/nginx/error.log 表示错误日志存放的位置
# notice 表示错误日志的格式
error_log /var/log/nginx/error.log notice;

http {
  # log_format 表示 日志定义的格式
  # main 是这种格式的名称
  # $ 开头的都是nginx的一些内置变量，我们也可以自定义一些变量
  # 比如 $remote_addr：访问者ip；$time_local: nginx本地时间
  log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
  # accss_log 表示访问日志
  # /var/log/nginx/access.log 表示访问日志存放的位置
  # main 表示日志存储的格式，即上面定义的main中的格式
  access_log /var/log/nginx/access.log main;
  
  server {
    # 虚拟主机中也可以定义日志
    listen 80;
    server_name www.funny.com;
    
    # server访问日志存储的位置
    access_log /var/log/nginx/www.funny.com.access.log main;
  }
}
```

关于内置变量的含义可以查看官方文档和一些三方的文档：

- [nginx.org - 内置变量](https://nginx.org/en/docs/http/ngx_http_core_module.html#var_status)
- [nginx 40问 - Rewrite全局变量是什么？](https://mp.weixin.qq.com/s/Y4UumhTuHy0MBlyFMxOPCg)
- [Nginx配置语法 - SHERlocked93@掘金](https://juejin.cn/post/6844904144235413512#heading-13)

