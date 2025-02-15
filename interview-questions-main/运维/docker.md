1. Docker 和虚拟机有啥不同？
   答：Docker 是轻量级的沙盒，在其中运行的只是应用，虚拟机里面还有额外的系统。
2. Docker 安全么？
   答：Docker 利用了 Linux 内核中很多安全特性来保证不同容器之间的隔离，并且通过
   签名机制来对镜像进行验证。大量生产环境的部署证明，Docker 虽然隔离性无法与虚
   拟机相比，但仍然具有极高的安全性。
3. 如何清理后台停止的容器？
   答：可以使用 sudo docker rm $sudo( docker ps -a -q) 命令。
4. 如何查看镜像支持的环境变量？
   答：可以使用 docker run IMAGE env 命令。
5. 当启动容器的时候提示：exec format error？如何解决问题
   答：检查启动命令是否有可执行权限，进入容器手工运行脚本进行排查。
6. 本地的镜像文件都存放在哪里？
   答：与 Docker 相关的本地资源都存放在/var/lib/docker/目录下，其中 container 目录
   存放容器信息，graph 目录存放镜像信息，aufs 目录下存放具体的内容文件。
7. 如何退出一个镜像的 bash，而不终止它？
   答：按 Ctrl + P + Q。
8. 退出容器时候自动删除?
   答：使用 –rm 选项，例如 sudo docker run –rm -it ubuntu
9. 怎么快速查看本地的镜像和容器？
   答：可以通过 docker images 来快速查看本地镜像；通过 docker ps -a 快速查看本地容器。
   镜像相关：
10. 如何批量清理临时镜像文件？
    答：可以使用 sudo docker rmi $(sudo docker images -q -f danging=true)命令
11. 如何查看镜像支持的环境变量？
    答：使用 sudo docker run IMAGE env
12. 本地的镜像文件都存放在哪里
    答：于 Docker 相关的本地资源存放在/var/lib/docker/目录下，其中 container 目录存放
    容器信息，graph 目录存放镜像信息，aufs 目录下存放具体的镜像底层文件。
13. 构建 Docker 镜像应该遵循哪些原则？
    答：整体远侧上，尽量保持镜像功能的明确和内容的精简，要点包括：
     尽量选取满足需求但较小的基础系统镜像，建议选择 debian:wheezy 镜像，仅有
    86MB 大小
     清理编译生成文件、安装包的缓存等临时文件
     安装各个软件时候要指定准确的版本号，并避免引入不需要的依赖
     从安全的角度考虑，应用尽量使用系统的库和依赖
     使用 Dockerfile 创建镜像时候要添加.dockerignore 文件或使用干净的工作目录
    容器相关
14. 容器退出后，通过 docker ps 命令查看不到，数据会丢失么？
    答：容器退出后会处于终止（exited）状态，此时可以通过 docker ps -a 查看，其中数
    据不会丢失，还可以通过 docker start 来启动，只有删除容器才会清除数据。
15. 如何停止所有正在运行的容器？
    答：使用 docker kill $(sudo docker ps -q)

# docker的工作原理是什么，讲一下

docker是一个Client-Server结构的系统，docker守护进程运行在宿主机上，守护进程从客户端接受命令并管理运行在主机上的容器，容器是一个运行时环境，这就是我们说的集装箱。

# docker的组成包含哪几大部分

一个完整的docker有以下几个部分组成：

1、docker client，客户端，为用户提供一系列可执行命令，用户用这些命令实现跟 docker daemon 交互；

2、docker daemon，守护进程，一般在宿主主机后台运行，等待接收来自客户端的请求消息；

3、docker image，镜像，镜像run之后就生成为docker容器；

4、docker container，容器，一个系统级别的服务，拥有自己的ip和系统目录结构；运行容器前需要本地存在对应的镜像，如果本地不存在该镜像则就去镜像仓库下载。

docker 使用客户端-服务器 (C/S) 架构模式，使用远程api来管理和创建docker容器。docker 容器通过 docker 镜像来创建。容器与镜像的关系类似于面向对象编程中的对象与类。

# 

# docker与传统虚拟机的区别什么？

1、传统虚拟机是需要安装整个操作系统的，然后再在上面安装业务应用，启动应用，通常需要几分钟去启动应用，而docker是直接使用镜像来运行业务容器的，其容器启动属于秒级别；

2、Docker需要的资源更少，Docker在操作系统级别进行虚拟化，Docker容器和内核交互，几乎没有性能损耗，而虚拟机运行着整个操作系统，占用物理机的资源就比较多;

