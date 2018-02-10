---
layout: file
title: docker镜像迁移
date: 2018-02-10 16:36:21
tags: 
 - docker
 - 知识类
---
### docker镜像迁移

#### 从FROM获取docker镜像文件
```linux
## 查询docker下的镜像文件列表
docker images
REPOSITORY   TAG   IMAGE ID    CREATED      SIZE
pgbi/kong-dashboard      latest              05418433409c        12 days ago     91.2MB
kong                                            latest              399d655a9074        2 weeks ago         325MB
postgres                                        9.4                 92cb571ca65a        7 weeks ago         263MB
goodraincloudframeworks/docker-kong-dashboard   latest              1fbde1bdb34f        8 months ago        325MB

## 保存其中的某个镜像文件

docker save -o postgresql.tar postgresql

## 查询当前保存的文件

ls
postgres.tar

```
#### 将tar 文件迁移到TO机器
> FTP工具拷贝

#### 在TO加载镜像文件

```linux
## 加载镜像
docker load -i postgres.tar
4bcdffd70da2: Loading layer 129.3 MB/129.3 MB
740247c87968: Loading layer 345.1 kB/345.1 kB
8e68806cca8a: Loading layer 3.178 MB/3.178 MB
1b98abe192d2: Loading layer 20.31 MB/20.31 MB
bcc6a77e9f72: Loading layer 1.536 kB/1.536 kB
24b339fdccfc: Loading layer 8.704 kB/8.704 kB
cec2e9297839: Loading layer 120.5 MB/120.5 MB
88d3d7da20da: Loading layer 25.09 kB/25.09 kB
b29d8d9cf54a: Loading layer 2.048 kB/2.048 kB
164aac234ac5: Loading layer 3.072 kB/3.072 kB
d7736c0ac75f: Loading layer 7.168 kB/7.168 kB
90c0e779880d: Loading layer 1.536 kB/1.536 kB
Loaded image: postgres:9.4r    512 B/1.536 kB

## 查询当前加载的镜像
docker images
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
kong                                            latest              399d655a9074        2 weeks ago         325.3 MB
postgres                                        9.4                 92cb571ca65a        7 weeks ago         262.7 MB
goodraincloudframeworks/docker-kong-dashboard   latest              1fbde1bdb34f        8 months ago        324.5 MB


## 启动镜像
docker run XXXXX
```