# 重启Docker命令

sudo service docker restart   重新启动Docker命令

iptables -I INPUT 1 -p tcp --dport 80 -j ACCEPT     开启防火墙80端口访问权限

------





# **Docker第一批笔记**

## **Docker**为什么出现？

一款产品：开发--上线 两套环境！应用环境，应用配置！
开发---运维。问题：我在我的电脑上可以运行！版本更新等，导致服务不可用！对于运维来说，考验就十分巨大！

环境配置十分麻烦，每一个机器都要配置环境（集群Redis、ES、Hadoop。。。）费时费力。
发布一个项目（ jar+（Redis  MySql   jdk ES））项目能不能都带上环境安装打包（Docker干的事情）
之前在服务器配置一个应用的环境Redis MySQL  JDK ES Hadoop，配置超级麻烦，不能够跨平台Windows，最后发布到Linux。
传统：开发给jar包，运维来做！
现在：开发打包部署上线，一套流程做完！
Docker给以上的问题提出了解决方案！





Docker历史：
2010年，美国成立一家公司dotCloud，做一些pass的云计算服务，LXC有关的容器技术。
他们将自己的技术（容器化技术）命名就是Docker
Docker刚刚诞生的时候，没有引起行业的注意，dotCloud，就活不下去！

开源
开发源代码
2013年，Docker开源！
Docker越来越多得人发现了docker的优点，Docker每个月都会更新一个版本。
2014年4月9日，Docker1.0发布！

Docker为什么这么火？十分的轻巧！

在容器技术出来之前，我们都是使用虚拟机技术！
虚拟机在window中装一个Vmware，通过这个软件我们可以虚拟出来一台或者多台电脑！笨重！
虚拟机也是属于虚拟化技术，Docker容器技术，也是一种虚拟化技术!

vm:linux centos原生镜像（一个电脑）  隔离，需要开启多个虚拟机
docker：隔离  镜像（最核心的环境 4m+jdk+mysql）十分的小巧，运行镜像就可以了，小巧，几个M  KB  秒级启动！



聊聊Docker
Docker基于Go语言开发！开源项目
官网：https://www.docker.com/
文档地址：https://docs.docker.com/   Docker的文档超级详细的！
仓库地址：https://hub.docker.com/    可以把我们的镜像发布上去

------



## Docker能干嘛？

虚拟化技术：
1.资源占用十分多
2.冗余步骤多
3.启动很慢
容器技术：
容器技术不是模拟的一个完整的操作系统

比较Docker和虚拟机技术的不同：
传统的虚拟机，虚拟出一条硬件，运行一个完整的操作系统，然后在这个系统上安装和运行软件。
容器内的应用直接运行在宿主机的内核中，容器是没有自己的内核的，也没有虚拟机模拟我们的硬件，所以就轻便了。
每个容器间是互相隔离，每个容器都有一个属于自己的文件系统，互补影响。

应用更快速的交付和部署
传统：一堆帮助文档，安装程序。
Docker：打包镜像发布测试，一键运行

更便捷的升级和扩缩容
使用了Docker之后，我们部署应用就和搭积木一样！
项目打包为一个镜像，扩展  服务器A 出问题了！ 服务器B一键扩展

更简单的系统运维
在容器化之后，我们的开发，测试环境都是高度一致的

更高效的计算资源利用
Docker是内核级别的虚拟化，可以再一个物理机上运行很多的容器实例！服务器的性能可以被压榨到极致！

------



## Docker安装

Docker的基本组成：
镜像（image）：
docker镜像就好比是一个模板，可以通过这个模板创建容器服务，tomcat镜像===》run====》tomcat01容器（提供服务）通过这个镜像可以创建多个容器（最终服务运行或者项目运行就是在容器中）。
容器（container）：
Docker利用容器技术，独立运行一个或者一组应用，通过镜像来创建的。
启动、停止、删除、基本命令！
目前就可以把这个容器理解为就是一个简易的linux系统
仓库（repository）
仓库就是存放镜像的地方
仓库分为公有仓库和私用仓库！
Docker Hub（默认是国外）
阿里云...都有容器服务器（配置镜像加速！）

安装Docker：
环境准备：
1.需要会一点点的linux的基础
2.CentOS7
3.我们使用Xshell连接远程服务器进行操作！

安装步骤：
1.卸载旧版版
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
2.安装需要的安装包
yum install -y yum-utils

3.设置镜像仓库，基本上都是使用阿里云内部仓库。等会还要配置阿里云镜像加速可以使这个仓库更快。
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo（默认是国外的，死慢死慢的，千万不要用。百度搜索 Docker的阿里云镜像地址）
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo (阿里云镜像地址)，百度搜索docker阿里云镜像地址，一搜一大堆。

3.更新yum软件包索引
yum makecache fast

3.安装docker  下面下载的是docker-ce 社区版       还有ee企业版
yum install docker-ce docker-ce-cli containerd.io

4.启动docker
systemctl start docker

5.查看docker是否安装成功
docker --version

6.测试Docker
docker run hello-world
  Unable to find image 'hello-world:latest' locally
  6.1命令执行之后会在docker寻找镜像，但是没有寻找到
  latest: Pulling from library/hello-world
  6.2就会远程拉去镜像，拉取官方的 library/下面的hello-world
  		            0e03bdcc26d7:PullcompleteDigest:sha256:6a65f928fb91fcfbc963f7aa6d57c8eeb4	26ad9a20c7ee045538ef34847f44f1
Status: Downloaded newer image for hello-world:latest
  6.3拉取完成后就会有这样的一个签名信息
  Hello from Docker!
  6.4然后通过run运行起来了，最后打印一句话Hello from Docker!表示运行成功

7.查看一下下载的这个 hello-wrld镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        4 months ago        13.3kB

了解：卸载docker
yum remove docker-ce docker-ce-cli containerd.io  卸载依赖
rm -rf /var/lib/docker                删除资源
 /var/lib/docker             docker磨人的工作路径

阿里云镜像加速
容器与镜像服务---》镜像加速器   -----》选择系统
1.sudo mkdir -p /etc/docker
2.sudo tee /etc/docker/daemon.json <<-'EOF'
   {
    "registry-mirrors": ["https://wls87jpi.mirror.aliyuncs.com"]
    }
    EOF
3.sudo systemctl daemon-reload
4.sudo systemctl restart docker
***在终端一步步指向上面四步就把阿里云镜像加速配置成功了！***



底层原理
Docker是怎么工作的？
Docker是一个Client-Server结构的系统，Docker的守护进程运行在主机上。通过Socket从客户端访问！
DockerServer接收到Docker-Client的指令，就会执行这个命令

Docker为什么比VM快？
1.Docker有着比虚拟机更少的抽象层
2.docker利用的是宿主机的内核，vm需要是Guest OS(自己搭建一个这样的环境)。
新建一个容器的时候，docker不需要像虚拟机一样重新加载一个操作系统，避免引导。虚拟机是加载Guest OS这种级别的，而docker是利用宿主机的操作系统，省略了这个复杂的过程  秒级！

------



## Docker的常用命令

帮助命令
docker version   显示docker的版本信息 
docker info       显示docker的系统信息，包括镜像和容器的数量
docker 命令 --help   万能命令

### 镜像命令

  docker images   查看所有本地主机上的镜像
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  hello-world         latest              bf756fb1ae65        4 months ago        13.3kB
   REPOSITORY  镜像的仓库源
   TAG                 镜像的标签
   IMAGE ID       镜像的id
   CREATED       镜像的创建时间
   SIZE               镜像的大小 

-a, --all             列出所有镜像
  -q, --quiet      只显示镜像的id

 docker search  搜索镜像
NAME                              DESCRIPTION                                     STARS(收藏数)               OFFICIAL            AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   9544                [OK]                
mariadb                           MariaDB is a community-developed fork of MyS…   3466                [OK]
可选项，通过收藏来过滤

--filter=STARS=3000   搜索出来的镜像就是STAR等于3000的
NAME                DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql               MySQL is a widely used, open-source relation…   9544                [OK]                
mariadb             MariaDB is a community-developed fork of MyS…   3466   

docker pull 镜像名[:tag]                下载镜像

[root@192 ~]# docker pull mysql
Using default tag: latest                        如果不写tag，默认就是latest，表示最新版本
latest: Pulling from library/mysql          
afb6ec6fdc1c: Pull complete           分层下载，  docker image的核心，联合文件系统
0bdc5971ba40: Pull complete     
97ae94a2c729: Pull complete 
f777521d340e: Pull complete 
1393ff7fc871: Pull complete 
a499b89994d9: Pull complete 
7ebe8eefbafe: Pull complete 
597069368ef1: Pull complete 
ce39a5501878: Pull complete 
7d545bca14bf: Pull complete 
211e5bb2ae7b: Pull complete 
5914e537c077: Pull complete 
Digest: sha256:a31a277d8d39450220c722c1302a345c84206e7fd4cdb619e7face046e89031d     签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest            真实地址

docker.io/library/mysql:latest     等价于   docker pull mysql

指定版本下载
[root@192 ~]# docker pull mysql:5.7
5.7: Pulling from library/mysql
afb6ec6fdc1c: Already exists 
0bdc5971ba40: Already exists 
97ae94a2c729: Already exists 
f777521d340e: Already exists 
1393ff7fc871: Already exists 
a499b89994d9: Already exists 
7ebe8eefbafe: Already exists 
4eec965ae405: Pull complete 
a531a782d709: Pull complete 
270aeddb45e3: Pull complete 
b25569b61008: Pull complete 
Digest: sha256:d16d9ef7a4ecb29efcd1ba46d5a82bda3c28bd18c0f1e3b86ba54816211e1ac4
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7

docker rmi  删除镜像
docker rmi -f 容器id          强制删除指定的镜像
docker rmi -f 容器id  容器id 容器id         删除多个镜像
docker rmi -f $(docker images -aq)    删除全部镜像
docker tag 镜像id 镜像名称:版本    修改镜像名称版本，会产生一个新的镜像

------



### 容器命令

说明：我们有了镜像才可以创建容器，linux，下载一个centos镜像来测
docker pull centos





新建容器并启动
docker run [可选参数] images
参数说明
--name="Name"  容器名字 tomcat01   tomcat02，用来区分容器
-d                          后台方式运行
-it                      使用交互方式运行，进入容器查看内容
-p                          指定容器的端口  -p 8080:8080
      -p ip:主机端口:容器端口
      -p：主机端口:容器端口             (映射到容器端口)常用
      -p：容器端口
       容器端口
-P(大p)                  随机指定端口

测试：docker run -it centos /bin/bash     启动并进入容器
          exit            容器停止运行并退出
         ctrl+p+q    容器不停止退出

列出所有运行的容器
docker ps  当前运行的
docker ps -a  曾经运行的
docker ps -n 显示最近创建的容器
docker ps -q 只显示容器的编号

删除容器
docker rm 容器id     删除指定容器         不能删除正在运行的容器    如果要强制删除用rm -f
docker rm -f $(docker ps -aq)   删除所有容器 
docker ps -a -q|xargs docker rm  删除所有的容器

