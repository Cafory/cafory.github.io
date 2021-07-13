---
title: Docker使用（一）
tags: Docker
---

# 简介

Docker 是一个开源的应用容器引擎，基于 [Go 语言](https://www.runoob.com/go/go-tutorial.html) 并遵从 Apache2.0 协议开源。

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

Docker 从 17.03 版本之后分为 CE（Community Edition: 社区版） 和 EE（Enterprise Edition: 企业版），我们用社区版就可以了。

<!--more-->

# 安装


详细教程可见菜鸟教程的[Docker 教程 ](https://www.runoob.com/docker/docker-tutorial.html)以及[C语言中文网的教程](http://c.biancheng.net/view/3121.html))。

测试使用的平台是Windows，首先需要在___程序___里面开启___Hyper-V___服务。***Hyper-V***见微软官网介绍：[Windows 10 上的 Hyper-V 简介](https://docs.microsoft.com/zh-cn/virtualization/hyper-v-on-windows/about/)。

![开启Hyper-V](https://gitee.com/cafory/images-store/raw/master/Image/image-20210713145516449.png)



完成Hyper-V的安装后，下载[Docker Desktop for Windows](https://www.docker.com/products/docker-desktop)并安装。

安装完成后重启电脑，打开Docker，即可完成安装。

![Docker主界面](https://gitee.com/cafory/images-store/raw/master/Image/image-20210713153221139.png)

在Windows命令行窗口输入`docker version`，出现如下界面即安装成功。

![image-20210713154053337](https://gitee.com/cafory/images-store/raw/master/Image/image-20210713154053337.png)


#### **可能出现的问题**

+ WSL 2 未能完全安装  

![image-20210713152321271](https://gitee.com/cafory/images-store/raw/master/Image/image-20210713152321271.png)

> 按照[在 Windows 10 上安装 WSL ](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10#step-4---download-the-linux-kernel-update-package)文档重新安装WSL 2，重启Docker即可。  

# 使用


本节主要介绍Docker的使用方法，内容来源于**菜鸟教程**以及**C语言中文网** 。

## 一些概念

Docker包括三个基本概念：  

>+ **镜像(Image)**：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。
>+ **容器（Container）**：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
>+ **仓库（Repository）**：仓库可看成一个代码控制中心，用来保存镜像。

   

Docker架构如下表所示：  

| 概念                   | 说明                                                         |
| :--------------------------- | :----------------------------------------------------------- |
| Docker 镜像(Images)    | Docker 镜像是用于创建 Docker 容器的模板，比如 Ubuntu 系统。  |
| Docker 容器(Container) | 容器是独立运行的一个或一组应用，是镜像运行时的实体。         |
| Docker 客户端(Client)  | Docker 客户端通过命令行或者其他工具使用 Docker SDK (https://docs.docker.com/develop/sdk/) 与 Docker 的守护进程通信。 |
| Docker 主机(Host)      | 一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。       |
| Docker Registry        | Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。Docker Hub([https://hub.docker.com](https://hub.docker.com/)) 提供了庞大的镜像集合供使用。一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 **<仓库名>:<标签>** 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 **latest** 作为默认标签。 |
| Docker Machine         | Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。 |

## 镜像（Image）


如果曾经做过 VM 管理员，则可以把 [Docker](http://c.biancheng.net/docker/) 镜像理解为 VM 模板，VM 模板就像停止运行的 VM，而 Docker 镜像就像停止运行的容器；而作为一名研发人员，则可以将镜像理解为类（Class）。

创建**容器**需要从**镜像仓库**拉取镜像至本地。常见的镜像仓库为Docker Hub，也可以使用其他仓库。

通过拉取至本地的镜像可以创建启动一个或多个容器。

通过 **<仓库名>:<标签>** 在镜像仓库定位我们需要的镜像并拉取至本地，拉取指令为：

> `docker pull  <repository>:<tag>`  
>
> 或者  
>
> `docker image pull  <repository>:<tag>`  

​     

如果希望从第三方镜像仓库服务获取镜像（非 Docker Hub），则需要在镜像仓库名称前加上第三方镜像仓库服务的 DNS 名称。

> `docker image pull DNS/<repository>:<tag>`  

   

查看镜像使用以下指令：

> `docker image ls`  
>
> 添加过滤器
>
> `docker image ls --filter [选项] `  
>
> [选项]可为:
>
> - dangling：可以指定 true 或者 false，仅返回悬虚镜像（true），或者非悬虚镜像（false）。
> - before：需要镜像名称或者 ID 作为参数，返回在之前被创建的全部镜像。
> - since：与 before 类似，不过返回的是指定镜像之后创建的全部镜像。
> - label：根据标注（label）的名称或者值，对镜像进行过滤。docker image ls命令输出中不显示标注内容。

​      

删除镜像使用如下命令：

> `docker image rm IMAGE-ID`  

执行该命令删除**IMAGE-ID**对应的镜像。需要注意的是，在执行该命令之前，需要停止并删除与该镜像关联的容器。

  

## 容器（Container）


当我们安装 Docker 的时候，会涉及两个主要组件：Docker 客户端和 Docker daemon（有时也被称为“服务端”或者“引擎”）。

daemon 实现了 Docker 引擎的 API。

使用 Linux 默认安装时，客户端与 daemon 之间的通信是通过本地 IPC/UNIX Socket 完成的（/var/run/docker.sock）；在 Windows 上是通过名为 npipe:////./pipe/docker_engine 的管道（pipe）完成的。

可以使用`docker version`命令来检测客户端和服务端是否都已经成功运行，并且可以互相通信。

在使用容器之前，需要先使用`doker version`命令查看客户端与服务端是否都已经正常运行。

  

执行以下命令以启动容器：

> `docker [container] run <options> <im- age>:<tag> <app>`  

例如执行

> `docker run ubuntu:15.10 /bin/echo "Hello world"`
>
> 解释如下：
>
> + `ubuntu:15.10`：镜像名称
>
> + `/bin/echo "Hello world"`：运行容器时启动的程序或者执行的命令

运行结果如下所示：

![image-20210713202327496](https://gitee.com/cafory/images-store/raw/master/Image/image-20210713202327496.png)

  

通过使用参数 `-i -t`使得容器可以具有交互能力，执行如下命令：

> `docker run -it ubuntu /bin/bash`
>
> 参数解释：
>
> - **-t:** 在新容器内指定一个伪终端或终端。
> - **-i:** 允许你对容器内的标准输入 (STDIN) 进行交互。

出现以下结果：

![image-20210713202538492](https://gitee.com/cafory/images-store/raw/master/Image/image-20210713202538492.png)

由`root@f9c4a3b68843`可知已经进入一个Ubuntu容器。

  

如果想要退出容器执行`exit`命令即可。此时则会结束容器运行。如果想要运行已经结束运行的容器，可以执行以下命令：

> `docker start CONTAINER-ID`

执行此命令需要指导容器的ID，可以通过 以下命令查看容器ID：

> `docker ps`：查看正在运行的容器  
>
> `docker ps -a`：查看所有容器  

执行结果如下图所示：  

![image-20210713203427840](https://gitee.com/cafory/images-store/raw/master/Image/image-20210713203427840.png)

 执行启动容器指令结果如下图所示：

![image-20210713204036600](https://gitee.com/cafory/images-store/raw/master/Image/image-20210713204036600.png)

  

有时候我们希望容器可以后台运行，此时只需要在`docker run`后面加上`-d`参数即可，完整指令如下所示：  

> `docker run -itd ubuntu /bin/bash`  

执行结果如下图所示：  

![image-20210713204709406](https://gitee.com/cafory/images-store/raw/master/Image/image-20210713204709406.png)

执行此指令默认不会进入容器内，如果想要进入容器，可以执行以下命令：  

> + `docker attach CONTAINER-ID`：此命令进入容器，退出容器后，容器停止
> + `docker exec CONTAINER-ID`：此命令进入容器，退出容器后，容器继续后台运行

当然如果不在容器里面无法通过`exit`命令停止容器，此时我们可以通过如下命令停止容器：

> `docker stop CONTAINER-ID`   



删除容器可以使用如下指令：

> `docker rm -f CONAINTER-ID`  



