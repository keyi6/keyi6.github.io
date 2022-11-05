---
title: install oracle on Mac os X | mac下oracle的最佳实践
date: 2018-04-11 18:52:23
tags:
	- docker
	- oracle
---

想在 mac OS 下安装 oracle？so easy！虽然 oracle 在 os X 下并没有 release，但是有了 docker 我们可以轻松实现。

## 安装docker
参考[这个](https://yeasy.gitbooks.io/docker_practice/content/install/mac.html)

## 在docker里安装Oracle
1. 先下载oracle的image
```sh
docker pull wnameless/oracle-xe-11g
```
2. 运行并开放窗口
```sh
docker run -d -p 49160:22 -p 49161:1521 wnameless/oracle-xe-11g
```
3. 记下对应的IMAGE ID
```sh
docker image ls
```
好了，现在可以在终端里通过
```sh
docker exec -it 你的IMAGEID /bin/bash
```
这条命令在你的终端里跑oracle了。

但是，如果想使用GUI怎么办呢？
## 用SQL developer连接到你的oracle
去[下载SQL developer](http://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html)，然后用ssh连接到你的oracle。
默认信息为
```
hostname: localhost
port: 49161
sid: xe
username: system
password: oracle
```

注意连接的时候还有写Host和Port，用下面这条命令就能查询到你的oracle在的Host和Port。
```sh
docker ps
```


