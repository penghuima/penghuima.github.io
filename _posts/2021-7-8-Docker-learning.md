---
layout: post
title: "Docker 学习记录"
categories: Docker
tags: Docker linux
---

* content
{:toc}
Docker 是一个操作系统级的虚拟化技术，是基于 LXC 技术构建的轻量级容器引擎。Docker为 DevOps 时代软件开发运维一体化提供了标准“集装箱”，开发者可以将应用及其依赖打包到可移植的容器中，实现在不同基础环境之间的无缝迁移。与传统的虚拟化技术相比，Docker 具有更高效的系统资源利用率、更快速的启动时间、提供一致的运行环境、更轻松的迁移等众多优势。Docker 已成为目前最为热门的开源项目之一，并以其为核心形成了繁荣的生态圈，在大型软件企业和初创公司中都得到大规模实践应用。

<!--more-->

### Hello Docker

在容器中输出 Hello Docker

```shell
$ docker pull busybox:latest   #这一步可以省略，docker run 的时候会自己主动去拉取
$ docker run --name container_name busybox:latest echo "Hello Docker"
```

- 第一条命令：获取一个名为 busybox:latest 的镜像。这条命令会从 Docker Hub 官方镜像仓库获取一个名为busybox:latest 镜像( busybox 的最新版)，并把它下载到宿主机。其中 busybox 是最小的 Linux 系统。
- 第二条命令： 创建并启动一个容器，并执行相应命令。首先，**--name** 设置容器的名字为 container_name，然后为容器指定了 busybox:latest 作为启动镜像，最后设置了该容器的启动命令为 echo "Hello Docker"。容器启动并输出 Hello Docker 后，就退出运行了。

```shell
$ docker ps -a
CONTAINER ID   IMAGE            COMMAND                 CREATED         STATUS                     PORTS     NAMES
7fc749ddc809   busybox:latest   "echo 'Hello Docker'"   6 seconds ago   Exited (0) 5 seconds ago             container_name
```

在这个实例中，我们并不需要进入容器执行“程序”，因为我们设置了容器的启动命令，也就是 echo "Hello Docker"。**在容器启动时会在容器中执行“启动命令”**，在本例中，启动命令 COMMAND 就是 echo "Hello Docker" ，执行输出了 Hello Docker ，已经达到了我们的要求，所以就无需进入容器内部执行了，且容器执行结束后也退出运行了。

**如果忘记命令参数，记得多使用 help 帮助**

```shell
$ docker --help
```

### Docker 镜像拉取

#### docker pull 

```shell
docker pull [OPTIONS] <仓库名>：<标签>
docker pull busybox:1.27  #拉取busybox:1.27镜像
```

**注意:**如果 tag 值为空，即没有指定标签，就会使用默认 tag ，也就是 latest ，如果 tag 值不为空，就使用指定的tag 标签

一般情况下，会在官方镜像仓库 Docker Hub 中寻找名为 repoName 的仓库，如果仓库不存在，返回错误信息。如果仓库存在，就从仓库中拉取对应 tag 的镜像。例如：如果执行 docker pull ubuntu:14.04 ，那么将从 ubuntu仓库中拉取 tag 为 14.04 的镜像 。（在官方镜像仓库 Docker Hub 中有很多个镜像仓库，一般情况下会将同一类型的镜像放在同一个仓库中，例如在一个 ubuntu 仓库中由很多个 ubuntu 镜像组成，包括 ubuntu:14.04、ubuntu:16.04 、ubuntu:latest 等等镜像）。

当将**镜像一层层下载**完成后，存储到本地。

#### docker images

查看本地镜像列表

#### docker rmi

删除镜像,如果想要使用 docker rmi 删除一个镜像，需要注意需要先将使用该镜像的容器删除掉，否则该镜像不能删除成功。当然也可以使用 docker rmi -f 强制删除该镜像！

