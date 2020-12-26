---
title: "SSL"
date: 2020-12-26T09:06:39+08:00
draft: false
---

## 生成服务端keystore（密钥和证书）

keytool -keystore server.kestore.jks -alias server -validity 365 -storepass blw -kepass blw -genkey -dname \* CH=CA,OU=eBay,O=eBay,L=SH,ST=SH,C=CN\*

## 生成客户端keystore（密钥和证书）

keytool -keystore client.kestore.jks -alias client -validity 365 -storepass blw -kepass blw -gankey -dname \* CH=CA,OU=eBay,O=eBay,L=SH,ST=SH,C=CN\*

## 将CA证书导入服务端truststore

keytool -v -keystore server.truststore.jks -alias CARoot -import -file cat.crt -storepass blw


## 将CA证书导入客户端truststore

keytool -v -keystore client.truststore.jks -alias CARoot -import -file cat.crt -storepass blw


## 导出服务端证书

keytool -keystore server.keystore.jks -alias server -certreq -file server.crt -storepass blw

## 用CA证书给服务端证书签名

openssl x509 -req -CA ca.crt -CAkey ca.key -in server.crt -out server-signed.crt -days 365 -CAcreateserial -passin pass:blw

## 将CA证书导入服务器keystore

keytool -keystore server.keystore.jks -alias CARoot -import -file ca.crt -storepass blw

## 将已签名服务端证书导入服务端keystore

keytool -keystore server.keystore.jks -alias server -import -file server.signed.crt -storepass blw

## 验证broker SSL是否已经生效

openssl s_client -debug -connect localhost:9093 -tls1