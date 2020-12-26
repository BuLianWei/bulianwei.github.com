---
title: "Trouble"
date: 2020-11-16T22:12:47+08:00
draft: false
---

1.  Could not transfer artifact org.apache.flink:flink-runtime_2.11:jar:1.10.0 from/to central (https://repo.maven.apache.org/maven2): GET request of: org/apache/flink/flink-runtime_2.11/1.10.0/flink-runtime_2.11-1.10.0.jar from central failed: Premature end of Content-Length delimited message body (expected: 12,008,735; received: 2,379,621) -> [Help 1]

   由于之前从网上加载依赖包没有加载完全，导致本地库中的包不完全，所以没有办法重新加载依赖。可以直接找到相应的包目录将目录删除，然后从新下载。

   本文中直接删除`.m2/repository/org/apache/flink/flink-runtime_2.11/1.10.0 `目录即可  或者 mvn clean install -e -U 强制性更新（强制性从远程仓库拉取包文件）

2. 

