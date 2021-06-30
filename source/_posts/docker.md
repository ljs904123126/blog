---
title: Docker分享
date: 2021-05-17 17:17:19
tags: docker
category: linux
---

# Docker分享

## Docker 是什么? 为什么要使用?

### Docker 是什么

Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。它是目前最流行的 Linux 容器解决方案。

Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。

总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

### 为什么要使用docker

#### Docker 对于研发

1. 作为开发者而言，Docker 镜像机制可以帮你快速创建一个可用的开发环境，我经常会遇到，开发者可能需要各种开源依赖组件或者环境，但对于一些稍微复杂点的软件，可能需要几个小时甚至半天时间，有docker 镜像后，任何开发者都可以秒级创建开发环境

2. 一致的运行环境，避免“我机器上可以运行”，Docker 镜像并不会因为环境的变化而不能运行，也不会在不同的电脑上有不同的运行结果。可以给测试人员提交含有应用的 Docker 镜像，这样便不再会发生“在我机器上是可以运行的”这种事情，很大程度上减轻了开发人员测试人员互相检查机器环境设置带来的时间成本。

#### Docker 对于运维

1. **更高效的利用系统资源**。

   由于容器不需要进行硬件虚拟以及运行完整操作系统等额外开销，`Docker` 对系统资源的利用率更高。无论是应用执行速度、内存损耗或者文件存储速度，都要比传统虚拟机技术更高效。因此，相比虚拟机技术，一个相同配置的主机，往往可以运行更多数量的应用。

