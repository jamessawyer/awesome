## 1.images, Containers and Ports

常用命令：

- **`docker pull [imageName]`**: 下载镜像，比如 **`docker pull nginx`**, 默认下载最新版本镜像

- **`docker images`**: 显示已经下载的所有镜像(或者使用 **`docker image ls`**)

  ```bash
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  postgres            alpine              cf624d6ff064        4 months ago        157MB
  postgres            latest              73119b8892f9        7 months ago        314MB
  ```

- **`docker run [imageName:imageTag]`**: 运行docker

  ```bash
  # latest 就是 imageTag
  docker run nginx:latest
  
  # detach 模式
  docker run -d nginx
  ```

- **`docker container ls`**: 列举正在运行的容器

  ```
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
  1223747aa469        nginx               "/docker-entrypoint.…"   6 seconds ago       Up 5 seconds        80/tcp              trusting_cori
  ```

- **`docker ps`**: 相比上面列举正在运行容器的方式，这一种用法更常见

  ```bash
  # 这个命令有很多其它的指令
  docker ps --help
  # 打印
  Options:
    -a, --all             Show all containers (default shows just running)
    -f, --filter filter   Filter output based on conditions provided
        --format string   Pretty-print containers using a Go template
    -n, --last int        Show n last created containers (includes all states) (default -1)
    -l, --latest          Show the latest created container (includes all states)
        --no-trunc        Don't truncate output
    -q, --quiet           Only display numeric IDs
    -s, --size            Display total file sizes
    
  # 比如默认的打印格式不好看 可以自定义格式
  docker ps -a --format="ID\t{{.ID}}\nNAME\t{{.Names}}\nIMAGE\t{{.Image}}\nPORT\t{{.Ports}}\nCOMMAND\t{{.Command}}\nCREATED\t{{.CreatedAt}}\nSTATUS\t{{.Status}}\n"
  
  # 可以将上面的字符串设置为一个变量
  export FORMAT="ID\t{{.ID}}\nNAME\t{{.Names}}\nIMAGE\t{{.Image}}\nPORT\t{{.Ports}}\nCOMMAND\t{{.Command}}\nCREATED\t{{.CreatedAt}}\nSTATUS\t{{.Status}}\n"
  # 使用
  docker ps -a --format=$FORMAT
  ```

- **`docker ps -a`**: 列举所有的容器，包括未运行的

- **`docker run -p [your_port:image_port] ...`**: 暴露**端口**， 还可以同时将镜像映射到多个端口中

  ```bash
  # 上面nginx 默认端口是 80， 如果想对外暴露的是 8080 端口
  # image_port 是固定的， your_port可以随意写，只要和其它images向外暴露的不一样即可
  docker run -d -p 8080:80 nginx
  # 运行后 可以打开浏览器 http://localhost:8080/ 查看
  
  # 映射多个端口
  docker run -d -p 3000:80 -p 8080:80 nginx
  ```

- **`docker stop [container_id | container_name]`**: 停止运行docker，可以使用docker容器id或者names; 与之对应的是 **`docker start [container_id | container_name]`**

  ```bash
  # 查看容器状态
  docker ps
  
  # 使用container id停止
  docker stop 87279c50c8d9
  
  # 使用names停止 一般这个name 没有通过--name指定的话 docker会随机指定一个
  docker stop keen_galileo
  ```

- **`docker rm [container_id]`**: 移除容器,⚠️：正在运行的容器是不会被移除的（可以使用 **`docker rm -f [container]`** 强制移除）

  ```bash
  # 可以使用使用下面命令 查看所有的下载的images id
  docker ps -aq
  # 打印结果
  160cb4a987ed
  87279c50c8d9
  c576b0d7db53
  
  # 移除某一个容器
  docker rm 160cb4a987ed
  
  # 移除所有
  docker rm $(docker ps -aq)
  ```

- **`docker run --name [custom_image_name] ...`**: 自定义运行的镜像名，推荐使用--name， **因为可能同时开启同一个image容器，使用自定义名，可以快速的开启，关闭，查找容器**

  ```bash
  docker run -d --name website -p 8080:80 nginx
  
  # 查看
  docker ps
  # 打印
  NAMES
  website
  
  # 关闭容器
  docker stop website
  
  # 启动容器
  docker start website
  ```



## 2. Volumes

作用：

- 容器和容器之间共享

- 用于容器(Container)向Host共享数据: 默认情况，容器中的数据，会随着容器的移除而移除，**为了持久化，会将Container中的数据映射到Host 磁盘中**
- 用于Host向容器中同步数据，比如将Host中的 **网页** 同步到nginx容器中的 **`/usr/share/nginx/html`** 目录中

```bash
# 将本地 /Users/kobe/Desktop/website 同步到 nginx容器的 /usr/share/nginx/html 目录
# /Users/kobe/Desktop/website 中包含 index.html
# -v 表示 volumn
# :ro 表示 read only 表示该volumn是只读的
docker run -d -p 8080:80 --name website -v /Users/kobe/Desktop/website:/usr/share/nginx/html:ro nginx

# 现在访问 localhost:8080/index.html 可以看到网页
```

如果想容器和Host之间相互可以读写，则将上面的 **`:ro`** 去掉：