3、Docker更轻量，Docker的架构可以共用一个内核与共享应用程序库，所占内存极小;同样的硬件环境，Docker运行的镜像数远多于虚拟机数量，对系统的利用率非常高;

4、与虚拟机相比，Docker隔离性更弱，Docker属于进程之间的隔离，虚拟机可实现系统级别隔离;

5、Docker的安全性也更弱，Docker的租户root和宿主机root相同，一旦容器内的用户从普通用户权限提升为root权限，它就直接具备了宿主机的root权限，进而可进行无限制的操作。虚拟机租户root权限和宿主机的root虚拟机权限是分离的，并且虚拟机利用如Intel的VT-d和VT-x的ring-1硬件隔离技术，这种技术可以防止虚拟机突破和彼此交互，而容器至今还没有任何形式的硬件隔离;

6、Docker的集中化管理工具还不算成熟，各种虚拟化技术都有成熟的管理工具，比如：VMware vCenter提供完备的虚拟机管理能力;

7、Docker对业务的高可用支持是通过快速重新部署实现的，虚拟化具备负载均衡，高可用、容错、迁移和数据保护等经过生产实践检验的成熟保障机制，Vmware可承诺虚拟机99.999%高可用，保证业务连续性;

8、虚拟化创建是分钟级别的，Docker容器创建是秒级别的，Docker的快速迭代性，决定了无论是开发、测试、部署都可以节省大量时间;

9、虚拟机可以通过镜像实现环境交付的一致性，但镜像分发无法体系化，Docker在Dockerfile中记录了容器构建过程，可在集群中实现快速分发和快速部署。

# 

# docker技术的三大核心概念是什么？

镜像：镜像是一种轻量级、可执行的独立软件包，它包含运行某个软件所需的所有内容，我们把应用程序和配置依赖打包好形成一个可交付的运行环境(包括代码、运行时需要的库、环境变量和配置文件等)，这个打包好的运行环境就是image镜像文件。

容器：容器是基于镜像创建的，是镜像运行起来之后的一个实例，容器才是真正运行业务程序的地方。如果把镜像比作程序里面的类，那么容器就是对象。

镜像仓库：存放镜像的地方，研发工程师打包好镜像之后需要把镜像上传到镜像仓库中去，然后就可以运行有仓库权限的人拉取镜像来运行容器了。

# centos镜像几个G，但是docker centos镜像才几百兆，这是为什么？

一个完整的Linux操作系统包含Linux内核和rootfs根文件系统，即我们熟悉的/dev、/proc/、/bin等目录。我们平时看到的centOS除了rootfs，还会选装很多软件，服务，图形桌面等，所以centOS镜像有好几个G也不足为奇。

而对于容器镜像而言，所有容器都是共享宿主机的Linux 内核的，而对于docker镜像而言，docker镜像只需要提供一个很小的rootfs即可，只需要包含最基本的命令，工具，程序库即可，所有docker镜像才会这么小。

# 

# 讲一下镜像的分层结构以及为什么要使用镜像的分层结构？

一个新的镜像其实是从 base 镜像一层一层叠加生成的。每安装一个软件，dockerfile中使用RUN命令，就会在现有镜像的基础上增加一层，这样一层一层的叠加最后构成整个镜像。所以我们docker pull拉取一个镜像的时候会看到docker是一层层拉取的。

分层机构最大的一个好处就是 ： 共享资源。比如：有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像；同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

讲一下容器的copy-on-write特性，修改容器里面的内容会修改镜像吗？

我们知道，镜像是分层的，镜像的每一层都可以被共享，同时，镜像是只读的。当一个容器启动时，一个新的可写层被加载到镜像的顶部，这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。

所有对容器的改动 - 无论添加、删除、还是修改文件，都只会发生在容器层中，因为只有容器层是可写的，容器层下面的所有镜像层都是只读的。镜像层数量可能会很多，所有镜像层会联合在一起组成一个统一的文件系统。如果不同层中有一个相同路径的文件，比如 /a，上层的 /a 会覆盖下层的 /a，也就是说用户只能访问到上层中的文件 /a。在容器层中，用户看到的是一个叠加之后的文件系统。

添加文件时：在容器中创建文件时，新文件被添加到容器层中。

读取文件：在容器中读取某个文件时，Docker 会从上往下依次在各镜像层中查找此文件。一旦找到，立即将其复制到容器层，然后打开并读入内存。

修改文件：在容器中修改已存在的文件时，Docker 会从上往下依次在各镜像层中查找此文件。一旦找到，立即将其复制到容器层，然后修改之。

删除文件：在容器中删除文件时，Docker 也是从上往下依次在镜像层中查找此文件。找到后，会在容器层中记录下此删除操作。

