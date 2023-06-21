wsl2基本设置：
- https://www.bilibili.com/video/BV1BV4y1Z7v4/?spm_id_from=333.1007.top_right_bar_window_custom_collection.content.click&vd_source=1b80f8e85d6b313a57c6e37edadd9d91



ubuntu账号

jinx
abc123

https://www.bilibili.com/video/BV1BV4y1Z7v4/?buvid=Z3496C37636EC3734B5ABF6D7D902C8FE7D7&is_story_h5=false&mid=sTIW1oq7DMcmVneBM52UmA%3D%3D&p=1&plat_id=116&share_from=ugc&share_medium=iphone&share_plat=ios&share_session_id=6F493168-6814-4139-9208-7C2CF69D0DBF&share_source=COPY&share_tag=s_i&timestamp=1684144319&unique_k=08SQR8v&up_id=23664899

https://codec.wang/blog/setup-wsl-for-frontend



## Windows Terminal运行wsl报错“系统找不到指定的文件。 [已退出进程,代码为 4294967295 (0xffffffff)]”的修复

https://wengeblog.com/posts/a20/

wsl进入磁盘

```bash
cd /mnt/c/xxx # 进入C盘xxx文件夹
cd /mnt/d/xxx # 进入D盘xxx文件夹
```



浏览ubuntu当前目录：

```bash
explorer.exe .
```

https://blog.csdn.net/sexyluna/article/details/106432454





## Zsh

https://www.jianshu.com/p/1f98a10ed63a



## 端口转发

windows本地IP

```
172.16.16.48
```

https://www.notevm.com/a/6283.html

https://www.notevm.com/a/6359.html

在 `linux`中

```bash
# 查看ubuntu IP
ifconfig

172.30.145.14
```

在 `powershell`  中

```bash
#增加转发规则，以80端口为例
netsh interface portproxy add v4tov4 listenport=80 listenaddress=0.0.0.0 connectport=80 connectaddress=172.30.145.14

# 把Linux系统里面的80端口转发到本机的88端口
netsh interface portproxy add v4tov4 listenport=88 listenaddress=0.0.0.0 connectport=80 connectaddress=172.30.145.14

# 查看已转发的端口
netsh interface portproxy show all

# 删除单条转发规则
netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=88

# 重置转发规则（删除所有规则）
netsh interface portproxy reset
```

设置admin 端口转发

```
netsh interface portproxy add v4tov4 listenport=8082 listenaddress=0.0.0.0 connectport=8082 connectaddress=172.30.145.14
```

现在就可以在windows侧访问wsl2 IP了：

```
172.16.16.48:8082
```



👌👌下面这种方式证明有效：（关于如何执行ps1文件，可以youtube上查找）

https://blog.csdn.net/isea533/article/details/130334713

```js
# windows powershell 的位置
C:\Windows\System32\WindowsPowershell\v1.0

wsl-netsh-set 80
fw-port-set 80

wsl-netsh-set 8082
fw-port-set 8082
netsh interface portproxy show all
```

bard AI方法

```bash
# powershell
wsl hostname -I

netsh interface portproxy add v4tov4 listenport=<PORT> listenaddress=0.0.0.0 connectport=<PORT> connectaddress=<WSL2_IP>


netsh interface portproxy add v4tov4 listenport=8082 listenaddress=0.0.0.0 connectport=8082 connectaddress=xxx.xx.234.150

netsh interface portproxy show v4tov4
```