启动和停止容器的操作
docker start 容器id         启动容器
docker restart 容器id       重启容器
docker stop 容器id          停止当前正在运行的容器
docker kill 容器id             强制停止当前容器

常用其他命令
docker run -d 镜像名
问题  docker ps  发现centos停止了。常见的坑，docker容器使用后台运行，就必须要有一个前台进程，docker发现没有应用了，就会自动停止了。
nginx:容器启动后，发现自己没有提供服务，就会立刻停止，就是没有程序了

查看日志命令
docker logs -f -t --tail 容器 ，没有日志
自己编写一段shell脚本 "whlie true;do echo kuangshen;slepp 1;done "
-tf   显示日志
--tail number   显示日志条数
docker logs -tf --tail 10 容器id
tail -f logs/catalina.out    一直显示tomcat日志

查看容器中的进程信息
docker top 容器id

查看镜像的元数据
 docker inspect 容器id     比较重要

进入当前正在运行的容器
我们通常容器都是使用后台方式运行的，这个时候需要进入容器修改一些配置
命令
docker exec -it 容器id /bin/bash     进入容器后开启一个新的终端，可以再里面操作(常用)

首先，docker run -it centos 的意思是，为centos这个镜像创建一个容器

```shell
         -it就等于 -i和-t，这两个参数的作用是，为该docker创建一个伪终端，这样就可以进入到容器的交互模式？（也就是直接进入到容器里面）
         后面的/bin/bash的作用是表示载入容器后运行bash ,docker中必须要保持一个进程的运行，要不然整个容器启动后就会马上kill itself，这个/bin/bash就表示启动容器后启动bash。
```

docker attach 容器id          进入容器正在执行的终端，不会启动新的进程

从容器内拷贝文件到主机上
docker cp 容器id:容器内路径 . 

------



## Docker安装Nginx

```shell
# 1.搜索镜像 search 最好去docker搜索，可以看到帮助文档信息
# 2.下载镜像 publl [root@192 home]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat              9.0                 1b6b1fe7261e        12 days ago         647MB
tomcat              latest              1b6b1fe7261e        12 days ago         647MB
nginx               latest              9beeba249f3e        13 days ago         127MB
centos              latest              470671670cac        4 months ago        237MB

# -d 后台运行
# --name 给容器命名
# -p 宿主机端口，容器内部端口
[root@192 home]# docker run -d --name nginx01 -p 3344:80 nginx
efaeb853b6087fc7db211238f4af61ded62bde15934fc1e882116ac7d8969841
[root@192 home]# docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
efaeb853b608        nginx               "nginx -g 'daemon of…"   12 seconds ago      Up 8 seconds        0.0.0.0:3344->80/tcp   nginx01
[root@192 home]# curl localhost:3344

# 3.运行测试
[root@192 home]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat              9.0                 1b6b1fe7261e        12 days ago         647MB
tomcat              latest              1b6b1fe7261e        12 days ago         647MB
nginx               latest              9beeba249f3e        13 days ago         127MB
centos              latest              470671670cac        4 months ago        237MB
[root@192 home]# docker run -d --name nginx01 -p 3344:80 nginx
efaeb853b6087fc7db211238f4af61ded62bde15934fc1e882116ac7d8969841
[root@192 home]# docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
efaeb853b608        nginx               "nginx -g 'daemon of…"   12 seconds ago      Up 8 seconds        0.0.0.0:3344->80/tcp   nginx01
[root@192 home]# curl localhost:3344
# 进入容器
[root@192 home]# docker exec -it nginx01 /bin/bash
root@efaeb853b608:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
root@efaeb853b608:/# cd /etc/nginx
root@efaeb853b608:/etc/nginx# ls
conf.d	fastcgi_params	koi-utf  koi-win  mime.types  modules  nginx.conf  scgi_params	uwsgi_params  win-utf

```

端口暴露的情况

![1590722566013](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1590722566013.png)

------



## Docker安装Tomcat

```shell
# 官方的使用
docker run -it -rm tomcat:9.0
# 我们之前的启动都是后台，停止了容器之后，容器还是可以查到    docker run -it -rm tomcat:9.0 一般用来测试，用完就自动删除

#下载启动
docker pull tomcat:9.0
#后台运行
docker run -d -p 3355:8080 --name tomcat01 tomcat
#发现问题：1.linux命令少了，2.没有webapps。阿里云镜像的原因，默认是最小的镜像，所有不必要的都剔除掉保证最小可运行的环境。
```

------



## Docker安装es+kibana

```shell
# es暴露的端口很多
# es十分耗内存
# es的数据一般需要放置到安全目录！挂载
# --net somenetwork   网络配置

# 下载启动elasticsearch
docker run -d --name elasticsearch  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.2
# 启动了linux服务器就卡住了，因为非常耗内存  
# 访问一下，测试有没有成功
[root@192 home]# curl localhost:9200
{
  "name" : "1792e1c95fb8",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "ar8r8BAKSQqUCPIHM5lrrw",
  "version" : {
    "number" : "7.6.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
    "build_date" : "2020-03-26T06:34:37.794943Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

#docker stats 查看cpu的状态
```

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1590732766009.png)

```shell
# 增加内存限制，修改配置文件 -e 环境配置修改
docker run -d --name elasticsearch02  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="Xms64m -Xmx
512m" elasticsearch:7.6.2
```

------
## 可视化(基本的管理镜像工具)

- portainer(先用这个)，不是最佳选择

```shell
docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

**什么是portainer**

Docker图形化界面管理工具！提供一个后台面板供我们操作！

```shel
docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

访问测试：http://192.168.64.131:8088/            外网:8088

通过他来访问了

------

# Docker镜像讲解

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需要的所有内容，包括代码、运行时、库、环境变量和配置文件。

所有的应用直接打包docker镜像，就可以直接跑起来！

**如何得到镜像：**

- 从远程仓库下载
- 朋友拷贝给你
- 自己制作一个镜像 DockerFile

## Docker镜像加载原理

> UnionFS(联合文件系统)

我们下载的时候看到的一层层的就是这个！

UnionFS（联合文件系统）：Union文件系统(UnionFS)是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下（unite serveral directories into a single virtual filesystem）。Union文件系统是Docker镜像的基础，镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应有镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

------



## Coommit镜像

```shel
docker commit 提交容器成为一个新的镜像
# 命令和git原理类似
docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名:[TAG]   TAG：版本标签
```

实战测试

```shell
# 1.启动一个默认的tomcat，
# 2.发现这个默认的tomcat是没有webapps应用了，镜像的原因，官方的镜像默认webapps下面是没有文件的！
# 3.我自己拷贝进去了基本的文件
# 4.将我们操作过的容器通过commit提交为一个镜像，我们以后就使用修改过的镜像即可，这就是我们自己的一个修改过的镜像。
```

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1590976502787.png)

```shell
如果想要保存当前容器的状态，就可以通过commit来提交，获得一个镜像。
```



**到了这里才入门Docker**

------

# 容器数据卷

## 什么是数据卷

**Docker的理念回顾**

将应用和环境打包成一个镜像！

数据？如果数据都在容器中，那么我们容器删除，数据就会丢失！==需求：数据可以持久化==
MySQL，容器删了，数据库就删了，就该跑路了！==需求：MySQL数据可以储存在本地！==

容器之间可以有一个数据共享的技术！Docker容器中产生的数据，同步到本地！
这就是卷技术！目录的挂载，将容器内的目录，挂载到linux上面！

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1590980705151.png)

**好处：容器的持久化和同步操作。多个容器之间也可以数据共享。**

------



## 使用数据卷

> 方式一：直接使用命令挂载  -v

```shell
docker run -it -v 主机目录，容器内的目录

# 测试
[root@192 home]# docker run -it -v /home/ceshi:/home centos /bin/bash
# 启动起来时候我们可以通过docker inspect 容器id 查看详细情况
```

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1590982733717.png)

**测试文件的同步**

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1590983255862.png)

**再来测试**

1.停止容器

2.宿主机上修改文件

3.启动容器

4.容器内的数据依旧是同步的

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1590984238383.png)

**好处：我们以后修改只需要在本地修改即可，容器会自动同步**

------



## 实战：安装MySQL 挂载

问题：MySQL的数据持久化问题！

```shell
#获取镜像
[root@192 home]# docker pull mysql:5.7
#运行容器，需要做数据挂载     #安装启动mysql   需要配置密码的，这是注意点！
#官方测试：docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
#启动我们的mysql
-d 后台运行
-p 端口映射
-v 数据卷挂载
-e 环境配置
--name 容器名字
[root@192 home]# docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

#启动完成之后，我们在本地使用Navicat来连接测试一下
#Navicat连接到服务器的3310 ---- 3310和容器内的3306映射，这个时候我们就可以连接上了！
#在本地测试创建一个数据库，查看一下映射的路径是否正确
```

**就算删除容器，我们挂载到本地的数据卷依旧存在，这就实现了容器数据持久化功能**

------



## 具名挂载和匿名挂载

```shell
#匿名挂载
-v 容器内路径！
docker run -d -P --name nginx01 -v /ect/nginx nginx
#查看所有的volume的情况
[root@192 /]# docker volume ls
DRIVER              VOLUME NAME
local               137f4756cd34d4152896f730c7c2d4e87ee9ca6926092868213d9ec34ded0a71
local               996ed9fbd8a7b6bed0ac5cce054f1488b83069651d3681ccb58c6b02cb5741d0
local               92836b742caa930761c5cd7a570ec7e6e551c3090e6d708610eadf3609539d95
#这里发现，这种就是匿名挂载，我们在-v的时候，只写了容器内的路径，没有写容器外的路径！


#具名挂载
[root@192 home]# docker run -d -P --name nginx03 -v juming-nginx:/etc/nginx nginx
dcad3b459ba771284f4a49896a6284d819b6d3bd5c407a5ffb5f6188ef3a138b
[root@192 home]# docker volume ls
DRIVER              VOLUME NAME
local               30a198c349dfd7939b625cbc27a6155265bdb10d5cac69c4f23c3d92008ab0b9
local               137f4756cd34d4152896f730c7c2d4e87ee9ca6926092868213d9ec34ded0a71
local               996ed9fbd8a7b6bed0ac5cce054f1488b83069651d3681ccb58c6b02cb5741d0
local               juming-nginx
# 通过 -v 卷名:容器内路径
#查看一下这个卷
```

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591001998989.png)

所有的docker容器内的卷，没有指定目录的情况下都在`/var/lib/docker/volumes/xxx/_data`文件下

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591002525987.png)

我们通过具名挂载可以方便找到我们的数据卷，大多数情况下在使用的`具名挂载`

```shell
# 如何确定是具名挂载还是匿名挂载，还是指定路径挂载！
-v 容器内路径             #匿名挂载
-v 卷名：容器内路径        #具名挂载
-v /宿主机路径:容器内路径   #指定路径挂载
```

**拓展：**

```shell
# 通过-v 容器内路径:ro || rw 改变读写权限
ro readonly    #只读
rw readarite   #可读可写
#一旦设置了容器读写权限，容器对挂载出来的内容就有限定了！
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:rw nginx
#ro 只要看到ro就说明这个路径只能通过宿主机操作，容器内部无法操作
```

