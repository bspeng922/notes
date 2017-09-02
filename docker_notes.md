# Docker


要理解 Docker 内部构建，需要理解以下三种部件：

Docker 镜像 - Docker images
Docker 仓库 - Docker registeries
Docker 容器 - Docker containers
Docker 镜像
Docker 镜像是 Docker 容器运行时的只读模板，每一个镜像由一系列的层 (layers) 组成。Docker 使用 UnionFS 来将这些层联合到单独的镜像中。UnionFS 允许独立文件系统中的文件和文件夹(称之为分支)被透明覆盖，形成一个单独连贯的文件系统。正因为有了这些层的存在，Docker 是如此的轻量。当你改变了一个 Docker 镜像，比如升级到某个程序到新的版本，一个新的层会被创建。因此，不用替换整个原先的镜像或者重新建立(在使用虚拟机的时候你可能会这么做)，只是一个新 的层被添加或升级了。现在你不用重新发布整个镜像，只需要升级，层使得分发 Docker 镜像变得简单和快速。

Docker 仓库
Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。同样的，Docker 仓库也有公有和私有的概念。公有的 Docker 仓库名字是 Docker Hub。Docker Hub 提供了庞大的镜像集合供使用。这些镜像可以是自己创建，或者在别人的镜像基础上创建。Docker 仓库是 Docker 的分发部分。

Docker 容器
Docker 容器和文件夹很类似，一个Docker容器包含了所有的某个应用运行所需要的环境。每一个 Docker 容器都是从 Docker 镜像创建的。Docker 容器可以运行、开始、停止、移动和删除。每一个 Docker 容器都是独立和安全的应用平台，Docker 容器是 Docker 的运行部分。



docker run 创建 Docker 容器时，可以用 --net 选项指定容器的网络模式，Docker 有以下 4 种网络模式：

host 模式，使用 --net=host 指定。
container 模式，使用 --net=container:NAMEorID 指定。
none 模式，使用 --net=none 指定。
bridge 模式，使用 --net=bridge 指定，默认设置。


host 模式
如果启动容器的时候使用 host 模式，那么这个容器将不会获得一个独立的 Network Namespace，而是和宿主机共用一个 Network Namespace。容器将不会虚拟出自己的网卡，配置自己的 IP 等，而是使用宿主机的 IP 和端口。

container 模式
这个模式指定新创建的容器和已经存在的一个容器共享一个 Network Namespace，而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的 IP，而是和一个指定的容器共享 IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。两个容器的进程可以通过 lo 网卡设备通信。

none模式
这个模式和前两个不同。在这种模式下，Docker 容器拥有自己的 Network Namespace，但是，并不为 Docker容器进行任何网络配置。也就是说，这个 Docker 容器没有网卡、IP、路由等信息。需要我们自己为 Docker 容器添加网卡、配置 IP 等。

bridge模式
bridge 模式是 Docker 默认的网络设置，此模式会为每一个容器分配 Network Namespace、设置 IP 等，并将一个主机上的 Docker 容器连接到一个虚拟网桥上。当 Docker server 启动时，会在主机上创建一个名为 docker0 的虚拟网桥，此主机上启动的 Docker 容器会连接到这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。接下来就要为容器分配 IP 了，Docker 会从 RFC1918 所定义的私有 IP 网段中，选择一个和宿主机不同的IP地址和子网分配给 docker0，连接到 docker0 的容器就从这个子网中选择一个未占用的 IP 使用。如一般 Docker 会使用 172.17.0.0/16 这个网段，并将 172.17.42.1/16 分配给 docker0 网桥（在主机上使用 ifconfig 命令是可以看到 docker0 的，可以认为它是网桥的管理接口，在宿主机上作为一块虚拟网卡使用）



在docker容器中运行hello world
$docker run learn/tutorial echo "hello word"

在容器中安装新的程序
$docker run learn/tutorial apt-get install -y ping

保存对容器的修改
docker ps -l命令获得安装完ping命令之后容器的id
docker commit 698 learn/ping

运行新的镜像
$docker run lean/ping ping www.google.com

检查运行中的镜像
docker ps命令可以查看所有正在运行中的容器列表
docker inspect命令我们可以查看更详细的关于某一个容器的信息
$ docker inspect efe

发布自己的镜像
docker images命令可以列出所有安装过的镜像。
docker push命令可以将某一个镜像发布到官方网站。
你只能将镜像发布到自己的空间下面。



## 运行容器

docker run -d -it -p 8009:8000 django /bin/bash

docker run - 运行一个容器
-d - 后台运行
-t - 分配一个（伪）tty (link is external)
-i - 交互模式 (so we can interact with it)
-p - 端口映射
django - 使用django镜像
/bin/bash - 运行命令 bash shell

```
[root@controller ~]# docker run -d -it -p 8009:8000 django /bin/bash
d2d2c81f6b330259dc2b4a7b0c5cb0050b60b63384819f149f3201063bd0f76c
[root@controller ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                            PORTS                    NAMES
d2d2c81f6b33        django              "/bin/bash"         16 seconds ago      Up 14 seconds                     0.0.0.0:8009->8000/tcp   prickly_feynman
deff027ce837        django              "/bin/bash"         10 minutes ago      Exited (127) About a minute ago                            hungry_archimedes
8d1d62f710de        django:latest       "/bin/bash"         3 days ago          Exited (127) 3 days ago                                    suspicious_newton
1ac62ac4c5b5        hello-world         "/hello"            3 days ago          Exited (0) 3 days ago                                      agitated_northcutt
[root@controller ~]# docker port d2d2c81f6b33
8000/tcp -> 0.0.0.0:8009
```


## Dockerfile

下面列出了 Dockerfile 中最常用的指令，完整列表和说明可参看官方文档。

FROM
指定 base 镜像。

MAINTAINER
设置镜像的作者，可以是任意字符串。

COPY
将文件从 build context 复制到镜像。
COPY 支持两种形式：

COPY src dest

COPY ["src", "dest"]

注意：src 只能指定 build context 中的文件或目录。

ADD
与 COPY 类似，从 build context 复制文件到镜像。不同的是，如果 src 是归档文件（tar, zip, tgz, xz 等），文件会被自动解压到 dest。

ENV
设置环境变量，环境变量可被后面的指令使用。例如：
...

ENV MY_VERSION 1.3

RUN apt-get install -y mypackage=$MY_VERSION

...


EXPOSE
指定容器中的进程会监听某个端口，Docker 可以将该端口暴露出来。我们会在容器网络部分详细讨论。

VOLUME
将文件或目录声明为 volume。我们会在容器存储部分详细讨论。

WORKDIR
为后面的 RUN, CMD, ENTRYPOINT, ADD 或 COPY 指令设置镜像中的当前工作目录。

RUN
在容器中运行指定的命令。

CMD
容器启动时运行指定的命令。
Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效。CMD 可以被 docker run 之后的参数替换。

ENTRYPOINT
设置容器启动时运行的命令。
Dockerfile 中可以有多个 ENTRYPOINT 指令，但只有最后一个生效。CMD 或 docker run 之后的参数会被当做参数传递给 ENTRYPOINT。

