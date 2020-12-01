## 1. Nginx文件结构

下面是部分目录结构

- **`/etc/nginx/nginx.conf`**: Nginx主配置文件 ✨
- **`/etc/nginx/conf.d/`** 进行子配置的配置项 ✨
- **`/etc/nginx/mime.types`**: 设置http协议的Content-Type和扩展名对应关系
- **`/usr/share/doc/..`**: Nginx的手册和帮助文件
- **`/usr/share/nginx/html/..`**: 存放静态网页文件✨
- **`/usr/sbin/nginx`**: Nginx服务的启动管理的终端命令
- **`/usr/lib64/nginx/modules`**: Nginx模块目录
- **`/usr/lib/systemd/system/nginx.service`**: 用于配置系统守护进程管理器管理方式
- **`/var/log/nginx`**: Nginx的日志目录



## 2. Nginx配置语法

**`/etc/nginx/nginx.conf`** 配置文件内容结构：

```
main        # 全局配置，对全局生效
├── events  # 配置影响 Nginx 服务器或与用户的网络连接
├── http    # 配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置
│   ├── upstream # 配置后端服务器具体地址，负载均衡配置不可或缺的部分
│   ├── server   # 配置虚拟主机的相关参数，一个 http 块中可以有多个 server 块
│   ├── server
│   │   ├── location  # server 块可以包含多个 location 块，location 指令用于匹配 uri
│   │   ├── location
│   │   └── ...
│   └── ...
└── ...
```

>  语法规则

1. 配置文件由指令和指令块构成
2. 每条指令以 `;` 分号结尾，指令与参数间以空格符号分割
3. 指令块以 `{}` 大括号将多条指令组织在一起
4. `include` 语句允许组合多个配置文件以提升可维护性
5. 使用 `#` 符号添加注释，提高可读性
6. 使用 `$` 符号使用变量
7. 部分指令的参数支持正则表达式

示例：

```nginx
user         nginx; # 运行用户，默认即是nginx，可以不进行设置
worker_processes 1; # Nginx进程数，一般设为为何CPU核数一样 或者设置为 `auto`
error_log /var/log/nginx/error.log warn; # Nginx的错误日志存放目录
pid       /var/run/nginx.pid; # Nginx服务启动时的pid存放位置

events {
  use epoll; # 使用epoll的I/O模型(如果你不知道Nginx该使用哪种轮询方法，会自动选择一个最适合你操作系统的)
  worker_connections 1024; # 每个进程允许最大并发数
}

# 配置使用最频繁的部分，代理、缓存、日志定义等绝大多数功能和第三方模块的配置都在这里
http {
  # 设置日志模式
  log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
  
  # Nginx 访问日志存放的位置
  access_log /var/log/nginx/access.log main;
  
  sendfile            on;  # 开启搞笑传输模式
  tcp_nopush          on;  # 减少网络报文段的数量
  tcp_nodelay         on;
  keepalive_tiemout   65;  # 保存链接的时间，也叫超时时间，单位秒
  types_hash_max_size 2048
    
  include             /etc/nginx/mime.types; # 文件扩展名与类型映射表
  default_type        application/octet-stream; # 默认文件类型
  
  include /etc/nginx/conf.d/*.conf;   # 加载子配置项
    
  
  server {
    listen 80; # 配置监听的端口
    server_name localhost; #配置的域名
    
    location / {
      root   /usr/share/nginx/html;     # 网站根目录
      index  index.html index.htm;      # 默认首页文件
      deny   172.168.22.11;              # 禁止访问的ip地址，可以为 `all`
      allow  172.168.33.44;             # 允许访问的ip地址，可以为 `all`
    }
    
    error_page 500 502 503 504 /50x.html; # 默认50x对应的页面
    error_page 400 404 error.html; # 默认40x错误对应的页面
  }

}
```

**`server`** 块可以包含多个 **`location`** 块，location指令用于匹配uri,语法：

```nginx
location [ = | ~ | ~* | ^~] uri {
	# ...
}
```

其中：