------



## 初识DockerFile

DockerFile就是用来构建docker镜像的构建文件！其实就是一段命令脚本！

通过这个脚本可以生成镜像，镜像是一层一层的，脚本是一个个的命令，每个命令都是一层！

```shell
#创建一个dockerfile文件，名字可以随机，建议Dockerfile
#文件中的内容    指令(大写)    参数
FORM centos
VOLUME ["volume01","volume02"]
CMD echo"-----end----"
CMD /bin/bash
#这里的每个命令，都是镜像的一层！
```

```shell
#启动一下自己写的容器
```

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591004897529.png)

这个卷和外部一定有一个同步的目录

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591004978191.png)

查看一下卷挂载的路径

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591005575595.png)

测试一下刚才的目录是否能同步

![1591005748639](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591005748639.png)

这种方式我们未来使用的十分多，因为我们通常会构建自己的镜像！

假设构建镜像时没有挂载卷，要手动镜像挂载 -v 卷名:容器内路径！

------

## 数据卷容器

多个mysql同步数据！

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591006520001.png)
**ps:被挂载的叫做父容器**

```shell
#启动3个容器，通过我们刚才自己写的镜像启动
docker run -it --name docker01 kuangshen/centos:1.0
docker run -it --name docker02 --volumes-from docker01 kuangshen/centos:1.0
docker run -it --name docker03 --volumes-from docker01 kuangshen/centos:1.0
```

![1591065584893](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591065584893.png)

```shell
# volumes-from 实现父子容器数据共享  这里继承的是docker01作为父容器。数据双向拷贝共享
# 有多个容器指向一个父容器，如果父容器被删除，不会影响别的容器数据拷贝共享
docker run -it --name docker02 --volumes-from docker01 kuangshen/centos:1.0

```

**多个mysql实现数据共享**

```shell
docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v
/home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

docker run -d -p 3310:3306 -e MYSQL_ROOT_PASSWORD=123456 --name mysql02 --volumes-from mysql01 mysql:5.7
#以上就实现了两个mysql数据共享了。用这个方法就可以实现多个容器之间数据共享
```

***结论***
容器之间配置信息的传递，数据卷容器的生命周期一直持续到没有容器使用为止。
但是一旦持久化到了本地，这个时候，本地的数据是不会删除的

------

# DockerFile

## DockerFile介绍

dockerfile是用来构建docker镜像的文件！命令参数脚本！
构建步骤:

1. 编写一个dockerfile文件
2. docker build构建成为一个镜像
3. docker run运行镜像
4. docker push发布镜像(DockerHub、阿里云镜像仓库)

查看一下官方是怎么做的:
![1591070070120](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591070070120.png)

![1591070164783](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591070164783.png)

很多官方的镜像都是基础包，很多功能都没有，我们通常会自己搭建自己的镜像！
官方既然可以制作镜像，我们自己也可以！

------



## DockerFile构建过程

**基础知识:**

1. 每个保留关键字(指令)都是必须是大写字母

2. 执行从上到下顺序执行

3. #表示注释

4. 每一个指令都会创建提交一个新的镜像层，并提交
   ![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591071341721.png)

   dockerfile是面向开发的，我们以后要发布项目，做镜像，就需要编写dockerfile文件，这个文件十分简单！DockerFile必须要掌握！

   步骤:开发、部署、运维。。。。缺一不可

   DockerFile:构建文件，定义了一切的步骤，源代码
   Dockerimages:通过DockerFile构建生成的镜像，最终发布和运行的产品
   Docker容器:容器就是镜像运行起来提供服务器

   ------

   

## Docker的指令

```SHELL
FROM         # 基础镜像，一切从这里开始构建
MAINTAINER   #镜像是谁写的，姓名+邮箱
RUN          #镜像构建的时候需要运行的命令
ADD          #步骤:搭建一个有tomcat的镜像，添加一个tomcat压缩包。
WORKDIR      #镜像的工作目录
VOLUME       #挂载卷，挂载的目录
EXPOSE       #暴露端口配置。跟 -p 是一个意思
CMD          #指定这个容器启动的时候要运行的命令,只有最后一个会生效，可被替代
ENTRYPOINT   #指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD      #当构建一个被继承DockerFile，这个时候就会运行ONBUILD指令，触发指令(了解，他没有细说)
COPY         #类似ADD，将我们的文件拷贝到镜像中
ENV          #构建的时候，设置环境变量 跟 -e 是一个意思
```

![1591076402284](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591076402284.png)
![1591076426809](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591076426809.png)
ps：以前我们只会使用别人的，现在我们知道这些指令后，学习自己写镜像。



------



## **实战：制作centOS**

Docker Hub中99%镜像都是从这个基础镜像过来的 FROM scratch,然后配置需要的软件和配置来进行的构建

![1591077494387](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591077494387.png)

> 创建一个自己的centos

```SHELL
# 1.编写DockerFile的文件
FROM centos
MAINTAINER xiangzhiming<xzmadmin@163.com>
ENV MYPATH /usr/local
WORKDIR $MYPATH
RUN yum -y install vim
RUN yum -y install net-tools
EXPOSE 80
CMD echo $MYPATH
CMD echo "------end-------"
CMD /bin/bash 

# 2.通过这个文件构建镜像
# -f dockerfile文件路径
# -t tagged版本信息
# 命令 docker build -f mydockerfile-centos -t mycentos:0.1 .

Successfully built 8b1f83668c68
Successfully tagged mycentos:0.1
# 3.测试运行
```

对比之前的官方的centos
![1591079593619](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591079593619.png)
**我在官方基础上做的镜像：**
![1591079836395](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591079836395.png)

我们可以列出本地进行的变更历史
![1591080095771](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591080095771.png)

> CMD 和 ENTRYPOINT的区别

```shell
CMD           #指定这个容器启动的时候要运行的命令，只有最后一个会生效，会被替代
ENTRYPOINT    #指定这个容器启动的时候要运行的命令，可以追加命令
```

测试cmd

```shell
# 编写dockerfile文件
[root@localhost dockerfile]# vi dockerfile-cmd-test
FROM centos
CMD ["ls","-a"]
# 构建镜像
[root@localhost dockerfile]# docker build -f dockerfile-cmd-test -t cmdtest .
# run运行，发现我们的ls -a生效。  一启动就会执行里面的cmd命令，只有最后一个命令会生效
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
# 想追加一个命令 -l 期望的是返回ls -al，列表的详细数据
docker: Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused "exec: \"-l\": executable file not found in $PATH": unknown.
ERRO[0000] error waiting for container: context canceled 
#cmd的情况下，-l替换了CMD["ld","-a"]命令，-l不是命令，所以报错
```

![1591081979691](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591081979691.png)

测试ENTRYPOINT

```shell
[root@localhost dockerfile]# vi docerfile-cmd-entrypoint
FROM centos
ENTRYPOINT ["ls","-a"]

[root@localhost dockerfile]# docker build -f docerfile-cmd-entrypoint -t entorypoint-test .
Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM centos
 ---> 470671670cac
Step 2/2 : ENTRYPOINT ["ls","-a"]
 ---> Running in 725a5429825f
Removing intermediate container 725a5429825f
 ---> fe69939b1e1f
Successfully built fe69939b1e1f
Successfully tagged entorypoint-test:latest
[root@localhost dockerfile]# docker run fe69939b1e1f
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
#我们的追加命令，是直接拼接在我们的ENTRYPOINT命令的后面
[root@localhost dockerfile]# docker run fe69939b1e1f -l
total 0
drwxr-xr-x.   1 root root   6 Jun  2 07:17 .
drwxr-xr-x.   1 root root   6 Jun  2 07:17 ..
-rwxr-xr-x.   1 root root   0 Jun  2 07:17 .dockerenv
lrwxrwxrwx.   1 root root   7 May 11  2019 bin -> usr/bin
drwxr-xr-x.   5 root root 340 Jun  2 07:17 dev
drwxr-xr-x.   1 root root  66 Jun  2 07:17 etc
drwxr-xr-x.   2 root root   6 May 11  2019 home
lrwxrwxrwx.   1 root root   7 May 11  2019 lib -> usr/lib
lrwxrwxrwx.   1 root root   9 May 11  2019 lib64 -> usr/lib64
drwx------.   2 root root   6 Jan 13 21:48 lost+found
drwxr-xr-x.   2 root root   6 May 11  2019 media
drwxr-xr-x.   2 root root   6 May 11  2019 mnt
drwxr-xr-x.   2 root root   6 May 11  2019 opt
dr-xr-xr-x. 167 root root   0 Jun  2 07:17 proc
dr-xr-x---.   2 root root 162 Jan 13 21:49 root
drwxr-xr-x.  11 root root 163 Jan 13 21:49 run
lrwxrwxrwx.   1 root root   8 May 11  2019 sbin -> usr/sbin
drwxr-xr-x.   2 root root   6 May 11  2019 srv
dr-xr-xr-x.  13 root root   0 Jun  2 01:10 sys
drwxrwxrwt.   7 root root 145 Jan 13 21:49 tmp
drwxr-xr-x.  12 root root 144 Jan 13 21:49 usr
drwxr-xr-x.  20 root root 262 Jan 13 21:49 var
[root@localhost dockerfile]# 
```

DockerFile中很多命令都十分相似，我们要了解它们的区别，我们最好的学习就是对比它们然后测试！

------



## 实战:制做Tomcat镜像

1.准备镜像文件：tomcat压缩包，jdk压缩包
![1591090354426](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591090354426.png)

2.编写DockerFile文件，官方命名`Dockerfile`，build会自动寻找这个文件，就不需要-f 指定了！

```shell
FROM centos
MAINTAINER xzmadmin<xzmadmin@163.com>

COPY readme.txt /xzmag/user/localhost/readme.txt

ADD jdk-7u80-linux-x64.tar.gz /xzmag/one/localhost/
ADD apache-tomcat-7.0.104.tar.gz /xzmag/one/localhost/

RUN yum -y install vim

ENV MYPATH /xzmag/one/localhost
WORKDIR $MYPATH

#WORKDIR /xzmag/one/localhost

ENV JAVA_HOME /xzmag/one/localhost/jdk1.7.0_80
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

ENV CATALINA_HOME /xzmag/one/localhost/apache-tomcat-7.0.104
ENV CATALINA_BASH /xzmag/one/localhost/apache-tomcat-7.0.104

ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080


CMD /xzmag/one/localhost/apache-tomcat-7.0.104/bin/startup.sh && tail -F /xzmag/one/localhost/apache-tomcat-7.0.104/logs/catalina.out

```

3.构建镜像

```shell
# docker build -t diytomcat .
docker build -f /root/dockerfile/文件名 -t mycentos:1.0.0 .     指定dockerfile文件构建
```

4.启动镜像

```shell
docker run -d -p 9090:8080 --name xzmtomcat -v /home/kuangshen/build/tomcat/test:/usr/local/apache-tomcat-9.0.22/webapps/test -v /home/kuangshen/build/tomcat/tomcatlogs:/usr/local/apache-tomcat-9.0.22/logs diytomcat
```

