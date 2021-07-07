[centos 安装docker](https://docs.docker.com/engine/install/centos/)

如果之前安装过docker, 想重新安装，可以先卸载旧版本：

 ```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
 ```



```bash
# 安装 yum-utils 提供 yum-config-manager 工具
sudo install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
  
# 安装最新docker engine
sudo yum install docker-ce docker-ce-cli containerd.io

# 如果想安装别的版本 可先查询可安装的版本
yum list docker-ce --showduplicates | sort -r
# 然后进行安装
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

启动docker:

```bash
sudo systemctl start docker

# 测试
docker run hello-world
```

安装docker-compose:

先安装  `pip`:

```bash
yum install -y epel-release
yum install -y python-pip
pip install --upgrade pip
```



```
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```



如果