只有当需要修改时才复制一份数据，这种特性被称作 Copy-on-Write。可见，容器层保存的是镜像变化的部分，不会对镜像本身进行任何修改。

# 简单描述一下Dockerfile的整个构建镜像过程

好的。

1、首先，创建一个目录用于存放应用程序以及构建过程中使用到的各个文件等；

2、然后，在这个目录下创建一个Dockerfile文件，一般建议Dockerfile的文件名就是Dockerfile；

3、编写Dockerfile文件，编写指令，如，使用FROM 指令指定基础镜像，COPY指令复制文件，RUN指令指定要运行的命令，ENV设置环境变量，EXPOSE指定容器要暴露的端口，WORKDIR设置当前工作目录，CMD容器启动时运行命令，等等指令构建镜像；

4、Dockerfile编写完成就可以构建镜像了，使用docker build -t 镜像名:tag . 命令来构建镜像，最后一个点是表示当前目录，docker会默认寻找当前目录下的Dockerfile文件来构建镜像，如果不使用默认，可以使用-f参数来指定dockerfile文件，如：docker build -t 镜像名:tag -f /xx/xxx/Dockerfile ；

5、使用docker build命令构建之后，docker就会将当前目录下所有的文件发送给docker daemon，顺序执行Dockerfile文件里的指令，在这过程中会生成临时容器，在临时容器里面安装RUN指定的命令，安装成功后，docker底层会使用类似于docker commit命令来将容器保存为镜像，然后删除临时容器，以此类推，一层层的构建镜像，运行临时容器安装软件，直到最后的镜像构建成功。

# Dockerfile构建镜像出现异常，如何排查？

首先，Dockerfile是一层一层的构建镜像，期间会产生一个或多个临时容器，构建过程中其实就是在临时容器里面安装应用，如果因为临时容器安装应用出现异常导致镜像构建失败，这时容器虽然被清理掉了，但是期间构建的中间镜像还在，那么我们可以根据异常时上一层已经构建好的临时镜像，将临时镜像运行为容器，然后在容器里面运行安装命令来定位具体的异常。

————————————————

### Dockerfile的基本指令有哪些？

```
FROM 		指定基础镜像（必须为第一个指令，因为需要指定使用哪个基础镜像来构建镜像）；
MAINTAINER 设置镜像作者相关信息，如作者名字，日期，邮件，联系方式等；
COPY 		复制文件到镜像；
ADD 		复制文件到镜像（ADD与COPY的区别在于，ADD会自动解压tar、zip、tgz、xz等归档文件，而COPY不会，同时ADD指令还可以接一个url下载文件地址，一般建议使用COPY复制文件即可，文件在宿主机上是什么样子复制到镜像里面就是什么样子这样比较好）；
ENV 		设置环境变量；
EXPOSE 		暴露容器进程的端口，仅仅是提示别人容器使用的哪个端口，没有过多作用；
VOLUME 		数据卷持久化，挂载一个目录；
WORKDIR 	设置工作目录，如果目录不在，则会自动创建目录；
RUN 		在容器中运行命令，RUN指令会创建新的镜像层，RUN指令经常被用于安装软件包；
CMD 		指定容器启动时默认运行哪些命令，如果有多个CMD，则只有最后一个生效，另外，CMD指令可以被docker run之后的参数替换；
ENTRYOINT 	指定容器启动时运行哪些命令，如果有多个ENTRYOINT，则只有最后一个生效，另外，如果Dockerfile中同时存在CMD和ENTRYOINT，那么CMD或docker run之后的参数将被当做参数传递给ENTRYOINT；

```

# 在 Dockerfile 中，可以通过 CMD 和 ENTRYPOINT 两个指令来定义容器启动时要执行的命令，它们之间的区别如下：

1. CMD 指令：

- - 使用格式为 CMD [command, parameter1, parameter2, ...]。
  - 可以为 Docker 镜像设置默认命令和参数，但是如果用户指定了启动容器时要执行的命令，则会覆盖默认设置。
  - 在 Dockerfile 中可以有多个 CMD 指令，但只有最后一个会生效。

1. ENTRYPOINT 指令：

- - 使用格式为 ENTRYPOINT [command, parameter1, parameter2, ...]。
  - 与 CMD 类似，ENTRYPOINT 也定义了默认的要执行的命令和参数，但是与 CMD 不同的是，ENTRYPOINT 被定义的命令及参数不会被覆盖，而会作为基础命令。
  - 如果在 docker run 时指定了新的命令，这个新的命令会作为参数附加在 ENTRYPOINT 命令的后面执行。

