---
layout: post
title: 第007章-Dockerfile
categories: [Docker 容器]
description: 
keywords: Dockerfile.md
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Dockerfile

## Introduction

Dockerfile是一个文本文件，里面包含了若干条指令，每条指令描述了构建镜像的细节。简单来说，它就是由一系列指令和参数构成的脚本文件，从而构建出一个新的镜像文件。



**编写规范如下：**

①、每条指令（每行开头关键字）都必须是大写字母；

②、执行顺序是按照编写顺序从上到下；

③、# 表示注释；



## Grammar

| Grammar    | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| FROM       | 指定基础镜像；                                               |
| MAINTAINER | 设置镜像作者；                                               |
| WORKDIR    | 设置构建镜像的工作目录，如果目录不存在则自动创建；           |
| ADD        | 与COPY类似，都是将文件从docker context拷贝到镜像，不同的是当文件是归档类型（tar、tar.gz、zip等）时，会自动解压到目标路径； |
| COPY       | 将文件从docker context拷贝到镜像；                           |
| RUN        | 在容器中运行指定的指令；                                     |
| ENTRYPOINT | 设置容器启动时运行的命令，当设置多条ENTRYPOINT时，只有最后一条生效。 |
| CMD        | 设置容器启动时运行的指令，当设置多条CMD指令时，只有最后一条生效； |
| ENV        | 设置环境变量，并可被后面的指令使用；                         |
| EXPORSE    | 指定容器中进程会监听的端口，docker可将该端口暴露出来；       |
| USER       |                                                              |
| VOLUME     | 将文件或目录设置为volume；                                   |



### FROM

指定基础镜像，放在第一行，其格式如下

 ```dockerfile
FROM <image>    
FROM <image>:<tag>   
FROM <image>:<digest>  
 ```



```dockerfile
#依赖官方基准镜像（centos:lastest）
FROM centos 
# 若想构建一个最小的镜像，不想基于其他任何镜像时，可使用如下方式
FROM scratch
#指定具体版本号
FROM tomcat:9.0-jdk8-openjdk 
```



### MAINTAINER

通常表示镜像来自哪个机构。类似还有比如 LABEL 标签，展示镜像的一些说明信息，不会对镜像有实际影响。



### LABEL

镜像元数据，可以设置镜像的任何元数据，格式如下：

```dockerfile
LABEL <key>=<value> <key>=<value> <key>=<value>
# 或  
LABEL <key>=<value>     
LABEL <key>=<value>     
LABEL <key>=<value>      
```

利用docker inspect [IMAGE_NAME|IMAGE_ID]命令进行查看镜像元素据。

 ```sh
$ docker inspect [imageName|imageId]
 ```



### WORKDIR

用于在容器内设置一个工作目录，通过WORKDIR设置工作目录后，Dockerfile 中其后的命令RUN、CMD、ENTRYPOINT、ADD、COPY等命令都会在该目录下执行。

如果指定路径不存在，该指令也会自动创建该目录。

```dockerfile
WORKDIR /opt/docker/workdir 
```



### COPY

复制文件，主要就是构建镜像时，进行拷贝文件到镜像的指定路径下，格式为：

```dockerfile
COPY <源路径>...  <目标路径>   
COPY ["<源路径1>",...  "<目标路径>"]
```



### ADD

更高级的复制文件，ADD指令和COPY的格式和性质基本一致。但是在COPY基础上增加了一些功能。比如<源路径>可以是一个 URL，这种情况下，Docker引擎会试图去下载这个链接的文件放到<目标路径>去。



### VOLUME

定义匿名卷，创建挂载点，即向基于所构建镜像创始的容器添加卷，一个卷可以存在于一个或多个容器的指定目录，该目录可以绕过联合文件系统，并具有以下功能：

卷可以容器间共享和重用

容器并不一定要和其它容器共享卷

修改卷后会立即生效

对卷的修改不会对镜像产生影响

卷会一直存在，直到没有任何容器在使用它

VOLUME可以将源代码、数据或其它内容添加到镜像中，而又不并提交到镜像中，并使多个容器间共享这些内容。
```dockerfile
VOLUME ["/data"]   
```




### ARG

该命令用于设置构建参数，该参数在容器运行时是获取不到的，只有在构建时才能获取。这也是其和ENV的区别。

```dockerfile
ARG <name>[=<default  value>]    
```



### RUN

在镜像的构建过程中执行特定的命令，并生成一个中间镜像。