- **`=`** : 表示精确匹配路径，用于不含正则表达式的 uri 前，如果匹配成功，则不再进行后续的查找
- **`^~`**: 用于不含正则表达式的uri 前，表示如果该符号后面的字符是最佳匹配，采用该规则，不再进行后续的查找
- **`~`**： 表示用该符号后面的正则去匹配路径，不区分大小写
- **`~*`**：表示用该符号后面的正则去匹配路径，不区分大小写。跟 **`~`** 一样，优先级比较低，如有多个location的正则能匹配的话，则使用正则表达式最长的那个

如果uri包含正则表达式，则必须要有 **`~ | ~*`** 标志。



> 全局变量

Nginx中有一些全局变量，可以在配置的任何位置使用：

| 全局变量名       | 功能                                                         |
| ---------------- | ------------------------------------------------------------ |
| $host            | 请求信息中的 Host,如果请求中没有Host行，则等于设置的服务器名，不包含端口 |
| $request_method  | 客户端请求方法，如 GET、POST                                 |
| $remote_addr     | 客户端的 IP 地址                                             |
| $args            | 请求中的参数                                                 |
| $arg_PARAMETER   | GET 请求中变量名 PARAMETER 参数的值，例如：$http_user_agent(User_Agent值)，$http_referer... |
| $content_length  | 请求头中的 Content-length字段                                |
| $http_user_agent | 客户端agent信息                                              |
| $http_cookie     | 客户端cookie信息                                             |
| $remote_port     | 客户端的端口                                                 |
| $server_protocol | 请求使用的协议，比如 HTTP/1.0、HTTP/1.1                      |
| $server_addr     | 服务器地址                                                   |
| $server_name     | 服务器名称                                                   |
| $server_port     | 服务器的端口号                                               |
| $scheme          | HTTP协议（http, https）                                      |

还有些 **nginx内置预定义变量** 可以自行了解。



## 3.设置二级域名虚拟地址

购买阿里云或者腾讯云服务器后，可以在 **域名管理 -> 解析 -> 添加记录** 中添加二级域名，配置后云提供商会把二级域名也解析到我们配置的服务器IP上，然后我们在Nginx上配置一下虚拟主机的方格纹监听，就可以闹到从这个二级域名过来的请求了。

比如现在购买的域名为 **`life.com`**,配置一个二级域名 **`my.life.com`**。由于默认配置文件 **`/etc/nginx/nginx.conf`** 中的 **`http`**模块中有这样搞一个指令 **`include /etc/nginx/conf.d/*.conf`**,也就是说 **`conf.d`** 文件夹下所有的 **`*.conf`** 文件都会作为子配置项被引入配置文件中。为了维护方便，可以在 **`/etc/nginx/conf.d`** 文件夹中新建一个 **`my.life.com.conf`** 文件（可以使用 **`cp default.conf my.life.com.conf`**命令拷贝一份默认配置）：

```nginx
# /etc/nginx/conf.d/my.life.com.conf

server {
	listen 80;
	server_name my.life.com;
	location / {
    # 在/usr/share/nginx/html 新建一个目录叫 my
		root /usr/share/nginx/html/my;
		index index.html
	}
}
```

在 **`/usr/share/nginx/html`** 文件夹下新建一个 **`my`** 文件夹，新建文件 `index.html`, 内容随便写点，改完后 **`nginx -s reload`** 重新加载，浏览器中输入二级域名 **`my.life.com`** 即可访问。



## 4.配置反向代理

**反向代理是工作中最常用的服务器功能，经常被用来解决跨域问题**。

进入 Nginx的主配置文件：

```bash
vim /etc/nginx/nginx.conf
```

为了方便可以使用vim **`:set nu`** 显示行号。去 **`http`** 模块的 `server` 块中的 `location /`, 增加一行将默认网址重定向到B站的 **`proxy_pass`** 配置：

```nginx
http {
	server {
		listen 80;
		server_name *.life.com;
		
		location / {
			proxy_pass http://www.bilibili.com;
		}
	}
}
```

