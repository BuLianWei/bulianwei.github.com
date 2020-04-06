---
title: 自建CA证书认证
date: 2020-04-05 11:21:00
categories: Linux,Web
tags: Linux,Web
---


# 自建CA证书认证

CA证书认证通常包含三部分，CA认证服务器、业务服务器、客户端，也可以简单分成两部分CA/业务服务器，和客户端

## CA服务器

### 生成私钥(pem)
openssl genrsa -out cakey.pem -des 2048

>gen:生成	 rsa:加密算法	out:输出	des:秘钥加密口令(可加可不加)	2048:秘钥生成长度(2048 bits)，默认1024

### 生成根证书签发请求(csr)
openssl req -new -key cakey.pem -out cakey.csr

### 生成自签发根CA证书
创建CA的方式有多种，可以分别使用req、x509、ca 等伪命令来创建和签发CA证书

- req
openssl req -new -x509 -key cakey.pem -days 365 -out cacert.pem

- x509
openssl x509 -req -in cakey.csr -signkey key.pem -out cacert.pem

- ca
openssl ca -selfsign -keyfile cakey.pem -in cakey.csr -batch -out cacert.pem


>这是采用 etc/pki/tls/openssl.cnf 默认文件的配置方式生成CA证书，是比较规范推荐的生成方式，
>采用默认配置 -key 、-signkey、-keyfile等属性可以不用添加，系统会到默认文件夹下自己加载
>使用默认配置文件方式必须提前按照规定创建好index.txt和serial文件，其他文件和文件夹的命名也必须一致，看[相关重要文件夹](#相关重要文件夹)

### 获取serial
获取客户端/服务端证书的serial
openssl x509 -in client.crt/server.crt -noout -subject

### 吊销证书
对比获取到的serial序列号是否与index数据库中的信息一致，一致吊销证书
openssl ca -revoke serial.pem
serial为new_certs_dir文件夹下要吊销的证书编号与在客户端或者服务端查出的serial一致

### 刷新吊销列表
第一次要生成吊销证书序号`echo 00 > /etc/pki/CA/crlnumber`
openssl ca -gencrl -out crl.crl



## 业务服务端

### 生成私钥(pem)
openssl genrsa -out server.pem -des 2048

### 生成证书签发请求(csr)
openssl req -new -key server.pem -out server.csr

### 由根CA颁发证书
openssl ca -in server.csr -out server.crt -days 100

## 客户端

### 生成私钥(pem)
openssl genrsa -out client.pem -des 2048

### 生成证书签发请求(csr)
openssl req -new -key client.pem -out client.csr

### 由根CA颁发证书
openssl ca -in client.csr -out client.crt -days 100




## 相关重要文件夹

dir     				 	   =   		   /etc/pki/CA							
certs  				   	= 		      \$dir/certs							  //证书归档文件夹
database			     =			  \$dir/index.txt					    //证书数据库
new_certs_dir		 =		  	\$dir/newcerts					   //新颁发证书文件夹
certificates			  = 		 	\$dir/cacet.pem					 //Root CA
serial						=		      \$dir/serial						 	//下一个颁发证书的序列号，由16进制数字构成最低2位
private_key	          =   		  \$dir/private/cakey.pem	  //私钥


## 其他命令

- 生成用户密码口令
openssl passwd -1 -salt
-1 为使用的加密算法(hash)，-salt是加盐(最长8bits)，可以随机或者自己在后面指定

- 加密
openssl enc -e -des3 -a -salt -in encode.txt -out encode.enc
	enc：加密/解密	-e：加密	-des3：加密算法	-salt：加盐	-in：输入文件	-out：输出文件

- 解密
openssl enc -d -des3 -a -salt -in encode.enc -out encode.encd
-d：解密



> 从私钥中获取公钥
> openssl rsa -in cakey.pem -pubout -out cakey.pubkey
> 显示证书信息
> openssl x509 -in server.csr -noout -text 
> 查看crl文件
> openssl crl -in crl.crl -noout -text
>
> tls：传输层安全协议 Transport Layer Security的缩写
> ssl：安全套接字层 Secure Socket Layer的缩写
> csr：是Certificate Signing Request的缩写，即证书签名请求，这不是证书
> crt：即 certificate的缩写，即证书。linux/unix常用的证书后缀，一般为ascii文件
>
> cer：是crt的替代形式，windows常用证书的后缀，一般为二进制文件。证书中没有私钥
>
> pem：Privacy Enhanced Mail,打开看文本格式,以"-----BEGIN..."开头, "-----END..."结尾,内容是BASE64编码.