```dockerfile
RUN <command>   
# 或者   
RUN ["executable", "param1", "param2"]  
```



第一种后边直接跟shell命令，在linux操作系统上默认 /bin/sh -c；在windows操作系统上默认 cmd /S /C。

第二种是类似于函数调用，可将executable理解成为可执行文件，后面就是两个参数。

 ```dockerfile
# Shell 命令格式
RUN /bin/bash  -c 'source $*HOME*/.bashrc;  echo $*HOME *  
# Exec命令格式（官方推荐）
RUN ["/bin/bash", "-c", "echo  hello"]  
 ```



多行命令不要写多个RUN，原因是Dockerfile中每一个指令都会建立一层.多少个RUN就构建了多少层镜像，会造成镜像的臃肿、多层，不仅仅增加了构件部署的时间，还容易出错。

多个RUN命令可以使用&&符号连接，RUN书写时的换行符是\。



### ENTRYPOINT

容器启动时执行的命令；

ENTRYPOINT 用于给容器配置一个可执行程序。也就是说，每次使用镜像创建容器时，通过 ENTRYPOINT 指定的程序都会被设置为默认程序。ENTRYPOINT 有以下两种形式：

```dockerfile
ENTRYPOINT ["executable", "param1", "param2"]   
ENTRYPOINT command param1 param2  
```



ENTRYPOINT 与 CMD 非常类似，不同的是通过docker run执行的命令不会覆盖 ENTRYPOINT，而docker run命令中指定的任何参数，都会被当做参数再次传递给ENTRYPOINT。**Dockerfile 中只允许有一个 ENTRYPOINT 命令，多指定时会覆盖前面的设置，而只执行最后的ENTRYPOINT 指令。**

docker run运行容器时指定的参数都会被传递给ENTRYPOINT，且会覆盖 CMD 命令指定的参数。如，执行docker run <image> -d时，-d 参数将被传递给入口点。也可以通过docker run --entrypoint重写 ENTRYPOINT 入口点。如：可以像下面这样指定一个容器执行程序：

 ```dockerfile
ENTRYPOINT ["/usr/bin/nginx"] 
 ```



### CMD

容器启动后执行默认的命令或参数，第一种和第二种都是可执行文件加上参数的形式，第三种是shell写法。

**和ENTRYPOINT 命令一样，也是只有最后一个 CMD 命令会被执行，但是如果容器启动时附加指令，则CMD会被忽略。**如果启动时附加命令，则会执行附加的命令(下图附加 ls 命令)，而不执行Dockerfile 中的CMD 命令。也就是说 ENTRYPOINT 指令一定会执行，但是 CMD 指令不一定会执行。



```dockerfile
# 推荐使用 Exec 格式。
CMD ["executable","param1","param2"]   
CMD ["param1","param2"]   
CMD command param1 param2  
```



这里边包括参数的一定要用双引号，就是双引号",不能是单引号。千万不能写成单引号。原因是参数传递后，docker解析的是一个JSON array。

 ```dockerfile
CMD [ "sh", "-c", "echo  $*HOME*" ]   
CMD [ "echo", "$*HOME*"  ]  
 ```



#### RUN & CMD & ENTRYPOINT

- RUN: 在镜像构建时执行命令，比如 RUN yum -y install vim；
- ENTRYPOINT:容器启动时执行的命令；
- CMD:容器启动后执行默认的命令或参数；



1、RUN

执行命令并创建新的镜像层，通常用于镜像构建中软件包安装等操作。

2、CMD

为容器指定默认的启动执行命令，此命令会在容器启动且docker run没有指定其他命令时运行，也就是说CMD中的命令是可以在docker run中被其他命令所覆盖的。

3、ENTRYPOINT

和CMD很像，不同的是ENTRYPOINT指定的命令一定会被执行，即使docker run中指定了其他命令，这也就意味着ENTRYPOINT可以用于让容器以应用程序或服务的形式运行。



### ENV

设置环境变量，之后的命令都可以用此变量进行赋值，格式如下：

```dockerfile
ENV <key> <value>   
ENV <key>=<value>  
...     
```



尽量使用环境常量，这样可以提高程序的可维护性。



### EXPOSE

为镜像设置监听端口，容器运行时会监听改端口。

```dockerfile
EXPOSE <port> [<port>/<protocol>...]  
# 为镜像设置监听端口，同时，也能指定协议名   
EXPOSE 80/udp  
```