```shell
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

#### 保存镜像 & 加载镜像

使用 docker save 和 docker load 命令，实现镜像备份和恢复

```shell
$ docker pull busybox:latest
#1.将busybox:latest镜像保存到tar包
$ docker save busybox:latest >busybox.tar
#删除busybox:latest镜像
$ docker rmi busybox:latest
#2.从tar包加载busybox:latest镜像
$ docker load <busybox.tar
```

如果想要将本机的alpine 镜像传到另一台机器

- 首先，在本机执行 docker save alpine > alpine-latest.tar 将 alpine 镜像保存成一个 tar 文件。

- 然后我们将 alpine-latest.tar 文件复制到了到了另一个机器上，然后在该机器上执行 docker load < alpine-latest.tar，这样就可以使用 tar 包将镜像加载进来了。如下图所示，可以通过 docker images alpine 查看到alpine:latest 镜像。

如果我们结合这两个命令以及 ssh 甚至 pv 的话，利用 Linux 强大的管道，我们可以写一个命令完成从一个机器将镜像迁移到另一个机器，并且带进度条的功能： 

```shell
$ docker save <镜像名> | bzip2 | pv | ssh <用户名>@<主机名> 'cat | docker load'
```

### Docker 容器运行

如果将 Docker 比作一艘轮船，那容器就是轮船上一个一个的集装箱，而镜像就是组成集装箱的基本材料。

启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（stopped）的容器重新启动。

#### docker run

```shell
docker run [OPTIONS] 镜像名 [COMMAND] [ARG]
#基于镜像新建容器
$ docker run --name container_name busybox:latest echo "Hello Docker"
#重新启动处于终止状态的容器
$ docker start [OPTIONS] 容器 [容器2...]
```

Docker run 背后的工作包括：

1. 检查本地是否存在指定的镜像，不存在就从公有仓库下载启动；
2. 利用镜像创建并启动一个容器；
3. 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层；
4. 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去；
5. 从地址池配置一个 ip 地址给容器；
6. 执行用户指定的启动命令；
7. 执行完毕后容器被终止。

上面的容器输出 Hello Docker 就退出运行处于终止状态了，我们新启动一个容器，并为容器分配一个终端，与用户交互。

注意：这里镜像选择 ubuntu 而不是 busybox ，busybox 没有 /bin/bash ，会报错

```shell
$ docker run -it --name container_1 ubuntu /bin/bash
root@b5c4fe4c6a61:/# echo "hello docker"    #进入容器
hello docker
root@b5c4fe4c6a61:/# exit
exit                                        #退出容器
```

其中，**-i** 选项告诉 Docker 保持标准输入输出流对容器开放，**-t** 选项让 Docker 分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上。

##### /bin/bash

当我们使用命令 docker run --name container_name busybox:latest echo "Hello Docker" 输出 Hello Docker 时，会发现容器输出后就立刻终止了，而且用 docker start 重启容器后也会很快终止。

这是因为 **Docker容器是一个进程，实际上它以 sh 作为主进程。如果主进程停止了，那么容器也就停止了。而如果容器的“启动命令”执行完之后，由于主进程没有命令继续执行，所以主进程会停止，容器也就因此而停止了**

> Q1：思考如何让容器启动后不立即终止

显然如果容器的 sh 主进程不停止，这个容器就不会停止！

> A1: 将启动命令设置为“启动一直运行的子进程”
>
> **docker run --name container_name  -it  ubuntu /bin/bash**   #进入容器
>
> **docker run --name container_name  -itd  ubuntu /bin/bash** #不进入容器

**执行完这条命令后，创建并启动容器之后，执行 /bin/bash，会启动一个子进程，此时父进程（也就是容器的主进程 sh）会进入 sleep 状态，由于 sleep 状态不是终止状态，所以容器会继续运行。**

> Q2:为什么在容器中输入 exit 或者执行 ctrl D 后，容器将会终止呢？

> A2:这是因为 exit 会退出（结束）当前进程，也就是 /bin/bash，由于子进程结束，sh 主进程恢复到运行态，然而由于没有命令需要继续执行，所以 sh 主进程结，因此容器终止。

> Q3: docker run --name container_name  -it  ubuntu /bin/bash  进入一个容器并 exit 退出之后，如何下次再重新进入容器呢？
>
> A: docker start 啊，  只要第一次创建的时候有通过 -it 创建子进程  /bin/bash 即可，下次 docker start 的时候也不会立即终止，并可以通过 docker exec 进入容器  

##### exec

docker exec进入一个容器内部

```shell
$ docker exec [options] containerName|containerId command [arg]
```

docker exec 命令可以在一个运行的容器内部执行一条命令，如执行docker exec <容器> mkdir dir1后，就在容器中创建了一个 dir1 的文件夹。

除此以外，还可以在容器中启动一个新的 bash，如执行了 docker exec -it <容器> /bin/bash，在容器内部启动了一个新的 bash 终端，并使用 **-it** 为其分配一个伪终端绑定到标准输出上。

##### attach

```shell
$ docker attach containerName|containerId
```

**attach与exec的主要区别**

1. attach 直接进入容器“启动命令”的终端，不会启动新的进程；
2. exec 则是在容器中打开新的终端，该终端与“启动命令”的终端不是同一个，并且可以启动新的进程；
3. 如果想直接在终端中查看容器“启动命令”的输出，用 attach；其他情况使用 exec

#### docker  ps

Docker 中有这样一条命令 docker ps，可以查看容器的信息，包括容器 id，基础镜像，启动命令，创建时间，当前状态，端口号，容器名字。

如果不加任何参数，只执行 docker ps，将会显示所有运行中的容器。

如果 docker ps –a 命令，可以查看 Docker 环境中所有的容器，包括已经停止的容器。

如果只想显示容器 id ,则执行 docker ps –aq ，在**快速删除所有容器**时，你会用的到的

```shell
$ docker stop $(docker ps -aq)
$ docker rm $(docker ps -aq)
```

#### docker rm

删除容器，等容器终止了再删掉，全删

```shell
$ docker stop $(docker ps -aq)
$ docker rm $(docker ps -aq)
```

#### 容器导入导出

```shell
docker run --name busyboxContainer busybox echo "hello"
#1.然后将busyboxContainer导出为容器快照：busybox.tar
docker export busyboxContainer >busybox.tar
#2.最后使用该容器快照导入镜像，镜像名为busybox:v1.0。
cat busybox.tar |docker import - busybox:v1.0
```

**docker export和docker save的区别**

首先，两者的操作对象不同。`docker save`是将一个镜像保存为一个`tar`包，而`docker export`是将一个容器快照保存为一个`tar`包。

然后，`docker export`导出的容器快照文件将丢弃所有的历史记录和元数据信息，即仅保存容器当时的快照状态；而`docker save`保存的镜像存储文件将保存完整记录，体积也要大

### Docker Dockerfile

#### Dockerfile简介

镜像的定制实际上就是定制每一层所添加的配置、文件。那么如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，就可以解决镜像构建透明性、体积等问题。这个脚本就是Dockerfile。

Dockerfile 描述了组装镜像的步骤，其中每一条命令都是单独执行的，除了`FROM`指令外，其他每一条指令都在上一条指定所生成的镜像基础上执行，执行完会生成一个新的镜像层，新的镜像层覆盖在原来的镜像层之上，从而形成了新的镜像。Dockerfile 所生成的最终镜像就是在基础叠加镜像上一层层的镜像层组成的。

在 Dockerfile 中，指令不区分大小写，但是为了与参数区分，推荐大写。Docker 会顺序执行 Dockerfile 中的指令，**第一条必须是 FROM 指令，它用于指定构建镜像的基础镜像**。在 Dockerfile 中，以 # 开头的行是注释。

下面介绍使用 Dockerfile 构建一个镜像，步骤如下：

- 首先创建一个空文件夹：mkdir newdir；
- 然后进入该文件夹：cd newdir；
- 在该文件夹下创建一个名为 Dockerfile 的文件，根据实际需求补全 Dockerfile 的内容；
- 使用 Dockerfile 构建一个镜像：docker build -t testimage .（注意这个小数点）其中 -t 指定新镜像的镜像名。

#### FROM、RUN

**FROM**：定制的镜像都是基于 FROM 的镜像，

**RUN**：用于执行后面跟着的命令行命令。有以下俩种格式：

shell 格式：

```shell
RUN <命令行命令>
# <命令行命令> 等同于，在终端操作的 shell 命令。
```

exec 格式：

```shell
RUN ["可执行文件", "参数1", "参数2"]
# 例如：
# RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
```

举一个实例，使用 Dockerfile 构建一个名为 testimage 的镜像，该镜像具备 ubuntu:latest 的运行环境，而且在镜像的/目录下创建了一个 dir1 文件夹

```shell
mkdir newdir         #先创建一个新的空文件夹
cd newdir            #进入这个新文件夹中
touch Dockerfile     #创建一个Dockerfile文件
#补全Dockerfile的内容（为了方便展示，这里用的是echo向Dockerfile中输入内容）
echo "FROM ubuntu:latest" > Dockerfile
echo "RUN mkdir /dir1" >> Dockerfile
#使用该Dockerfile构建一个名为testimage的镜像
docker build -t testimage .
```

执行 docker build 命令，指定使用 Dockerfile 构建一个镜像。执行结果如下所示：

```shell
[root@localhost newdir]# docker build -t testimage .
Sending build context to Docker daemon 2.048 kB
Step 1/2 : FROM ubuntu
 ---> 14f60031763d