5.访问测试tomcat
6.发布项目(由于做了卷挂载，我们直接在本地编写项目)

```shell
<?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://java.sun.com/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                               http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
           version="2.5">
  </web-app>
```

```shell
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>hello.cuijiayue</title>
</head>
<body>
Hello World!<br/>
<%
System.out.println("----my test web logs----");
%>
</body>
</html>
```

项目部署成功，直接访问成功！

------



# 发布自己的镜像

## 发布到DcokerHub

***前提准备***.去dockerhub注册自己的账号  ，地址：https://hub.docker.com/

```shell
docer login -u 用户名
[root@192 tomcat]# docker login -u bkxzmadmin
Password:                            #这个地方输入密码不会显示
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded              #登录成功

#开始上传文件之前修改镜像仓库
docker tag 镜像id hubdocker用户名/现在的镜像仓库名:版本号
#传递镜像
docker push 镜像仓库名:版本号

```

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591168732593.png)
![1591168875800](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591168875800.png)
![1591169036770](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591169036770.png)
![1591169166103](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591169166103.png)
![1591169980846](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591169980846.png)

------





## 发布到阿里云

1.登录阿里云，找到容器镜像服务
2.创建命名空间
![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591237390548.png)
3.创建容器镜像仓库
![1591237539482](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591237539482.png)

5.阿里云发布镜像指南
![1591239560298](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591239560298.png)

![1591242942741](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591242942741.png)

![1591242979729](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591242979729.png)
***亲测步骤教程：***
1.登录阿里云新建仓库，复制阿里云仓库中的地址
![1591243390484](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591243390484.png)

2.修改Docker中镜像仓库，直接粘贴第一步复制好的内容，
3.然后在上传。

------





# Docker网络

docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)     #列出所有容器ip地址







## **理解Docker0**

![1591262194517](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591262194517.png)
这三个网络分别代表不同的环境

```shell
#问题   docker是如何处理容器网络访问的？
```

![1591248944396](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591248944396.png)

```shell
# docker run -d -P --name tomcat01 tomcat  

#查看容器的内部网络地址 ip add ，发现容器启动时会得到一个eth0@if11 IP地址，这是docker分配的！
[root@192 ~]# docker exec -it tomcat01 ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
10: eth0@if11: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
[root@192 ~]# 

#linux能ping通容器内部！
[root@192 ~]# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.050 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.098 ms
^C
--- 172.17.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.050/0.069/0.098/0.021 ms
[root@192 ~]# 
#说明

```

> **原理**

1.我们每启动一个docker容器，docker就会给docker容器分配一个IP，我们只要安装了docker，就会有一个网卡 docker0
桥接模式，使用的技术evth-pair技术！

2.启动一个tomcat01，多了一个网卡![1591250285678](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591250285678.png)

3.在启动一个tomcat02，又多了一对个网卡，还有一个在容器，并且连着号
![1591250412796](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591250412796.png)

```shell
# 我们发现这个容器带来的网卡，都是一对一对的。
# evth-pair技术，就是一对的虚拟设备接口，他们都是成对出现的，一端连着协议，一端彼此相连
# 正因为有这个特性，evth-pair充当桥梁，连接各种虚拟网络设备的
```

4.测试一下tomcat01和tomcat02是否可以ping通！结果是ping通了
![1591258744259](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591258744259.png)

```shell
# docker exec -it tomcat02 ping 127.0.0.2
# 结论：容器之间是可以ping通的。
```



**网络模型图：**
![1591261160194](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591261160194.png)

tomcat01 ping tomcat02不是直接连，而是发请求到docker0，然后docker0在注册表找要tomcat02的IP或者用广播这两种方法来转发tomcat01的请求来ping通tomcat02

**结论：**tomcat01和tomcat02是共用的一个路由器，Docker0.
所有的容器在不指定网络的情况下，都是Docker0来路由的，docker会给容器分配一个默认的可用IP

------



## --link

![1591262779843](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591262779843.png)

上面我们使用容器名称ping容器名称，失败！我们做一些操作，让他们能用名称ping通

```shell
# 创建容器的时候用--link指定要ping的容器
[root@192 ~]# docker run -d -P --name tomcat04 --link tomcat01 tomcat       
7e88d5e678e95dd198b7ea2aa025b82cc2add9f505da607679ef6a60a398124c
[root@192 ~]# docker exec -it tomcat04 ping tomcat01                        
PING tomcat01 (172.17.0.3) 56(84) bytes of data.
64 bytes from tomcat01 (172.17.0.3): icmp_seq=1 ttl=64 time=0.164 ms
64 bytes from tomcat01 (172.17.0.3): icmp_seq=2 ttl=64 time=0.081 ms
64 bytes from tomcat01 (172.17.0.3): icmp_seq=3 ttl=64 time=0.082 ms
^C
--- tomcat01 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.081/0.109/0.164/0.038 ms
[root@192 ~]# 

#但是反向ping，tomcat01 ping tomcat04的时候又ping不通了
[root@192 ~]# docker exec -it tomcat01 ping tomcat04
ping: tomcat04: Name or service not known
[root@192 ~]# 

```

```shell
[root@192 ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
54209a1df4c9        bridge              bridge              local
e37418650af5        host                host                local
416b96f648d3        none                null                local
---
[root@192 ~]# docker inspect 54209a1df4c9
```

![1591264582680](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591264582680.png)

其实这个tomcat04就是在本地配置了tomcat01的配置

```shell
#查看tomcat04的配置，发现里面配置了tomcat01
[root@192 ~]# docker exec -it tomcat04 cat /etc/host
cat: /etc/host: No such file or directory
[root@192 ~]# docker exec -it tomcat04 cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.3	tomcat01 2510f402d599        #这里
172.17.0.5	7e88d5e678e9
[root@192 ~]#    
```

我们现在已经不建议使用--link了，因为这样太笨重了。建议使用的是自定义网络，就是不适用docker0。
docker0的问题：不支持容器名连接访问。

------



## 自定义网络

> 查看所有的docker网络    docker network ls

![1591427827484](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591427827484.png)
***网络模式 :***
1.bridge：桥接   在docker上面搭桥   默认的docker0和各个容器间就是使用的桥接
2.none：不配置网络
3.host：和宿主机共享网络
4.container：容器内网络连通(局限性很大，用的少，不建议使用了！)
*我们自己定义的网络也使用birdge 桥接模式*****

**测试**

```shell
# 我们直接启动的命令 --net bridge ，这个--net bridge就是我们的docker0，只是他在网络里面叫做bridge
docker run -a -P --name tomcat01    我们原来的启动的容器会有一个默认参数 --net bridge
docker run -a -P --name tomcat01 --net bridge tomcat

#docker0的特点，1.默认的  2.域名不能访问的，  --link可以打通连接，但不建议使用了
#我们可以自定义一个网络！
# --driver  默认的就是桥接
# --subnet  子网掩码地址
# --gateway 网关
[root@192 ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
5bada08cc606da861b4755303a1a143a1924f4f277bbb1f9e5ea3b7851343318
[root@192 ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
453f606f087a        bridge              bridge              local
e37418650af5        host                host                local
5bada08cc606        mynet               bridge              local
416b96f648d3        none                null                local
[root@192 ~]# 


```



我们自己的网络就创建好了！！！！

![1591429376081](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591429376081.png)
我们以后的服务就可以放在我们自己的网络里。

```SHELL
[root@192 ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "5bada08cc606da861b4755303a1a143a1924f4f277bbb1f9e5ea3b7851343318",
        "Created": "2020-06-06T15:37:55.576968075+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "8baeddee6ed3e538e346ddc85a153cf4dea77dd70e20f6896978da1a32903d52": {
                "Name": "tomcat-net-02",
                "EndpointID": "53abc6beb212e41502f381f76f7a07834725138445df7ddc0790b9422f58c266",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "9d16363e16ca5d7323cdc823c7e52a842785dee16597ca14079db31c7d8e7985": {
                "Name": "tomcat-net-01",
                "EndpointID": "3922a25323325fbb19c8009f49b4721aeba6b6e877e5ac3c26adbd8625e4bc7c",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]


# 再次测试ping连接
[root@192 ~]# docker exec -it tomcat-net-01 ping 192.168.0.3
PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=0.151 ms
64 bytes from 192.168.0.3: icmp_seq=2 ttl=64 time=0.076 ms
^C
--- 192.168.0.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1ms
rtt min/avg/max/mdev = 0.076/0.113/0.151/0.038 ms

#现在不使用--link，也可以使名字ping通
[root@192 ~]# docker exec -it tomcat-net-01 ping tomcat-net-02
PING tomcat-net-02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.058 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.067 ms
^C
--- tomcat-net-02 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 2ms
rtt min/avg/max/mdev = 0.058/0.062/0.067/0.009 ms
[root@192 ~]# 

```

我们自定义的网络，docker都已经帮我们维护好了对应的关系，推荐我们平时这样使用网络！

------



## 网络连通

![1591430408444](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591430408444.png)
![1591430550202](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591430550202.png)

```shell
# 测试tomcat01 打通到 mynet
[root@192 ~]# docker network connect mynet tomcat01

# 连通之后就是将tomcat01 放到了mynet网络下
#这种方式在官方叫做一个容器两个ip
```

![1591430745469](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1591430745469.png)

```shell
# tomcat01已经打通了
[root@192 ~]# docker exec -it tomcat01 ping tomcat-net-01
PING tomcat-net-01 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=102 ms
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.077 ms
^C
--- tomcat-net-01 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 2ms
rtt min/avg/max/mdev = 0.077/50.948/101.819/50.871 ms

#tomcat02是依旧打不通的
[root@192 ~]# docker exec -it tomcat02 ping tomcat-net-01
ping: tomcat-net-01: Name or service not known
[root@192 ~]# 
```

结论：假设要跨网络去操作别的容器，就需要使用docker networkect连通！

------

## 实战：Redis集群部署

背景：六个容器，三台主机，每个主机一个备份机，当主机遭遇不测，备份机马上顶替主机工作