# 如何进入容器？使用哪个命令

进入容器有两种方法：docker attach、docker exec；

docker attach命令是attach到容器启动命令的终端，docker exec 是另外在容器里面启动一个TTY终端。

docker run -d centos /bin/bash -c "while true;do sleep 2;echo I_am_a_container;done"

3274412d88ca4f1d1292f6d28d46f39c14c733da5a4085c11c6a854d30d1cde0

docker attach 3274412d88ca4f						#attach进入容器

Ctrl + c  退出，Ctrl + c会直接关闭容器终端，这样容器没有进程一直在前台运行就会死掉了

Ctrl + pq 退出（不会关闭容器终端停止容器，仅退出）

```
docker run -d centos /bin/bash -c "while true;do sleep 2;echo I_am_a_container;done"
3274412d88ca4f1d1292f6d28d46f39c14c733da5a4085c11c6a854d30d1cde0
docker attach 3274412d88ca4f						#attach进入容器
Ctrl + c  退出，Ctrl + c会直接关闭容器终端，这样容器没有进程一直在前台运行就会死掉了
Ctrl + pq 退出（不会关闭容器终端停止容器，仅退出）

docker exec -it 3274412d88ca /bin/bash				#exec进入容器	
[root@3274412d88ca /]# ps -ef						#进入到容器了开启了一个bash进程
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 05:31 ?        00:00:01 /bin/bash -c while true;do sleep 2;echo I_am_a_container;done
root        306      0  1 05:41 pts/0    00:00:00 /bin/bash
root        322      1  0 05:41 ?        00:00:00 /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 2
root        323    306  0 05:41 pts/0    00:00:00 ps -ef
[root@3274412d88ca /]#exit							#退出容器，仅退出我们自己的bash窗口

```

小结：attach是直接进入容器启动命令时的终端，不会启动新的进程；exec则是在容器里面打开新的终端，会启动新的进程；一般建议使用exec进入容器。

# 如何run起来dockerfile

编写好 Dockerfile 后，您可以通过以下步骤将其构建成一个 Docker 镜像并运行起来：

1. 使用命令行终端进入到包含 Dockerfile 的目录中。
2. 使用以下命令构建 Docker 镜像：

```
docker build -t image_name:tag .
```

其中，image_name 是您为镜像指定的名称，tag 是版本标签，. 表示当前目录。

1. 构建完成后，您可以使用以下命令运行 Docker 镜像：

```
docker run -d -p host_port:container_port image_name:tag
```

其中，host_port 是您想要映射到主机的端口，container_port 是 Docker 容器内部运行的端口。

1. 您可以通过以下命令查看正在运行的容器：

docker ps

# 

# Dockerfile 的常用命令如下： 

⚫ FROM：继承基础镜像 

⚫ MAINTAINER：镜像制作作者的信息，已弃用，使用LABEL替代 

⚫ LABEL：k=v形式，将一些元数据添加至镜像 

⚫ RUN：用来执行shell命令 

⚫ EXPOSE：暴露端口号 

⚫ CMD：启动容器默认执行的命令，会被覆盖 

⚫ ENTRYPOINT：启动容器真正执行的命令，不会被覆盖 

⚫ ENV：配置环境变量 

⚫ ADD：复制文件到容器，一般拷贝文件，压缩包自动解压 

⚫ COPY：复制文件到容器，一般拷贝目录 

⚫ WORKDIR：设置容器的工作目录 

⚫ USER：容器使用的用户 

⚫ ARG：设置编译镜像时传入的参数

​                      

### 启动 docker-compose 文件

要启动 docker-compose.yml 文件中定义的服务，可以在包含该文件的目录下运行以下命令：

```
Bash
1docker-compose up -d
```

这里 -d 参数表示以后台模式启动服务。如果你想在前台运行服务并看到输出日志，可以省略 -d。

### 关闭 docker-compose 文件中的服务

要停止并关闭 docker-compose.yml 文件中定义的所有服务，可以在相同目录下运行：

```
Bash
docker-compose down
```

默认情况下，docker-compose down 会停止所有服务、删除网络、并移除由 docker-compose up 创建的容器和网络。如果你只想停止服务而不移除容器、网络等，可以使用 docker-compose stop 命令：

```
Bash1docker-compose stop
```

docker-compose stop 命令仅停止服务，但保留容器，这样你可以在以后使用 docker-compose start 命令快速重启服务：

```
Bash
1docker-compose start
```

记得在执行这些命令之前，确保你的工作目录正确，即你处于包含 docker-compose.yml 文件的目录中。