改完保存退出后， **`nginx -s reload`** 重新加载，进入默认网址，那么现在就会之间跳转到B站了，实现了一个简单的代理。实际使用中，可以将请求转发到本机另一台服务器上，也可以根据访问的路径跳转到不同端口的服务器中。

比如我们监听 **`9001`** 端口，然后把方格纹不同路径的请求进行反向代理：

1. 把访问 **`http://127.0.0.1:9001/edu`** 的请求转发到 **`http://127.0.0.1:8080`**
2. 把访问 **`http://127.0.0.1:9001/vod`** 的请求转发到 **`http://127.0.0.1:8081`**

打开主配置文件，然后在http模块下增加一个server块：

```nginx
server {
	listen 9001;
	server_name *.life.com;
	
	location ~ /edu/ {
		proxy_pass http://127.0.0.1:8080;
	}
	
	location ~ /vod/ {
		proxy_pass http://127.0.0.1:8081;
	}
}
```

反向代理还有一些其它的指令：

1. **`proxy_set_header`**: 在将客户端请求发送给后端服务器之前，更改来自客户端的请求头信息
2. **`proxy_read_timeout`**: 配置Nginx与后端代理服务器舱室建立连接的超时时间
3. **`proxy_read_timeout`**: 配置Nginx向后端服务器组发出read请求后，等待相应的超时时间
4. **`proxy_send_timeout`**: 配置Nginx向后端服务器组发出write请求后，等待相应的超时时间
5. **`proxy_redirect`**: 用于修改后端服务器返回的响应头中的Location和Refresh



## 5. 跨域CORS配置

现在前后端分离的项目，经常本地的前端服务，需要访问不同的后端地址，不可避免遇到跨域问题。

假设现在配置了2个二级域名 **`aa.life.com & bb.life.com`**, 都指向云服务器地址。虽然对应IP是一样的，但是 **`aa.life.com`** 域名发出的请求访问 **`bb.life.com`** 域名的请求还是跨域了， **因为访问的host不一致**。



> 使用反向代理解决跨域

在前端服务地址为 `aa.life.com` 的页面请求 `bb.life.com` 的后端服务导致的跨域，可以这样配置：

```nginx
server {
	listen 9001;
	server_name aa.life.com;
	
	location / {
		proxy_pass bb.life.com;
	}
}
```

这样就将 `aa.life.com` 域名的请求全部代理到了 `bb.life.com`, 前端的请求都被我们用服务器代理到了后端地址下，绕过了跨域。

这里对静态文件的请求和后端服务的请求都以 `bb.life.com` 开始，不易区分，所以为了实现对后端服务请求的统一转发，我们通常会约定对后端服务的请求加上 **`/apis/`** 前缀或者其它path来对静态资源的请求加以区分，此时我们可以这样配置：

```nginx
# 请求跨域，约定代理后端服务请求path以 /apis/ 开头
location ^~/apis/ {
  # 这里重写了请求，将正则匹配中的第一个分组的path拼接到真正的请求后面，并用break停止后续匹配
  rewrite ^/apis/(.*)$ /$1 break;
  proxy_pass bb.life.com;
  
  # 连个域名之间cookie的传递与回写
  proxy_cookie_domain bb.life.com aa.life.com;
}
```

这样，静态资源我们使用 **`aa.life.com/xx.html`**,动态资源我们使用 **`aa.life.com/apis/getSomething`**， 浏览器页面看起来仍然访问的前端服务器，绕过了浏览器的同源策略，毕竟我们看起来并没有跨域。

也可以统一一点，直接把前后端服务器地址直接都转发到另一个 **`server.life.com`**,只通过在后面添加的path来区分请求的是静态资源还是后端服务。



> 2.配置 header 解决跨域

当浏览器访问跨源服务器时，也可以在跨域的服务器上直接设置Nginx，从而前端就可以无感地开发，不用把实际上访问后端的地址改成前端服务的地址，这样可适用性更高。

比如前端站点是 **`aa.life.com`**,这个地址下的前端页面请求是， **`bb.life.com`** 下的资源，比如前者的 **`aa.life.com/index.html`** 内容是这样的：