```shell
# 创建网卡  因为是集群所以要在同一个网络
docker network create redis --subnet 172.38.0.0/16

# 通过脚本创建六个redis配置
for port in $(seq 1 6); \
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat <<EOF>/mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done



# 创建redis命令
docker run -p 6371:6379 -p 16371:16379 --name redis-1 \
-v /mydata/redis/node-1/data:/data \
-v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

docker run -p 6376:6379 -p 16376:16379 --name redis-6 \
-v /mydata/redis/node-6/data:/data \
-v /mydata/redis/node-6/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.16 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

#创建集群
/data # redis-cli --cluster create 172.38.0.11:6379 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1
>>> Performing hash slots allocation on 7 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.38.0.15:6379 to 172.38.0.11:6379
Adding replica 172.38.0.16:6379 to 172.38.0.12:6379
Adding replica 172.38.0.11:6379 to 172.38.0.13:6379
Adding extra replicas...
Adding replica 172.38.0.14:6379 to 172.38.0.11:6379
M: 5cf4403356b7ffd4fda9bae0438b621c697c793e 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
S: 5cf4403356b7ffd4fda9bae0438b621c697c793e 172.38.0.11:6379
   replicates a15ec7686df04340c09bbb78921d89d83dc92c95
M: 8633cb8734510363333e5ea9d8bdb1893023d895 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
M: a15ec7686df04340c09bbb78921d89d83dc92c95 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
S: a05ee964d54ff39b8e15b7442e9ed6f49f46b401 172.38.0.14:6379
   replicates 5cf4403356b7ffd4fda9bae0438b621c697c793e
S: 1f5c0f3947c1d105dabf4b6375c1b8ba99ce178e 172.38.0.15:6379
   replicates 5cf4403356b7ffd4fda9bae0438b621c697c793e
S: 12ce74aa957d69a2554debd044b15fc67e227591 172.38.0.16:6379
   replicates 8633cb8734510363333e5ea9d8bdb1893023d895
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 172.38.0.11:6379)
M: 5cf4403356b7ffd4fda9bae0438b621c697c793e 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
   2 additional replica(s)
M: a15ec7686df04340c09bbb78921d89d83dc92c95 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
S: 1f5c0f3947c1d105dabf4b6375c1b8ba99ce178e 172.38.0.15:6379
   slots: (0 slots) slave
   replicates 5cf4403356b7ffd4fda9bae0438b621c697c793e
S: a05ee964d54ff39b8e15b7442e9ed6f49f46b401 172.38.0.14:6379
   slots: (0 slots) slave
   replicates 5cf4403356b7ffd4fda9bae0438b621c697c793e
M: 8633cb8734510363333e5ea9d8bdb1893023d895 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 12ce74aa957d69a2554debd044b15fc67e227591 172.38.0.16:6379
   slots: (0 slots) slave
   replicates 8633cb8734510363333e5ea9d8bdb1893023d895
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
/data # 


```





***算了，Redis集群没学过，看不懂，不记了。连Redis的指令都不知道啥意思***

------





## SpringBoot微服务打包Docker镜像

1.构架springboot项目
2.打包应用
3.编写dockerfile
4.构建镜像
5.发布运行





# Docekr卸载

1、查看当前docker状态

