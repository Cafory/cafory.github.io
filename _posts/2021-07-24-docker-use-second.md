---
title: Docker使用（二）
tags: Docker
---

# 前言

在很多时候我们需要将宿主机的文件复制到Docker容器里面，或者在Docker启动时挂载宿主机的一个目录。本节主要记录Docker复制宿主机目录及挂载宿主机目录的命令以及方法。

<!--more-->

# 复制


## 宿主机 -> 容器

从宿主机拷贝文件至容器可以执行以下命令：

```shell
docker cp <file path in host>  <containerID> ：<file path in container>
```

以下以Windows 下 _Docker Desktop_为例，演示从宿主机拷贝文件至容器的功能。

首先打开控制台，查看所有镜像。
> `docker ps -a`

![image-20210723210045225](https://gitee.com/cafory/images-store/raw/master/Image/image-20210723210045225.png)



发现容器有点多，那么可以执行以下命令删除全部容器：

> `docker rm $(docker ps -a -q)`

![image-20210723210148390](https://gitee.com/cafory/images-store/raw/master/Image/image-20210723210148390.png)

删除之后，新建一个测试容器

> `docker run --name cptest -itd ubuntu:latest /bin/bash`
>
> + --name <name of container> : 指定容器名称 

![image-20210723210642107](https://gitee.com/cafory/images-store/raw/master/Image/image-20210723210642107.png)

可以看到已经创建了名为***cptest***的容器。

执行以下命令

``` shell
[In Host]: docker exec -it cptest /bin/bash  # 打开容器
[In Container]: pwd    	                     # 查看当前路径
[In Container]: mkdir /cptest                # 创建测试文件夹
[In Container]: ls                           # 查看当前文件夹
```

结果如下所示：

![image-20210723211551263](https://gitee.com/cafory/images-store/raw/master/Image/image-20210723211551263.png)

OK，目前已经将测试文件夹创建完毕，接下来执行如下命令复制测试文件***cptest.txt***以及测试文件夹***cptest***。

``` shell
[In Host]: docker cp .\cptest.txt cptest:/cptest/     # 复制文件
[In Host]: docker cp .\cptest\ cptest:/cptest/        # 复制文件夹
[In Host]: docker exec -it cptest /bin/bash           # 打开容器
[In Container]: cd /cptest/                           # 打开测试文件夹
[In Container]: ls                                    # 查看文件夹内容
```

结果如下所示：

![image-20210723212515621](https://gitee.com/cafory/images-store/raw/master/Image/image-20210723212515621.png)

可以发现宿主机的文件及文件夹已经全部复制到容器中去了。

## 容器 -> 宿主机

执行以下命令可以将容器内的文件或者目录复制到宿主机。

``` shell
docker cp <container>:<path in container>  <path in host>
```

将容器***cptest***中`/cptest/cptest.txt`复制到宿主机`./cptest/`下。命令如下所示：

```shell
[In Host]: docker cp cptest:/cptest/cptest.txt ./cptest/
```

![image-20210724132509726](https://gitee.com/cafory/images-store/raw/master/Image/image-20210724132509726.png)

在宿主机目标文件夹下可以看到该文件。

![image-20210724132604164](https://gitee.com/cafory/images-store/raw/master/Image/image-20210724132604164.png)



# 挂载

## 正常挂载

通过**挂载**可以使得容器共享宿主机的目录及文件，命令如下：

```shell
# 通过-v参数指定挂载的目录
[In Host]: docker run -it -v <path in host (abs)>:<path in container><:permission> <image> <command/app> 
```

例如将宿主机`./volumetest`目录挂载至容器**volumetest**， 可以执行以下命令：

```shell
[In Host]: docker run -it -v C:\Users\XJTU-IAIR-09\volumetest/volumetest/:/volumetest ubuntu:latest /bin/bash
```

![image-20210724142623776](https://gitee.com/cafory/images-store/raw/master/Image/image-20210724142623776.png)

如上所示，即完成目录的挂载，在容器内的操作也会影响宿主机的目录及文件。

默认挂载的路径权限为读写。如果指定为只读可以修改`<:permission>`为`:ro`。



## 数据卷挂载

数据卷：“一个专门用来提供数据卷供其它容器挂载的正常的容器”。其他的容器启动可以直接挂载数据卷容器中定义的挂载信息。可以使用`--volumes-from <container>`在启动容器时指定需要挂载的数据卷。

```shell
# 创建数据卷
[In Host]: docker run -v C:\Users\XJTU-IAIR-09\volumetest:/volumetest  --name dataVol ubuntu:latest /bin/bash 
# 利用--volumes-from  从数据卷挂载
[In Host]: docker run -it --volumes-from dataVol ubuntu:latest /bin/bash
```

![image-20210724142547709](https://gitee.com/cafory/images-store/raw/master/Image/image-20210724142547709.png)

在容器查看，可以看到宿主机的测试文件夹已经挂载至新的容器中。