```html
<html>
<body>
	<h1>aa.life.com</h1>
	<script type="text/javascript">
		var http = new XMLHttpRequest()
		http.open('GET', 'http://bb.life.com', true)
		http.send()
	</script>
</body>
</html>
```

打开浏览器我们将看到这样的辰颐物语 **`Access to XMLHttpRequest at 'http://bb.life.com/index.html' from 'http://aa.life.com' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the request resouce.`**

很显然这里是跨域请求，在浏览器中直接访问 **`http://bb.life.com/index.html`** 是可以访问的，但在 **`aa.life.com`**的html中访问就会出现跨域问题。

在 **`/etc/nginx/conf.d/`** 文件夹下新建一个配置文件，对应二级域名 **`bb.life.com`**:

```nginx
# /etc/nginx/conf.d/bb.life.com.conf

server {
	listen 80;
	server_name bb.life.com;
	
	# 全局变量获得当前请求origin，带cookie的请求不支持
	add_header 'Access-Control-Allow-Origin' $http_origin;
	# 为 true 可带上 cookie
	add_header 'Access-Control-Allow-Credentials' 'true';
	# 允许请求方法
	add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
	# 允许请求的header，可以是 *
	add_header 'Access-Control-Allow-Headers' $http_access_control_request_header;
	add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
  
  if ($request_method = 'OPTIONS') {
    # OPTIONS 请求的有效期，在有效期内不用发出另一条预检请求
    add_header 'Access-Control-Max-Age' 1728000;
    add_header 'Content-Type' 'text/plain; charset=utf-8';
    add_header 'Content-Length' 0;
    
    return 204; # 200 也可以
  }
  
  location / {
    root /usr/share/nginx/html/bb;
    index index.html;
  }
}
```

然后 **`nginx -s reload`** 重新加载配置。这是再访问 `aa.life.com/index.html` 结果如下，请求出现了我们刚刚配置的Header。

关于跨域相关文章：

