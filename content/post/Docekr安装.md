---
title: "Docekr安装"
date: 2020-12-25T08:22:55+08:00
draft: false
---

## 安装Docker

本次安装在 CentOS 7上使用 Yum 安装
本次使用阿里镜像源安装
阿里镜像地址
`https://developer.aliyun.com/mirror/`
### 准备环境
- 查找 docker-ce repo 库
在目录`https://mirrors.aliyun.com/docker-ce/linux/centos/`下找到**docker-ce.repo** 右键复制地址链接
- 下载 docker-ce repo 库
```shell
wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
或
```shell
curl -o /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
- 检查是否已经安装好 docker-ce repo 库
`yum repolist|grep docker`
### 安装
使用 yum 进行安装
```shell
yum install -y docker-ce
```
使用命令
`rpm -ql docker-ce`
可以查看安装的 docker-ce 的目录文件
### 修改配置
修改 docker.service 配置文件
```shell
vim /usr/lib/systemd/system/docker.service
```
添加一下

```shell
ExecStartPost=/usr/sbin/iptables -P FORWARD ACCEPT //k8s 的报文转发依赖ACCEPT 启动 docker 时会自动设置为 DROP 需要在启动时 再设置为 ACCEPT
Enviroment="HTTPS_PROXY=https://xxx.xxx" //设置要代理的地址，可以不配置
Enviroment="NO_PROXY=127.0.0.1/8,192.0.0.1/16" //设置不代理某些地址，可以不配置
```
配置完执行一下命令重新 load 一下配置文件
```shell
systemctl daemon-reload
```

### 启动服务
使用一下命令启动 docker 服务
```shell
systemctl start docker
```
如果服务启动不起来
可以使用`systemctl status docker` 或者 `journalctl -xe`查看启动过程中报错信息
检查是否启动成功
`docker info`
拉取镜像 hello-world 并启动

```shell
docker pull hello-world
docker run -it hello-world
--------------------
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

### 设置开机自启
docker 服务设置开机自启
```shell
systemctl enable docker
```

### 安装k8s时 要检查配置
设置 bridge 值为1
查看 bridge 值
`sysctl -a |grep bridge`
如果`net.bridge.bridge-nf-call-ip6tables = 0 net.bridge.bridge-nf-call-iptables = 0`不为1时
使用一下命令设置

```shell
cat > /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```
配置过后加载文件使其生效
```shell
sysctl -p /etc/sysctl.d/k8s.conf
```