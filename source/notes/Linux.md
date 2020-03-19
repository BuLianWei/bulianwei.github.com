

#Linux

### 1. 挂载

~~~
mount -t iso9660 -o ro /dev/cdrom /mnt/cdrom  //将文件类型为iso9660的文件 以只读（ro）方式从 /dev/cdrom 挂载到/mnt/cdrom
~~~

### 2. 设置开机自动挂载
~~~
vi /etc/fstab
/dev/cdrom	/mnt/cdrom	iso9660	defaults	0	0
~~~

### 3. 关闭防火墙
~~~
chkconfig iptables --list
chkconfig iptables  off   //重启时也自动关闭
~~~
### 4. 设置yum本地源
~~~
cd /etc/yum.repos.d/
修改baseurl=file:///或http://
~~~

### 5. 将自己的包配置成yum库
~~~
进入到repo目录
执行命令：createrepo  .  
~~~

### 6. rename 批量重命名
~~~
rename .a .b *.a   //将所有.a结尾的文件重命名成.b结尾的文件
~~~

### 7. 服务
~~~
service httpd status  //查看服务状态
service httpd start   //开启服务
service httpd restart //重启服务
service httpd stop  //关闭服务
~~~

### 8. 关闭SELinux
~~~
vi /etc/selinux/config
修改SE Linux=disabled
~~~


### 9. 添加字符串到文件
~~~
“>” 将一条命令重定向到一个文件，会覆盖原文件；
“>>” 将一条命令追加到一个文件，不会覆盖，在文件末尾添加；
~~~

### 10. vi快捷键

#### 一般模式下：
~~~
y	复制，3yy 复制三行；
d	删除， 5dd 删除附近五行；
~~~
#### 命令行模式：
~~~
%s／abc／efg	字符串替换。将所有的abc替换成efg
／abc	查找字符串abc ，按n查找下一个，N查找上一个；
~~~
### 11. 修改文件权限
~~~
chmod	u+／-r	文件		给文件拥有者添加／减少 可读权限（u为拥有者，g为用户组，o其他用户）
chmod	777		文件		修改文件为所有人可读可写可执行（r：4，w：2，x：1）
chmod	-r	把文件夹及其下子文件修改为一致；
chown 用户 目录或文件名	修改文件所属用户
~~~
### 12. 用户管理

#### 添加用户：
~~~
useradd	用户名	添加用户
passwd	用户名	修改用户密码
~~~
### 13. 查看大小

#### 查看文件夹：
~~~
du	-sh	文件夹
~~~
#### 查看分区：
~~~
df	-h
~~~

### 14. 跟踪日志文件
~~~
tail -10	文件		跟踪显示后10行
tail	-f	文件		实时跟踪显示文件（只跟踪文件indo号）
tail	-F	文件		实时跟踪显示文件 （跟踪文件的名称）
~~~
### 15. cut
~~~
cut	-d	‘ ：’	-f	1	截取以 ：分割的第一个
~~~
### 16. sort
~~~
sort	-t	' : '	-k  2nr	  将用  ：分割的字符串以第二列数字倒序排列
~~~
### 17. sed
~~~
sed	‘2d’	filename	删除filename第二行	不会改变源文件里面的数据加sed -i 会改变源文件里面的数据
sed	‘/test/’d 	filename	删除匹配test的行
sed	‘2，$d’	filename	删除filename2到结束
sed	‘s/aa/bb/g’	filename	全局将带有aa的行替换成bb
~~~
### 18. awk
~~~
awk	-F	‘ ：’  ‘ { print $1 "," $7 }   将以 ：分割的字符串打印第1和第7列中间用 ，分割
~~~
### 19. expor
~~~
export 定义的变量只对本会话和子会话生效（bash），要想使在子对话定义的变量在父会话中生效，要使用source  /etc/profile**1.添加字符串到文件**
“>” 将一条命令重定向到一个文件，会覆盖原文件；
“>>” 将一条命令追加到一个文件，不会覆盖，在文件末尾添加；
~~~

### 20. 安装centos mini版本后应该处理的问题

#### 1. ifconfig后没有eth0 网卡
~~~
vi /etc/sysconfig/network-scripts/ifcfg-eth0 
ONBOOT=no 改为 yes
~~~

#### 2. 安装ssh
~~~
yum install openssh-server  安装ssh服务
yum install opens-clients    安装ssh客户端
查看ssh是否启动
netstat -antp | grep ssh    可以看到22号端口是否启动
~~~

#### 3. 永久关闭防火墙
~~~
chkconfig  iptables off
service iptables stop （重启以后不会再生效，不能永久关闭）
~~~

#### 4. 修改主机名
~~~
vi  /etc/sysconfig/network
hostname=xxxx     xxx改为需要的名称
~~~
#### 5. 增加用户并为用户添加密码
~~~
adduser   xxx    xxx为用户
passwd    xxx    xxx为用户
~~~
#### 6. 为用户添加权限
~~~
vi /etc/sudoers
找到root all=（all） all
添加xxx  all=（all） all
~~~
#### 7. 当复制一个虚拟机时网卡eth0启动不了
~~~
vi /etc/udev/rules.d/70-persistent-net.rules
将eth0的删除，将eth1的改成eth0
或者查看eth1的序列号将网卡ifcfg-eth0的序列号改成eth1的
~~~
#### 8. 添加几台虚拟机的网址主机名映射
~~~
vi /etc/hosts
添加如192.168.1.101  hadoop01
~~~
#### 9. 为几台虚拟机设置免密登录

### 21. Linux下轻量级的集群管理利器ClusterShell