- [都是跨域问题 - 记一次图片上传踩坑 - 掘金](https://juejin.im/post/6844904047409889288)

## 6. 开启GZIP压缩

gzip可以大大压缩网页，节省带宽。



> 1.Nginx配置gzip

一般在请求html和css等静态资源的时候，支持的浏览器在Request请求静态资源的时候，会加上 **`Accept-Encoding: gzip`** 这个header，表示自己支持gzip的压缩方式，Nginx在拿到这个请求的时候，如果有相应配置，就会返回经过gzip压缩过的文件给浏览器，并在response响应的时候加上 **`content-encoding: gzip`** 来告诉浏览器自己采用的压缩方式（因为浏览器在传给服务器的时候一般还告诉服务器自己支持好几种压缩方式），浏览器拿到压缩的文件后，根据自己的解压方式进行解析。

为了方便管理，可以在 **`/etc/nginx/conf.d/`** 文件夹中新建配置文件 **`gzip.conf`**:

```nginx
# /ect/nginx/conf.d/gzip.conf

gzip on; # 默认off， 是否开启gzip
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

# 上面两个开启基本就能跑起来，下面的可以了解
gzip_static on;
gzip_proxied any;
gzip_vary on;
gzip_comp_level 6;
gzip_buffers 16 8k;
# 如果文件小于1kb 会导致越压缩越大
# 这里表示低于 1kb 的文件就不要gzip压缩了
gzip_min_length 1k;
gzip_http_version 1.1;
```

上面配置的含义：

- **`gzip_types`**: 要采用gzip压缩的MIME文件类型，其中 `text/html` 被系统强制启用；
- **`gzip_static`**: 默认off，该模块启用后，Nginx首先检查是否存在请求静态文件的gz结尾的文件，如果有则直接返回该 **`.gz`** 文件内容；
- **`gzip_proxied`**: 默认off， Nginx做为反向代理时启用，用于设置启用或禁用从代理服务器上收到相应内容gzip压缩
- **`gzip_vary`**: 用于在响应消息头中添加 **`Vary: Accept-Encoding`**, 使代理服务器根据请求头中的 **`Accept-Encoding`** 识别是否启用gzip压缩
- **`gzip_comp_level`**:  gzip压缩比， 压缩机别是 `1-9`， `1`压缩级别最低，`9` 最高，级别越高压缩率越大，压缩时间越长，建议 `4-6`
- **`gzip_buffers`**: 获取多少内存用于缓存压缩结果， `16 8k` 表示以 `8k*16` 为单位获得
- **`gzip_min_length`**: 允许压缩的页面最小字节数，页面字节数从header头中的 `Content-Length` 中进行获取。默认值为0，不管页面多大都压缩。建议设置成大于1k的字节数，小于1k可能会越压越大
- **`gzip_http_version`**: 默认是 `1.1`, 启用gzip所需的http最低版本

这个配置可以插入到http模块整个服务器的配置里，也可以插入到需要使用的虚拟主机的 `server` 或者下面的 `location` 模块中，当然上面的写法，会被 `include` 到 `http` 模块中。

配置之后请求返回的header里面多一个 **`Content-Encoding: gzip`**, 返回信息被压缩了。



> 2.Webpack的gzip配置

当前前端项目使用Webpack进行打包的时候，也可以开启gzip压缩：

```js
const CompressionWebpackPlugin = require('compression-webpack-plugin')

module.exports = {
	// gzip配置
  configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') { // 生产环境
      return {
        plugins: [new CompressionWebpackPlugin({
          test: /\.js$|\.html$|\.css/, // 匹配文件名
          threshold: 10240,  // 文件压缩阈值，对超过10Kb的进行压缩
          deleteOriginalAssets: false // 是否删除源文件
        })]
      }
    }
  }
}
```

超过10kb的js,html,css文件以 `.gz` 结尾。使用前端进行gzip压缩，避免Nginx压缩文件，消耗服务器的计算资源，响应会增加客户端的请求时间，得不偿失。

将前端打包之后的高压缩等级文件作为静态资源放在服务器上，Nginx会优先查找这些压缩之后的文件返回给客户端，响度昂与把压缩文件的动作从Nginx提前给webpack打包的时候完成。



## 7. 配置负载均衡

负载均衡的意思是把负载合理地分发到多个服务器上个，实现压力分流的目的。

主要配置：

```nginx
http {
  upstream myserver {
    # ip_hash; # ip_hash方式
    # fair; # fair方式
    server 127.0.0.1:8081; # 负载均衡目的服务器
    server 127.0.0.1:8080;
    server 127.0.0.1:8082 weight=10; # weight 权重 默认是1
  }
  
  server {
    location / {
      proxy_pass http://myserver;
      proxy_connect_timeout 10;
    }
  }
}
```

Nginx提供了好几种分配方式，默认为轮询，就是轮流来。有以下几种分配方式：

1. **轮询**，默认方式，每个请求按时间顺序逐一分配到不同的后端服务器，如果服务器挂了，能自动剔除；
2. **weight**， 权重分配，指定轮询几率，权重越高，被访问的概率越大，用于后台服务器性能不均的情况
3. **ip_hash**, 每个请求按访问IP的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决动态网页session共享问题。负载均衡每次请求都会重新定位到服务器集群中的某一个，那么已经登录某一个服务器的用户再重新定位到另一个服务器，其登录信息将会丢失，这样显然是不妥的。
4. **fair（第三方）**,按后端服务器的响应时间分配，响应时间短的优先分配，依赖第三方插件 `nginx-upstream-fair`, 需要先安装。



## 8. 配置动静分离

将动态和静态请求分开。分开方式有2种：

1. 纯粹把静态文件独立成单独的域名，放在独立的服务器上，**主流推崇方案**
2. 动态跟静态文件混合在一起发布，通过Nginx配置分开

通过location制定个不同的后缀名实现不同的请求转发。通过 `expires` 参数设置，可以使浏览器缓存过期时间，减少与服务器之前的请求和浏览。`expires` 具体定义：是给一个资源设定一个国企时间，也就是说无序去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。此种方式非常适合不经常变动的资源。（如果经常更新文件，不建议使用 expires来缓存）。假设设置为 `3d`,表示3天内访问这个URL，发送一个请求，比对服务器该文件最后更新时间没有发生变化，则不会从服务器抓取，返回**状态码304**，如果修改，则直接从服务器重新下载，返回状态码200.

```nginx
server {
	location /www/ {
		root /data/;
		index index.html index.htm;
	}
	
	location /image/ {
		root /data/;
		autoindex on;
	}
}
```



## 9. 配置高可用集群（双机热备）

当主Nginx服务器宕机后，切换到备份Nginx服务器，需要安装一个插件 **`keepalived`**:

```bash
# 假设使用linux服务器
yum install keepalived -y
```

然后编辑 **`/etc/keepalived/keepalive.conf`** 配置文件，并在配置文件中增高架 **`vrrp_script`** 定义一个外文检测机制，并在 **`vrrp_instance`** 中通过定义 **`track_script`** 来追踪脚本执行过程，实现节点转移：

```json
global_defs {
	notification_email {
		acassen@firewall.loc
	}
	notication_eamil_from Alexandre@firewall.loc
	smtp_server 127.0.0.1
	smtp_connect_timeout 30 // 上面配置的是邮件，没啥用
	router_id LVS_DEVEL // 当前服务器名字，用hostname命令来查看
}

vrrp_script chk_maintainace { // 检测机制的脚本名称为chk_maintainace
	script "[[ -e/etc/keepalived/down ]] && exit 1 || exit 0" // 可以是脚本路径或脚本命令
	// script "/etc/keepalived/nginx_check.sh" // 比如这样搞的脚本路径
	interval 2 // 每2秒检测一次
	weight -20 // 当脚本执行成立，那么把当前服务器优先级改为-20
}

vrrp_instancVI_1 { // 每一个vrrp_instance就是定义一个虚拟路由器
	state MASTER  // 主机为MASTER，备用机为BACKUP
	interface eth0    // 网卡名字，可以从ifconfig中查找
	virtual_router_id 51 // 虚拟路由的id号，一般小于255，主备机id需要一样
	priority 100      // 优先级，master的优先级比backup的大
	advert_int 1      // 默认心跳间隔
	authentication {  // 认证机制
		auth_type PASS
		auth_pass 1111 // 密码
	}
	virtual_ipaddress {  // 虚拟地址vip
		172.16.2.8
	}
}
```

其中检测脚本 **`nginx_check.sh`**,这里提供一个：

```sh
#!/bin/bash
A=`ps -C nginx --no-header | wc -l`
if [ $A -eq 0 ];then
    /usr/sbin/nginx # 尝试重新启动nginx
    sleep 2         # 睡眠2秒
    if [ `ps -C nginx --no-header | wc -l` -eq 0 ];then
        killall keepalived # 启动失败，将keepalived服务杀死。将vip漂移到其它备份节点
    fi
fi
```

赋值一份到备份服务器，备份Nginx的配置要将 `state` 后改为 `BACKUP`， `prority` 改为比主机小。

设置完毕后各自 `service keepalived start` 启动，经过访问成功之后，可以把master机的keepalived停掉，此时Master机就不再是主机了 `service keepalived stop`, 看访问虚拟IP时是否能够自动切换到备用机器 `ip addr`.

再次启动Master的keepalived，此时vip又编导主机上了。



## 10. 适配PC或移动设备

根据用户设备不同返回不同样式的站点，根据 **`user-agent`** 来判断返回PC还是H5站点。

先在 **`/usr/share/nginx/html`** 文件夹下创建2个文件夹 **`pc & mobile`**, 然后分别创建一个 `index.html`,随便写一点什么内容：

```sh
cd /usr/share/nginx/html
mkdir pc mobile
cd pc
vim index.html   # 随便写点比如 hello pc!
cd ../mobile
vim index.html   # 随便写点比如 hello mobile!
```

然后和设置二级域名一样，在 **`/etc/nginx/conf.d`** 文件夹下新建一个配置文件 `aa.life.com.conf` (这里只是做个演示)

```nginx
# etc/nginx/conf.d/aa.life.com.conf

server {
	listen 80;
	server_name aa.life.com;
	
	location / {
		root /usr/share/nginx/html/pc;
		if ($http_user_agent ~* '(Android|webOS|iPhone|iPod|BlackBerry)') {
			root /usr/share/nginx/html/mobile;
		}
		index index.html;
	}
}
```

这里主要多一个 `if` 语句，然后使用 `$http_user_agent` 全局变量来判断用户请求的 `user_agent`, 指向不同的 **`root`** 路径，返回对应的站点。

测试时，可以使用浏览器进行模拟，记住要刷新页面。



## 11. 配置HTTPS

首先申请SSL证书，申请成功后，下载证书的压缩文件，里面有个nginx文件夹，把 `xx.crt` 和 `xxx.key` 文件拷贝到服务器目录，再配置下：

```nginx
server {
  listen 443 ssl http2 default_server; # SSL 访问端口号为443
  server_name life.com; # 填写绑定证书的域名
  
  ssl_certificate /etc/nginx/https/1_life.com.crt; # 证书文件地址
  ssl_certificate_key /etc/nginx/https/2_life.com.key; # 私钥文件地址
  ssl_session_timeout 10m; # 10min
  
  ssl_protocols TLSv1 TLSv1.1 TLS1.2; #请按照以下协议配置
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
  ssl_prefer_server_ciphers on;
  
  location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
  }
}
```

写完后 **`nginx -t -q`** 校验一下，没问题就 **`nginx -s reload`**, 现在访问 `https://life.com` 就能访问HTTPS版本的网站了。

一般还可以加上几个增强安全性的命令：

```nginx
add_header X-Frame-Options DENY; # 减少点击劫持
add_header X-Content-Type-Options nosniff; # 进制服务器自动解析资源类型
add_header X-Xss-Protection 1; # 防XSS攻击
```



## 12. 一些常用技巧

### 1.静态服务

```nginx
server {
	listen 80;
	server_name static.life.com;
	charset utf-8; # 防止中文文件名乱码
	
	location /download {
		alias /usr/share/nginx/html/static; # 静态资源目录
		
		autoindex on; # 开启静态资源列目录
		autoindex_exact_size off; # 默认on:显示文件的确切大小，单位byte；off：显示文件的大概大小，单位Kb，Mb，Gb
		autoindex_localtime off; # 默认off：显示文件时间为GMT时间；on：显示文件时间为服务器时间
	}
}
```



### 2.图片防盗链

```nginx
server {
  listen 80;
  server_name *.life.com;
  
  # 图片防盗链
  location ~* \.(gif|jpg|jpeg|png|bmp|swf)$ {
     # 只允许本机 IP 外链引用，将百度和谷歌也加入白名单
    valid_referers none blocked server_names ~\.google\. ~\.baidu\. *.qq.com;
    
    if ($invalid_referer) {
      return 403;
    }
  }
}
```



### 3.请求过滤

```nginx
# 非指定请求全返回 403
if ($request_method !~ ^(GET|POST|HEAD)$) {
  return 403;
}

location / {
  # IP访问限制（只允许IP是 192.168.0.2 机器访问）
  allow 192.168.0.2;
  deny all;
  
  root html;
  index index.html;
}
```



### 4.配置图片、字体等静态文件缓存

由于图片、字体、音频、视频等静态文件在打包的时候通常会增加hash，所以缓存可以设置的长一点，先设置强制缓存，再设置协商缓存；如果存在没有hash值的静态文件，建议不设置强制缓存，仅通过协商缓存来判断是否需要使用缓存。

```nginx
# 图片缓存时间设置
location ~ .*\.(css|js|jpg|png|gif|swf|woff|woff2|eot|svg|ttf|otf|mp3|m4a|aac|txt)$ {
	expires 10d;
}

# 如果不希望缓存
expires -1;
```



### 5.单页面项目history路由配置

```nginx
server {
  listen 80;
  server_name aa.life.com;
  
  location / {
    root /usr/share/nginx/html/dist; # vue打包后的文件夹
    index index.html;
    try_files $uri $uri/ /index.html @rewrites;
    
    expires -1;
    add_header Cache-Control no-cache;
  }
  
  # 接口转发，如果需要的话
  #location ~ ^/api {
  #  proxy_pass http://bb.life.com;
  #}
  
  location @rewrites {
    rewrite ^(.+)$ /index.html break;
  }
}
```



### 6.HTTP请求转发到HTTPS

配置完HTTPS后，浏览器还可以访问 HTTP的地址 `http://life.com`,可以做一个 `301` 跳转，把对应域名的HTTP请求重定向到HTTPS上

```nginx
server {
  listen 80;
  server_name www.life.com;
  
  # 单域名重定向
  if ($host = 'www.life.com') {
    return 301 https://www.life.com$request_uri;
  }
  
  # 全局非https协议重定向
  if ($scheme != 'https') {
    return 301 https://$server_name$request_uri;
  }
  
  # 或者全部重定向
  return 301 https://$server_name$request_uri;
  
  # 以上配置选择自己需要的即可，不用全部加
}
```



### 7. 泛域名路径分离

非常实用技巧，经常有时候我们可能需要配置一些二级或者三级域名，希望通过nginx自动指向对应目录，比如：

1. `test1.doc.life.com` 自动指向 `/usr/share/nginx/html/doc/test1` 服务器地址
2. `test2.doc.life.com` 自动指向 `/usr/share/nginx/html/doc/test2` 服务器地址

```nginx
server {
	listen 80;
	server_name ~^([\w-]+\.doc\.life\.com$);
	
	root /usr/share/nginx/html/doc/$1;
}
```



### 8.泛域名转发

和之前功能类似，有时候我们希望把二级或者三级域名链接重写到我们希望的路径，让后端就可以更具路由解析不同的规则：

1. `test1.serv.life.com/api?name=a` 自动转发到 `127.0.0.1:8080/test1/api?name=a`
2. `test2.serv.life.com/api?name=a` 自动转发到 `127.0.0.1:8080/test2/api?name=a`

```nginx
server {
  listen 80;
  server_name ~^([\w-]+\.doc\.life\.com$);
  
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header X-Nginx-Proxy true;
    proxy_pass http://127.0.0.1:8080/$1$request_uri;
  }
}
```



## 13. 最佳实践

1. 为了使nginx配置易于维护，建议为每个服务器创建一个单独的配置文件，存储在 `/etc/nginx/conf.d` 目录，根据需求可以创建任意多个独立的配置文件
2. 独立的配置文件，建议遵循以下命名约定 **`<服务>.conf`**, 比如域名是 `life.com`, 那么你的配置文件应该是 **`/etc/nginx/conf.d/life.com.conf`**, 如果部署多个服务，也可以在文件名中加上NGINX转发的端口号，比如 `life.com.8080.conf`, 如果是二级域名，也建议加上 `aa.life.com.conf`
3. 常用的，复用频率比较高的配置可以放到 **`/etc/nginx/snippets`** 文件夹，在Nginx的配置文件中需要用到的位置就 `include` 进去，以功能来命名，并在每个snippet配置文件的开头注水表明主要功能和引入位置，方便管理。比如之前的 `gzip|cors` 等常规用配置，都可以设置在snippet
4. Nginx日志相关的目录，以 `域名.type.log` 命名（比如 `bb.life.com.access.log` & `bb.life.com.error.log`）位于 **`/var/log/nginx`** 目录中，为每个独立的服务配置不同的访问权限和错误日志文件，这样查找错误时，会更加方便快捷。



本文来自：（自己摘抄一遍，仅为了快速了解其中的概念）

- [Nginx 从入门到实践，万字详解 - SHERlocked93 掘金](https://juejin.im/post/6844904144235413512)



2020年10月21日13:38:01

