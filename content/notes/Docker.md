---
title: Docker
---

# Docker

## 命令

### 镜像
+ docker images	//列出本地镜像
  + -a	//列出本地所有镜像
  + -q	//只显示镜像ID
  + --digests	//显示镜像的摘要信息
  + --no-trunc	//显示完整的镜像信息

+ docker search	//查找镜像
	+ --no-trunc	//显示完整的镜像描述
	+ -s	//列出收藏数不小于制定值的镜像
	+ --automated	//只列出automated build 类型的镜像

+ docker pull	//下载镜像

+ docker rmi	//删除镜像
	+ -f	//强制删除

+ docker commit	//提交容器副本使之成为一个新镜像
	+ -m	//提交信息
	+ -a	//作者

### 容器

#### 运行容器

docker run  [params] imagename [command]

 使用镜像创建并启动容器

params:

+ -i	//保存容器，通常与t使用
+ -t	//为容器重新分配一个熟人终端，通常与-i同用
+ -d	//后台运行容器
+ --name	//为容器指定名字
+ -p	//指定端口映射
+ -P（大写）	//随机端口映射
+ -v	//添加数据卷  ／宿主机绝对路径:/容器内目录    宿主机或容器的目录不存在时 会自动创建

举例：docker run -it --name mycentos -p 80:80 centos bash



> -it ：交互式，退出后容器也会退出
> -id：守护式，退出后容器不会退出


#### 查看容器

docker ps	[params] 
列出现在正在运行的容器
params：
+ -a	//列出所有的容器
+ -l	//显示最近创建的容器
+ -n	//显示最近n个创建的容器
+ -q	//只显示容器编号
+ --no-trunc	//不截断输出

举例：docker ps -a

#### 进入运行的容器

+ docker exec [command] containerid|containername [command]
  在运行的容器中执行命令
  exit退出后容器不会退出

  举例：docker exec -it centos bash

+ docker attach	
  进入运行着的容器
  
#### 启动容器

docker start	
启动容器


#### 重启容器

docker restart	
重启容器

#### 停止容器

docker stop	
关闭容器

#### 删除容器

docker rm 
删除容器


#### 其他命令
+ docker top	//查看容器内运行的进程
+ docker inspect	//查看容器内部细节
+ docker cp	//从容器内拷贝内容到主机上
+ docker kill	//杀掉容器
+ docker logs	//查看日志
	+ -t	//加入时间戳
	+ -f	//跟进最新日志打印
	+ --tail	//显示最新多少条


## 容器数据卷
容器的持久化，容器间继承+数据共享
### 数据卷
docker run -v /宿主机绝对路径:/容器内目录 镜像名

宿主机或容器的目录不存在时 会自动创建

### 数据卷容器
docker run --volumes-from 父容器



## DockerFile
是一个由一系列命令和参数组成的构建Docker镜像的文件
### 关键字
+ FROM	//父镜像，依赖镜像
+ MAINTAINER	//镜像维护姓名和邮箱
+ RUN	//容器构建时需要执行的命令
+ EXPOSE	//当前容器对外暴露出的端口
+ WORKDIR	//指定容器创建后，登陆进来的工作目录
+ ENV	//用来在构建镜像过程中设置的环境变量
+ ADD	//将宿主目录下的文件加载进容器里（自动处理url和解压）
+ COPY	//将宿主目录下的文件复制到容器相关目录
+ VOLUME	//容器数据卷，用于数据共享和持久化
+ CMD	//指定容器启动时要运行的命令（多个命令时只有最后一个生效，运行时指定的命令耶会覆盖cmd 命令）
+ ENTRYPOINT	//指定容器启动时要运行的命令，运行时指定的命令会被追加）
+ ONBUILD	//当构建一个被继承的DockerFile时的运行命令，父镜像的onbuild 在子镜像被build时触发










