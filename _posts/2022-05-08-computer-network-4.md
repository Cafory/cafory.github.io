---
title: 计算机网络-04
tags: Network


---

# 第四章 网络层：数据平面

在本章和下一章中 ， 我们将学习网络层实际是怎样实现主机到主机的通信服务的 。 我们将看到与运输层和应用层不同的是 ， 在网络中的每一台主机和路由器中都有一个网络层部分。 正因如此 ，网络层协议是协议桟中最具挑战性 （ 因而也是最有趣）的部分。网络层在协议栈中毋庸置疑是最复杂的层次 ， 因此我们将在这里用大量篇幅来讨论 。的确因为涉及的内容太多 ， 我们要用两章的篇幅来讨论网络层 。 我们将看到网络层能够被分解为两个相互作用的部分 ， 即数据平面和控制平面 。



<!--more-->

## 4.1 网络层概述

> 网络层是[OSI参考模型](https://baike.baidu.com/item/OSI参考模型)中的第三层，介于[传输层](https://baike.baidu.com/item/传输层/4329536)和[数据链路层](https://baike.baidu.com/item/数据链路层/4329290)之间，它在数据链路层提供的两个相邻端点之间的数据帧的传送功能上，进一步管理网络中的[数据通信](https://baike.baidu.com/item/数据通信/897073)，将数据设法从源端经过若干个中间[节点](https://baike.baidu.com/item/节点/865052)传送到目的端，从而向传输层提供最基本的端到端的[数据传送](https://baike.baidu.com/item/数据传送/500685)服务。主要内容有：虚电路分组交换和[数据报](https://baike.baidu.com/item/数据报)分组交换、[路由选择](https://baike.baidu.com/item/路由选择/10824858)[算法](https://baike.baidu.com/item/算法/209025)、阻塞控制方法、[X.25协议](https://baike.baidu.com/item/X.25协议)、综合业务数据网（ISDN）、[异步传输模式](https://baike.baidu.com/item/异步传输模式/511955)（ATM）及网际互连原理与实现。【来源：[百度百科](https://baike.baidu.com/item/%E7%BD%91%E7%BB%9C%E5%B1%82/4329439?fr=aladdin)】

图 4-1 显示了一个简单网络， 其中有 H1 和 H2 两台主机 ， 在 H1 与 H2 之间的路径上有几台路由器。假设 H1 正在向 H2 发送信息 ，考虑这些主机与中间路由器的网络层所起的作用 。 H1 中的网络层取得来自于 H1 运输层的报文段 ， 将每个报文段封装成一个数据报 ， 然后向相邻路由器 R1 发送该数据报 。 在接收方主机 H2, 网络层接收来自相邻路由器R2 的数据报 ， 提取出运输层报文段 ， 并将其向上交付给 H2 的运输层 。 每台路由器的数据平面的主要作用是从其输入链路向其输出链路转发数据报 ； 控制平面的主要作用是协调这些本地的每路由器转发动作 ， 使得数据报沿着源和目的地主机之间的路由器路径最终进行端到端传送 。

![image-20220508110500681](https://gitee.com/cafory/images-store/raw/master/Image/202205081105844.png/)

### 4.1.1 转发和路由选择 ： 数据平面和控制平面

网络层的作用从表面上看极为简单 ， 即将分组从一台发送主机移动到一台接收主机 。为此 ，需要使用两种重要的网络层功能：

+ **转发。**当一个分组到达某路由器的一条输入链路时 ，该路由器必须将该分组移动到适当的输出链路 。例如 ， 在图 4-1 中来自主机 H1 到路由器 R1 的一个分组， 必须向到达 H2 的路径上的下一台路由器转发。 如我们将看到的那样 ，转发是在数据平面中实现的唯一功能（ 尽管是最为常见和重要的功能） 。 在最为常见的场合（ 我们将在 4. 4 节中讨论）， 分组也可能被现有的路由器阻挡 （ 例如 ，该分组来源于一个已知的恶意主机 ， 或者该分组发向一个被禁止的目的主机 ）， 或者可能是冗余的并经过多条出链路发送 。

+ **路由选择。**当分组从发送方流向接收方时 ，网络层必须决定这些分组所采用的路由或路径 。 计算这些路径的算法被称为**路由选择算法 ( routing algorithm )** 。例如，在图 4-1 中一个路由选择算法将决定分组从 H1 到 H2 流动所遵循的路径 。 路由选择在网络层的控制平面中实现 。

> 在讨论网络层时 ，许多作者经常交替使用转发和路由选择这两个术语 。 我们在本书中
> 将更为精确地使用这些术语 。 **转发 ( forwarding)** 是指将分组从一个输入链路接口转移到适当的输出链路接口的路由器本地动作。 转发发生的时间尺度很短(通常为几纳秒) ， 因此通常用硬件来实现 。 **路由选择 ( routing)** 是指确定分组从源到目的地所采取的端到端路径的网络范围处理过程 。 路由选择发生的时间尺度长得多(通常为几秒) ， 因此通常用软件来实现 。 用驾驶的例子进行类比 ，考虑在 1.3.1 节中旅行者所历经的从宾夕法尼亚州到佛罗里达州的行程 。 在这个行程中 ，那位驾驶员在到佛罗里达州的途中经过了许多立交桥 。 我们能够认为转发就像通过单个立交桥的过程： 一辆汽车从其道路上进入立交桥的一个入口 ， 并且决定应当走哪条路来离开该立交桥 。 我们可以把路由选择看作是规划从宾夕法尼亚州到佛罗里达州行程的过程： 在着手行程之前 ，驾驶员已经查阅了地图并在许多可能的路径中选择一条 ， 其中每条路径都由一系列经立交桥连接的路段组成 。

每台网络路由器中有一个关键元素是它的**转发表 ( forwarding table )** 。 路由器检査到达分组首部的一个或多个字段值 ，进而使用这些首部值在其转发表中索引 ，通过这种方法来转发分组 。 这些值对应存储在转发表项中的值 ， 指出了该分组将被转发的路由器的输出链路接口。例如在图 4-2 中 ， 一个首部字段值为 0111 的分组到达路由器。 该路由器在它的转发表中索引 ， 并确定该分组的输出链路接口是接口 2 。 该路由器则在内部将该分组转发到接口 2 。 在 4.2 节中 ， 我们深入路由器内部， 更为详细地研究这种转发功能 。 转发是由网络层的数据平面执行的主要功能 。

![image-20220508110928016](https://gitee.com/cafory/images-store/raw/master/Image/202205081109112.png/)

+ **控制平面： 传统的方法**

你也许现在想知道路由器中的转发表一开始是如何配置的 。这是一个关键问题， 它揭示了路由选择和转发间的重要相互作用关系 。 如图 4-2 所示，路由选择算法决定了插入该路由器转发表的内容 。 在这个例子中 ，路由选择算法运行在每台路由器中 ， 并且在每台路由器中都包含转发和路由选择两种功能 。 如我们将在 5. 3 节和 5. 4 节中所见， 在一台路由
器中的路由选择算法与在其他路由器中的路由选择算法通信 ， 以计算出它的转发表的值。这种通信是如何执行的呢 ？通过根据路由选择协议交换包含路由选择信息的路由选择报文 ！ 我们将在 5.2-5. 4 节讨论路由选择算法和协议 。

+ **控制平面： SDN 方法**

图 4-2 中显示的实现路由选择功能的方法 ， 是路由选择厂商在其产品中采用的传统方法 ，至少最近还是如此 。使用该方法 ， 每台路由器都有一个与其他路由器的路由选择组件通信的路由选择组件。 然而， 对人类能够手动配置转发表的观察启发我们 ， 对于控制平面功能来说， 也许存在其他方式来确定数据平面转发表的内容 。

![image-20220508111435767](https://gitee.com/cafory/images-store/raw/master/Image/202205081114886.png/)

图 4-3 显示了从路由器物理上分离的另一种方法 ，远程控制器计算和分发转发表以供每台路由器所使用 。 注意到图 4-2 和图 4-3 的数据平面组件是相同的 。而在图 4-3 中 ， 控制平面路由选择功能与物理的路由器是分离的， 即路由选择设备仅执行转发 ，而远程控制器计算并分发转发表 。 远程控制器可能实现在具有高可靠性和冗余的远程数据中心中 ， 并可能由 ISP 或某些第三方管理 。 器计算并分发转发表 。 远程控制器可能实现在具有高可靠性和冗余的远程数据中心中 ， 并可能由 ISP 或某些第三方管理 。 路由器和远程控制器是如何通信的呢 ？通过交换包含转发表和其他路由选择信息的报文 。 显示在图 4-3 中的控制平面方法是**软件定义网络 （ Software-Defined Networking, SDN）** 的本质， 因为计算转发表并与路由器交互的控制器是用软件实现的， 故网络是 “ 软件定义” 的 。 这些软件实现也越来越开放 ， 换言之类似于 Linux操作系统代码，这些代码可为公众所用， 允许 ISP （ 以及网络研究者和学生） 去创新并对控制网络层功能的软件提出更改建议 。 我们将在 5. 5 节中讨论 SDN 控制平面 。

### 4.1.2 网络服务模型

在钻研网络层的数据平面之前 ， 我们将以开阔的视野来专注于我们引入的新东西并考虑网络层可能提供的不同类型的服务。 当位于发送主机的运输层向网络传输分组时 （ 即在发送主机中将分组向下交给网络层 ），运输层能够指望网络层将该分组交付给目的地吗?当发送多个分组时 ， 它们会按发送顺序按序交付给接收主机的运输层吗 ？ 发送两个连续分组的时间间隔与接收到这两个分组的时间间隔相同吗 ？网络层会提供关于网络中拥塞的反馈信息吗 ？ 在发送主机与接收主机中连接运输层通道的抽象视图 （特性 ） 是什么 ？ 对这些
问题和其他问题的答案由网络层提供的服务模型所决定 。 **网络服务模型 （ network service model ）** 定义了分组在发送与接收端系统之间的端到端运输特性 。

我们现在考虑网络层能提供的某些可能的服务。 这些服务可能包括 ：

+ **确保交付。** 该服务确保分组将最终到达目的地 。
+ **具有时延上界的确保交付。** 该服务不仅确保分组的交付 ，而且在特定的主机到主机时延上界内 （ 例如在 100ms 内 ） 交付。
+ **有序分组交付。** 该服务确保分组以它们发送的顺序到达目的地 。
+ **确保最小带宽 。** 这种网络层服务模仿在发送和接收主机之间一条特定比特率（ 例如1 Mbps ） 的传输链路的行为。只要发送主机以低于特定比特率的速率传输比特（ 作为分组的组成部分 ）， 则所有分组最终会交付到目的主机 。
+ **安全性 。** 网络层能够在源加密所有数据报并在目的地解密这些分组， 从而对所有运输层报文段提供机密性 。

这只是网络层能够提供的服务的部分列表， 有无数种可能的服务变种 。

因特网的网络层提供了单一的服务 ，称为尽力而为服务 （ best ・ effort service ） 。使用尽力而为服务 ， 传送的分组既不能保证以它们发送的顺序被接收 ， 也不能保证它们最终交付 ； 既不能保证端到端时延 ， 也不能保证有最小的带宽 。 尽力而为服务看起来是根本无服务的一种委婉说法 ， 即一个没有向目的地交付分组的网络也符合尽力而为交付服务的定义 ！ 其他的网络体系结构已定义和实现了超过因特网尽力而为服务的服务模型 。

令人感兴趣的是 ， 尽管有这些研发良好的供选方案 ， 但因特网的基本尽力而为服务模型与适当带宽供给相结合已被证明超过 “ 足够好 ” ，能够用于大量的应用， 包括诸如 Netflix 、 IP 语音和视频等流式视频服务 ， 以及诸如 Skype 和 Facetime 等实时会议应用 。

### 第四章概述

在提供了网络层的概述后 ， 我们将在本章后续几节中讨论网络层的数据平面组件。我们在这里顺便提到 ，许多计算机网络研究者和从业人员经常互换地使用**转发**和**交换**这两个术语 。 我们在这本教科书中也将互换使用这些术语 。 在我们开始讨论术语的主题时 ，还需要指岀经常互换使用的两个其他术语， 但我们将更为小心地使用它们。 我们将约定术语分组交换机是指一台通用分组交换设备 ， 它根据分组首部字段中的值 ， 从输入链路接口到输出链路接口转移分组 。 某些分组交换机称为**链路层交换机 （ link-layer switch ）**
（ 在第 6 章仔细学习 ）， 基于链路层帧中的字段值做出转发决定 ，这些交换机因此被称为链路层 （第 2 层 ）设备 。其他分组交换机称为**路由器 （ router）**, 基于网络层数据报中的首部字段值做岀转发决定 。 路由器因此是网络层 （第 3 层 ）设备 。 （ 为了全面理解这种重要区别 ， 你可能要回顾 1.5.2 节， 在那里我们讨论了网络层数据报和链路层帧及其关系 。 ） 因为在本章中我们关注的是网络层 ， 所以我们将主要使用术语路由器来代替交换机 。



##  4.2 路由器工作原理

既然我们已经概述了网络层中的数据平面和控制平面 、 转发与路由选择之间的重要区别以及网络层的服务与功能， 我们将注意力转向网络层的转发功能， 即实际将分组从一台路由器的入链路传送到适当的出链路 。

图 4-4 显示了一个通用路由器体系结构的总体视图 ， 其中标识了一台路由器的 4 个组件。

![image-20220508112304419](https://gitee.com/cafory/images-store/raw/master/Image/202205081123488.png/)

+ **输入端口。**输入端口 （ input port） 执行几项重要功能 。 它在路由器中**执行终结入物理链路的物理层功能**，这显示在图 4-4 中输入端口部分最左侧的方框与输出端口部分最右侧的方框中。 它还要与位于入链路远端的数据链路层交互来**执行数据链路层功能**，这显示在输入与输出端口部分中间的方框中。也许更为重要的是 ， 在输入端口还要**执行查找功能**，这显示在输入端口最右侧的方框中。正是在这里，通过查询转发表决定路由器的输出端口 ， 到达的分组通过路由器的交换结构转发到输出端口。 控制分组（ 如携带路由选择协议信息的分组） 从输入端口转发到路由选择处理器。 注意这里的 “ 端口” 一词， 指的是路由器的物理输入和输出接口 ，这完全不同于第 2 、 3 章中所讨论的与网络应用程序和套接字相关联的软件端口。
+ **交换结构。**输出端口存储从交换结构接收的分组， 并通过执行必要的链路层和物理层功能在输出链路上传输这些分组 。 当一条链路是双向的时 （ 即承载两个方向的流量），输出端口通常与该链路的输入端口成对出现在同一线路卡上。
+ **输出端口。**输出端口存储从交换结构接收的分组， 并通过执行必要的链路层和物理层功能在输出链路上传输这些分组 。 当一条链路是双向的时 （ 即承载两个方向的流量），输出端口通常与该链路的输入端口成对出现在同一线路卡上。
+ **路由选择处理器。**路由选择处理器执行控制平面功能 。 在传统的路由器中 ， 它执行路由选择协议（ 我们将在 5. 3 节和 5. 4 节学习 ），维护路由选择表与关联链路状态信息 ， 并为该路由器计算转发表 。 在 SDN 路由器中 ，路由选择处理器 （ 在其他活动中 ）负责与远程控制器通信 ，目的是接收由远程控制器计算的转发表项， 并在该路由器的输入端口安装这些表项 。 路由选择处理器还执行网络管理功能， 我们将在5.7 节学习相关内容 。

路由器的输入端口、 输出端口和交换结构几乎总是用硬件实现， 如图 4-4 所示 。为了理解为何需要用硬件实现，考虑具有 lOGbps 输入链路和 64 字节的 IP 数据报 ， 其输入端口在另一个数据报到达前仅有 51.2ns 来处理数据报 。 如果 N 个端口结合在一块线路卡上（ 因为实践中常常这样做 ）， 数据报处理流水线必须以 N 倍速率运行，这远快过软件实现的速率 。 转发硬件既能够使用路由器厂商自己的硬件设计来实现， 也能够使用购买的商用硅片的硬件设计来实现 。

当数据平面以纳秒时间尺度运行时 ，路由器的控制功能以毫秒或秒时间尺度运行，这些控制功能包括执行路由选择协议 、 对上线或下线的连接链路进行响应 、 与远程控制器通信 （ 在 SDN 场合 ） 和执行管理功能 。 因而这些**控制平面 （ control plane ）** 的功能通常用软件实现并在路由选择处理器 （通常是一种传统的 CPU） 上执行 。

深入探讨路由器的内部细节之前 ， 我们转向本章开头的那个类比 ， 其中分组转发好比汽车进入和离开立交桥 。假定该立交桥是环状交叉路， 在汽车进入该环状交叉路前 ，需要做一点处理 。 我们来考虑一下对于这种处理需要什么信息 。

+ **基于目的地转发。**假设汽车停在一个入口站上并指示它的最终目的地 （ 并非在本地环状交叉路，而是其旅途的最终目的地 ） 。入口站的一名服务人员查找最终目的地,决定通向最后目的地的环状交叉路的出口 ， 并告诉驾驶员要走哪个出口。
+ **通用转发。**除了目的地之外 ， 服务人员也能够基于许多其他因素确定汽车的出口匝道 。例如 ， 所选择的出口匝道可能与该汽车的起点如发行该车牌照的州有关。 来自某些州的汽车可能被引导使用某个出口匝道（经过一条慢速道路通向目的地 ），而来自其他州的汽车可能被引导使用一个不同的岀口匝道（经过一条高速路通向目的地 ） 。 基于汽车的模型 、品牌和寿命 ， 可能做出相同的决定 。 或者认为不适合上路的汽车可能被阻止并且不允许通过环状交叉路 。 就通用转发来说，许多因素都会对服务人员为给定汽车选择出口匝道产生影响。

一旦汽车进入环状交叉路（该环状交叉路可能挤满了从其他输入道路进入的其他汽车， 朝着其他环状交叉路出口前进）， 并且最终离开预定的环状交叉路出口匝道， 在这里可能遇到了从该岀口离开环状交叉路的其他汽车 。

在这个类比中 ， 我们能够在图 4-4 中识别最重要的路由器组件 ： 入口道路和入口站对应于输入端口 （ 具有查找功能以决定本地输出端口 ）；环状交叉路对应于交换结构 ；环状交叉路出口匝道对应于输出端口。借助于这个类比 ， 我们可以考虑瓶颈可能出现的地方 。如果汽车以极快的速率到达（ 例如 ，该环状交叉路位于德国或意大利 ！）而车站服务人员很慢 ， 将发生什么情况 ？这些服务人员必须工作得多快 ， 以确保在入口路上没有车辆拥堵 ？甚至对于极快的服务人员 ， 如果汽车在环状交叉路上开得很慢 ， 将发生什么情况 ， 拥堵仍会出现吗 ？ 如果大多数进入的汽车都要在相同的出口匝道离开环状交叉路， 将发生什么情况 ， 在岀口匝道或别的什么地方会出现拥堵吗 ？ 如果我们要为不同的汽车分配优先权 ， 或先行阻挡某些汽车进入环状交叉路，环状交叉路将如何运行？这些全都与路由器和交换机设计者面对的问题形成类比 。

### 4.2.1 输入端口处理和基于目的地转发

图 4-5 中显示了一个更详细的输入处理的视图 。 如前面讨论的那样 ，输入端口的线路端接功能与链路层处理实现了用于各个输入链路的物理层和链路层 。 在输入端口中执行的查找对于路由器运行是至关重要的 。 正是在这个地方 ，路由器使用转发表来查找输出端口 ， 使得到达的分组能经过交换结构转发到该输出端口。 转发表是由路由选择处理器计算和更新的（ 使用路由选择协议与其他网络路由器中的路由选择处理器进行交互 ）， 或者转发表接收来自远程 SDN 控制器的内容 。 转发表从路由选择处理器经过独立总线（ 例如一个 PCI 总线） 复制到线路卡 ， 在图 4-4 中该总线由从路由选择处理器到输入线路卡的虚线所指示 。使用在每个输入端口的影子副本 ，转发决策能在每个输入端口本地做出 ， 无须基于每个分组调用集中式路由选择处理器 ， 因此避免了集中式处理的瓶颈 。

![image-20220508113355895](https://gitee.com/cafory/images-store/raw/master/Image/202205081133952.png/)

现在我们来考虑 “ 最简单” 的情况 ， 一个入分组基于该分组的目的地址交换到输岀端口。在 32 比特 IP 地址的情况下 ，转发表的蛮力实现将针对每个目的地址有一个表项 。因为有超过 40 亿个可能的地址 ，选择这种方法总体上是不可行的 。

作为一个说明怎样处理规模问题的例子 ， 假设我们的路由器具有 4 条链路，编号 0 到3, 分组以如下方式转发到链路接口:

![image-20220508113454839](https://gitee.com/cafory/images-store/raw/master/Image/202205081134904.png/)

显然 ， 对于这个例子 ， 在路由器的转发表中没有必要有 40 亿个表项 。例如 ， 我们能够有一个如下仅包括 4 个表项的转发表：

![image-20220508113520954](https://gitee.com/cafory/images-store/raw/master/Image/202205081135994.png/)

使用这种风格的转发表，路由器用分组目的地址的**前缀 ( prefix )** 与该表中的表项进行匹配； 如果存在一个匹配项， 则路由器向与该匹配项相关联的链路转发分组 。例如 ， 假设分组的目的地址是 11001000 00010111 00010110 10100001，因为该地址的 21 比特前缀匹配该表的第一项， 所以路由器向链路接口 0 转发该分组 。 如果一个前缀不匹配前 3 项中的任何一项， 则路由器向链路接口 3 转发该分组 。 尽管听起来足够简单 ， 但这里还是有重要的微妙之处 。你可能已经注意到一个目的地址可能与不止一个表项相匹配 。例如 ， 地址11001000 00010111 00011000 10101010 的前 24 比特与表中的第二项匹配，而该地址的前21 比特与表中的第三项匹配 。 当有多个匹配时 ，该路由器使用**最长前缀匹配规则 ( longest prefix matching rule)** ； 即在该表中寻找最长的匹配项， 并向与最长前缀匹配相关联的链路接口转发分组 。 当在 4. 3 节中详细学习因特网编址时 ， 我们将完全明白使用这种最长前缀匹配规则的理由 。

假定转发表已经存在 ， 从概念上讲表査找是简单的，硬件逻辑只是搜索转发表查找最长前缀匹配 。但在吉比特速率下 ，这种查找必须在纳秒级执行(回想我们前面 10Gbps 链路和一个 64 字节 IP 数据报的例子)。 因此 ， 不仅必须要用硬件执行查找 ，而且需要对大型转发表使用超出简单线性搜索的技术 ； 快速查找算法的综述能够在 ［Gupta 2001, Ruiz-
Sanchez 2011］ 中找到。同时必须对内存访问时间给予特别关注 ，这导致用嵌入式片上
DRAM 和更快的 SRAM (用作一种 DRAM 缓存)内存来设计 。 实践中也经常使用**三态内容可寻址存储器 ( Tenary Content Address Memory, TCAM)** 来查找。使用 TCAM ，一个 32 比特 IP 地址被放入内存 ， TCAM 在基本常数时间内返回对该地址的转发表项的内容 。Cisco Catalyst 6500 和 7500 系列路由器及交换机能够保存 100 多万 TCAM 转发表项。

一旦通过查找确定了某分组的输出端口 ， 则该分组就能够发送进入交换结构 。 在某些设计中 ， 如果来自其他输入端口的分组当前正在使用该交换结构 ， 一个分组可能会在进入交换结构时被暂时阻塞 。 因此 ， 一个被阻塞的分组必须要在输入端口处排队， 并等待稍后被及时调度以通过交换结构 。 我们稍后将仔细观察分组（ 位于输入端口与输出端口中 ）的阻塞 、 排队与调度 。 尽管 “ 查找 ” 在输入端口处理中可认为是最为重要的动作 ， 但必须采取许多其他动作 ： ①必须出现物理层和链路层处理， 如前面所讨论的那样 ； ②必须检查分组的版本号、 检验和以及寿命字段 （这些我们将在 4. 3 节中学习 ）， 并且重写后两个字段；③必须更新用于网络管理的计数器 （ 如接收到的 IP 数据报的数目） 。

在结束输入端口处理的讨论之前 ， 注意到输入端口查找目的 IP 地址 （ “匹配 ” ）， 然后发送该分组进入交换结构 （ “动作” ）的步骤是一种更为一般的 “匹配加动作” 抽象的特定情况 ，这种抽象在许多网络设备中执行，而不仅在路由器中。 在链路层交换机 （ 在第 6章讨论） 中 ，除了发送帧进入交换结构去往输出端口外 ，还要查找链路层目的地址 ， 并采取几个动作。 在防火墙 （ 在第 8 章讨论） 中 ，首部匹配给定准则 （ 例如源 / 目的 IP 地址和运输层端口号的某种组合 ）的入分组可能被阻止转发 ，而防火墙是一种过滤所选择的入分组的设备 。 在**网络地址转换器 （ NAT）** 中 ， 一个运输层端口号匹配某给定值的入分组， 在转发 （ 动作 ） 前其端口号将被重写。 的确， “匹配加动作” 抽象不仅作用大 ，而且在网络设备中无所不在 ， 并且对于我们将在 4.4 节中学习的通用转发是至关重要的 。

### 4.2.2 交换

交换结构位于一台路由器的核心部位， 因为正是通过这种交换结构, 分组才能实际地从一个输入端口交换 （ 即转发 ）到一个输出端口中。交换可以用许多方式完成 ，如图 4-6所示 。

![image-20220508114159444](https://gitee.com/cafory/images-store/raw/master/Image/202205081141528.png/)

+ **经内存交换 。**最简单、 最早的路由器是传统的计算机 ， 在输入端口与输出端口之间的交换是在 CPU （路由选择处理器 ）的直接控制下完成的 。 输入与输出端口的功能就像在传统操作系统中的I/O设备一样 。一个分组到达一个输入端口时 ，该端口会先通过中断方式向路由选择处理器发出信号。于是 ，该分组从输入端口处被复制到处理器内存中。 路由选择处理器则从其首部中提取目的地址 ， 在转发表中找出适当的输出端口 ， 并将该分组复制到输出端口的缓存中。 在这种情况下，如果内存带宽为每秒可写进内存或从内存读出最多 B 个分组， 则总的转发吞吐量（ 分组从输入端口被传送到输出端口的总速率） 必然小于R/2 。也要注意到不能同时转发两个分组， 即使它们有不同的目的端口 ， 因为经过共享系统总线一次仅能执行一个内存读 /写。

  许多现代路由器通过内存进行交换 。 然而， 与早期路由器的一个主要差别是，目的地址的查找和将分组存储 （ 交换 ）进适当的内存存储位置是由输入线路卡来处理的 。 在某些方面，经内存交换的路由器看起来很像共享内存的多处理器 ，用一个线路卡上的处理将分组交换 （ 写 ）进适当的输出端口的内存中。

+ **经总线交换 。**在这种方法中 ，输入端口经一根共享总线将分组直接传送到输出端口 ， 不需要路由选择处理器的干预 。’ 通常按以下方式完成该任务 ：让输入端口为分组预先计划一个交换机内部标签（首部）， 指示本地输出端口 ， 使分组在总线上传送和传输到输出端口。 该分组能由所有输出端口收到 ， 但只有与该标签匹配的端口才能保存该分组 。 然后标签在输出端口被去除， 因为其仅用于交换机内部来跨越总线 。 如果多个分组同时到达路由器 ， 每个位于不同的输出端口 ，除了一个分组外所有其他分组必须等待 ， 因为一次只有一个分组能够跨越总线 。 因为每个分组必须跨过单一总线， 故路由器的交换带宽受总线速率的限制 ； 在环状交叉路的类比中 ，这相当于环状交叉路一次仅包含一辆汽车 。 尽管如此 ， 对于运行在小型局域网和企业网中的路由器来说，通过总线交换通常足够用了。

+ **经互联网络交换 。**克服单一、共享式总线带宽限制的一种方法是 ， 使用一个更复杂的互联网络， 例如过去在多处理器计算机体系结构中用来互联多个处理器的网络 。 纵横式交换机就是一种由 2/V 条总线组成的互联网络， 它连接 /V 个输入端口与 N 个输岀端口 ， 如图 4-6 所示 。 每条垂直的总线在交叉点与每条水平的总线交叉，交叉点通过交换结构控制器 （ 其逻辑是交换结构自身的一部分 ）能够在任何时候开启和闭合。当某分组到达端口 A, 需要转发到端口 Y 时 ， 交换机控制器闭合总线 A 和 Y 交叉部位的交叉点 ， 然后端口 A 在其总线上发送该分组，该分组仅由总线 Y 接收 。 注意到来自端口 B 的一个分组在同一时间能够转发到端口 X, 因为 A 到 Y 和 B 到 X 的分组使用不同的输入和输岀总线 。 因此 ， 与前面两种交换方法不同 ，纵横式网络能够并行转发多个分组 。 纵横式交换机是**非阻塞的 （non­-blocking）** ， 即只要没有其他分组当前被转发到该输出端口 ，转发到输出端口的分组将不会被到达输出端口的分组阻塞 。 然而， 如果来自两个不同输入端口的两个分组其目的地为根同的输出端口 ， 则一个分组必须在输入端等待 ， 因为在某个时刻经给定总线仅能够发送一个分组 。

> 更为复杂的互联网络使用多级交换元素， 以使来自不同输入端口的分组通过交换结构同时朝着相同的输出端口前行 。 Cisco CRS 利用了一种三级非阻塞交换策略 。 路由器的交换能力也能够通过并行运行多种交换结构进行扩展。 在这种方法中 ，输入端口和输出端口被连接到并行运行的 N 个交换结构 。一个输入端口将一个分组分成 K 个较小的块 ，并且通过 N 个交换结构中的 K 个发送（ “喷射 ” ）这些块到所选择的输出端口 ，输岀端口再将 K 个块装配还原成初始的分组 。

### 4.2.3 输出端口处理

如图 4-7 中所示，输出端口处理取出已经存放在输出端口内存中的分组并将其发送到输出链路上。 这包括选择和取岀排队的分组进行传输， 执行所需的链路层和物理层传输功能 。

![image-20220508115006572](https://gitee.com/cafory/images-store/raw/master/Image/202205081150627.png/)

### 4.2.4 何处出现排队

如果我们考虑显示在图 4-6 中的输入和输出端口功能及其配置， 下列情况是一目了然的： 在输入端口和输出端口处都可以形成分组队列 ， 就像在环状交叉路的类比中我们讨论过的情况 ， 即汽车可能等待在流量交叉点的入口和出口。 排队的位置和程度 （ 或者在输入端口排队， 或者在输岀端口排队） 将取决于流量负载 、交换结构的相对速率和线路速率 。 我们现在更为详细一点考虑这些队列,因为随着这些队列的增长，路由器的缓存空间最终将会耗尽 ， 并且当无内存可用于存储到达的分组时将会出现**丢包 （ packet loss ）** 。 回想前面的讨论， 我们说过分组 “ 在网络中丢失 ” 或 “ 被路由器丢弃 ”。 **正是在一台路由器的这些队列中 ，这些分组被实际丢弃或丢失 。**

假定输入线路速度与输出线路速度 （ 传输速率） 是相同的， 均为$R_{line}$（ 单位为每秒分组数 ）， 并且有 N 个输入端口和 N 个输出端口。为进一步简化讨论， 假设所有分组具有相同的固定长度 ， 分组以同步的方式到达输入端口。 这就是说， 在任何链路发送分组的时间等于在任何链路接收分组的时间， 在这样的时间间隔内 ， 在一个输入链路上能够到达 0 个或 1 个分组 。 定义交换结构传送速率$R_{switch}$ 为从输入端口到输出端口能够移动分组的速率。 如果$R_{swicth}$ 比$R_{line}$快 N 倍 ， 则在输入端口处仅会出现微不足道的排队 。 这是因为即使在最坏情况下 ， 所有 N 条输入线路都在接收分组， 并且所有的分组将被转发到相同的输出端口 ， 每批 N 个分组（ 每个输入端口一个分组） 也能够在下一批到达前通过交换结构处理完毕 。

+ **输入排队**

如果交换结构不能快得 （相对于输入线路速度而言） 使所有到达分组无时延地通过它传送， 会发生什么情况呢 ？ 在这种情况下， 在输入端口也将岀现分组排队， 因为到达的分组必须加入输入端口队列中 ， 以等待通过交换结构传送到输出端口。为了举例说明这种排队的重要后果 ，考虑纵横式交换结构 ， 并假定 ： ①所有链路速度相同 ； ②一个分组能以一条输入链路接收一个分组所用的相同的时间量， 从任意一个输入端口传送到给定的输出端口 ； ③分组按 FCFS 方式 ， 从一指定输入队列移动到其要求的输出队列中。只要其输出端口不同 ， 多个分组可以被并行传送 。 然而， 如果位于两个输入队列前端的两个分组是发往同一输出队列的， 则其中的一个分组将被阻塞 ， 且必须在输入队列中等待 ， 因为交换结构一次只能传送一个分组到某指定端口。

![image-20220508133448978](https://gitee.com/cafory/images-store/raw/master/Image/202205081334055.png/)

图 4-8 显示了一个例子 ， 其中在输入队列前端的两个分组（ 带深色阴影 ）要发往同一个右上角输出端口。假定该交换结构决定发送左上角队列前端的分组 。 在这种情况下 ， 左下角队列中的深色阴影分组必须等待 。但不仅该分组要等待 ， 左下角队列中排在该分组后面的浅色阴影分组也要等待 ， 即使右中侧输出端口 （ 浅色阴影分组的目的地 ） 中无竞争。 这种现象叫作输入排队交换机中的**线路前部（Head-Of-the-Line, HOL） 阻塞** ， 即在一个输入队列中排队的分组必须等待通过交换结构发送（ 即使输出端口是空闲的）， 因为它被位于线路前部的另一个分组所阻塞 。 ［Karol 1987 ］ 指出 ，由于HOL 阻塞 ， 只要输入链路上的分组到达速率达到其容量的 58%，在某些假设前提下 ，输入队列长度就将无限制地增大（ 不严格地讲，这等同于说将出现大量的丢包 ） 。 

+ **输出排队**

我们接下来考虑在交换机的输出端口是否会出现排队 。再次假定 $R_{switch}$比$R_{line}$快 N 倍，并且到达 N 个输入端口的每个端口的分组， 其目的地是相同的输出端口。 在这种情况下，在向输出链路发送一个分组的时间内 ， 将有 N 个新分组到达该输出端口 （ 厲个输入端口的每个都到达 1 个 ） 。 因为输出端口在一个单位时间（该分组的传输时间） 内仅能传输一个分组，这 N 个到达分组必须排队（等待 ）经输岀链路传输 。 在正好传输 N 个分组（这些分组是前面正在排队的） 之一的时间中 ， 可能又到达 N 个分组，等等 。 所以 ， 分组队列能够在输岀端口形成 ， 即使交换结构比端口线路速率快 N倍。 最终， 排队的分组数量能够变得足够大 ，耗尽输出端口的可用内存 。

当没有足够的内存来缓存一个入分组时 ， 就必须做出决定 ：要么丢弃到达的分组（采用一种称为**弃尾 （ drop-tail）** 的策略），要么删除一个或多个已排队的分组为新来的分组腾
出空间 。 在某些情况下 ， 在缓存填满之前便丢弃一个分组（ 或在其首部加上标记）的做法是有利的，这可以向发送方提供一个拥塞信号。 已经提出和分析了许多分组丢弃与标记策略 。这些策略统称为**主动队列管理（ Active Queue Management , AQM） 算法** 。 **随机早期检测 （ Random Early Detection, RED）** 算法是得到最广泛研究和实现的 AQM 算法之一 。

![image-20220508135554669](https://gitee.com/cafory/images-store/raw/master/Image/202205081355742.png/)

在图 4 -9 中图示了输出端口的排队情况。 在时刻每个入端输入端口都到达了一个分组， 每个分组都是发往最上侧的输岀端口。假定线路速度相同 ， 交换机以 3 倍于线路速度的速度运行， 一个时间单位 （ 即接收或发送一个分组所需的时间） 以后，所有三个初始分组都被传送到输出端口 ，并排队等待传输 。 在下一个时间单位中，这三个分组中的一个将通过输出链路发送出去。 在这个例子中 ， 又有两个新分组已到达交换机的入端；这些分组之一要发往最上侧的输岀端口。 这样的后果是 ，输出端口的**分组调度 （ packet scheduler ）** 在这些排队分组中选择一个分组来传输。

假定需要路由器缓存来吸收流量负载的波动 ， 一个自然而然的问题就是需要多少缓存 。多年以来 ，用于缓存长度的经验方法是 ［RFC 3439］， 缓存数量 （ B） 应当等于平均往返时延 （ RTT，比如说 250ms） 乘以链路的容量 （ C） 。 这个结果是基于相对少量的TCP 流的排队动态性分析得到的 。 因此 ， 一条具有 250ms RTT 的10Gbps 链路需要的缓存量等于$B = RTT\cdot C=2.5Gb$ 。 然而， 最近的理论和试验研究表明 ， 当有大量的 TCP 流 （ N 条 ） 流过一条链路时 ，缓存所需要的数量是 $B = RTT\cdot C / \sqrt N$ 对于通常有大量流经过的大型主干路由器链路， N 的值可能非常大 ， 所需的缓存长度的减小相当明显 。 

### 4.2.5 分组调度

现在我们转而讨论确定次序的问题， 即排队的分组如何经输出链路传输的问题 。以前你自己无疑在许多场合都排长队等待过， 并观察过等待的客户怎样被服务 ， 你无疑也熟悉路由器中常用的许多排队规则。 有一种是先来先服务 （ FCFS, 也称之为先进先出（FIFO）） 。 这是英国人人共知的规则 ，用于病人就诊 、公交车站和市场中的有序 FCFS 队列。 （ 哦 ， 你排队了吗 ？） 有些国家基于优先权运转， 即给一类等待客户超越其他等待客户的优先权服务。也有循环排队， 其中客户也被划分为类别 （ 与在优先权队列一样 ）， 但每类用户依次序提供服务。

+ **先进先出**

图 4-10 显示了对于先进先出 （ First-In-First-Out, FIFO） 链路调度规则的排队模型的抽象 。 如果链路当前正忙于传输另一个分组， 到达链路输出队列的分组要排队等待传输 。 如果没有足够的缓存空间来容纳到达的分组，队列的分组丢弃策略则确定该分组是否将被丢弃 （ 丢失 ） 或者从队列中去除其他分组以便为到达的分组腾出空间， 如前所述 。 在下面的讨论中 ， 我们将忽视分组丢弃 。 当一个分组通过输出链路完全传输（ 也就是接收服务 ） 时 ， 从队列中去除它 。

![image-20220508140341299](https://gitee.com/cafory/images-store/raw/master/Image/202205081403353.png/)

FIFO （ 也称为先来先服务 ， FCFS） 调度规则按照分组到达输出链路队列的相同次序来选择分组在链路上传输 。 我们都很熟悉服务中心的 FIFO 排队， 在那里到达的顾客加入单一等待队列的最后 ， 保持次序 ， 然后当他们到达队伍的前面时就接受服务。

图 4-11 显示了运行中的 FIFO 队列。分组的到达由上部时间线上带编号的箭头来指示，用编号指示了分组到达的次序 。各个分组的离开表示在下部时间线的下面 。分组在服务中 （被传输）花费的时间是通过这两个时间线之间的阴影矩形来指示的 。假定在这个例子中传输每个分组用去 3 个单位时间 。利用 FIFO 规则 ， 分组按照到达的相同次序离开 。注意在分组 4 离开之后 ， 在分组 5 到达之前链路保持空闲（ 因为分组 1~4 已经被传输并从队列中去除） 。

![image-20220508140435684](https://gitee.com/cafory/images-store/raw/master/Image/202205081404757.png/)

+ **优先权排队**

在**优先权排队（ priority queuing）** 规则下 ， 到达输出链路的分组被分类放入输出队列中的优先权类， 如图 4-12 所示 。 在实践中 ，网络操作员可以配置一个队列 ，这样携带网络管理信息的分组（ 例如 ，由源或目的 TCP/UDP 端口号所标识）获得超过用户流量的优先权 ； 此外 ， 基于 IP 的实时话音分组可能获得超过非实时流量（ 如 SMTP 或 IMAP 电子邮件分组）的优先权 。 每个优先权类通常都有自己的队列。 当选择一个分组传输时 ， 优先权排队规则将从队列为非空（ 也就是有分组等待传输）的最高优先权类中传输一个分组 。 在同一优先权类的分组之间的选择通常以FIFO 方式完成 。

![image-20220508140641437](https://gitee.com/cafory/images-store/raw/master/Image/202205081406499.png/)

图 4-13 描述了有两个优先权类的一个优先权队列的操作。分组 1 、 3 和 4 属于高优先权类， 分组 2 和 5 属于低优先权类 。分组 1 到达并发现链路是空闲的， 就开始传输 。 在分组 1 的传输过程中 ， 分组 2 和 3 到达， 并分别在低优先权和高优先权队列中排队 。 在传输完分组 1 后 ， 分组 3 （ 一个高优先权的分组）被选择在分组 2 （ 尽管它到达得较早 ， 但它是一个低优先权分组） 之前传输 。 在分组 3 的传输结束后 ， 分组 2开始传输 。分组 4 （ 一个高优先权分组） 在分组 2 （ 一个低优先权分组）的传输过程中到达 。 在**非抢占式优先权排队（ non-preemptive priority queuing）** 规则下 ， 一旦分组开始传输， 就不能打断 。 在这种情况下 ， 分组 4 排队等待传输， 并在分组 2 传输完成之后开始传输 。

![image-20220508140920227](https://gitee.com/cafory/images-store/raw/master/Image/202205081409295.png/)

+ **循环和加权公平排队**

在**循环排队规则 ( round robin queuing discipline)** 下 ， 分组像使用优先权排队那样被分类 。 然而， 在类之间不存在严格的服务优先权 ， 循环调度器在这些类之间轮流提供服务。 在最简单形式的循环调度中 ，类 1 的分组被传输， 接着是类 2 的分组， 接着又是类 1的分组， 再接着又是类 2 的分组，等等 。一个所谓的**保持工作排队 ( work conserving queuing)** 规则在有(任何类的)分组排队等待传输时 ， 不允许链路保持空闲 。 当寻找给定类的分组但是没有找到时 ， 保持工作的循环规则将立即检查循环序列中的下一个类。

图 4-4 描述了一个两类循环队列的操作。 在这个例子中 ， 分组 1 、 2 和 4 属于第一类，分组 3 和 5 属于第二类 。分组 1 一到达输出队列就立即开始传输 。分组 2 和 3 在分组 1的传输过程中到达， 因此排队等待传输 。 在分组 1 传输后 ，链路调度器查找类 2 的分组，因此传输分组 3 。在分组 3 传输完成后 ，调度器查找类 1 的分组， 因此传输分组 2 。 在分组 2传输完成后 ， 分组 4 是唯一排队的分组， 因此在分组 2 后立刻传输分组 4 。

![image-20220508141149502](https://gitee.com/cafory/images-store/raw/master/Image/202205081411558.png/)

一种通用形式的循环排队已经广泛地实现在路由器中 ， 它就是所谓的**加权公平排队( Weighted Fair Queuing, WFQ )** 规则。图 4-15 对 WFQ 进行了描述 。其中 ， 到达的分组被分类并在合适的每个类的等待区域排队 。与使用循环调度一样 ， WFQ 调度器也以循环的方式为各个类提供服务 ， 即首先服务第 1 类， 然后服务第 2 类， 接着再服务第 3 类， 然后（ 假设有 3 个类别 ）重复这种服务模式 。 WFQ 也是一种保持工作排队规则 ，因此在发现一个空的类队列时 ， 它立即移向服务序列中的下一个类 。

![image-20220508141301655](https://gitee.com/cafory/images-store/raw/master/Image/202205081413710.png/)

WFQ 和循环排队的不同之处在于 ， 每个类在任何时间间隔内可能收到不同数量的服务。具体而言， 每个类 $i$被分配一个权叫$w_i$。使用 WFQ 方式 ， 在类 $i$有分组要发送的任何时间间隔中 ，第 $i$类将确保接收到的服务部分等于$w_{i}/(\sum w_{j})$， 式中分母中的和是计算所有有分组排队等待传输的类别得到的 。 在最坏的情况下 ， 即使所有的类都有分组排队，第  $i$类仍然保证分配到带宽的 $w_{i}/(\sum w_{j})$ 部分。 因此 ， 对于一条传输速率为 R 的链路，第，类总能获得至少为$w_{i}/(\sum w_{j})$ 的吞吐量 。 我们对 WFQ 的描述理想化了 ， 因为没有考虑这样的事实 ： 分组是离散的数据单元 ， 并且不能打断一个分组的传输来开始传输另一个分组。



## 4.3 网际协议： IPv4 、 寻址 、 IPv6 及其他

到目前为止 ， 我们在第 4 章中对网络层的学习 ， 包括网络层的数据平面和控制平面组件概念 ，转发和路由选择之间的区别 ， 各种网络服务模型的标识和对路由器内部的观察，并未提及任何特定的计算机网络体系结构或协议 。 在这节中 ， 我们将关注点转向今天的因特网网络层的关键方面和著名的网际协议 （ IP） 。

今天有两个版本的 IP 正在使用 。 在 4.3.1 节中 ， 我们首先研究广泛部署的 IP 版本 4，这通常简单地称为 IPv4 。 在 4. 3.5 节中 ， 我们将仔细考察 IP 版本 6 ，它已经被提议替代 IPv4 。 在中间， 我们将主要学习因特网编址 ，这是一个看起来相当枯燥和面向细节的主题， 但是这对理解因特网网络层如何工作是至关重要的 。 掌握 IP 编址就是掌握因特网的网络层 ！

### 4.3.1 IPv4 数据报格式

前面讲过网络层分组被称为数据报 。我们以概述 IPv4 数据报的语法和语义开始对 IP 的学习。你也许认为没有什么比一个分组的比特的语法和语义更加枯燥无味的了。 无论如何 ， 数据报在因特网中起 着重要作用， 每个网络行业的学生和专业 人员都需要理解它 、吸收它并掌握它 。IPv4数据报格式如图 4-16 所示 。

![image-20220508141821973](https://gitee.com/cafory/images-store/raw/master/Image/202205081418041.png)

IPv4 数据报中的关键字段如下 ：

+ **版本 （ 号 ） 。** 这 4 比特规定了数据报的 IP 协议版本 。 通过查看版本号 ，路由器能够确定如何解释 IP 数据报的剩余部分。
+ **首部长度 。** 因为一个 IPv4 数据报可包含一些可变数量的选项（这些选项包括在IPv4 数据报首部中 ）， 故需要用这 4 比特来确定 IP 数据报中载荷（ 例如在这个数据报中被封装的运输层报文段 ） 实际开始的地方 。 大多数 IP 数据报不包含选项，所以一般的 IP 数据报具有 20 字节的首部 。
+ **服务类型 。** 服务类型 （ TOS） 比特包含在 IPv4 首部中 ， 以便使不同类型的 IP 数据报 （ 例如 ， 一些特别要求低时延 、 高吞吐量或可靠性的数据报 ）能相互区别开来 。
+ **数据报长度 。** 这是 IP 数据报的总长度 （首部加上数据 ）， 以字节计 。 因为该字段长为 16 比特， 所以 IP 数据报的理论最大长度为 65 535 字节 。 然而， 数据报很少有超过 1500 字节的，该长度使得 IP 数据报能容纳最大长度以太网帧的载荷字段 。
+ **标识 、 标志 、 片偏移 。** 这三个字段与所谓 IP 分片有关 ，这是一个我们将很快要考虑的主题 。 有趣的是 ， 新版本的 IP （ 即 IPv6） 不允许在路由器上对分组分片 。
+ **寿命。** 寿命 （ Time-To-Live， TTL） 字段用来确保数据报不会永远（ 如由于长时间的路由选择环路） 在网络中循环 。 每当一台路由器处理数据报时 ，该字段的值减1。若 TTL 字段减为 0， 则该数据报必须丢弃 。
+ **协议 。** 该字段通常仅当一个 IP 数据报到达其最终目的地时才会有用 。 该字段值指示了 IP 数据报的数据部分应交给哪个特定的运输层协议 。例如 ， 值为 6 表明数据部分要交给 TCP, 而值为 17 表明数据要交给 UDP 。注意在 IP 数据报中的协议号所起的作用，类似于运输层报文段中端口号字段所起的作用 。协议号是将网络层与运输层绑定到一起的黏合剂 ，而端口号是将运输层和应用层绑定到一起的黏合剂。
+ **首部检验和。**首部检验和用于帮助路由器检测收到的 IP 数据报中的比特错误 。 首部检验和是这样计算的： 将首部中的每 2 个字节当作一个数 ，用反码算术对这些数求和。如在 3. 3 节讨论的那样 ，该和的反码（被称为因特网检验和 ） 存放在检验和字段中。 路由器要对每个收到的 IP 数据报计算其首部检验和 ， 如果数据报首部中携带的检验和与计算得到的检验和不一致， 则检测岀是个差错 。 路由器一般会丢弃检测出错误的数据报 。 注意到在每台路由器上必须重新计算检验和并再次存放到原处 ， 因为 TTL 字段以及可能的选项字段会改变。此时 ， 一个经常问的问题是 ： 为什么TCP/IP 在运输层与网络层都执行差错检测 ？这种重复检测有几种原因 。 首先 ， 注意到在 IP 层只对 IP 首部计算了检验和 ，而 TCP/UDP 检验和是对整个 TCP/UDP 报文段进行的 。其次 ，TCP/UDP 与 IP 不一定都必须属于同一个协议栈 。
+ **源和目的 IP 地址 。**当某源生成一个数据报时 ， 它在源 IP 字段中插入它的 IP 地址 ， 在目的 IP 地址字段中插入其最终目的地的地址 。 通常源主机通过 DNS 查找来决定目的地址 ， 如在第 2 章中讨论的那样 。
+ **选项 。** 选项字段允许 IP 首部被扩展 。 首部选项意味着很少使用， 因此决定对每个数据报首部不包括选项字段中的信息 ，这样能够节约开销 。 然而， 少量选项的存在的确使问题复杂了 ， 因为数据报首部长度可变 ， 故不能预先确定数据字段从何处开始 。 而且还因为有些数据报要求处理选项，而有些数据报则不要求 ， 故导致一台路由器处理一个 IP 数据报所需的时间变化可能很大 。 这些考虑对于高性能路由器和主机上的 IP 处理来说特别重要 。 由于这样或那样的原因 ， 在 IPv6 首部中已去掉了 IP 选项， 如 4.3.5 节中讨论的那样 。
+ **数据 （ 有效载荷） 。** 我们来看看最后也是最重要的字段.这是数据报存在的首要理由！ 在大多数情况下 ， IP 数据报中的数据字段包含要交付给目的地的运输层报文段 （ TCP 或 UDP） 。 然而，该数据字段也可承载其他类型的数据 ， 如 ICMP 报文（ 在 5. 6 节中讨论） 。

注意到一个 IP 数据报有总长为 20 字节的首部（ 假设无选项） 。 如果数据报承载一个TCP 报文段 ， 则每个 （ 无分片的） 数据报共承载了总长 40 字节的首部（ 20 字节的 IP 首部加上 20 字节的 TCP 首部） 以及应用层报文 。

###  4.3.2 IPv4 数据报分片

在第 6 章中我们将看到 ， 并不是所有链路层协议都能承载相同长度的网络层分组 。 有的协议能承载大数据报 ，而有的协议只能承载小分组 。例如 ， 以太网帧能够承载不超过1500 字节的数据 ，而某些广域网链路的帧可承载不超过 576 字节的数据 。一个链路层帧能承载的最大数据量叫作**最大传送单元 （ Maximum Transmission Unit, MTU）**。 因为每个 IP 数据报封装在链路层帧中从一台路由器传输到下一台路由器 ， 故链路层协议的 MTU 严格地限制着 IP 数据报的长度 。 对 IP 数据报长度具有严格限制并不是主要问题 。 问题在于在发送方与目的地路径上的每段链路可能使用不同的链路层协议， 且每种协议可能具有不同的 MTU 。

为了更好地理解这一转发问题， 想象你是一台互联几条链路的路由器 ， 且每条链路运行具有不同 MTU 的链路层协议 。假定你从某条链路收到一个IP数据报 ，通过检查转发表确定出链路， 并且该条出链路的 MTU 比该 IP 数据报的长度要小 。 此时你会感到慌乱 ， 如何将这个过大的 IP 分组挤进链路层帧的有效载荷字段呢 ？解决该问题的方法是将 IP 数据报中的数据分片成两个或更多个较小的 IP 数据报 ，用单独的链路层帧封装这些较小的 IP 数据报 ， 然后通过输出链路发送这些帧 。 每个这些较小的数据报都称为**片( fragment )** 。

片在其到达目的地运输层以前需要重新组装 。 TCP 与 UDP 的确都希望从网络层收到完整的 、 未分片的报文 。 IPv4 的设计者感到在路由器中重新组装数据报会给协议带来相当大的复杂性并且影响路由器的性能 。 （ 如果你是一台路由器 ， 你愿意将重新组装报文片放在你必须要做的各种各样工作的首位吗 ？） 为坚持网络内核保持简单的原则 ，IPv4 的设计者决定将数据报的重新组装工作放到端系统中 ，而不是放到网络路由器中。

当一台目的主机从相同源收到一系列数据报时 ， 它需要确定这些数据报中的某些是否是一些原来较大的数据报的片 。 如果某些数据报是这些片的话， 则它必须进一步确定何时收到了最后一片， 并且如何将这些接收到的片拼接到一起以形成初始的数据报 。为了让目的主机执行这些重新组装任务 ， IPv4 的设计者将**标识 、 标志和片偏移**字段放在 IP 数据报首部中。 当生成一个数据报时 ， 发送主机在为该数据报设置源和目的地址的同时贴上标识号。发送主机通常将它发送的每个数据报的标识号加 1 。 当某路由器需要对一个数据报分片时 ， 形成的每个数据报 （ 即片） 具有初始数据报的源地址 、 目的地址与标识号。 当目的地从同一发送主机收到一系列数据报时 ， 它能够检查数据报的标识号以确定哪些数据报实际上是同一较大数据报的片 。 由于 IP 是一种不可靠的服务 ， 一个或多个片可能永远到达不了目的地 。 因为这种原因 ， 为了让目的主机绝对地相信它已收到了初始数据报的最后一个片， 最后一个片的标志比特被设为 0, 而所有其他片的标志比特被设为 1 。另外 ， 为了让目的主机确定是否丢失了一个片（ 且能按正确的顺序重新组装片）， 使用偏移字段指定该片应放在初始 IP 数据报的哪个位置 。

![image-20220508145009218](https://gitee.com/cafory/images-store/raw/master/Image/202205081450298.png/)

图 4-17 图示了一个例子 。一个4000 字节的数据报 （ 20 字节 IP 首部加上 3980 字节 IP 有效载荷） 到达一台路由器 ， 且必须被转发到一条 MTU为 1500 字节的链路上。 这就意味着初始数据报中 3980 字节数据必须被分配为 3 个独立的片（ 其中的每个片也是一个 IP 数据报 ） 。

### 4.3.3 IPv4 编址

在讨论 IP 编址之前 ， 我们需要简述一下主机与路由器连入网络的方法 。一台主机通常只有一条链路连接到网络； 当主机中的 IP 想发送一个数据报时 ， 它就在该链路上发送 。主机与物理链路之间的边界叫作**接口 （ interface）** 。 现在考虑一台路由器及其接口。 因为路由器的任务是从链路上接收数据报并从某些其他链路转发出去 ，路由器必须拥有两条或更多条链路与它连接 。 路由器与它的任意一条链路之间的边界也叫作接口。一台路由器因此有多个接口 ， 每个接口有其链路 。 因为每台主机与路由器都能发送和接收 IP数据报 ， IP 要求每台主机和路由器接口拥有自己的 IP 地址 。 因此 ， **从技术上讲， 一个 IP地址与一个接口相关联，而不是与包括该接口的主机或路由器相关联** 。

每个 IP 地址长度为 32 比特（等价为 4 字节）， 因此总共有屮个 （ 或大约 40 亿个 ）可能的 IP 地址 。 这些地址通常按所谓**点分十进制记法 （ dotted-decimal notation）** 书写 ， 即地址中的每个字节用它的十进制形式书写 ， 各字节间以句点隔开 。例如 ，考虑 IP 地址`193.32.216.9`，193 是该地址的第一个 8 比特的十进制等价数 ， 32 是该地址的第二个 8 比特的十进制等价数 ， 依次类推 。 因此 ， 地址 `193.32.216.9` 的二进制记法是:

$$11000001 \space 00100000 \space 11011000 \space 00001001$$

在全球因特网中的每台主机和路由器上的每个接口 ，都必须有一个全球唯一的 IP 地址 （ 在 NAT 后面的接口除外 ， 在 4.3.4 节中讨论） 。 然而，这些地址不能随意地自由选择 。一个接口的 IP 地址的一部分需要由其连接的子网来决定 。

图 4-18 提供了一个 IP 编址与接口的例子 。 在该图中 ， 一台路由器 （ 具有 3 个接口 ）用于互联 7 台主机 。仔细观察分配给主机和路由器接口的 IP 地址 ， 有几点需要注意 。图 4-18 中左上侧的 3 台主机以及它们连接的路由器接口 ，都有一个形如 `223.1.1.xxx` 的 IP地址 。 这就是说， 在它们的 IP 地址中 ， 最左侧的 24 比特是相同的 。这 4 个接口也通过一个并不包含路由器的网络互联起来 。 该网络可能由一个以太网 LAN 互联， 在此情况下，这些接口将通过一台以太网交换机互联， 或者通过一个无线接入点互联 。 我们此时将这种无路由器连接这些主机的网络表示为一朵云 ， 在第 6 、 7 章中再深入这些网络的内部 。

![image-20220508145512788](https://gitee.com/cafory/images-store/raw/master/Image/202205081455872.png/)

用 IP 的术语来说， 互联这 3 个主机接口与 1 个路由器接口的网络形成一个**子网 （ subnet）** [RFC950] 。 （ 在因特网文献中 ， 子网也称为 IP 网络或直接称为网络 。 ） IP 编址为这个子网分配一个地址 `223.1.1.0/24`, 其中的/24 记法 ， 有时称为子网掩码 （ network mask ） , 指示 32 比特中的最左侧 24 比特定义了子网地址 。 因此子网 223. 1. 1. 0/24由 3 个主机接口 （ `223. 1.1. 1` 、 `223. 1.1.2`和 `223.1.1.3` ） 和 1 个路由器接口（`223.1.1.4`） 组成 。任何其他要连到`223. 1. 1. 0/24` 网络的主机都要求其地址具有 `223. 1. 1. xxx` 的形式 。 图 4-18 中显示了另外两个网络： `223. 1. 2. 0/24` 网络与`223. 1. 3. 0/24` 子网 。 图 4 ・ 19 图示了在图4-18 中存在的 3 个 IP 子网 。

![image-20220508150106754](https://gitee.com/cafory/images-store/raw/master/Image/202205081501843.png/)

一个子网的 IP 定义并不局限于连接多台主机到一个路由器接口的以太网段。为了搞清其中的道理， 可考虑图 4-20, 图中显示了 3 台通过点对点链路彼此互联的路由器。 每台路由器有 3 个接口 ， 每条点对点链路使用一个 ， 一个用于直接将路由器连接到一对主机的广播链路 。 这里出现了几个子网呢 ？ 3 个子网 `223.1.1.0/24` 、 `223. 1. 2. 0/24` 和 `223. 1.3.0/24` 类似于我们在图 4-18 中遇到的子网 。但注意到在本例中还有其他 3 个子网： 一个子网是 `223.1.9. 0/24`， 用于连接路由器 R1 与 R2 的接口 ； 另外一个子网是 `223.1. 8.0/24`， 用于连接路由器 R2 与 R3 的接口 ；第三个子网是 `223. 1.7.0/24`， 用于连接路由器 R3 与 R1的接口。 对于一个路由器和主机的通用互联系统， 我们能够使用下列有效方法定义系统中的子网:

> 为了确定子网， 分开主机和路由器的每个接口 ， 产生几个隔离的网络岛 ， 使用接口端接这些隔离的网络的端点 。 这些隔离的网络中的每一个都叫作一个**子网( subnet)** 。

![image-20220508150619215](https://gitee.com/cafory/images-store/raw/master/Image/202205081506294.png/)

如果我们将该过程用于图 4-20 中的互联系统上 ， 会得到 6 个岛或子网 。

从上述讨论显然可以看出 ， 一个具有多个以太网段和点对点链路的组织(如一个公司或学术机构)将具有多个子网， 在给定子网上的所有设备都具有相同的子网地址 。原则上 ， 不同的子网能够具有完全不同的子网地址 。 然而， 在实践中 ， 它们的子网地址经常有许多共同之处 。为了理解其中的道理， 我们来关注在全球因特网中是如何处理编址的 。

因特网的地址分配策略被称为**无类别域间路由选择 ( Classless Inlerdomain Routing，CIDR)** [RFC 4632]。CIDR 将子网寻址的概念一般化了。 当使用子网寻址时 ， 32 比特的IP 地址被划分为两部分 ， 并且也具有点分十进制数形式 $a.b.c.d/x$， 其中$x$指示了地址的第一部分中的比特数 。

形式为 $a.b.c.d/x$的地址的 $x$ 最高比特构成了 IP 地址的网络部分 ， 并且经常被称为该地址的**前缀 (prefix)** (或网络前缀)。一个组织通常被分配一块连续的地址 ， 即具有相同前缀的一段地址(参见 “ 实践原则”)。 在这种情况下 ，该组织内部的设备的 IP 地址将共享共同的前缀 。 当我们在 5. 4 节中论及因特网的 BGP 路由选择协议时 ， 将看到该组织网络外部的路由器仅考虑前面的前缀比特咒。 这就是说， 当该组织外部的一台路由器转发一个数据报 ， 且该数据报的目的地址位于该组织的内部时 ， 仅需要考虑该地址的前面尤比特 。这相当大地减少了在这些路由器中转发表的长度 ， 因为形式为 $a.b.c.d/x$ 的单一表项足以将数据报转发到该组织内的任何目的地 。

一个地址的剩余 32 -x 比特可认为是用于区分该组织内部设备的， 其中的所有设备具有相同的网络前缀 。 当该组织内部的路由器转发分组时 ， 才会考虑这些比特 。 这些较低阶
比特可能（ 或可能不 ） 具有另外的子网结构 ， 如前面所讨论的那样 。例如 ， 假设某CIDR化的地址 $a.b.c.d/21$的前 21 比特定义了该组织的网络前缀， 它对该组织中所有主机的 IP地址来说是共同的 。其余的 11 比特标识了该组织内的主机 。 该组织的内部结构可以采用这样的方式 ， 使用最右边的 11 比特在该组织中划分子网， 就像前面所讨论的那样 。例如，$a.b.c.d/24$可能表示该组织内的特定子网 。

在 CIDR 被采用之前 ， IP 地址的网络部分被限制为长度为 8 、 16 或 24 比特，这是一种称为**分类编址 （classful addressing）** 的编址方案 ，这是因为具有 8 、 16 和 24 比特子网地址的子网分别被称为 A 、 B 和 C 类网络 。一个 IP 地址的网络部分正好为 1 、 2 或 3 字节的要求 ， 已经在支持数量迅速增加的具有小规模或中等规模子网的组织方面出现了问题 。一个C 类 （ /24 ） 子网仅能容纳多达 $2^8 -2 = 254$（$2^8=256$， 其中的两个地址预留用于特殊用途） 台主机 ，这对于许多组织来说太小了。 然而一个 B 类（ /16 ） 子网可支持多达 65 534台主机 ， 又太大了。 在分类编址方法下 ， 比方说一个有 2000 台主机的组织通常被分给一个 B 类（ /16 ） 地址 。 这就导致了 B 类地址空间的迅速损耗以及所分配的地址空间的利用率低下。例如 ， 为具有 2000 台主机的组织分配一个 B 类地址 ， 就具有足以支持多达65 534 个接口的地址空间， 剩下的超过 63 000 个地址却不能被其他组织使用 。

如果还不提及另一种类型的 IP 地址 ， 即 IP 广播地址 `255. 255. 255. 255`， 那将是我们的疏漏 。 当一台主机发出一个目的地址为 `255. 255. 255. 255` 的数据报时 ，该报文会交付给同一个网络中的所有主机 。 路由器也会有选择地向邻近的子网转发该报文 （虽然它们通常不这样做 ） 。

现在我们已经详细地学习了 IP 编址 ，需要知道主机或子网最初是如何得到它们的地址的 。 我们先看一个组织是如何为其设备得到一个地址块的， 然后再看一个设备 （ 如一台主机 ） 是如何从某组织的地址块中分配到一个地址的 。

+ **获取一块地址**

为了获取一块 IP 地址用于一个组织的子网内 ， 某网络管理员也许首先会与他的 ISP 联系，该 ISP 可能会从已分给它的更大地址块中提供一些地址 。例如 ，该 ISP 也许自己已被分配了地址块 `200. 23. 16. 0/20` 。 该 ISP 可以依次将该地址块分成 8 个长度相等的连续地址块 ， 为本 ISP 支持的最多达 8 个组织中的一个分配这些地址块中的一块 ， 如下所示 。 （ 为了便于查看， 我们已将这些地址的网络部分加了下划线 。 ）

![image-20220508152152790](https://gitee.com/cafory/images-store/raw/master/Image/202205081521859.png/)

尽管从一个 ISP 获取一组地址是种得到一块地址的方法 ， 但这不是唯一的方法 。 显然 ， 必须还有一种方法供 ISP 本身得到一块地址"是否有一个全球性的权威机构 ， 它具有管理 IP 地址空间并向各 ISP 和其他组织分配地址块的最终责任呢 ？的确有一个 ！ IP 地址由**因特网名字和编号分配机构 （ Internet Corporation for Assigned Names and Numbers，ICANN）** 管理，管理规则基于 ［RFC 7020］ 。 非营利的 ICANN 组织 的作用不仅是分配 IP 地址 ，还管理 DNS 根服务器。 它还有一项容易引起争论的工作 ， 即分配域名与解决域名纷争。 ICANN 向区域性因特网注册机构 （ 如 ARIN 、 RIPE 、APNIC 和 LACNIC） 分配地址 ，这些机构一起形成了 ICANN 的地址支持组织 ， 处理本区域内的地址分配 / 管理 。

+ **获取主机地址 ： 动态主机配置协议**

某组织一旦获得了一块地址 ， 它就可为本组织内的主机与路由器接口逐个分配 IP 地址 。 系统管理员通常手工配置路由器中的IP 地址 （ 常常在远程通过网络管理工具进行配置） 。主机地址也能手动配置， 但是这项任务目前更多的是使用动态主机配置协议 **（ Dynamic Host Configuration, DHCP）**来完成 。 DHCP 允许主机自动获取 （被分配） 一个 IP 地址 。 网络管理员能够配置 DHCP，以使某给定主机每次与网络连接时能得到一个相同的 IP 地址 ， 或者某主机将被分配一个**临时的 IP 地址 （ tempomry IP address ）** , 每次与网络连接时该地址也许是不同的 。 除了主机 IP 地址分配外 ， DHCP 还允许一台主机得知其他信息 ， 例如它的子网掩码 、 它的第一跳路由器地址 （ 常称为默认网关 ） 与它的本地DNS 服务器的地址 。

由于 DHCP 具有将主机连接进一个网络的网络相关方面的自动能力 ， 故它又常被称为**即插即用协议（ plug-and-play protocol）** 或**零配置 （ zeroconf） 协议** 。 这种能力对于网络管理员来说非常有吸引力 ， 否则他将不得不手工执行这些任务 ！

DHCP 是一个客户-服务器协议 。 客户通常是新到达的主机 ， 它要获得包括自身使用的 IP 地址在内的网络配置信息 。 在最简单场合下 ， 每个子网（ 在图 4-20 的编址意义下 ）将具有一台 DHCP 服务器。 如果在某子网中没有服务器 ， 则需要一个 DHCP 中继代理（通
常是一台路由器 ），这个代理知道用于该网络的 DHCP 服务器的地址 。 图 4-23 显示了连接到子网 `223. 1.2/2 4` 的一台 DHCP 服务器 ， 具有一台提供中继代理服务的路由器 ， 它为连接到子网 `223. 1. 1/24` 和 `223. 1 . 3/24` 的到达客户提供 DHCP 服务。 在我们下面的讨论中,将假定 DHCP 服务器在该子网上是可供使用的 。

![image-20220508152710585](https://gitee.com/cafory/images-store/raw/master/Image/202205081527683.png/)

对于一台新到达的主机而言，针对图 4-23 所示的网络设置， DHCP 协议是一个 4 个步骤的过程， 如图 4-24 中所示 。 在这幅图中 ， yiaddr （表示 “你的因特网地址 ”之意 ） 指示分配给该新到达客户的地址 。

这 4 个步骤是:

+ **DHCP 服务器发现 。**一台新到达的主机的首要任务是发现一个要与其交互的 DHCP服务器。 这可通过使用 **DHCP 发现报文 （ DHCP discover message）** 来完成 ， 客户在UDP 分组中向端口 67 发送该发现报文 。 该 UDP 分组封装在一个 IP 数据报中。但是这个数据报应发给谁呢 ？ 主机甚至不知道它所连接网络的 IP 地址 ， 更不用说用于该网络的 DHCP 服务器地址了。 在这种情况下 ， DHCP 客户生成包含 DHCP 发现报文的IP 数据报 ， 其中使用广播目的地址 `255.255.255.255` 并且使用 “ 本主机 ” 源 IP 地址`0.0. 0.0` 。 DHCP 客户将该 IP 数据报传递给链路层 ，链路层然后将该帧广播到所有与该子网连接的节点。
+ **DHCP 服务器提供 。** DHCP 服务器收到一个 DHCP 发现报文时 ，用 **DHCP 提供报文 ( DHCP offer message )** 向客户做出响应 ，该报文向该子网的所有节点广播 ，然使用 IP 广播地址 `255. 255. 255. 255` (你也许要思考一下这个服务器为何也必须采用广播)。 因为在子网中可能存在几个 DHCP 服务器 ，该客户也许会发现它处于能在几个提供者之间进行选择的优越位置 。 每台服务器提供的报文包含有收到的发现报文的事务 ID 、 向客户推荐的 IP 地址 、 网络掩码以及 IP **地址租用期 ( address lease time )** , 即 IP 地址有效的时间量 。 服务器租用期通常设置为几小时或几天 。
+ **DHCP 请求 。** 新到达的客户从一个或多个服务器提供中选择一个 ， 并向选中的服务器提供用 **DHCP 请求报文 ( DHCP request message ) 进行响应** ， 回显配置的参数 。
+ **DHCP ACK。**服务器用 **DHCP ACK 报文 (DHCP ACK message)** 寸对DHCP 请求报文
  进行响应 ，证实所要求的参数 。

一旦客户收到 DHCP ACK 后 ， 交互便完成了 ， 并且该客户能够在租用期内使用 DHCP分配的 IP 地址 。 因为客户可能在该租用期超时后还希望使用这个地址 ， 所以 DHCP 还提供了一种机制以允许客户更新它对一个 IP 地址的租用 。

从移动性角度看， DHCP 确实有非常严重的缺陷 。 因为每当节点连到一个新子网，要从 DHCP 得到一个新的 IP 地址 ， 当一个移动节点在子网之间移动时 ， 就不能维持与远程应用之间的 TCP 连接 。 在第 6 章中 ， 我们将研究移动 IP, 它是一种对 IP 基础设施的扩展，允许移动节点在网络之间移动时使用其单一永久的地址。

### 4.3.4 网络地址转换

讨论了有关因特网地址和 IPv4 数据报格式后 ， 我们现在可清楚地认识到每个 IP 使能的设备都需要一个 IP 地址 。 随着所谓小型办公室 、 家庭办公室 ( Small Office , Home Office, SOHO) 子网的大量出现，看起来意味着每当一个 SOHO 想安装一个 LAN 以互联多台机器时 ，需要 ISP 分配一组地址以供该 SOHO 的所有 IP 设备(包括电话 、 平板电脑 、游戏设备 、 IPTV 、 打印机等)使用 。 如果该子网变大了 ， 则需要分配一块较大的地址 。但如果 ISP 已经为 SOHO 网络的当前地址范围分配过一块连续地址该怎么办呢 ？ 并且 ， 家庭主人一般要(或应该需要)首先知道的管理 IP 地址的典型方法有哪些呢 ？ 幸运的是 ，有一种简单的方法越来越广泛地用在这些场合 ：**网络地址转换 ( Network Address Translation , NAT)** 。

图 4-25 显示了一台 NAT 使能路由器的运行情况。位于家中的 NAT 使能的路由器有一个接口 ，该接口是图 4 ・ 25 中右侧所示家庭网络的一部分。 在家庭网络内的编址就像我们在上面看到的完全一样 ， 其中的所有 4 个接口都具有相同的网络地址 `10. 0. 0/24` 。 地址空间 `10. 0. 0. 0/8` 是在 ［RFC 1918］ 中保留的三部分 IP 地址空间之一 ，这些地址用于如图 4-25 中的家庭网络等**专用网络 ( private network )** 或**具有专用地址的地域 ( realm with private address)** 。 具有专用地址的地域是指其地址仅对该网络中的设备有意义的网络 。为了明白它为什么重要，考虑有数十万家庭网络这样的事实 ，许多使用了相同的地址空间`10. 0. 0. 0/24`。 在一个给定家庭网络中的设备能够使用 `10. 0. 0. 0/24` 编址彼此发送分组 。然而，转发到家庭网络之外进入更大的全球因特网的分组显然不能使用这些地址(或作为源地址 ， 或作为目的地址) ， 因为有数十万的网络使用着这块地址 。 这就是说， `10.0.0.0/24` 地址仅在给定的网络中才有意义。但是如果专用地址仅在给定的网络中才有意义的话，当向或从全球因特网发送或接收分组时如何处理编址问题呢 ， 地址在何处才必须是唯一的呢 ？答案在于理解 NAT 。

![image-20220509191659664](https://gitee.com/cafory/images-store/raw/master/Image/202205091916766.png/)

NAT 使能路由器对于外部世界来说甚至不像一台路由器。 相反 NAT 路由器对外界的行为就如同一个**具有单一 IP 地址的单一设备** 。 在图 4-25 中 ， 所有离开家庭路由器流向更大因特网的报文都拥有一个源 IP 地址 `138.76.29.7`， 且所有进入家庭的报文都拥有同一个目的 IP 地址 `138.76. 29. 7` 。从本质上讲， NAT 使能路由器对外界隐藏了家庭网络的细节 。(另外 ， 你也许想知道家庭网络计算机是从哪儿得到其地址 ，路由器又是从哪儿得到它的单一 IP 地址的 。 在通常的情况下 ，答案是相同的， 即 DHCP！路由器从 ISP 的 DHCP 服务器得到它的地址 ， 并且路由器运行一个 DHCP 服务器 ， 为位于 NAT-DHCP 路由器控制的家庭网络地址空间中的计算机提供地址 。 )

如果从广域网到达 NAT 路由器的所有数据报都有相同的目的 IP 地址 （特别是对 NAT路由器广域网一侧的接口 ），那么该路由器怎样知道它应将某个分组转发给哪个内部主机呢 ？ 技巧就是使用 NAT 路由器上的一张 **NAT 转换表（ NAT translation table ）** ， 并且在表项中包含了端口号及其 IP 地址 。

考虑图 4-25 中的例子 。假设一个用户坐在家庭网络主机 `10. 0. 0. 1` 后 ，请求 IP 地址为`128.119. 40. 186` 的某台 Web 服务器 （端口 80 ） 上的一个 Web 页面 。主机 `10. 0. 0. 1` 为其指派了 （ 任意 ） 源端口号 3345 并将该数据报发送到 LAN 中。 NAT 路由器收到该数据报，为该数据报生成一个新的源端口号 5001，将源 IP 替代为其广域网一侧接口的 IP 地址`138. 76. 29.7`， 且将源端口 3345 更换为新端口 5001 。 当生成一个新的源端口号时 ， NAT 路由器可选择任意一个当前未在 NAT 转换表中的源端口号。 （ 注意到因为端口号字段为 16比特长， NAT 协议可支持超过 60 000 个并行使用路由器广域网一侧单个 IP 地址的连接 ！）路由器中的 NAT 也在它的 NAT 转换表中增加一表项 。 Web 服务器并不知道刚到达的包含HTTP 请求的数据报已被 NAT 路由器进行了改装， 它会发回一个响应报文 ， 其目的地址是NAT 路由器的 IP 地址 ， 其目的端口是 5001 。 当该报文到达 NAT 路由器时 ，路由器使用目的 IP 地址与目的端口号从 NAT 转换表中检索出家庭网络浏览器使用的适当 IP 地址（10. 0. 0.1） 和目的端口号 （ 3345 ） 。于是 ，路由器重写该数据报的目的 IP 地址与目的端口号 ， 并向家庭网络转发该数据报 。

NAT 在近年来已得到了广泛的应用 。但是 NAT 并非没有贬低者 。 首先 ， 有人认为端口号是用于进程寻址的，而不是用于主机寻址的 。 这种违规用法对于运行在家庭网络中的服务器来说确实会引起问题， 因为正如我们在第 2 章所见， 服务器进程在周知端口号上等待入请求 ， 并且 P2P 协议中的对等方在充当服务器时需要接受入连接 。 对这些问题的技术解决方案包括 **NAT 穿越（ NAT traversal ）**工具和通用即插即用 （ Universal Plug and Play， UPnP）。UPnP 是一种允许主机发现和配置邻近NAT 的协议  。

其次 ，纯粹的体系结构者提出了更为“原理性的 ”反对 NAT 的意见 。 这时 ， 关注焦点在于路由器是指第三层 （ 即网络层 ）设备 ， 并且应当处理只能达到网络层的分组 。NAT违反主机应当直接彼此对话这个原则 ， 没有干涉节点修改 IP 地址 ， 更不用说端口号。但不管喜欢与否 ， NAT 已成为因特网的一个重要组件 ， 成为所谓**中间盒** [Sekar 2011]，它运行在网络层并具有与路由器十分不同的功能 。中间盒并不执行传统的数据报转发 ，而是执行诸如 NAT 、 流量流的负载均衡 、 流量防火墙 （ 参见下面的插入框内容 ）等功能 。 我们将在随后的 4. 4 节学习的通用转发范例 ，除了传统的路由器转发外 ，还允许一些这样的中间盒功能， 从而以通用 、 综合的方式完成转发。

### 4.3.5 IPv6

由于新的子网和 IP 节点以惊人的增长率连到因特网上（ 并被分配唯一的 IP 地址 ）， 32 比特的 IP 地址空间即将用尽 。为了应对这种对大 IP 地址空间的需求 ， 开发了一种新的 IP 协议， 即 IPv6 。 IPv6 的设计者还利用这次机会 ， 在 IPv4积累的运行经验基础上加进和强化了 IPv4 的其他方面 。

尽管在 20 世纪 90 年代中期对 IPv4 地址耗尽的估计表明 ， IPv4 地址空间耗尽的期限还有可观的时间， 但人们认识到 ， 如此大规模地部署一项新技术将需要可观的时间， 因此研发 IP 版本 6 （IPv6）的工作开始了 [RFC 1752] 。 （ 一个经常问的问题是: IPv5 出了什么情况？ 人们最初预想 ST- 2 协议将成为 IPv5,，但 ST-2 后来被舍弃了。 ） 有关IPv6 的优秀信息来源见 [Huitema 1998] 。

![image-20220509193949950](https://gitee.com/cafory/images-store/raw/master/Image/202205091939025.png/)

+ **IPv6 数据报格式**

  IPv6 数据报的格式如图 4-26 所示 。
  IPv6 中引入的最重要的变化显示在其数据报格式中 ：

  + **扩大的地址容量 。**IPv6 将 IP 地址长度从 32 比特增加到 128 比特 。 这就确保全世界将不会用尽 IP 地址 。 现在 ， 地球上的每个沙砾都可以用 IP 地址寻址了。 除了单播与多播地址以外 ， IPv6 还引入了一种称为**任播地址 （ anycast address）** 的新型地址 ，这种地址可以使数据报交付给一组主机中的任意一个。
  + **简化高效的 40 字节首部 。** 如下面讨论的那样 ，许多 IPv4 字段已被舍弃或作为选项 。 因而所形成的 40 字节定长首部允许路由器更快地处理 IP 数据报 。一种新的选项编码允许进行更灵活的选项处理 。
  + **流标签 。** IPv6 有一个难以捉摸的**流 （ flow）** 定义。 RFC 2460 中描述道，该字段可用于“给属于特殊流的分组加上标签，这些特殊流是发送方要求进行特殊处理的流 ， 如一种非默认服务质量或需要实时服务的流 ”。

  如上所述， 比较图 4-26 与图 4-16 就可看岀 ， IPv6 数据报的结构更简单、 更高效 。 以下是在 IPv6 中定义的字段 。
  
  + **版本 。** 该 4 比特字段用于标识 IP 版本号。 毫不奇怪 ， IPv6 将该字段值设为 6 。注意到将该字段值置为 4 并不能创建一个合法的 IPv4 数据报 。
  + **流量类型 。** 该 8 比特字段与我们在 IPv4 中看到的 TOS 字段的含义相似。
  + **流标签 。** 如上面讨论过的那样 ，该 20 比特的字段用于标识一条数据报的流 ，能够对一条流中的某些数据报给出优先权 ， 或者它能够用来对来自某些应用（ 例如 IP话音）的数据报给岀更高的优先权 ， 以优于来自其他应用（ 例如 SMTP 电子邮件 ）的数据报 。
  + **有效载荷长度 。** 该 16 比特值作为一个无符号整数 ，给出了 IPv6 数据报中跟在定长的 40 字节数据报首部后面的字节数量 。
  + **下一个首部 。** 该字段标识数据报中的内容 （ 数据字段 ）需要交付给哪个协议（ 如TCP 或 UDP） 。 该字段使用与 IPv4 首部中协议字段相同的值。
  + **跳限制。** 转发数据报的每台路由器将对该字段的内容减 1 。 如果跳限制计数达到0， 则该数据报将被丢弃 。
  + **源地址和目的地址 。** IPv6 128 比特地址的各种格式在 RFC 4291 中进行了描述 。
  + **数据 。** 这是 IPv6 数据报的有效载荷部分。 当数据报到达目的地时 ，该有效载荷就从 IP 数据报中移出 ， 并交给在下一个首部字段中指定的协议处理 。
  
+ **从 IPv4 到 IPv6 的迁移**

详情见书P229。



## 4.4 通用转发和 SDN

回顾 4.2. 1 节将基于目的地转发的特征总结为两个步骤： 查找目的 IP 地址 （ “匹配 ” ）， 然后将分组发送到有特定输出端口的交换结构 （ “动作” ） 。 我们现在考虑一种更有意义的通用 “匹配加动作” 范式 ， 其中能够对协议栈的多个首部字段进行 “匹配 ” ，这些首部字段是与不同层次的不同协议相关联的 。“动作” 能够包括 ： 将分组转发到一个或多个输出端口 （ 就像在基于目的地转发中一样 ），跨越多个通向服务的离开接口进行负载均衡分组（ 就像在负载均衡中一样 ），重写首部值 （ 就像在 NAT 中一样 ）， 有意识地阻挡 /丢弃某个分组（ 就像在防火墙中一样 ）， 为进一步处理和动作而向某个特定的服务器发送一个分组（ 就像在 DPI — 样 ），等等 。

图 4-28 显示了位于每台分组交换机中的一张匹配加动作表，该表由远程控制器计算 、安装和更新 。 我们注意到虽然在各台分组交换机中的控制组件可以相互作用（ 例如以类似于图 4-2 中的方式 ）， 但实践中通用匹配加动作能力是通过计算 、 安装和更新这些表的远
程控制器实现的 。 花几分钟比较图 4-2 、 图 4-3 和图 4-28, 你能看出图 4-2 和图 4-3 中显示的基于目的地转发与图 4-28 中显示的通用转发有什么相似和差异吗 ？

![image-20220509195955329](https://gitee.com/cafory/images-store/raw/master/Image/202205091959443.png/)

我们后续对通用转发的讨论将基于 OpenFlow ， OpenFlow 是一个得到高度认可和成功的标准 ， 它已经成为匹配加动作转发抽象 、 控制器以及更为一般的 SDN 革命等概念的先驱。 我们将主要考虑 OpenFlow 1.0, 该标准以特别清晰和简明的方式引入了关键的 SDN 抽象和功能 。

匹配加动作转发表在 OpenFlow 中称为**流表 （ flow table）** ， 它的每个表项包括:

+ **首部字段值的集合。**入分组将与之匹配 。与基于目的地转发的情况一样 ， 基于硬件匹配在 TCAM 内存中执行得最为迅速。匹配不上流表项的分组将被丢弃或发送到远程控制器做更多处理 。
+ **计数器集合 （ 当分组与流表项匹配时更新计数器 ） 。**这些计数器可以包括已经与该表项匹配的分组数量， 以及自从该表项上次更新以来的时间 。
+ 当分组匹配流表项时**所采取的动作集合**。 这些动作可能将分组转发到给定的输出端口 ， 丢弃该分组 、 复制该分组和将它们发送到多个输岀端口 ， 和/ 或重写所选的首部字段 。

### 4.4.1 匹配

图 4-29 显示了 11 个分组首部字段和入端口 ID, 该 ID 能被 OpenFlow 1.0 中的匹配加动作规则所匹配 。前面 1.5.2 节讲过， 到达一台分组交换机的一个链路层 （第二层 ） 帧将包含一个网络层 （第三层 ） 数据报作为其有效载荷，该载荷通常依次将包含一个运输层（第四层 ） 报文段 。 第一个观察是 ， OpenFlow 的匹配抽象允许对来自三个层次的协议首部
所选择的字段进行匹配（ 因此相当勇敢地违反了我们在 1.5 节中学习的分层原则 ） 。 因为
我们还没有涉及链路层 ，用如下的说法也就足够了 ： 显示在图 4-29 中的源和目的 MAC 地址是与帧的发送和接收接口相关联的链路层地址 ；通过基于以太网地址而不是 IP 地址进
行转发 ， 我们看到 OpenFlow 使能的设备能够等价于路由器 （第三层设备 ）转发数据报以及交换机 （第二层设备 ）转发帧 。以太网类型字段对应于较高层协议（ 例如 IP）, 利用该字段分解该帧的载荷， 并且 VLAN 字段与所谓虚拟局域网相关联， 我们将在第 6 章中学习VLAN 。 OpenFlow 1.0 规范中匹配的 12 个值在最近的 OpenFlow 规范中已经增加到 41 个。

![image-20220509200756373](https://gitee.com/cafory/images-store/raw/master/Image/202205092007444.png/)

入端口是指分组交换机上接收分组的输入端口。 在 4.3. 1 节中 ， 我们已经讨论过该分组的 IP 源地址 、 IP 目的地址 、 IP 协议字段和 IP 服务类型字段 。 运输层源和目的端口号字段也能匹配 。

流表项也可以有通配符 。例如 ， 在一个流表中 IP 地址 `128.119.*.*`将匹配其地址的前 16 比特为 128. 119 的任何数据报所对应的地址字段 。 每个流表项也具有相应的优先权 。
如果一个分组匹配多个流表项，选定的匹配和对应的动作将是其中有最高优先权的那个。

最后 ， 我们观察到并非一个 IP 首部中的所有字段都能被匹配 。例如 OpenFlow 并不允许基于 TTL 字段或数据报长度字段的匹配 。为什么有些字段允许匹配，而有些字段不允许呢 ？ 毫无疑问， 与功能和复杂性有关。 选择一种抽象的 “ 艺术 ” 是提供足够的功能来完成某种任务 （ 在这种情况下是实现 、 配置和管理宽泛的网络层功能， 以前这些一直是通过各种各样的网络层设备来实现的）， 不必用如此详尽和一般性的 “ 超负荷 ” 抽象，这种抽象
已经变得臃肿和不可用 。

### 4.4.2 动作

如图 4-28 中所见， 每个流表项都有零个或多个动作列表，这些动作决定了应用于与流表项匹配的分组的处理 。 如果有多个动作 ， 它们以在表中规定的次序执行 。

其中最为重要的动作可能是 ：

+ **转发。**一个入分组可以转发到一个特定的物理输岀端口 ， 广播到所有端口 （ 分组到达的端口除外 ）， 或通过所选的端口集合进行多播 。 该分组可能被封装并发送到用于该设备的远程控制器。 该控制器则可能（ 或可能不 ） 对该分组采取某些动作，包括安装新的流表项， 以及可能将该分组返回给该设备以在更新的流表规则集合下进行转发。
+ **丢弃 。** 没有动作的流表项表明某个匹配的分组应当被丢弃 。
+ **修改字段 。** 在分组被转发到所选的输出端口之前 ， 分组首部 10 个字段 （ 图 4-29 中
  显示的除IP 协议字段外的所有第二、三、 四层的字段 ） 中的值可以重写。