Step 2/2 : RUN mkdir dir1
 ---> Running in c5117d908931
 ---> cb0193727724
Removing intermediate container c5117d908931
Successfully built cb0193727724
```

FROM ubuntu:latest 指定 ubuntu:latest 作为基础镜像，也就是将 ubuntu:latest 镜像的所有镜像层放置在testimage 镜像的最下面。然后执行 RUN mkdir dir1 指令，前面我们说过，执行 RUN 指令时，会在之前指令创建出的镜像的基础上创建一个临时容器，在这里的容器 Id 为 c5117d908931，并在容器中运行命令。在命令结束运行后提交新容器为新镜像，并删除临时创建的容器 c5117d908931

**注意:** Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。

因此以 **&&** 符号连接命令，这样执行后，只会创建 1 层镜像

```shell
FROM debian:jessie
RUN buildDeps='gcc libc6-dev make' \
&& apt-get update \
&& apt-get install -y $buildDeps \
&& wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
&& mkdir -p /usr/src/redis \
&& tar -xzf redis.tar.gz -C /usr/src/redis --strip-component
s=1 \
&& make -C /usr/src/redis \
&& make -C /usr/src/redis install \
&& rm -rf /var/lib/apt/lists/* \
&& rm redis.tar.gz \
&& rm -r /usr/src/redis \
&& apt-get purge -y --auto-remove $buildDeps
```

上面上述过程，一共创建2层镜像

在 Dockerfile 的编写过程中一定要牢记一点：**镜像的每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。**

#### docker build

在 Dockerfile 所在的文件夹下执行 docker build -t myimage . 这条命令，然后镜像就被构建了。

在 Linux 中，小数点`.`代表着当前目录。所以 docker build -t myimage . 中小数点`.`其实就是将当前目录设置为上下文路径。

执行 docker build 后，会首先将上下文目录的所有文件都打包，然后传给 Docker daemon ，这样 Docker daemon 收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

**解析**：由于 docker 的运行模式是 C/S。我们本机是 C，docker 引擎是 S。实际的构建过程是在 docker 引擎下完成的，所以这个时候无法用到我们本机的文件。这就需要把我们本机的指定目录下的文件一起打包提供给 docker 引擎使用。如果未说明最后一个参数，那么默认上下文路径就是 Dockerfile 所在的位置。

**注意**：上下文路径下不要放无用的文件，因为会一起打包发送给 docker 引擎，如果文件过多会造成过程缓慢

##### COPY

COPY 复制文件； 格式：COPY <源路径> <目标路径>；

COPY  指令将从构建上下文目录中 <源路径> 的文件或目录复制到新的一层的镜像内的 <目标路径> 位置。<源路径>所指定的源必须在上下文中，即必须是上下文根目录的相对路径！<目标路径> 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用  WORKDIR 指令来指定，后面介绍）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建目录。

##### ADD

ADD 更高级的文件复制； 格式：ADD <源路径> <目标路径>；

ADD 与 COPY 指令在功能上十分相似，但是在 COPY 的基础上增加了一些功能。比如，源路径可以是一个指向一个网络文件的 URL，这种情况下，Docker 引擎会试图下载这个`URL`指向的文件到<目标路径>去。

此外，当<源路径>为一个 tar 压缩文件时，**该压缩文件在被复制到容器中时会被解压提取**。但是使用 COPY 指令只会将 tar 压缩文件拷贝到<目标路径>中。

如果你只需要 tar 包中的文件内容而不需要 tar 包，不要先 COPY ./hello.txt.tar.gz ，然后 RUN tar –xvf hello.txt.tar.gz && rm hello.txt.tar.gz  。请直接使用ADD指令，ADD ./hello.txt.tar.gz

#### CMD

类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:

- CMD 在docker run 时运行。
- RUN 是在 docker build。

**作用**：**为启动的容器指定默认要运行的程序**，程序运行结束，容器也就结束。CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。(会被覆盖)

格式：

```shell
CMD ["<可执行文件或命令>","<param1>","<param2>",...] 
CMD ["<param1>","<param2>",...]  # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
```

#### ENTRYPOINT

类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。（不会被覆盖）

**优点**：在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。

格式：

```shell
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
```

可以搭配 CMD 命令使用：一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参，以下示例会提到。

假设已通过 Dockerfile 构建了 nginx:test 镜像：

```shell
FROM nginx
ENTRYPOINT ["nginx", "-c"] # 定参
CMD ["/etc/nginx/nginx.conf"] # 变参 
```

- 不传参运行

```shell
$ docker run  nginx:test
```

容器内会默认运行以下命令，启动主进程。

```shell
$ nginx -c /etc/nginx/nginx.conf
```

- 传参运行

```shell
$ docker run  nginx:test -c /etc/nginx/new.conf
```

容器内会默认运行以下命令，启动主进程(/etc/nginx/new.conf:假设容器内已有此文件)

```shell
$ nginx -c /etc/nginx/new.conf
```

#### ENV

设置环境变量，定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。

格式：

```shell
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```

#### WORKDIR

指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作目录，必须是提前创建好的）。

docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在。格式：

```shell
WORKDIR <工作目录路径>
```

#### EXPOSE指令

EXPOSE 暴露端口； 格式：EXPOSE <端口1> [<端口2>...]

EXPOSE 指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。如果想要公开容器的端口，必须在 docker run 是指定 -p 参数去公开端口或者指定 -P 参数公开所有被 EXPOSE 的端口。

在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是  docker run -P 时，会自动随机映射 EXPOSE 的端口。

#### 实例

**使用 Dockerfile，创建一个拥有 java 和 tomcat 运行环境的镜像**

Dockerfile 的内容如下，首先使用 FROM 指定基础镜像为 ubuntu:latest 镜像。然后使用 WORKDIR 设置当前的工作目录为 /var/tmp 。接下来使用 RUN 命令将 jre.tar.gz 下载到工作目录，并解压文件，然后删除 jre.tar.gz ；然后用类似的方式处理 tomcat。

接下来使用 ENV 配置 java 与 tomcat 的环境变量，由于 tomcat 服务会默认监听 8080 端口，所以使用 EXPOSE暴露端口号。最后使用 ENTRYPOINT 设置启动命令，使 tomcat 服务随容器启动而启动

```shell
FROM ubuntu
WORKDIR /var/tmp
RUN apt-get update && \
    apt-get install -y wget && \
    wget --no-check-certificate --no-cookies --header "Cookie: o\fraclelicense=accept-securebackup-cookie"  http://download.o\fracle.com/otn-pub/java/jdk/8u144-b01/090f390dda5b47b9b721c7dfaa008135/jre-8u144-linux-x64.tar.gz && \
    tar -xzf jre-8u144-linux-x64.tar.gz && \
    rm jre-8u144-linux-x64.tar.gz
RUN wget "http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.0.45/bin/apache-tomcat-8.0.45.tar.gz" && \
    tar -xzf apache-tomcat-8.0.45.tar.gz && \
    rm apache-tomcat-8.0.45.tar.gz
ENV JAVA_HOME /var/tmp/jre1.8.0_144
ENV CATALINA_HOME /var/tmp/apache-tomcat-8.0.45
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin
EXPOSE 8080
ENTRYPOINT /var/tmp/apache-tomcat-8.0.45/bin/startup.sh && /bin/bash
```

### Docker 数据卷管理

数据可以分为两类：持久化的和非持久化的。在 docker 中，每个 docker 容器都有自己的非持久化存储，非持久化存储自动创建，它从属于容器，与容器的生命周期相同，也就是说，容器被删除后，非持久化数据也会被删除。

如果大家想将自己的容器数据变为持久化数据，则需要将数据存储在卷（Volime）上。通常来说，卷与容器是解耦的，也就是说，卷与容器不是紧密相连的，因此可以独立地创建并且管理。

#### 创建容器卷并挂载

Docker1.9 版本以后，Docker 提供了一条新的命令：docker volume，用来创建、查看、删除数据卷。而传统的docker run -v 创建一个数据卷的方式也仍然保留了下来。

可以使用 docker volume create 创建一个数据卷，并指定数据卷的名字。如下所示：下面这条命令将创建了一个名为 vo1 的数据卷。

```shell
$ docker volume create --name vo1
```

除此以外，还可以在创建新容器的时候，通过添加`-v`标签创建一个数据卷，如下所示：下面这条命令基于`ubuntu`镜像进行创建并启动了一个容器，然后创建了一个“随机名字”的`volume`，并挂载到容器的`/data`目录。

```shell
$ docker run -itd -v /data ubuntu /bin/bash
```

当然也可以在创建新容器的时候，指定数据卷的名字，如下所示：这个命令就创建了一个名为`vo2`的数据卷，并挂载到了容器的`/data`目录。

```sh
$ docker run -itd -v vo2:/data ubuntu /bin/bash
```

之前我们说过，“数据卷”的内容会保存在宿主机的一个指定的目录上，默认情况下，在创建数据卷时，会在宿主机中的 `/var/lib/docker/volume/` 下创建一个以“数据卷名”为名的目录，并将数据卷的内容保存在该目录下的 `/_data` 目录下（也就是将数据卷的内容保存在 `/var/lib/docker/volume/数据卷名/_data/` 中）。

**数据卷挂载到容器目录之后，任何被写到容器目录下地内容都会被写到卷中**

在日常的工作中，我们一般不会用第一种方式来创建一个数据卷，因为一个没有被容器使用的数据卷显然意义是不大的。如果需要指定数据卷的名字，用第三种方式来创建就比较好了。

**挂载宿主机目录**

下面将宿主机的 /host/dir 挂载到了容器的 /container/dir 目录。

```shell
$ docker run --name vocotainer1 -v /host/dir:/container/dir ubuntu
```

**注意:** 宿主机的目录和容器的目录必须使用绝对路径。如果宿主机不存在`/host/dir`目录，则会创建一个空文件夹。在`/host/dir`下的所有文件和文件夹都可以在容器中在`/container/dir`下被访问。如果镜像中本来就存在`/container/dir`文件夹，那么该文件夹下所有内容都会被删除，保证与宿主机中文件夹一致。

#### 共享容器卷

在使用 docker run 创建并启动一个新容器时，也可以使用 **--volumes-from**  标签使容器与已有的容器共享数据卷，这样可以实现容器之间的外部数据共享。

```shell
#1.创建一个名为container1的容器，并将本地主机的/dir1目录挂载到容器中的/codir1中。
docker run --name container1 -v /dir1:/codir1 ubuntu
#2.创建一个名为container2的容器，与container1共享数据卷。
docker run --name container2 --volumes-from container1 ubuntu
```

上面的命令创建了一个名为 cotainer2 的容器，并与 container1 共享数据卷。因为 container1 的挂载点在 /dir上，所以 cotainer2 的挂载点也将会是 /dir

#### 查看数据卷

在 Docker 中可以通过 **docker inspect** 查看容器（--type container）、镜像（--type image）、数据卷等的具体信息，为了区分，所以最好指定具体类型为容器。通过 **--type** 参数可以指定具体类型，而 --type container 就是声明具体类型为容器。

查看输出 container1 的数据卷名字

```shell
$ docker inspect --type container --format='{{range .Mounts}}{{.Name}}{{end}}' container1
```

```json
"Mounts": [
            {
                "Type": "volume",
                "Name": "vo1",
                "Source": "/var/lib/docker/volumes/vo1/_data",
                "Destination": "/codir1",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
```

#### 删除数据卷

数据卷是被设计用来持久化数据的，它的生命周期独立于容器，如果在创建容器时挂载了数据卷，执行 docker rm 删除容器时，并不会自动地将容器对应的数据卷删除掉。在`Docker`中也不存在垃圾回收这样的机制来处理没有任何容器引用的数据卷。

- **删除容器的同时删除数据卷**

```shell
$ docker rm -v containerId|containerName
```

**在删除容器时如果想要将容器对应的数据卷也同时删除掉**，可以使用指定`-v`标签，但是值得注意的是，这种方法也只会尝试地去删除容器对应的数据卷，如果该数据卷还被其他容器使用，那么将删除不成功，但是如果这个数据卷已经不被任何其他容器所使用了，那么数据卷将会被删除。

如果是`docker run -v /data --name container1 ubuntu`创建的数据卷（没有显示指定数据卷名），如果该数据卷没有被其他任何容器使用，那么在使用`docker rm -v container1`尝试删除`container1`容器以及对应的数据卷时，会把数据卷删除掉。

**但是如果用 docker run -v vo1:/data --name container1 ubuntu 创建的数据卷（显示指定数据卷名），也就是创建时指定了“数据卷名”，那么在使用 docker rm -v container1 尝试删除容器以及对应的数据卷时，不会将数据卷删除，只是解除了数据卷和容器的联系。如果要删除数据卷，还得在上述基础上，继续使用docker volume rm vo1**

- **删除没有被容器使用的数据卷**

在我们的工作中难免在删除容器时忘记删除了数据卷，当然我们可以通过`docker volume rm`一条条地尝试的去删除。但是`docker`提供了更加简便的方法。也就是:`docker volume prune`，如果执行这条命令，那么会将所有没有被容器使用的数据卷删除掉

#### 备份恢复数据卷

通过一个例子演示数据卷的备份与恢复，数据卷的备份就是通过 --volumes-from 参数创建新的容器与旧容器共享存储卷，然后在新容器挂载宿主机目录，将共享数据卷打包在挂载宿主机目录中。

```shell
# 创建一个vo1的数据卷，并在数据卷中添加1.txt文件
$ docker run --name vocontainer1 -v vo1:/dir1 ubuntu touch /dir1/1.txt
#1.将vo1数据卷的数据备份到宿主机的$(pwd)中,将容器的/backup路径挂载上去，并将容器内/dir1文件夹打包至/backup/backup.tar
$ docker run --volumes-from vocontainer1 -v $(pwd):/backup ubuntu tar -cvf /backup/backup.tar /dir1
#删除所有的容器以及它使用的数据卷
$ docker rm -vf $(docker ps -aq)
$docker volume rm vo1
#在次创建一个vo1的数据卷
$ docker run -itd --name vocontainer2 -v vo1:/dir1 ubuntu /bin/bash
#2.将保存在宿主机中备份文件的数据恢复到vocontainer2的/中
$ docker run --volumes-from vocontainer2 -v $(pwd):/backup ubuntu tar -xvf /backup/backup.tar -C /
```

启动容器时，使用`tar`命令将数据卷的备份文件`backup.tar`解压到`/backup`目录，由于该容器与`vocontainer2`容器共享一个数据卷，也就相当于将`backup.tar`(解压之后是 /dir1 )解压到了`vocontainer2`的`/backup`目录。 又因为`vocontainer2`将名为`vo1`的数据卷挂载到了`/dir1`上，所以实质上就将`vo1`的数据卷内容完全恢复了！

### Docker 私人仓库管理

#### 创建私人仓库

在官方镜像仓库 Docker Hub 中提供了创建私人仓库的镜像 Resposity（镜像仓库）：Registry，本例将以Registry:2 镜像为例，构建一个私人仓库。

```
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

只需要上面这一条命令，一个私人仓库就创建好了。从这条命令可以看出，这个私人仓库以容器的形式运行着。其中 --restart=always 是指在 Docker 服务重启或者 registry 容器退出时会重新启动。而 -p 是指将宿主机的 5000 端口映射到容器的 5000 端口，这样就可以通过`宿主机ip：5000`访问到`容器的5000端口`了。（registry 容器默认会监听5000端口）。-d 参数是指在后台运行。

当然还有其他的配置，例如 -v 指定私人仓库的存储位置，添加 -v /mnt/registry:/var/lib/registry 可以将私人仓库的存储位置设置为宿主机的 /mnt/registry。

#### 将镜像推送到私人仓库

- 使用docker tag 给镜像加上一个标签

如果想要将镜像推送到私人仓库而不是 Docker Hub，首先必须使用 docker tag 命令，使用主机名和端口来标记一个镜像，如下所示，为 ubuntu:latest 镜像加上一个 localhost:5000/my-ubuntu:latest 的标签。

```
docker tag ubuntu:latest localhost:5000/my-ubuntu
```

- ##### 使用docker push将镜像推送到私人仓库

使用 docker push 命令可以将镜像推送到仓库，默认情况下会将镜像推送到官方仓库 Docker Hub 中去，但是如果推送一个“用主机名和端口来标记”的镜像，那么就会推送到私人仓库。

```
docker push localhost:5000/my-ubuntu
```

#### 从私人仓库拉取镜像

除了推送以外，当然还可以从私人仓库拉取镜像，docker pull 可以从仓库拉取某个镜像，默认情况下，也是从官方仓库拉取。当我想从私人仓库拉取 my-ubuntu:latest 镜像。执行以下命令就行了。

```
docker pull localhost:5000/my-ubuntu
```

#### 管理私人仓库中的镜像

Docker 提供的 Registry 镜像没有提供查看镜像和删除镜像的指令，但是有第三方的软件可以提供这些功能，例如：harbor 。

#### 删除私人仓库

私人仓库实质上就是一个容器，所以删除私人仓库就是删除私人仓库对应的容器。我们可以使用 docker rm -f 强制删除删除它，但是这样删除之后，私人仓库中存储的镜像并不会被删除掉。如果你想在删除私人仓库的同时，也将镜像删除，需要添加 -v 参数，也就是 docker rm -f -v 。例如删除本地的私人仓库，可以执行以下语句：

```
docker rm -vf myregistry
```

