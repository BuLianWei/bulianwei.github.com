---
title: "Kubernetes安装"
date: 2020-12-25T16:24:58+08:00
draft: false
---

## Kubernetes 安装
本次安装环境 Centos 7 
本次使用阿里镜像源
`https://developer.aliyun.com/mirror`
### 准备 Yum repo 镜像库（master，node）
从阿里镜像源找到 kubernetes 
从目录里面找到 yum repo 地址
`https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/`
从目录里面找到 key 验证地址
`https://mirrors.aliyun.com/kubernetes/yum/doc/`
右键复制`https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg`和`https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg`地址作为 gpgkey 的验证地址 
以下是已经准备好的命令直接复制到服务器粘贴即可
```shell
cat > /etc/yum.repos.d/Kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes Repository
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg 
	https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
EOF
```
检查是否配置好 yum repo 库
`yum repolist|grep kubernetes`

### 安装 Kubernetes（master，node）
使用 Yum 安装 Kubernetes
```shell
yum install -y kubeadm kubectl kubelet
```
查看是否安装完全
`yum list installed |grep ^kube`
使用 rpm -ql 可以查看安装软件都生成了哪些目录文件
`rpm -ql kubeadm kubectl kubelet`

### 修改 kubelet 配置（master，node）
Kubernets 安装要求关闭 Swap，可是有些环境中由于内存不大，需要开启 Swap，所以这里需要修改配置，使开启 Swap 的同时，初始化 Kubernetes 也可以成功
```shell
cat > /etc/sysconfig/kubelet <<EOF
KUBELET_EXTRA_ARGS="--fail-swap-on=false"
EOF
```
### 拉取初始化镜像（master，node）
因为初始化时会加载 Kubernetes 一些底层的系统镜像，所以可以在初始化前先将镜像拉取下来。（另外初始化时拉取的镜像默认是从外网拉取，由于不可描述原因在没有进行代理会拉取失败，所以可以先从国内镜像源进行拉取）
使用国外镜像源拉取镜像(可以上外网的时候可以使用)
`kubeadm config images pull`
使用一下脚本下载阿里镜像源，并将镜像打包成 kubernetes需要的格式
```shell
images=`kubeadm config images list`
images=(${images//k8s.gcr.io\// })

for imageName in ${images[@]} ; do
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
    docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
```
### 初始化（master）
初始化只需要在 master 节点进行，其他 node 节点 需要 join ，因此 kubeadm init 命令只在 master 节点进行
初始化之前先看一下默认初始化的参数，看哪些需要修改
`kubeadm config print init-defaults`
这里主要是 kubernetesVersion 版本号会不一样
`kubeadm init -h` 可以查看初始化过程中使用的配置
--apiserver-advertise-address //次选项是设定 api server 的 ip 地址，如果宿主机上有多个网络时可以使用次配置来设定自己想要的 ip 地址
--kubernetes-version //此配置是修改 kubernetes 的版本号，一般来说都要修改，如果默认初始化的配置和安装的版本不一样时可以设定
--image-repository //此配置是配置镜像拉取的地址，默认是 k8s.gcr.io 次地址是 google 镜像的地址，由于不可描述的原因此地址不可访问，可以改成阿里或其他镜像源
`registry.cn-hangzhou.aliyuncs.com/google_containers`
--pod-network-cidr //此配置是配置 pod 网络的 ip 地址，此 ip 地址要和安装的网络插件一致，此次是使用 flannel 插件所以 pod 的网络要设置成 10.244.0.0/16
可以先使用`--dry-run`进行初始化试跑
`kubeadm init  --pod-network-cidr="10.244.0.0/16" --dry-run `
试跑过程发现报错
- [ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
此报错是因为我们的宿主机CPU核数不够，要求CPU至少2核，我们的宿主机只有1个核
**解决方案**
只要将宿主机核数调大就好
（一般物理机不会有此问题）
- [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
此处报错是因为`net.bridge.bridge-nf-call-ip6tables = 0 net.bridge.bridge-nf-call-iptables = 0`的值不为1
**解决方案**
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
- [ERROR Swap]: running with swap on is not supported. Please disable swap
此处报错是因为我们没用将 Swap 关掉引起的，如果我们不想将 Swap 关掉，还想可以顺利进行初始化，我们可以在初始化命令上添加
`--ignore-preflight-errors=Swap`
忽略掉此处报错

所以最后的初始化命令为
```shell
kubeadm init  --pod-network-cidr="10.244.0.0/16" --ignore-preflight-errors=Swap 
```
出现
`Your Kubernetes control-plane has initialized successfully!`
就表示初始化成功

初始化过后会显示要进行一下配置

```shell
---------------------------------------------------------
Your Kubernetes control-plane has initialized successfully!

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.56.106:6443 --token gr0k68.q8g7cezvkbk8n0wr \
    --discovery-token-ca-cert-hash sha256:63bb33710ba6d9387cd4c8537c40aa8a19d1d116a051af0d4508a3735504f866

```

### 修改 Kubectl 配置（master）
kubectl 是访问集群的 cli 工具，如果使用 kubectl 命令需要进行配置
一般情况下不使用 root 用户进行 kubectl 操作，所以切换到相应用户进行一下操作
```shell
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
如果测试环境可以直接使用 root 用户进行一下操作
```shell
  mkdir -p $HOME/.kube
  cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```
此后就可以使用
`kubectl get nodes`
查看有哪些 nodes，此处因为没有安装网络插件，所以 nodes 还是 NotReady 状态
其他 node 节点如果也要使用 kubectl 命令 需要将`/etc/kubernetes/admin.conf` 拷贝到 node 节点的`$HOME/.kube/config`

### 安装 flannel 网络插件（master）
Kubernetes 的 Pod 网络插件有好多，我们此次使用 flannel 
打开 flannel 的 GitHub 网站`https://github.com/coreos/flannel` 找到安装步骤
使用一下命令进行安装
```shell
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
安装完成后，使用
`kubectl get nodes`
即可查看 nodes 已经 Ready
使用
`kubectl get pods -n kube-system`
可以查看有哪些 pods 在运行
如果安装网络插件过程中报错
`The connection to the server raw.githubusercontent.com was refused - did you specify the right host or port?`
此处报错是因为，宿主机并不能解析`raw.githubusercontent.com`网址
**解决方案**
可以在hosts文件里面直接添加上网站和 ip 的映射
在网站`https://www.ipaddress.com/`找到`raw.githubusercontent.com`对应 ip `199.232.96.133`
使用一下命令进行 ip 和域名映射
```shell
echo "199.232.96.133 raw.githubusercontent.com" >> /etc/hosts
```

### 添加 Node 节点到集群（node）
如果要添加 node 节点到集群时，可以使用在初始化后生成的
```shell
kubeadm join 192.168.56.106:6443 --token gr0k68.q8g7cezvkbk8n0wr \
    --discovery-token-ca-cert-hash sha256:63bb33710ba6d9387cd4c8537c40aa8a19d1d116a051af0d4508a3735504f866
```










