---
layout: post
title: Dockerfile与docker-compose详解
categories: docker
description: Dockerfile,docker-compose介绍与常用命令
keywords: docker Dockerfile docker-compose 工具
---
## 引言
换东家了，发现这一年学了很多前端的东西，但对整套Web开发跟系统架构还不是很熟，以前的话有专门的运维，所以对于工具，服务器之类的基本没去了解过，过来这边以后发现除了前端以外，一些基本的工具及部署还是要明白的。  
没办法，开始恶补吧，不过也挺好的，这段时间正好心情不是很好，投入学习也是一种方式吧。目前这边前端需要接触的主要是以下几个技术。

- 前端框架：AngularJS
- 容器：docker
- 代理服务器：nginx
- 后端：python(Flask)
- 数据库：mongoDB  

打算一个个都写一下自己在学习过程中的感悟吧，也当是复习巩固了，但可能除了前端外不会纠结的特别深。

## 知识准备
这里就不在复述什么是Docker以及镜像，容器这些概念了，如果还不了解这些可以[点击这里](https://www.widuu.com/docker/index.html)

## Dockerfile
Dockerfile是我们用来定制镜像的文件。看过上面的文章可以知道镜像的定制是同分层来组成的，当我们执行一个操作的时候实际上是新建一层，并且进行了隔离。但这Docker的透明度跟臃肿的问题，而解决这个问题的方法就是Dockerfile。

**Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。**  
```
FROM nginx
RUN echo '<h1>This is your first Dockerfile</h1>' /usr/share/nginx/html/index.html
```

这是一个最简单的Dockerfile定制文件，它代表的意思是:  
- 基础镜像是nginx
- 执行shell命令并打印出相应的信息

定制镜像，顾名思义就是在基础镜像的上进行定制，所以我们必须要指定一个需要定制的基础镜像，这也导致了 **FROM** 关键字 **必须是Dockerfile文件的首行**。这里Docker会在Docker Store里面寻找对应的镜像，也可以自己制定基础镜像的获取地址。  
如果你想不基于任何镜像，定制一个纯空白的镜像可以使用scratch,这个镜像是一个虚拟的概念，它并不存在，如下:

```  
FROM scratch
...
```
### Dockerfile 类型
既然希望通过Dockerfile自动执行某些操作，那必然需要执行命令，RUN可以像在命令行中输入命令一样执行。它有如下两种形式：

1.shell(命令行)模式  
这种模式就像上面代码中写的一样，直接执行shell指令了   
2.exec模式  
RUN ["可执行文件", "参数1", "参数2"]  
这种模式就像我们代码中的函数一样。

接下来我们开始创建这个镜像，在Dockerfile文件目录下输入：  
```
docker build -t nginxdf .
```
通过控制台，我们可以看到我们的定制镜像出来了。  
**注意：**  
仔细看docker命令会发现在最后一个``` .```符号来表示当前目录，这是告诉docker它执行的上下文环境。  
>Docker 在运行时分为 Docker 引擎（也就是服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 Docker Remote API，而如 docker 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种 docker 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。也因为这种 C/S 设计，让我们操作远程服务器的 Docker 引擎变得轻而易举。  
>当构建的时候，用户会指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件

这段话非常重要，所以千万不要把Dockerfile放到你的根目录下进行编译，那整个镜像会把你的磁盘都打包了。。。

### 指令集合
下面是一些常用的Dockerfile指令：  
- COPY 复制文件
- ADD 复制文件（比COPY强大）
- CMD 容器启动命令
- ENTRYPOINT 入口点
- ENV 设置环境变量
- ARG 构建参数
- VOLUME 定义匿名卷
- EXPOSE 暴露端口
- WORKDIR 指定工作目录
- USER 指定当前用户
- HEALTHCHECK 健康检查
- ONBUILD 这个比较特殊,这个指令后面会跟着其他指令，并且在当前镜像创建时，后面的指令并不会执行，只有再以当前镜像为基础镜像定制新的镜像时才会执行。可以理解为父亲为儿子准备的指令 

指令具体含义可可以看[这里](http://seanlook.com/2014/11/17/dockerfile-introduction/)，就不赘述了。

最后我们看一个实际项目中的Dockerfile:  
```
FROM virtualnet.host.name.com/library/nginx:v1
COPY ./code /usr/share/nginx/html

RUN rm /etc/nginx/conf.d/*
COPY ./conf/nginx.conf /etc/nginx
COPY ./conf/geo.conf /etc/nginx/conf.d

COPY ./conf/default.conf /tmp/default.conf
COPY ./conf/default.local.conf /tmp/default.local.conf
COPY ./conf/default.local-test.conf /tmp/default.local-test.conf
COPY ./conf/default.product.conf /tmp/default.product.conf

ADD ./scripts/start.sh /sbin/entrypoint.sh

CMD ["/sbin/entrypoint.sh"]

```

现在是不是感觉，看着玩意终于看懂了呢~

## docker-compose  
Docker-compose使用来定义和运行复杂Docker的工具，它可以实现Doker容器集群的快速编排，比如在一个Web项目中,除了Web服务器本身，一般还需要后端的数据库服务容器，大吞吐量的应用甚至还要加上负载均衡的容器。  
安装指南：(http://www.itmuch.com/docker/20-docker-compose-install/)  
docker-compose允许通过docker-compose.yml(YAML格式)我们定义一组相关联的应用容器为一个项目。
这里的服务(service)和应用(project)指的是：  
- 服务：一个应用容器，也可以是多个运行相同镜像的实例。比如我们的Web服务容器，数据库服务容器，等等。
- 项目：一组服务（应用容器）关联而成的组合体。  

这两个概念会贯穿整个docker-compose，所以一定要好好理解一下。  


### docker-compose.yml  
docker-compose.yml 是YAML格式的文件，它是整个compose的核心。我们来看一个简单的服务定义。  
```
version:'3'
services:
    web:
        build:./myweb/
        ports:
        - "8000:8000"
    redis:
        image:"redis:3.0.7"
```  
这个项目包含了两个服务：  
web:利用当前目录myweb下的Dockerfile构建相应的镜像。
并且指定容器的暴露端口8000映射到主机端口8000上。  
redis:使用Docker store中redis镜像构建相应的容器，版本是3.0.7。  

### 配置
**build**  
指定 Dockerfile 所在文件夹的路径（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。 Compose 将会利用它自动构建这个镜像，然后使用这个镜像。  
```
version: '3'
services:

  webapp:
    build: ./dir
```  
你也可以使用 context 指令指定 Dockerfile 所在文件夹的路径。

使用 dockerfile 指令指定 Dockerfile 文件名。

使用 arg 指令指定构建镜像时的变量。
```
version: '3'
services:

  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```  
**cap_add, cap_drop**  
指定容器的内核能力（capacity）分配。

例如，让容器拥有所有能力可以指定为：
```
cap_add:
  - ALL
```  

**command**  
```
command: echo "hello world"
```
覆盖容器启动后默认执行的命令。  

**cgroup_parent**  
指定父 cgroup 组，意味着将继承该组的资源限制。

例如，创建了一个 cgroup 组名称为 cgroups_1。
```
cgroup_parent: cgroups_1
```  

**container_name**
指定容器名称。默认将会使用 项目名称_服务名称_序号 这样的格式  

注意: 指定容器名称后，该服务将无法进行扩展（scale），因为 Docker 不允许多个容器具有相同的名称。
```
container_name: docker-web-container
```

**devices**  
指定设备映射关系。
```
devices:
  - "/dev/ttyUSB1:/dev/ttyUSB0"
```

**depends_on**  
```
version:'3'
services:
    web:
        build:./myweb/
        depends_on:
            - redis
        ports:
        - "8000:8000"
    redis:
        image:"redis:3.0.7"
```  
解决容器的依赖、启动先后的问题。上面例子中会先启动 redis 再启动 web  


**dns**  
自定义 DNS 服务器。可以是一个值，也可以是一个列表。  
```
dns: 8.8.8.8

dns:
  - 8.8.8.8
  - 114.114.114.114
```

**dns_search**  
配置 DNS 搜索域。可以是一个值，也可以是一个列表。  
```
dns_search: example.com

dns_search:
  - domain1.example.com
  - domain2.example.com
```

**tmpfs**  
挂载一个 tmpfs 文件系统到容器。  
```
tmpfs: /run
tmpfs:
  - /run
  - /tmp
```

**env_file**  
从文件中获取环境变量，可以为单独的文件路径或列表。

如果通过 docker-compose -f FILE 方式来指定 Compose 模板文件，则 env_file 中变量的路径会基于模板文件路径。

如果有变量名称与 environment 指令冲突，则按照惯例，以后者为准。

环境变量文件中每一行必须符合格式，支持 # 开头的注释行。

```
env_file: .env

env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```

**environment**  
设置环境变量。你可以使用数组或字典两种格式。

只给定名称的变量会自动获取运行 Compose 主机上对应变量的值，可以用来防止泄露不必要的数据。  
```
environment:
  RACK_ENV: development
  SESSION_SECRET:

environment:
  - RACK_ENV=development
  - SESSION_SECRET
```
如果变量名称或者值中用到 true|false，yes|no 等表达 布尔 含义的词汇，最好放到引号里，避免 YAML 自动解析某些内容为对应的布尔语义。这些特定词汇，包括
```
y|Y|yes|Yes|YES|n|N|no|No|NO|true|True|TRUE|false|False|FALSE|on|On|ON|off|Off|OFF
```

**expose**  
暴露端口，但不映射到宿主机，只被连接的服务访问。

仅可以指定内部端口为参数  
```
expose:
 - "3000"
 - "8000"
```

**external_links**
链接到 docker-compose.yml 外部的容器，甚至并非 Compose 管理的外部容器。
```
external_links:
 - redis_1
 - project_db_1:mysql
 - project_db_1:postgresql
```

**extra_hosts**  
类似 Docker 中的 --add-host 参数，指定额外的 host 名称映射信息。  
```
extra_hosts:
 - "googledns:8.8.8.8"
 - "dockerhub:52.1.157.61"
```  
会在启动后的服务容器中 /etc/hosts 文件中添加如下两条条目。  
```
8.8.8.8 googledns
52.1.157.61 dockerhub
```  

**healthcheck**
通过命令检查容器是否健康运行。  
```
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
```  

**image**  
指定为镜像名称或镜像 ID。如果镜像在本地不存在，Compose 将会尝试拉取这个镜像。  
```
image: ubuntu
image: orchardup/postgresql
image: a4bc65fd
```  

**labels**  
为容器添加 Docker 元数据（metadata）信息。例如可以为容器添加辅助说明信息。  
```
labels:
  com.startupteam.description: "webapp for a startup team"
  com.startupteam.department: "devops department"
  com.startupteam.release: "rc3 for v1.0"
```  

**links**  
配置日志选项。  
```
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```

**network_mode**  
设置网络模式。使用和 docker run 的 --network 参数一样的值。
```
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```  

**networks**  
配置容器连接的网络。
```
version: "3"
services:

  some-service:
    networks:
     - some-network
     - other-network

networks:
  some-network:
  other-network:
```  

**pid**  
跟主机系统共享进程命名空间。打开该选项的容器之间，以及容器和宿主机系统之间可以通过进程 ID 来相互访问和操作。
```
pid: "host"
```  

**ports**  
暴露端口信息。

使用宿主端口：容器端口 (HOST:CONTAINER) 格式，或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。

*注意：当使用 HOST:CONTAINER 格式来映射端口时，如果你使用的容器端口小于 60 并且没放到引号里，可能会得到错误结果，因为 YAML 会自动解析 xx:yy 这种数字格式为 60 进制。为避免出现这种问题，建议数字串都采用引号包括起来的字符串格式。*  
```
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```  

**secrets**  
存储敏感数据，例如 mysql 服务密码。  
```
version: "3.1"
services:

mysql:
  image: mysql
  environment:
    MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
  secrets:
    - db_root_password
    - my_other_secret

secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```  

**volumes**  
数据卷所挂载路径设置。可以设置宿主机路径 （HOST:CONTAINER） 或加上访问模式 （HOST:CONTAINER:ro）。

该指令中路径支持相对路径。

通过volumes 关键字实现物理主机目录挂载到容器中的功能（同时删除Dockerfile中的ADD指令，不需要创建镜像时将代码打包进镜像，而是通过volums动态挂载，容器和物理host共享数据卷）  
```
volumes:
 - /var/lib/mysql
 - cache/:/tmp/cache
 - ~/configs:/etc/configs/:ro
```  

**ulimits**  
指定容器的 ulimits 限制值。

例如，指定最大进程数为 65535，指定文件句柄数为 20000（软限制，应用可以随时修改，不能超过硬限制） 和 40000（系统硬限制，只能 root 用户提高）。  
```
 ulimits:
    nproc: 65535
    nofile:
      soft: 20000
      hard: 40000
```
**sysctls**  
配置容器内核参数。  
```
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0

sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```  

**stop_signal**  
设置另一个信号来停止容器。在默认情况下使用的是 SIGTERM 停止容器。  
```
stop_signal: SIGUSR1
```  

### compose命令  
compose命令默认要在docker-compose.yml文件下执行。在控制台中输入：
```
docker-compose up
```  
该命令会在当前目录下查找docker-compose.yml文件，并启动文件中所有服务。  
在我们实际开发中，可能需要有不同的环境对应不同的配置，比如：  
docker-local-compose.yml 本地   
docker-test-compose.yml  测试  
docker-compose.yml  生产  

这时候只要加入-f来指定需要执行的是哪个具体文件就可以了  
```
docker-compose -p mybase_web_1 -f docker-compose.local.yml up -d myweb
```  

具体命令的含义，再开一章来写吧。