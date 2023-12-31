> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.liaoxuefeng.com](https://www.liaoxuefeng.com/article/1481991528644643)

> 一文读懂 Docker 原理

说起 Docker，基本上就是指容器。许多同学熟悉 Docker 的操作，却搞不懂到底什么是容器。本文就来讲讲 Docker 容器到底是个啥。

容器被称为轻量级的虚拟化技术，实际上是不准确的。确切地说，容器是一种对进程进行隔离的运行环境。

由于生产环境的容器几乎都是运行在 Linux 上的，因此，本文提到的进程、Docker 等概念或软件均以 Linux 平台为准。

我们知道进程是 Linux 操作系统执行任务的最小单元，一个时间同步服务是一个进程，一个 Java 服务是一个进程，一个 Nginx 服务是一个主进程 + 若干工作进程，总之，把一个系统比作一个办公室，进程就是一个个打工人：

![](https://www.liaoxuefeng.com/files/attachments/1481990731726912/l)

正常情况下，一个进程是能感知到其他进程的存在的，正如一个打工人放眼望去，办公室里还坐着一群其他打工人。进程的唯一标识是进程 ID，用数字 1、2、3…… 表示，好比打工人的工牌号，大家都各不一样。

而容器技术首先要解决的就是进程的隔离，即一个进程在运行的时候看不到其他进程。如何让一个打工人在工作时看不到其他打工人呢？方法是给这个打工人带一个 VR 眼镜，于是他看到的不是一个真实的办公室，而是一个虚拟的办公室。在这个虚拟办公室中，只有他一个打工人，没有别人。在 Linux 系统中，对一个进程进行隔离，主要是通过 Namespace 和 Cgroup 两大机制实现的。一个被隔离的进程，操作系统也会正常分配进程 ID，比如 12345，但是隔离进程自己看到的 ID 总是 1，好比打工人的工牌是 12345，但他自己通过 VR 眼镜看到的工牌号却是 1，感觉自己是 1 号员工似的：

![](https://www.liaoxuefeng.com/files/attachments/1481990767378499/l)

我们通过一个简单的 Python 程序就可以验证一下隔离进程的特点。我们编写一个简单的 HTTP 服务程序，针对 URL 为`/`、`/ps`、`/ls`分别返回自身进程 ID、所有进程 ID 和磁盘根目录列表：

[app.py](https://github.com/michaelliao/learn-docker/blob/master/simple-http-server/app.py)

如果我们正常启动这个 Python 程序，在浏览器中，可以看到进程 ID 为`10297`：

![](https://www.liaoxuefeng.com/files/attachments/1481990935150659/l)

用 / ps 查看所有进程，可以看到 1 号进程是`systemd`，还有很多其他进程：

![](https://www.liaoxuefeng.com/files/attachments/1481990985482307/l)

用 / ls 查看磁盘根目录，与当前系统根目录一致：

![](https://www.liaoxuefeng.com/files/attachments/1481991021133888/l)

现在，我们制作一个 Docker 镜像，然后以 Docker 模式启动这个 Python 服务程序，再看看进程 ID：

![](https://www.liaoxuefeng.com/files/attachments/1481991048396864/l)

从进程自己的视角看，它看到的进程 ID 总是`1`，并且，用`/ps`看不到其他进程，只能看到自己：

![](https://www.liaoxuefeng.com/files/attachments/1481991075659843/l)

再用`/ls`看一下磁盘，看到的也不是系统的根目录，而是 Docker 给挂载的一个虚拟的文件系统：

![](https://www.liaoxuefeng.com/files/attachments/1481991094534208/l)

但其实从操作系统看，这个 Docker 进程和其他进程一样，也有一个唯一的进程 ID 为`10475`：

![](https://www.liaoxuefeng.com/files/attachments/1481991121797187/l)

所以我们可以得出结论：

一个容器进程本质上是一个运行在沙盒中的隔离进程，由 Linux 系统本身负责隔离，Docker 只是提供了一系列工具，帮助我们设置好隔离环境后，启动这个进程。

最基本的隔离就是进程之间看不到彼此，这是由 Linux 的 Namespace 机制实现的。进程隔离的结果就是以隔离方式启动的进程看到的自身进程 ID 总是 1，且看不到系统的其他进程。

第二种隔离就是隔离系统真实的文件系统。Docker 利用 Linux 的 mount 机制，给每个隔离进程挂载了一个虚拟的文件系统，使得一个隔离进程只能访问这个虚拟的文件系统，无法看到系统真实的文件系统。至于这个虚拟的文件系统应该长什么样，这就是制作 Docker 镜像要考虑的问题。比如我们的 Python 程序要正常运行，需要一个 Python3 解释器，需要把用到的第三方库如`psutil`引入进来，这些复杂的工作被简化为一个`Dockerfile`，再由 Docker 把这些运行时的依赖打包，就形成了 Docker 镜像。我们可以把一个 Docker 镜像看作一个 zip 包，每启动一个进程，Docker 都会自动解压 zip 包，把它变成一个虚拟的文件系统。

第三种隔离就是网络协议栈的隔离，这个最不容易理解。

我们举个例子：在 Docker 中运行`docker run redis:latest`，然后在宿主机上写个程序连接`127.0.0.1:6379`，是无法连接到 Redis 的，因为 Redis 虽然监听`127.0.0.1:6379`这个端口，但 Linux 可以为进程隔离网络，Docker 默认启动的 Redis 进程拥有自己的网络名字空间，与宿主机不同：

```
┌──────────────┐ ┌─────────────────────────┐
│redis:        │ │app:                     │
│  listen: 6379│ │  connect: 127.0.0.1:6379│
├──────────────┤ ├─────────────────────────┤
│127.0.0.1:6379│ │        127.0.0.1        │
└──────────────┘ └─────────────────────────┘


```

要让宿主机访问到 Redis，可以用`-p 6379:6379`把 Redis 进程的端口号映射到宿主机，从而在宿主机上访问 Redis：

```
┌──────────────┐ ┌─────────────────────────┐
│redis:        │ │app:                     │
│  listen: 6379│ │  connect: 127.0.0.1:6379│
├──────────────┤ ├─────────────────────────┤
│127.0.0.1:6379│ │     127.0.0.1:6379      │
└──────────────┘ └─────────────────────────┘
            │                     ▲
            │                     │
            └─────────────────────┘


```

因此，在 Linux 的网络名字空间隔离下，Redis 进程和宿主机进程看到的 IP 地址`127.0.0.1`表面上一样，但实际上是不同的网络接口。

我们再看一个更复杂的例子。如果我们要运行 ZooKeeper 和 Kafka，先启动 ZooKeeper：

```
docker run -p 2181:2181 zookeeper:latest


```

再启动 Kafka，发现 Kafka 是无法连接 ZooKeeper 的，原因是，Kafka 试图连接的`127.0.0.1:2181`在它自己的网络接口上并不存在：

```
┌──────────────┐ ┌──────────────┐ ┌─────────────────────────────┐
│zookeeper:    │ │kafka:        │ │Host                         │
│  listen: 2181│ │  listen: 9092│ │                             │
├──────────────┤ ├──────────────┤ ├──────────────┬──────────────┤
│127.0.0.1:2181│ │127.0.0.1:9092│ │127.0.0.1:9092│127.0.0.1:2181│
└──────────────┘ └──────────────┘ └──────────────┴──────────────┘
            │                │                ▲              ▲
            │                └────────────────┘              │
            └────────────────────────────────────────────────┘


```

必须连接到 ZooKeeper 的`IP:2181`或者宿主机的`IP:2181`。直接指定 IP 并不是一个好的方式，我们应该利用 Docker Compose，把 ZooKeeper 和 Kafka 运行在同一个网络名字空间里，并通过`zookeeper:2181`来访问 ZooKeeper 端口，让 Docker 自动把 zookeeper 名字解析为动态分配的 IP 地址。`docker-compose.yml`参考配置如下：

[docker-compose.yml](https://developer.confluent.io/quickstart/kafka-docker/)

搞懂了容器的运行原理，我们才能更好地掌握 Docker 命令。掌握了 Docker，才能进一步学习 K8s——云原生操作系统。

在此推荐 Kubernetes 专家张磊的极客时间课程：[深入剖析 Kubernetes](http://gk.link/a/11pAd)，一门课程讲清楚容器技术、K8s 集群、K8s 编排，实属云原生时代必备技能。