![img](https://img2018.cnblogs.com/blog/761230/201909/761230-20190925092007697-366716325.png)

如果是运行状态则停掉

```shell
systemctl stop docker
```

2、查看yum安装的docker文件包

```shell
 yum list installed |grep docker
 
 查看docker相关的rpm源文件
 rpm -qa |grep docker
```

3、删除所有安装的docker文件包

```shell
yum -y remove rpm包名
```

 其他的docker相关的安装包同样删除操作，删完之后可以再查看下docker rpm源

```shell
rpm -qa |grep docker
```

删除docker的镜像文件，默认在/var/lib/docker目录下

![img](https://img2018.cnblogs.com/blog/761230/201909/761230-20190925092301219-851902288.png)

```shell
rm -rf /var/lib/docker
```

![img](https://img2018.cnblogs.com/blog/761230/201909/761230-20190925092333662-167764589.png)









# docker离线安装

1. 下载docker的安装文件

https://download.docker.com/linux/static/stable/x86_64/

下载的是：docker-18.06.3-ce.tgz 这个压缩文件



1. 将docker-18.06.3-ce.tgz文件上传到centos7-linux系统上，用ftp工具上传即可



1. 解压

```shell
[root@localhost java]# tar -zxvf docker-18.06.3-ce.tgz
```

1. 将解压出来的docker文件复制到 /usr/bin/ 目录下

```shell
[root@localhost java]# cp docker/* /usr/bin/
```

1. 进入**/etc/systemd/system/**目录,并创建**docker.service**文件

```shell
[root@localhost java]# cd /etc/systemd/system/
[root@localhost system]# touch docker.service
```

1. 打开**docker.service**文件,将以下内容复制

```shell
[root@localhost system]# vi docker.service
```

**注意**： --insecure-registry=192.168.200.128 此处改为你自己服务器ip

```shell
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd --selinux-enabled=false --insecure-registry=192.168.200.128
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
```

1. 给docker.service文件添加执行权限

```shell
[root@localhost system]# chmod 777 /etc/systemd/system/docker.service 
```

1. 重新加载配置文件（每次有修改docker.service文件时都要重新加载下）

```shell
[root@localhost system]# systemctl daemon-reload 
```

1. 启动

```shell
[root@localhost system]# systemctl start docker
```

1. 设置开机启动

```
[root@localhost system]# systemctl enable docker.service
```

1. 查看docker状态

```shell
[root@localhost system]# systemctl status docker
```

出现下面这个界面就代表docker安装成功。



1. 配置镜像加速器,默认是到国外拉取镜像速度慢,可以配置国内的镜像如：阿里、网易等等。下面配置一下网易的镜像加速器。打开docker的配置文件: /etc/docker/**daemon.json**文件：

```shell
[root@localhost docker]# vi /etc/docker/daemon.json
```

配置如下:

```json
{"registry-mirrors": ["http://hub-mirror.c.163.com"]}
```

配置完后**:wq**保存配置并**重启docker** 一定要重启不然加速是不会生效的！！！

```shell
[root@localhost docker]# service docker restart
```



# docker在线部署oracle并设置navicat连接



//拉取镜像
docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g

docker images

//创建容器
docker run -d -p 1521:1521 --name oracle11g registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g

//启动镜像
docker start oracle11g

//进入镜像
docker exec -it oracle11g bash

//切换管理员密码是helowin
su root

//修改profile，文件最底部添加三行。
vi /etc/profile

export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2
export ORACLE_SID=helowin
export PATH=$ORACLE_HOME/bin:$PATH

//创建软连接
ln -s $ORACLE_HOME/bin/sqlplus /usr/bin

//切换到oracle,！！！！！！！重点必须中间加   - 
su - oracle 

//连接数据库，添加navavat访问用户
[oracle@a8a161b66e1d ~]$ sqlplus /nolog

SQL*Plus: Release 11.2.0.1.0 Production on Wed Oct 10 12:54:06 2018

Copyright (c) 1982, 2009, Oracle.  All rights reserved.
SQL> conn / as sysdba
Connected.
SQL> alter user system identified by system;

User altered.

SQL> alter user sys identified by sys;

User altered.

SQL> create user ETS identified by ETS ;

User created.

SQL> grant connect,resource,dba to ETS ;

Grant succeeded.

 ![img](https://img-blog.csdnimg.cn/20200927133916152.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tlMXlpbmc=,size_16,color_FFFFFF,t_70)



如何配置psql，配置本地：

LS =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 121.78.***.***)(PORT = 1521))
    )
    (CONNECT_DATA =
       (SERVER = DEDICATED)
      (SERVICE_NAME = helowin)

   ）

）







# linux基本命令

cd /home      cd /要进入的目录
touch test.java     touch 需要创建的文件

pwd  以绝对路径的方式显示用户当前工作目录。命令将当前目录的全路径名称（从根目录）写入标准输出。全部目录使用`/`分隔。第一个`/`表示根目录，最后一个目录是当前目录。执行pwd命令可立刻得知您目前所在的工作目录的绝对路径名称

## 已使用过的命令

mv  jdk-8u251-linux-x64.tar.gz   /home/kuangshen/build/tomcat   移动文件
uname -a     查看操作系统多少位
 rm -rf           强制删除
yum install -y unzip zip    安装unzip命令

```shell
第一步下载　wget http://www.rarlab.com/rar/rarlinux-x64-5.3.0.tar.gz     如果linux不支持命令则去掉wget去访问地址下载
第二步解压　tar -zxvf rarlinux-x64-5.3.0.tar.gz
第三步     进入解压出的“rar”文件夹　cd rar 
第四步　   进行配置　make
       显示如下表示成功
mkdir -p /usr/local/bin

mkdir -p /usr/local/lib

cp rar unrar /usr/local/bin

cp rarfiles.lst /etc

cp default.sfx /usr/local/lib
可以用rar命令开始解压　
```



### 1、ls命令

就是 list 的缩写，通过 ls 命令不仅可以查看 linux 文件夹包含的文件，而且可以查看文件权限(包括目录、文件夹、文件权限)查看目录信息等等。

**常用参数搭配：**

```
ls -a 列出目录所有文件，包含以.开始的隐藏文件
ls -A 列出除.及..的其它文件
ls -r 反序排列
ls -t 以文件修改时间排序
ls -S 以文件大小排序
ls -h 以易读大小显示
ls -l 除了文件名之外，还将文件的权限、所有者、文件大小等信息详细列出来
```

**实例：**

(1) 按易读方式按时间反序排序，并显示文件详细信息

```
ls -lhrt
```

(2) 按大小反序显示文件详细信息

```
ls -lrS
```

(3)列出当前目录中所有以"t"开头的目录的详细内容

```
ls -l t*
```

(4) 列出文件绝对路径（不包含隐藏文件）

```
ls | sed "s:^:`pwd`/:"
```

(5) 列出文件绝对路径（包含隐藏文件）

```
find $pwd -maxdepth 1 | xargs ls -ld
```

### 2、cd 命令

cd(changeDirectory) 命令语法：

```
cd [目录名]
```

说明：切换当前目录至 dirName。

**实例：**

（1）进入要目录

```
cd /
```

（2）进入 "home" 目录

```
cd ~
```

（3）进入上一次工作路径

```
cd -
```

（4）把上个命令的参数作为cd参数使用。

```
cd !$
```

### 3、pwd 命令

pwd 命令用于查看当前工作目录路径。

**实例：**

（1）查看当前路径

```
pwd
```

（2）查看软链接的实际路径

```
pwd -P
```

### 4、mkdir 命令

mkdir 命令用于创建文件夹。

可用选项：

- **-m**: 对新建目录设置存取权限，也可以用 chmod 命令设置;
- **-p**: 可以是一个路径名称。此时若路径中的某些目录尚不存在,加上此选项后，系统将自动建立好那些尚不在的目录，即一次可以建立多个目录。

**实例：**

（1）当前工作目录下创建名为 t的文件夹

```
mkdir t
```

（2）在 tmp 目录下创建路径为 test/t1/t 的目录，若不存在，则创建：

```
mkdir -p /tmp/test/t1/t
```

### 5、rm 命令

删除一个目录中的一个或多个文件或目录，如果没有使用 -r 选项，则 rm 不会删除目录。如果使用 rm 来删除文件，通常仍可以将该文件恢复原状。

```
rm [选项] 文件…
```

**实例：**

（1）删除任何 .log 文件，删除前逐一询问确认：

```
rm -i *.log
```

（2）删除 test 子目录及子目录中所有档案删除，并且不用一一确认：

```
rm -rf test
```

（3）删除以 -f 开头的文件

```
rm -- -f*
```

### 6、rmdir 命令

从一个目录中删除一个或多个子目录项，删除某目录时也必须具有对其父目录的写权限。

**注意**：不能删除非空目录

**实例：**

（1）当 parent 子目录被删除后使它也成为空目录的话，则顺便一并删除：

```
rmdir -p parent/child/child11
```

### 7、mv 命令

移动文件或修改文件名，根据第二参数类型（如目录，则移动文件；如为文件则重命令该文件）。

当第二个参数为目录时，第一个参数可以是多个以空格分隔的文件或目录，然后移动第一个参数指定的多个文件到第二个参数指定的目录中。

**实例：**

（1）将文件 test.log 重命名为 test1.txt

```
mv test.log test1.txt
```

（2）将文件 log1.txt,log2.txt,log3.txt 移动到根的 test3 目录中

```
mv llog1.txt log2.txt log3.txt /test3
```

（3）将文件 file1 改名为 file2，如果 file2 已经存在，则询问是否覆盖

```
mv -i log1.txt log2.txt
```

（4）移动当前文件夹下的所有文件到上一级目录

```
mv * ../
```

### 8、cp 命令

将源文件复制至目标文件，或将多个源文件复制至目标目录。

注意：命令行复制，如果目标文件已经存在会提示是否覆盖，而在 shell 脚本中，如果不加 -i 参数，则不会提示，而是直接覆盖！

```
-i 提示
-r 复制目录及目录内所有项目
-a 复制的文件与原文件时间一样
```

**实例：**

（1）复制 a.txt 到 test 目录下，保持原文件时间，如果原文件存在提示是否覆盖。

```
cp -ai a.txt test
```

（2）为 a.txt 建议一个链接（快捷方式）

```
cp -s a.txt link_a.txt
```

### 9、cat 命令

cat 主要有三大功能：

1.一次显示整个文件:

```
cat filename
```

2.从键盘创建一个文件:

```
cat > filename
```

只能创建新文件，不能编辑已有文件。

3.将几个文件合并为一个文件:

```
cat file1 file2 > file
```

- -b 对非空输出行号
- -n 输出所有行号

**实例：**

（1）把 log2012.log 的文件内容加上行号后输入 log2013.log 这个文件里

```
cat -n log2012.log log2013.log
```

（2）把 log2012.log 和 log2013.log 的文件内容加上行号（空白行不加）之后将内容附加到 log.log 里

```
cat -b log2012.log log2013.log log.log
```

（3）使用 here doc 生成新文件

```
cat >log.txt <<EOF
>Hello
>World
>PWD=$(pwd)
>EOF
ls -l log.txt
cat log.txt
Hello
World
PWD=/opt/soft/test
```

（4）反向列示

```
tac log.txt
PWD=/opt/soft/test
World
Hello
```

### 10、more 命令

功能类似于 cat, more 会以一页一页的显示方便使用者逐页阅读，而最基本的指令就是按空白键（space）就往下一页显示，按 b 键就会往回（back）一页显示。

**命令参数：**

```
+n      从笫 n 行开始显示
-n       定义屏幕大小为n行
+/pattern 在每个档案显示前搜寻该字串（pattern），然后从该字串前两行之后开始显示 
-c       从顶部清屏，然后显示
-d       提示“Press space to continue，’q’ to quit（按空格键继续，按q键退出）”，禁用响铃功能
-l        忽略Ctrl+l（换页）字符
-p       通过清除窗口而不是滚屏来对文件进行换页，与-c选项相似
-s       把连续的多个空行显示为一行
-u       把文件内容中的下画线去掉
```

**常用操作命令：**

```
Enter    向下 n 行，需要定义。默认为 1 行
Ctrl+F   向下滚动一屏
空格键  向下滚动一屏
Ctrl+B  返回上一屏
=       输出当前行的行号
:f     输出文件名和当前行的行号
V      调用vi编辑器
!命令   调用Shell，并执行命令
q       退出more
```

**实例：**

（1）显示文件中从第3行起的内容

```
more +3 text.txt
```

（2）在所列出文件目录详细信息，借助管道使每次显示 5 行

```
ls -l | more -5
```

按空格显示下 5 行。

### 11、less 命令

less 与 more 类似，但使用 less 可以随意浏览文件，而 more 仅能向前移动，却不能向后移动，而且 less 在查看之前不会加载整个文件。

**常用命令参数：**

```
-i  忽略搜索时的大小写
-N  显示每行的行号
-o  <文件名> 将less 输出的内容在指定文件中保存起来
-s  显示连续空行为一行
/字符串：向下搜索“字符串”的功能
?字符串：向上搜索“字符串”的功能
n：重复前一个搜索（与 / 或 ? 有关）
N：反向重复前一个搜索（与 / 或 ? 有关）
-x <数字> 将“tab”键显示为规定的数字空格
b  向后翻一页
d  向后翻半页
h  显示帮助界面
Q  退出less 命令
u  向前滚动半页
y  向前滚动一行
空格键 滚动一行
回车键 滚动一页
[pagedown]： 向下翻动一页
[pageup]：   向上翻动一页
```

**实例：**

（1）ps 查看进程信息并通过 less 分页显示

```
ps -aux | less -N
```

（2）查看多个文件

```
less 1.log 2.log
```

可以使用 n 查看下一个，使用 p 查看前一个。

### 12、head 命令

head 用来显示档案的开头至标准输出中，默认 head 命令打印其相应文件的开头 10 行。

**常用参数：**

```
-n<行数> 显示的行数（行数为复数表示从最后向前数）
```

**实例：**

（1）显示 1.log 文件中前 20 行

```
head 1.log -n 20
```

（2）显示 1.log 文件前 20 字节

```
head -c 20 log2014.log
```

（3）显示 t.log最后 10 行

```
head -n -10 t.log
```

### 13、tail 命令

用于显示指定文件末尾内容，不指定文件时，作为输入信息进行处理。常用查看日志文件。

**常用参数：**

```
-f 循环读取（常用于查看递增的日志文件）
-n<行数> 显示行数（从后向前）
```

（1）循环读取逐渐增加的文件内容

```
ping 127.0.0.1 > ping.log &
```

后台运行：可使用 jobs -l 查看，也可使用 fg 将其移到前台运行。

```
tail -f ping.log
```

（查看日志）

### 14、which 命令

在 linux 要查找某个文件，但不知道放在哪里了，可以使用下面的一些命令来搜索：

```
which     查看可执行文件的位置。
whereis 查看文件的位置。
locate  配合数据库查看文件位置。
find        实际搜寻硬盘查询文件名称。
```

which 是在 PATH 就是指定的路径中，搜索某个系统命令的位置，并返回第一个搜索结果。使用 which 命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。

**常用参数：**

```
-n 　指定文件名长度，指定的长度必须大于或等于所有文件中最长的文件名。
```

**实例：**

（1）查看 ls 命令是否存在，执行哪个

```
which ls
```

（2）查看 which

```
which which
```

（3）查看 cd

```
which cd（显示不存在，因为 cd 是内建命令，而 which 查找显示是 PATH 中的命令）
```

查看当前 PATH 配置：

```
echo $PATH
```

或使用 env 查看所有环境变量及对应值

### 15、whereis 命令

whereis 命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息。whereis 及 locate 都是基于系统内建的数据库进行搜索，因此效率很高，而find则是遍历硬盘查找文件。

**常用参数：**

```
-b   定位可执行文件。
-m   定位帮助文件。
-s   定位源代码文件。
-u   搜索默认路径下除可执行文件、源代码文件、帮助文件以外的其它文件。
```

**实例：**

（1）查找 locate 程序相关文件

```
whereis locate
```

（2）查找 locate 的源码文件

```
whereis -s locate
```

（3）查找 lcoate 的帮助文件

```
whereis -m locate
```

### 16、locate 命令

locate 通过搜寻系统内建文档数据库达到快速找到档案，数据库由 updatedb 程序来更新，updatedb 是由 cron daemon 周期性调用的。默认情况下 locate 命令在搜寻数据库时比由整个由硬盘资料来搜寻资料来得快，但较差劲的是 locate 所找到的档案若是最近才建立或 刚更名的，可能会找不到，在内定值中，updatedb 每天会跑一次，可以由修改 crontab 来更新设定值 (etc/crontab)。

locate 与 find 命令相似，可以使用如 *、? 等进行正则匹配查找

**常用参数：**

```
-l num（要显示的行数）
-f   将特定的档案系统排除在外，如将proc排除在外
-r   使用正则运算式做为寻找条件
```

**实例：**

（1）查找和 pwd 相关的所有文件(文件名中包含 pwd）

```
locate pwd
```

（2）搜索 etc 目录下所有以 sh 开头的文件

```
locate /etc/sh
```

（3）查找 /var 目录下，以 reason 结尾的文件

```
locate -r '^/var.*reason$'（其中.表示一个字符，*表示任务多个；.*表示任意多个字符）
```

### 17、find 命令

用于在文件树中查找文件，并作出相应的处理。

命令格式：

```
find pathname -options [-print -exec -ok ...]
```

命令参数：

```
pathname: find命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。
-print： find命令将匹配的文件输出到标准输出。
-exec： find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {  } \;，注意{   }和\；之间的空格。
-ok： 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。
```

**命令选项：**

```
-name 按照文件名查找文件
-perm 按文件权限查找文件
-user 按文件属主查找文件
-group  按照文件所属的组来查找文件。
-type  查找某一类型的文件，诸如：
   b - 块设备文件
   d - 目录
   c - 字符设备文件
   l - 符号链接文件
   p - 管道文件
   f - 普通文件

-size n :[c] 查找文件长度为n块文件，带有c时表文件字节大小
-amin n   查找系统中最后N分钟访问的文件
-atime n  查找系统中最后n*24小时访问的文件
-cmin n   查找系统中最后N分钟被改变文件状态的文件
-ctime n  查找系统中最后n*24小时被改变文件状态的文件
-mmin n   查找系统中最后N分钟被改变文件数据的文件
-mtime n  查找系统中最后n*24小时被改变文件数据的文件
(用减号-来限定更改时间在距今n日以内的文件，而用加号+来限定更改时间在距今n日以前的文件。 )
-maxdepth n 最大查找目录深度
-prune 选项来指出需要忽略的目录。在使用-prune选项时要当心，因为如果你同时使用了-depth选项，那么-prune选项就会被find命令忽略
-newer 如果希望查找更改时间比某个文件新但比另一个文件旧的所有文件，可以使用-newer选项
```

**实例：**

（1）查找 48 小时内修改过的文件

```
find -atime -2
```

（2）在当前目录查找 以 .log 结尾的文件。 **.** 代表当前目录

```
find ./ -name '*.log'
```

（3）查找 /opt 目录下 权限为 777 的文件

```
find /opt -perm 777
```

（4）查找大于 1K 的文件

```
find -size +1000c
```

查找等于 1000 字符的文件

```
find -size 1000c 
```

-exec 参数后面跟的是 command 命令，它的终止是以 ; 为结束标志的，所以这句命令后面的分号是不可缺少的，考虑到各个系统中分号会有不同的意义，所以前面加反斜杠。{} 花括号代表前面find查找出来的文件名。

**实例：**

（5）在当前目录中查找更改时间在10日以前的文件并删除它们(无提醒）

```
find . -type f -mtime +10 -exec rm -f {} \;
```

（6）当前目录中查找所有文件名以.log结尾、更改时间在5日以上的文件，并删除它们，只不过在删除之前先给出提示。 按y键删除文件，按n键不删除

```
find . -name '*.log' mtime +5 -ok -exec rm {} \;
```

（7）当前目录下查找文件名以 passwd 开头，内容包含 "pkg" 字符的文件

```
find . -f -name 'passwd*' -exec grep "pkg" {} \;
```

（8）用 exec 选项执行 cp 命令

```
find . -name '*.log' -exec cp {} test3 \;
```

-xargs find 命令把匹配到的文件传递给 xargs 命令，而 xargs 命令每次只获取一部分文件而不是全部，不像 -exec 选项那样。这样它可以先处理最先获取的一部分文件，然后是下一批，并如此继续下去。

实例：

（9）查找当前目录下每个普通文件，然后使用 xargs 来判断文件类型

```
find . -type f -print | xargs file
```

（10）查找当前目录下所有以 js 结尾的并且其中包含 'editor' 字符的普通文件

```
find . -type f -name "*.js" -exec grep -lF 'ueditor' {} \;
find -type f -name '*.js' | xargs grep -lF 'editor'
```

（11）利用 xargs 执行 mv 命令

```
find . -name "*.log" | xargs -i mv {} test4
```

（12）用 grep 命令在当前目录下的所有普通文件中搜索 hostnames 这个词，并标出所在行：

```
find . -name \*(转义） -type f -print | xargs grep -n 'hostnames'
```

（13）查找当前目录中以一个小写字母开头，最后是 4 到 9 加上 .log 结束的文件：

```
find . -name '[a-z]*[4-9].log' -print
```

（14）在 test 目录查找不在 test4 子目录查找

```
find test -path 'test/test4' -prune -o -print
```

（15）实例1：查找更改时间比文件 log2012.log新但比文件 log2017.log 旧的文件

```
find -newer log2012.log ! -newer log2017.log
```

**使用 depth 选项：**

depth 选项可以使 find 命令向磁带上备份文件系统时，希望首先备份所有的文件，其次再备份子目录中的文件。

实例：find 命令从文件系统的根目录开始，查找一个名为 CON.FILE 的文件。 它将首先匹配所有的文件然后再进入子目录中查找

```
find / -name "CON.FILE" -depth -print
```

### 18、chmod 命令

用于改变 linux 系统文件或目录的访问权限。用它控制文件或目录的访问权限。该命令有两种用法。一种是包含字母和操作符表达式的文字设定法；另一种是包含数字的数字设定法。

每一文件或目录的访问权限都有三组，每组用三位表示，分别为文件属主的读、写和执行权限；与属主同组的用户的读、写和执行权限；系统中其他用户的读、写和执行权限。可使用 ls -l test.txt 查找。

以文件 log2012.log 为例：

```
-rw-r--r-- 1 root root 296K 11-13 06:03 log2012.log
```

第一列共有 10 个位置，第一个字符指定了文件类型。在通常意义上，一个目录也是一个文件。如果第一个字符是横线，表示是一个非目录的文件。如果是 d，表示是一个目录。从第二个字符开始到第十个 9 个字符，3 个字符一组，分别表示了 3 组用户对文件或者目录的权限。权限字符用横线代表空许可，r 代表只读，w 代表写，x 代表可执行。

常用参数：

```
-c 当发生改变时，报告处理信息
-R 处理指定目录以及其子目录下所有文件
```

权限范围：

```
u ：目录或者文件的当前的用户
g ：目录或者文件的当前的群组
o ：除了目录或者文件的当前用户或群组之外的用户或者群组
a ：所有的用户及群组
```

权限代号：

```
r ：读权限，用数字4表示
w ：写权限，用数字2表示
x ：执行权限，用数字1表示
- ：删除权限，用数字0表示
s ：特殊权限
```

实例：

（1）增加文件 t.log 所有用户可执行权限

```
chmod a+x t.log
```

（2）撤销原来所有的权限，然后使拥有者具有可读权限,并输出处理信息

```
chmod u=r t.log -c
```

（3）给 file 的属主分配读、写、执行(7)的权限，给file的所在组分配读、执行(5)的权限，给其他用户分配执行(1)的权限

```
chmod 751 t.log -c（或者：chmod u=rwx,g=rx,o=x t.log -c)
```

（4）将 test 目录及其子目录所有文件添加可读权限

```
chmod u+r,g+r,o+r -R text/ -c
```

19、tar 命令

用来压缩和解压文件。tar 本身不具有压缩功能，只具有打包功能，有关压缩及解压是调用其它的功能来完成。

弄清两个概念：打包和压缩。打包是指将一大堆文件或目录变成一个总的文件；压缩则是将一个大的文件通过一些压缩算法变成一个小文件

**常用参数：**

```
-c 建立新的压缩文件
-f 指定压缩文件
-r 添加文件到已经压缩文件包中
-u 添加改了和现有的文件到压缩包中
-x 从压缩包中抽取文件
-t 显示压缩文件中的内容
-z 支持gzip压缩
-j 支持bzip2压缩
-Z 支持compress解压文件
-v 显示操作过程
```

有关 gzip 及 bzip2 压缩:

```
gzip 实例：压缩 gzip fileName .tar.gz 和.tgz  解压：gunzip filename.gz 或 gzip -d filename.gz
          对应：tar zcvf filename.tar.gz     tar zxvf filename.tar.gz

bz2实例：压缩 bzip2 -z filename .tar.bz2 解压：bunzip filename.bz2或bzip -d filename.bz2
       对应：tar jcvf filename.tar.gz         解压：tar jxvf filename.tar.bz2
```

**实例：**

（1）将文件全部打包成 tar 包

```
tar -cvf log.tar 1.log,2.log 或tar -cvf log.*
```

（2）将 /etc 下的所有文件及目录打包到指定目录，并使用 gz 压缩

```
tar -zcvf /tmp/etc.tar.gz /etc
```

（3）查看刚打包的文件内容（一定加z，因为是使用 gzip 压缩的）

```
tar -ztvf /tmp/etc.tar.gz
```

（4）要压缩打包 /home, /etc ，但不要 /home/dmtsai

```
tar --exclude /home/dmtsai -zcvf myfile.tar.gz /home/* /etc
```

### 20、chown 命令

chown 将指定文件的拥有者改为指定的用户或组，用户可以是用户名或者用户 ID；组可以是组名或者组 ID；文件是以空格分开的要改变权限的文件列表，支持通配符。

```
-c 显示更改的部分的信息
-R 处理指定目录及子目录
```

**实例：**

（1）改变拥有者和群组 并显示改变信息

```
chown -c mail:mail log2012.log
```

（2）改变文件群组

```
chown -c :mail t.log
```

（3）改变文件夹及子文件目录属主及属组为 mail

```
chown -cR mail: test/
```

### 21、df 命令

显示磁盘空间使用情况。获取硬盘被占用了多少空间，目前还剩下多少空间等信息，如果没有文件名被指定，则所有当前被挂载的文件系统的可用空间将被显示。默认情况下，磁盘空间将以 1KB 为单位进行显示，除非环境变量 POSIXLY_CORRECT 被指定，那样将以512字节为单位进行显示：

```
-a 全部文件系统列表
-h 以方便阅读的方式显示信息
-i 显示inode信息
-k 区块为1024字节
-l 只显示本地磁盘
-T 列出文件系统类型
```

**实例：**

（1）显示磁盘使用情况

```
df -l
```

（2）以易读方式列出所有文件系统及其类型

```
df -haT
```

### 22、du 命令

du 命令也是查看使用空间的，但是与 df 命令不同的是 Linux du 命令是对文件和目录磁盘使用的空间的查看：

命令格式：

```
du [选项] [文件]
```

**常用参数：**

```
-a 显示目录中所有文件大小
-k 以KB为单位显示文件大小
-m 以MB为单位显示文件大小
-g 以GB为单位显示文件大小
-h 以易读方式显示文件大小
-s 仅显示总计
-c或--total  除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和
```

**实例：**

（1）以易读方式显示文件夹内及子文件夹大小

```
du -h scf/
```

（2）以易读方式显示文件夹内所有文件大小

```
du -ah scf/
```

（3）显示几个文件或目录各自占用磁盘空间的大小，还统计它们的总和

```
du -hc test/ scf/
```

（4）输出当前目录下各个子目录所使用的空间

```
du -hc --max-depth=1 scf/
```

### 23、ln 命令

功能是为文件在另外一个位置建立一个同步的链接，当在不同目录需要该问题时，就不需要为每一个目录创建同样的文件，通过 ln 创建的链接（link）减少磁盘占用量。

链接分类：软件链接及硬链接

软链接：

- 1.软链接，以路径的形式存在。类似于Windows操作系统中的快捷方式
- 2.软链接可以 跨文件系统 ，硬链接不可以
- 3.软链接可以对一个不存在的文件名进行链接
- 4.软链接可以对目录进行链接

硬链接:

- 1.硬链接，以文件副本的形式存在。但不占用实际空间。
- 2.不允许给目录创建硬链接
- 3.硬链接只有在同一个文件系统中才能创建

**需要注意：**

- 第一：ln命令会保持每一处链接文件的同步性，也就是说，不论你改动了哪一处，其它的文件都会发生相同的变化；
- 第二：ln的链接又分软链接和硬链接两种，软链接就是ln –s 源文件 目标文件，它只会在你选定的位置上生成一个文件的镜像，不会占用磁盘空间，硬链接 ln 源文件 目标文件，没有参数-s， 它会在你选定的位置上生成一个和源文件大小相同的文件，无论是软链接还是硬链接，文件都保持同步变化。
- 第三：ln指令用在链接文件或目录，如同时指定两个以上的文件或目录，且最后的目的地是一个已经存在的目录，则会把前面指定的所有文件或目录复制到该目录中。若同时指定多个文件或目录，且最后的目的地并非是一个已存在的目录，则会出现错误信息。

**常用参数：**

```
-b 删除，覆盖以前建立的链接
-s 软链接（符号链接）
-v 显示详细处理过程
```

**实例：**

（1）给文件创建软链接，并显示操作信息

```
ln -sv source.log link.log
```

（2）给文件创建硬链接，并显示操作信息

```
ln -v source.log link1.log
```

（3）给目录创建软链接

```
ln -sv /opt/soft/test/test3 /opt/soft/test/test5
```

### 24、date 命令
显示或设定系统的日期与时间。

命令参数：

```
-d<字符串> 　显示字符串所指的日期与时间。字符串前后必须加上双引号。
-s<字符串> 　根据字符串来设置日期与时间。字符串前后必须加上双引号。
-u 　显示GMT。
%H 小时(00-23)
%I 小时(00-12)
%M 分钟(以00-59来表示)
%s 总秒数。起算时间为1970-01-01 00:00:00 UTC。
%S 秒(以本地的惯用法来表示)
%a 星期的缩写。
%A 星期的完整名称。
%d 日期(以01-31来表示)。
%D 日期(含年月日)。
%m 月份(以01-12来表示)。
%y 年份(以00-99来表示)。
%Y 年份(以四位数来表示)。
```

**实例：**

（1）显示下一天

```
date +%Y%m%d --date="+1 day"  //显示下一天的日期
```

（2）-d参数使用

```
date -d "nov 22"  今年的 11 月 22 日是星期三
date -d '2 weeks' 2周后的日期
date -d 'next monday' (下周一的日期)
date -d next-day +%Y%m%d（明天的日期）或者：date -d tomorrow +%Y%m%d
date -d last-day +%Y%m%d(昨天的日期) 或者：date -d yesterday +%Y%m%d
date -d last-month +%Y%m(上个月是几月)
date -d next-month +%Y%m(下个月是几月)
```

### 25、cal 命令

可以用户显示公历（阳历）日历如只有一个参数，则表示年份(1-9999)，如有两个参数，则表示月份和年份：

常用参数：

```
-3 显示前一月，当前月，后一月三个月的日历
-m 显示星期一为第一列
-j 显示在当前年第几天
-y [year]显示当前年[year]份的日历
```

**实例：**

（1）显示指定年月日期

```
cal 9 2012
```

（2）显示2013年每个月日历

```
cal -y 2013
```

（3）将星期一做为第一列,显示前中后三月

```
cal -3m
```

### 26、grep 命令

强大的文本搜索命令，grep(Global Regular Expression Print) 全局正则表达式搜索。

grep 的工作方式是这样的，它在一个或多个文件中搜索字符串模板。如果模板包括空格，则必须被引用，模板后的所有字符串被看作文件名。搜索的结果被送到标准输出，不影响原文件内容。

命令格式：

```
grep [option] pattern file|dir
```

常用参数：

```
-A n --after-context显示匹配字符后n行
-B n --before-context显示匹配字符前n行
-C n --context 显示匹配字符前后n行
-c --count 计算符合样式的列数
-i 忽略大小写
-l 只列出文件内容符合指定的样式的文件名称
-f 从文件中读取关键词
-n 显示匹配内容的所在文件中行数
-R 递归查找文件夹
```

grep 的规则表达式:

```
^  #锚定行的开始 如：'^grep'匹配所有以grep开头的行。 
$  #锚定行的结束 如：'grep$'匹配所有以grep结尾的行。 
.  #匹配一个非换行符的字符 如：'gr.p'匹配gr后接一个任意字符，然后是p。  
*  #匹配零个或多个先前字符 如：'*grep'匹配所有一个或多个空格后紧跟grep的行。
.*   #一起用代表任意字符。  
[]   #匹配一个指定范围内的字符，如'[Gg]rep'匹配Grep和grep。 
[^]  #匹配一个不在指定范围内的字符，如：'[^A-FH-Z]rep'匹配不包含A-R和T-Z的一个字母开头，紧跟rep的行。  
\(..\)  #标记匹配字符，如'\(love\)'，love被标记为1。   
\<      #锚定单词的开始，如:'\<grep'匹配包含以grep开头的单词的行。
\>      #锚定单词的结束，如'grep\>'匹配包含以grep结尾的单词的行。
x\{m\}  #重复字符x，m次，如：'0\{5\}'匹配包含5个o的行。 
x\{m,\}  #重复字符x,至少m次，如：'o\{5,\}'匹配至少有5个o的行。  
x\{m,n\}  #重复字符x，至少m次，不多于n次，如：'o\{5,10\}'匹配5--10个o的行。  
\w    #匹配文字和数字字符，也就是[A-Za-z0-9]，如：'G\w*p'匹配以G后跟零个或多个文字或数字字符，然后是p。  
\W    #\w的反置形式，匹配一个或多个非单词字符，如点号句号等。  
\b    #单词锁定符，如: '\bgrep\b'只匹配grep。
```

**实例：**

（1）查找指定进程

```
ps -ef | grep svn
```

（2）查找指定进程个数

```
ps -ef | grep svn -c
```

（3）从文件中读取关键词

```
cat test1.txt | grep -f key.log
```

（4）从文件夹中递归查找以grep开头的行，并只列出文件

```
grep -lR '^grep' /tmp
```

（5）查找非x开关的行内容

```
grep '^[^x]' test.txt
```

（6）显示包含 ed 或者 at 字符的内容行

```
grep -E 'ed|at' test.txt
```

### 27、wc 命令

wc(word count)功能为统计指定的文件中字节数、字数、行数，并将统计结果输出

命令格式：

```
wc [option] file..
```

**命令参数：**

```
-c 统计字节数
-l 统计行数
-m 统计字符数
-w 统计词数，一个字被定义为由空白、跳格或换行字符分隔的字符串
```

**实例：**

（1）查找文件的 行数 单词数 字节数 文件名

```
wc text.txt
```

结果：

```
7     8     70     test.txt
```

（2）统计输出结果的行数

```
cat test.txt | wc -l
```

### 28、ps 命令

ps(process status)，用来查看当前运行的进程状态，一次性查看，如果需要动态连续结果使用 top

linux上进程有5种状态:

- \1. 运行(正在运行或在运行队列中等待)
- \2. 中断(休眠中, 受阻, 在等待某个条件的形成或接受到信号)
- \3. 不可中断(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生)
- \4. 僵死(进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放)
- \5. 停止(进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行)

ps 工具标识进程的5种状态码:

```
D 不可中断 uninterruptible sleep (usually IO)
R 运行 runnable (on run queue)
S 中断 sleeping
T 停止 traced or stopped
Z 僵死 a defunct (”zombie”) process
```

**命令参数：**

```
-A 显示所有进程
a 显示所有进程
-a 显示同一终端下所有进程
c 显示进程真实名称
e 显示环境变量
f 显示进程间的关系
r 显示当前终端运行的进程
-aux 显示所有包含其它使用的进程
```

**实例：**

（1）显示当前所有进程环境变量及进程间关系

```
ps -ef
```

（2）显示当前所有进程

```
ps -A
```

（3）与grep联用查找某进程

```
ps -aux | grep apache
```

（4）找出与 cron 与 syslog 这两个服务有关的 PID 号码

```
ps aux | grep '(cron|syslog)'
```

29、top 命令

显示当前系统正在执行的进程的相关信息，包括进程 ID、内存占用率、CPU 占用率等

**常用参数：**

```
-c 显示完整的进程命令
-s 保密模式
-p <进程号> 指定进程显示
-n <次数>循环显示次数
```

实例：

**（1）**

```
top - 14:06:23 up 70 days, 16:44,  2 users,  load average: 1.25, 1.32, 1.35
Tasks: 206 total,   1 running, 205 sleeping,   0 stopped,   0 zombie
Cpu(s):  5.9%us,  3.4%sy,  0.0%ni, 90.4%id,  0.0%wa,  0.0%hi,  0.2%si,  0.0%st
Mem:  32949016k total, 14411180k used, 18537836k free,   169884k buffers
Swap: 32764556k total,        0k used, 32764556k free,  3612636k cached
PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND  
28894 root      22   0 1501m 405m  10m S 52.2  1.3   2534:16 java  
```

前五行是当前系统情况整体的统计信息区。

**第一行，任务队列信息，同 uptime 命令的执行结果，具体参数说明情况如下：**

14:06:23 — 当前系统时间

up 70 days, 16:44 — 系统已经运行了70天16小时44分钟（在这期间系统没有重启过的吆！）

2 users — 当前有2个用户登录系统

load average: 1.15, 1.42, 1.44 — load average后面的三个数分别是1分钟、5分钟、15分钟的负载情况。

load average数据是每隔5秒钟检查一次活跃的进程数，然后按特定算法计算出的数值。如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转了。

**第二行，Tasks — 任务（进程），具体信息说明如下：**

系统现在共有206个进程，其中处于运行中的有1个，205个在休眠（sleep），stoped状态的有0个，zombie状态（僵尸）的有0个。

**第三行，cpu状态信息，具体属性说明如下：**

```
5.9%us — 用户空间占用CPU的百分比。
3.4% sy — 内核空间占用CPU的百分比。
0.0% ni — 改变过优先级的进程占用CPU的百分比
90.4% id — 空闲CPU百分比
0.0% wa — IO等待占用CPU的百分比
0.0% hi — 硬中断（Hardware IRQ）占用CPU的百分比
0.2% si — 软中断（Software Interrupts）占用CPU的百分比
```

**备注：**在这里CPU的使用比率和windows概念不同，需要理解linux系统用户空间和内核空间的相关知识！

第四行，内存状态，具体信息如下：

```
32949016k total — 物理内存总量（32GB）
14411180k used — 使用中的内存总量（14GB）
18537836k free — 空闲内存总量（18GB）
169884k buffers — 缓存的内存量 （169M）
```

**第五行，swap交换分区信息，具体信息说明如下：**

```
32764556k total — 交换区总量（32GB）
0k used — 使用的交换区总量（0K）
32764556k free — 空闲交换区总量（32GB）
3612636k cached — 缓冲的交换区总量（3.6GB）
```

**第六行，空行。**

**第七行以下：各进程（任务）的状态监控，项目列信息说明如下：**

```
PID — 进程id
USER — 进程所有者
PR — 进程优先级
NI — nice值。负值表示高优先级，正值表示低优先级
VIRT — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
RES — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
SHR — 共享内存大小，单位kb
S — 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
%CPU — 上次更新到现在的CPU时间占用百分比
%MEM — 进程使用的物理内存百分比
TIME+ — 进程使用的CPU时间总计，单位1/100秒
COMMAND — 进程名称（命令名/命令行）
```

**top 交互命令**

```
h 显示top交互命令帮助信息
c 切换显示命令名称和完整命令行
m 以内存使用率排序
P 根据CPU使用百分比大小进行排序
T 根据时间/累计时间进行排序
W 将当前设置写入~/.toprc文件中
o或者O 改变显示项目的顺序
```

### 30、kill 命令

发送指定的信号到相应进程。不指定型号将发送SIGTERM（15）终止指定进程。如果任无法终止该程序可用"-KILL" 参数，其发送的信号为SIGKILL(9) ，将强制结束进程，使用ps命令或者jobs 命令可以查看进程号。root用户将影响用户的进程，非root用户只能影响自己的进程。

**常用参数：**

```
-l  信号，若果不加信号的编号参数，则使用“-l”参数会列出全部的信号名称
-a  当处理当前进程时，不限制命令名和进程号的对应关系
-p  指定kill 命令只打印相关进程的进程号，而不发送任何信号
-s  指定发送信号
-u  指定用户
```

**实例：**

（1）先使用ps查找进程pro1，然后用kill杀掉

```
kill -9 $(ps -ef | grep pro1)
```

### 31、free 命令

显示系统内存使用情况，包括物理内存、交互区内存(swap)和内核缓冲区内存。

**命令参数：**

```
-b 以Byte显示内存使用情况
-k 以kb为单位显示内存使用情况
-m 以mb为单位显示内存使用情况
-g 以gb为单位显示内存使用情况
-s<间隔秒数> 持续显示内存
-t 显示内存使用总合
```

**实例：**

（1）显示内存使用情况

```
free
free -k
free -m
```

（2）以总和的形式显示内存的使用信息

```
free -t
```

（3）周期性查询内存使用情况

```
free -s 10
```




# Markdown 详细教程地址

- http://www.mamicode.com/info-detail-2794030.html



