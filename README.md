# hammal

Hammal 是运行于 cloudflare workers 上的 Docker 镜像加速工具，用于解决获取 Docker 官方镜像无法正常访问的问题。

文档： https://singee.atlassian.net/wiki/spaces/MAIN/pages/5079084/Cloudflare+Workers+Docker

# 离线安装 docker

### 下载 Docker 安装包

下载地址：https://download.docker.com/linux/static/stable/x86_64/docker-24.0.7.tgz

文件大小约：64MB

### 上传到服务器

```
[root@uat-0001 ~]# ll -h
总用量 82M
-rw-r--r--  1 root root  64M 12月 20 20:54 docker-24.0.7.tgz
```

### 解压软件包

```
[root@uat-0001 ~]# tar -zxvf docker-24.0.7.tgz
docker/
docker/docker
docker/docker-init
docker/dockerd
docker/runc
docker/ctr
docker/containerd-shim-runc-v2
docker/containerd
docker/docker-proxy
```

### 将 docker 目录下文件复制到/usr/bin 目录下

```
[root@uat-0001 ~]# ls docker
containerd  containerd-shim-runc-v2  ctr  docker  dockerd  docker-init  docker-proxy  runc
[root@uat-0001 ~]# cp docker/* /usr/bin/
```

### 创建 docker.service 文件

进入到/usr/lib/systemd/system/目录下，我们编辑创建 docker.service 文件，用于管理 docker 服务，复制黏贴如下内容即可。

```
[root@uat-0001 ~]# vim /usr/lib/systemd/system/docker.service
```

```ini
[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.com
After=network.target docker.socket
[Service]
Type=notify
EnvironmentFile=-/run/flannel/docker
WorkingDirectory=/usr/local/bin
ExecStart=/usr/bin/dockerd \
                -H tcp://0.0.0.0:4243 \
                -H unix:///var/run/docker.sock \
                --selinux-enabled=false \
                --log-opt max-size=100m
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

### 重新加载 daemon-reload

```
systemctl daemon-reload
```

### 启动 docker

```
systemctl start docker
```

### 查看服务启动状态

```
systemctl status docker
```

### 查看 Docker 版本

```
[root@uat-0001 ~]# docker version
Client:
 Version:           24.0.7
 API version:       1.43
 Go version:        go1.20.10
 Git commit:        afdd53b
 Built:             Thu Oct 26 09:04:00 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          24.0.7
  API version:      1.43 (minimum version 1.12)
  Go version:       go1.20.10
  Git commit:       311b9ff
  Built:            Thu Oct 26 09:05:28 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.7.6
  GitCommit:        091922f03c2762540fd057fba91260237ff86acb
 runc:
  Version:          1.1.9
  GitCommit:        v1.1.9-0-gccaecfc
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

### Debian 默认没有安装 iptables

> 检查是否安装 iptables

```
which iptables
```

> Ubuntu/Debian

```
sudo apt-get update
sudo apt-get install iptables
```

> CentOS/RHEL

```
sudo yum install iptables
```

> Arch Linux

```
sudo pacman -S iptables
```

> 查看 iptables 版本

```
iptables –-version

# 查看 iptables 状态
sudo systemctl status iptables

# 如果 iptables 未运行，尝试启动
sudo systemctl start iptables
```

> 创建软连接

If you’ve confirmed that ‘iptables’ is installed and the service is running, but you still encounter the error, it might not be in your PATH. You can add it to your PATH by modifying your shell’s profile configuration file. For example, if you’re using Bash, you can add the following line to your `~/.bashrc` or `~/.bash_profile`:

```
export PATH=$PATH:/sbin
```

After making this change, reload your shell or run

```
source ~/.bashrc
```

### 安装 docker-compose

```
curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-Linux-x86_64 > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

### 配置 docker 代理

```
cd /etc

mkdir docker

vim /etc/docker/daemon.json

{
  "registry-mirrors": [
    "https://dp.houhoukang.com"
  ]
}
```

### 安装 nginx

```
sudo apt install nginx

sudo systemctl start nginx

sudo systemctl status nginx

sudo ufw allow 'Nginx HTTP'
sudo ufw allow 'Nginx HTTPS'
```