```bash
docker run -d -p 8080:80 --name website -v /Users/kobe/Desktop/website:/usr/share/nginx/html nginx
```



- **`docker exec [OPTIONS] CONTAINER COMMAND [ARG...] `**: 在一个运行的容器中运行命令

  ```bash
  # 进入命令行界面
  docker exec -it website bash
  
  # 进入容器命令行
  # 查看容器中所有的文件和文件夹
  ls -la
  # 进入上面存放html的容器
  cd usr/share/nginx/html
  # 添加一个新的文件
  touch about.html
  
  # 此时检查做面的/Users/kobe/Desktop/website文件夹
  # 会发现容器中添加about.html 在这个文件夹中也出现了
  
  # 退出命令行
  exit
  ```

  - [docker exec](https://docs.docker.com/engine/reference/commandline/exec/)

- **`docker run --volumes-from [container]`**: 从其余的容器拷贝到当前容器 （**容器和容器之间共享**）

  ```bash
  # 将上面的website容器 拷贝一份为 website2
  docker run -d -p 8081:80 --name website2 --volumes-from website nginx
  
  # 现在访问 localhost:8080/index.html || localhost:8081/index.html 都可以看到网页
  ```

- **`docker exec [container_id | container_name] env`**: 显示某个容器所有的环境变量





## 3. Building Images 构建镜像

自定义镜像，自定义镜像应该包含应用运行的所有东西，比如OS(系统)，依赖包和自己的代码。

1. 首先在项目的根路径创建一个 **`Dockerfile`** 文件,添加配置，下面是最简单的一个配置

```dockerfile
# FROM 表示基于已有的某个镜像的基础之上再构建一个新的镜像
# 这里是基于`nginx:latest`
FROM nginx:latest
# 将当前文件夹中所有内容添加到容器中的 `/usr/share/nginx/html` 文件夹中
ADD . /usr/share/nginx/html
```

2. **`docker builder...`** 打包成一个新的镜像（可以使用 **`docker build --help`** 查看参数配置）

   ```bash
   # 假设执行当前命令行的路径是项目的根路径
   # `--tag wesite:latest`: 表示给新创建的镜像名和标签
   # `.`: 表示Dockerfile所在的路径
   docker build --tag wesite:latest .
   # 运行上面命令 会打印构建步骤
   Sending build context to Docker daemon  1.355MB
   Step 1/2 : FROM nginx:latest
    ---> f35646e83998
   Step 2/2 : ADD . /usr/share/nginx/html
    ---> 846d488a0618
   Successfully built 846d488a0618
   Successfully tagged website:latest
   
   # 查看电脑上存在的镜像
   docker image ls
   # 打印结果
   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   website             latest              846d488a0618        2 minutes ago       134MB
   nginx               latest              f35646e83998        47 hours ago        133MB
   
   # 运行自定义镜像 和之前运行nginx镜像一样
   docker run -d --name website -p 8080:80 website:latest
   
   # 查看当前运行的容器
   docker ps format=$FORMAT
   # 打印结果
   ID	a9e114dc4a6e
   NAME	website
   IMAGE	website:latest
   PORT	0.0.0.0:8080->80/tcp
   COMMAND	"/docker-entrypoint.…"
   CREATED	2020-10-15 14:16:10 +0700 +07
   STATUS	Up About a minute
   # 使用浏览器打开 http://localhost:8080 即可看到项目正常运行
   ```

> nodejs 示例

 现在创建一个 node express 项目，目录结构为

```bash
├── Dockerfile
├── index.js
├── node_modules
├── package.json
└── yarn.lock
```

其中 **`Dockerfile`** 为：

```dockerfile
# 基于node镜像
FROM node:latest
# 射者一个容器中运行的工作路径
WORKDIR /app
# 这里2个点的含义
# 表示将当前项目中的所有文件 添加到上面的 WORKDIR 路径中
ADD . .
# 运行，表示构建自定义镜像时运行的命令 用于下载依赖
RUN yarn install
# command 表示镜像运行后执行的命令
CMD node index.js
```

构建镜像：

```bash
docker build --tag user-service-api:latest .
```

运行镜像，生成容器：

```bash
docker run -d --name node-app -p 3000:3000 user-service-api:latest

# 查看
docker ps --format=$FORMAT
打印
ID	8973084000df
NAME	node-app
IMAGE	user-service-api:latest
PORT	0.0.0.0:3000->3000/tcp
COMMAND	"docker-entrypoint.s…"
CREATED	2020-10-15 14:40:35 +0700 +07
STATUS	Up 35 seconds
```



**`.dockerignore`** 文件，类似于 **`.gitignore`**, 表示构建镜像时忽略那些文件。

上面的nodejs示例中，会将 **`node_modules`** 文件也打包进去，可以在根目录新建一个 **`.dockerignore`** 文件：

```dockerfile
# node_modules 会在上面的 RUN yarn install 时添加，因此没有必要添加到镜像中
node_modules
# 开发用的 不必包含到创建的镜像中
Dockerfile
.git
```



from:

- [Docker and Kubernetes Tutorial - Amigoscode youtube](https://www.youtube.com/watch?v=bhBSlnQcq2k&t=2741s&ab_channel=Amigoscode)