2. **持续交付和部署**

   使用 `Docker` 可以通过定制应用镜像来实现持续集成、持续交付、部署。开发人员可以通过 [Dockerfile]() 来进行镜像构建，并结合 [持续集成(Continuous Integration)](https://en.wikipedia.org/wiki/Continuous_integration) 系统进行集成测试，而运维人员则可以直接在生产环境中快速部署该镜像，甚至结合 [持续部署(Continuous Delivery/Deployment)](https://en.wikipedia.org/wiki/Continuous_delivery) 系统进行自动部署。

3. **更轻松的迁移**

   由于 `Docker` 确保了执行环境的一致性，使得应用的迁移更加容易。

4. **轻松的维护和扩展**

   `Docker` 使用的分层存储以及镜像的技术，使得应用重复部分的复用更为容易，也使得应用的维护更新更加简单，基于基础镜像进一步扩展镜像也变得非常简单



## Docker 的基本原理和架构

### 容器的基础

​		Docker 底层的核心技术包括 Linux 上的命名空间（Namespaces）、控制组（Control groups）、Union 文件系统（Union file systems）和容器格式（Container format）。我们知道，传统的虚拟机通过在宿主主机中运行 hypervisor 来模拟一整套完整的硬件环境提供给虚拟机的操作系统。虚拟机系统看到的环境是可限制的，也是彼此隔离的。 这种直接的做法实现了对资源最完整的封装，但很多时候往往意味着系统资源的浪费。 例如，以宿主机和虚拟机系统都为 Linux 系统为例，虚拟机中运行的应用其实可以利用宿主机系统中的运行环境。

让某些进程在彼此隔离的命名空间中运行。大家虽然都共用一个内核和某些运行时环境（例如一些系统命令和系统库），但是彼此却看不到，都以为系统中只有自己的存在。这种机制就是容器（Container），利用命名空间来做权限的隔离控制，利用 cgroups 来做资源分配。

#### 命名空间

​		来实现进程等资源的隔离，命名空间是 Linux 内核一个强大的特性。每个容器都有自己单独的命名空间，运行在其中的应用都像是在独立的操作系统中运行一样。命名空间保证了容器之间彼此互不影响。

#### 控制组

​		容器资源使用的控制，控制组（[cgroups](https://en.wikipedia.org/wiki/Cgroups)）是 Linux 内核的一个特性，主要用来对共享资源进行隔离、限制、审计等。只有能控制分配到容器的资源，才能避免当多个容器同时运行时的对系统资源的竞争。

#### 文件系统

​		联合文件系统（[UnionFS](https://en.wikipedia.org/wiki/UnionFS)）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。联合文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。另外，不同 Docker 容器就可以共享一些基础的文件系统层，同时再加上自己独有的改动层，大大提高了存储的效率。



### Docker 组件架构

​		Docker 采用了 `C/S` 架构，包括客户端和服务端。

#### 服务端（Docker Daemon ）

Docker 守护进程 （`Daemon`）作为服务端接受来自客户端的请求，并处理这些请求（创建、运行、分发容器）。

##### 镜像（Image）

​		Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。比如官方镜像 `ubuntu:18.04` 就包含了完整的一套 Ubuntu 18.04 最小系统的 `root` 文件系统。

##### 容器（Container ）

​		容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 [命名空间](https://en.wikipedia.org/wiki/Linux_namespaces)。因此容器可以拥有自己的 `root` 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。

##### Dockerfile

​		镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

#### 客户端

​		客户端和服务端既可以运行在一个机器上，也可通过 `socket` 或者 `RESTful API` 来进行通信。Docker 客户端则为用户提供一系列可执行命令，用户用这些命令实现跟 Docker 守护进程交互。

#### 仓库（Registry）

​		镜像构建完成后，可以很容易的在当前宿主机上运行，但是，如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，[Docker Registry]() 就是这样的服务。

​		一个 **Docker Registry** 中可以包含多个 **仓库**（`Repository`）；每个仓库可以包含多个 **标签**（`Tag`）；每个标签对应一个镜像。

通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 `<仓库名>:<标签>` 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 `latest` 作为默认标签。



## Docker环境搭建和基本操作



### Docker安装

#### centos

安装命令如下：
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

也可以使用国内 daocloud 一键安装命令：
curl -sSL https://get.daocloud.io/docker | sh



#### macos

下载安装：https://download.docker.com/mac/stable/Docker.dmg



### 修改国内镜像

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://v5sxbuv2.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



### 普通用户执行

```shell
# 需要讲普通用户增加到docker组
sudo usermod -aG docker user
```



### Docker Hello World

```shell
#下载镜像
docker pull ubuntu:15.10

#查看下载的镜像
docker images

#使用镜像运行容器
docker run ubuntu:15.10 /bin/echo "Hello world"

#查看所有容器
docker ps -a

```



执行结果

```shell
[root@VM_0_5_centos spring-maven-docker]# docker pull ubuntu:15.10
15.10: Pulling from library/ubuntu
7dcf5a444392: Pull complete 
759aa75f3cee: Pull complete 
3fa871dc8a2b: Pull complete 
224c42ae46e7: Pull complete 
Digest: sha256:02521a2d079595241c6793b2044f02eecf294034f31d6e235ac4b2b54ffc41f3
Status: Downloaded newer image for ubuntu:15.10
docker.io/library/ubuntu:15.10

[root@VM_0_5_centos spring-maven-docker]# docker images
REPOSITORY               TAG            IMAGE ID       CREATED         SIZE
ubuntu                   latest         f63181f19b2f   4 weeks ago     72.9MB
adoptopenjdk/openjdk11   alpine         9e85955d5a16   2 months ago    345MB
openjdk                  8-jdk-alpine   a3562aa0b991   21 months ago   105MB
ubuntu                   15.10          9b9cb95443b5   4 years ago     137MB

[root@VM_0_5_centos spring-maven-docker]# docker run ubuntu:15.10 /bin/echo "Hello world"
Hello world

[root@VM_0_5_centos spring-maven-docker]# docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                       PORTS     NAMES
f39fece6603c   ubuntu:15.10   "/bin/echo 'Hello wo…"   57 seconds ago   Exited (0) 56 seconds ago              blissful_benz


```





### docker常用指令

官方文档：https://docs.docker.com/engine/reference/commandline/pull/

#### 镜像指令

```shell

docker images	查看所有本地主机上的镜像
docker search mysql	搜索镜像
docker pull mysql	下载镜像
docker pull mysql:5.7	指定版本下载
docker rmi -f ID/name	删除镜像
docker rmi -f 容器id	删除指定的镜像
docker rmi -f 镜像id 镜像id 镜像id 镜像id	删除多个镜像
docker rmi -f $(docker images -aq)	#删除全部镜像
docker save 将指定镜像保存成 tar 归档文件
docker load 导入使用 docker save 命令导出的镜像
```



#### 容器指令

```shell
docker ps 命令
	-a	#列出当前正在运行的容器+带出历史运行中的容器
	-n=?	#显示最近创建的容器
	-q	#只显示容器的编号
docker rm 容器id	#删除指定的容器，不能删除在运行的容器，如果要强制删除 rm -f
docker rm -f $(docker ps -aq)	#删除所有的容器
docker ps -a -q|xargs docker rm 	#删除所有的容器

docker start 容器id	#启动容器
docker restart 容器id	#重启容器
docker stop 容器id	#停止当前正在运行的容器
docker kill 容器id	#停止当前容器

############## 退出容器 需要进入容器内执行
exit	#直接容器停止并退出
ctrl + P + Q	#容器不停止退出


############## 进入正在运行中的容器，需要在run 时增加 -d 参数
#我们通常都是使用后台的方式运行的，需要进入容器，修改一些配置
#命令
#方法一：
	docker exec -it 容器id bashshell
	eg:docker exec -it 容器id /bin/bash
	ls
	ps -ef
#方法二：
	docker attach 容器id

docker exec	#进入容器后开启一个新的终端，可以在里面操作（常用）
docker attach	#进入容器正在执行的终端，不会启动新的进程
```





## SpringBoot服务完整实例

### 开发一个SpringBoot Web程序

#### 插件

GitHub：https://github.com/spotify/docker-maven-plugin

```xml
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>

    <plugin>
        <groupId>com.spotify</groupId>
        <artifactId>dockerfile-maven-plugin</artifactId>
        <version>1.4.13</version>
        <executions>
            <execution>
                <id>default</id>
                <goals>
                    <goal>build</goal>
                    <goal>push</goal>
                </goals>
            </execution>
        </executions>
        <configuration>
            <repository>${project.artifactId}</repository>
            <tag>${project.version}</tag>
            <buildArgs>
                <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
            </buildArgs>
        </configuration>
    </plugin>

</plugins>
```

增加插件白名单 setting.xml

```xml
<settings>
    <pluginGroups>
        <pluginGroup>com.spotify</pluginGroup>
    </pluginGroups>
</settings>
```



### 编写一个 Dockerfile 

需要想文件放置在根目录下

```dockerfile
FROM openjdk:8-jdk-alpine
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring
VOLUME /tmp
ARG JAR_FILE
ADD ${JAR_FILE} /app/app.jar
EXPOSE 8081
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app/app.jar"]
```



### 镜像制作和发布

打包命令

#### 使用maven插件

```shell
mvn clean package dockerfile:build 
```

#### 使用docker命令

```shell
 docker build --build-arg JAR_FILE=target/demo-application-0.0.1-SNAPSHOT.jar -t demo-application:0.0.1-SNAPSHOT .
```



#### 查看是否打包成功

```
[root@VM_0_5_centos spring-maven-docker]# docker images

REPOSITORY               TAG              IMAGE ID       CREATED          SIZE
demo-application         0.0.1-SNAPSHOT   655c8a29d03b   4 minutes ago    121MB
```



### 容器部署

```shell
docker run -p 8081:8081 -t demo-application:0.0.1-SNAPSHOT
```



### 测试效果

```http
http://148.70.127.52:8081/hello?name=panyumei
```





# 工具安装 

## Visual Studio Code Remote

### 插件

1. Remote - SSH
2. Docker