### USER

用于指定运行镜像所使用的用户，使用USER指定用户后，Dockerfile 中其后的命令RUN、CMD、ENTRYPOINT都将使用该用户。镜像构建完成后，通过docker run运行容器时，可以通过-u参数来覆盖所指定的用户。

```dockerfile
USER okong
```



## Build Process

```dockerfile
# 声明所使用的基础镜像
FROM ubuntu
# 指定构建镜像时的工作目录，后续命令都是基于此目录的，如果不存在则创建
WORKDIR /opt/soft/jdk
# 将jdk包复制并解压到/opt/soft/jdk目录下
ADD jdk-8u231-linux-x64.tar.gz /opt/soft/jdk/
# 设置环境变量
ENV JAVA_HOME=/opt/soft/jdk/jdk1.8.0_231
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV PATH=$JAVA_HOME/bin:$PATH
```



Docker镜像的创建过程：

1）Step 1：执行FROM，将ubuntu作为基础镜像，这里将tag为latest的ubuntu镜像拉取下来，镜像ID为**fb52e22af1b0**；

2）Step 2：执行WORKDIR，设置工作目录，这里其实是首先启动ID为fb52e22af1b0的临时容器，然后在其中创建/opt/soft/jdk目录，创建完毕后删除此临时容器，并将此容器保存为ID是**e5b5ee6e0f89**的镜像。

3）Step 3：执行ADD，将jdk-8u231-linux-x64.tar.gz拷贝并解压到/opt/soft/jdk目录下，并保存为ID是**f22f968c43cd**的镜像。

4）Step 4 ～ Step 6：执行ENV，先开启临时容器，然后设置环境变量，最后保存为镜像。

5）最终镜像构建成功，镜像ID为**cb70bf70e1e9**，tag为ubuntu-jdk:latest。



### Image Layer

在Dockerfile中，它的每条指令都会创建一个镜像层，执行操作后再将此镜像层保存。我们通过docker history可以很清楚看到这一点：

```sh
$ sudo docker history ubuntu-jdk
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
cb70bf70e1e9   22 minutes ago   /bin/sh -c #(nop)  ENV PATH=/opt/soft/jdk/jd…   0B
f04605d6e9c2   22 minutes ago   /bin/sh -c #(nop)  ENV CLASSPATH=.:/opt/soft…   0B
876227810405   22 minutes ago   /bin/sh -c #(nop)  ENV JAVA_HOME=/opt/soft/j…   0B
f22f968c43cd   22 minutes ago   /bin/sh -c #(nop) ADD file:610ae1ffb70fff692…   403MB
e5b5ee6e0f89   22 minutes ago   /bin/sh -c #(nop) WORKDIR /opt/soft/jdk         0B
fb52e22af1b0   12 days ago      /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>      12 days ago      /bin/sh -c #(nop) ADD file:d2abf27fe2e8b0b5f…   72.8MB
```



```sh
$ sudo  docker history ubuntu
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
fb52e22af1b0   12 days ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>      12 days ago   /bin/sh -c #(nop) ADD file:d2abf27fe2e8b0b5f…   72.8MB
```



ubuntu-jdk相比于ubuntu镜像多了很多层，这些层就是在执行WORKDIR、ADD、ENV指令时产生的。

可以通过`docker image ls -a`命令查看中间层镜像，中间层镜像一般都是会被重复利用的，无需重新构建，这样便能提升镜像构建效率。



## Example

### Simple Dockerfile

下面是一个简单的Dockerfile文件，可以通过build命令将其构建成一个docker镜像。

```dockerfile
FROM nginx
LABEL author="作者：xmm"
LABEL version="版本：v0.1"
LABEL desc="说明：修改nginx首页提示"
RUN echo 'hello,oKong' > /usr/share/nginx/html/index.html
CMD ["nginx", "-g", "daemon off;"]
```



### JDK Dockerfile

```dockerfile
# 声明所使用的基础镜像
FROM ubuntu
# 指定构建镜像时的工作目录，后续命令都是基于此目录的，如果不存在则创建
WORKDIR /opt/soft/jdk
# 将jdk包复制并解压到/opt/soft/jdk目录下
ADD jdk-8u231-linux-x64.tar.gz /opt/soft/jdk/
# 设置环境变量
ENV JAVA_HOME=/opt/soft/jdk/jdk1.8.0_231
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV PATH=$JAVA_HOME/bin:$PATH
```