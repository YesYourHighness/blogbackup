---
title: Docker
date: 2021-11-16 20:30:36
tags: 
- Docker
categories: 
- Docker
---

<center>
引言：Build, Ship and Run Any App, Anywhere!
</center>

<!--more-->

# Docker

因为有了Docker，运维与开发的界限越来越模糊，甚至现在的研发的定位就是**DevOps**

## 走进Docker

> 1、什么是Docker？

Docker是基于Go的开源容器项目：可以将Docker容器理解为一种**轻量级的虚拟机**

> 2、Docker有什么作用？

- 解耦应用程序与运行平台，可以保证快速的分发与部署；

实际中的应用比如：开发与运维交互，运维需要安装一系列前置环境（mysql、redis、kafka等等），并且对他们的版本也有要求，之前的安装配置方式费时费力；再比如需要对mysql服务集群进行快速的扩容和缩容；

> 3、Docker与虚拟机的区别

| 特性     | Docker容器 | 虚拟机 |
| -------- | ---------- | ------ |
| 启动速度 | 秒级       | 分钟级 |
| 内存使用 | 很少       | 较多   |
| 迁移性   | 优秀       | 一般   |

> 4、Docker容器与传统虚拟化技术的区别

**Docker技术是操作系统级别的虚拟化，虚拟机是硬件级别的虚拟化**

![Docker和传统的虚拟化方式的不同之处](http://img.yesmylord.cn//image-20211116165102160.png)

比方说Centos7是一个镜像文件，我们可以将其安装在VMWare上，这样就模拟了一个Linux操作系统；

现在Docker会将一个项目：包括运行文档、配置环境、运行环境统统打包为一个镜像，直接跑在我们的OS上（Docker相当于VMWare，项目镜像相当于Centos镜像）

---

实际上：Docker需要基本的Linux内核环境，所谓的操作系统级别的虚拟化，其实只是将APP所需要使用到的Linux内核支持拿了出来单独使用，这样做到的更小更快（Centos与Ubantu提出来的内核仅仅170M左右）。

所以实际上Docker是需要Linux的内核支持的，因此如果我们使用Windows去跑Docker，其实Docker也是先模拟了一个mini的Linux环境，然后在此之上运行程序的

## Docker三大核心概念

### 镜像Image

可以理解为一个**只读的模板**（类似于Java的类模板）

（一个镜像包含我们需要运行程序的必要环境）

### 容器Container

**Docker利用容器来运行应用**（类似于Java对象实例）

- 容器是**从镜像创建的应用运行实例**
- 容器之间相互隔离

注意：镜像本身是**只读**的，容器从镜像启动的时候，会在镜像的最上层创建一个**可写层**

### 仓库Repository

类似于代码仓库（Github、码云），**存放镜像的场所**

- 注册服务器是存放仓库的地方（比如一个注册服务器下，会有Ubantu的仓库、CentoOs的仓库等等）

## Docker的简易架构

如图所示，Docker的运行由三个构成：

![Docker Architecture diagram](http://img.yesmylord.cn//architecture.svg)

- **Client**：发起Docker命令
- **Docker Host**：docker安装的宿主机，运行着Docker这个后台进程（Docker是一个守护进程Daemon），宿主机内部有镜像（**本地镜像**）、容器
- **Register**：远程仓库，对于宿主机没有的本地镜像需要从远程仓库下载

## Docker相关命令

### 启动相关命令

```shell
# 启动Docker
systemctl start docker
# 重启
systemctl restart docker
# 关闭
systemctl stop docker
# 查看docker状态
systemctl status docker
# 开机启动Docker
systemctl enable docker
```

### 镜像相关命令

#### 1、获取镜像

我们可以去仓库下载镜像（类似于Git的操作）

```sh
docker pull name:[tag]

# name 即要下载的应用名称，tag为标签
# 如果不选择的话，默认是`lastest`，也就是最新版本的（所以我们一定要指定标签）
# 严格来说，如果没有指定仓库，所以默认会从registry.hub.docker.com下载，即此命令相当于：
docker pull registry.hub.docker.com/name:[tag]
# 如果需要安装非官方仓库的镜像的话，那么需要加上仓库名
```

demo：这里使用该命令下载`nginx`作为一个示例：

```shell
[root@slave1 ~]# docker pull nginx:1.20.1
1.20.1: Pulling from library/nginx
b380bbd43752: Already exists 
83acae5e2daa: Pull complete 
33715b419f9b: Pull complete 
eb08b4d557d8: Pull complete 
74d5bdecd955: Pull complete 
0820d7f25141: Pull complete 
Digest: sha256:a98c2360dcfe44e9987ed09d59421bb654cb6c4abe50a92ec9c912f252461483
Status: Downloaded newer image for nginx:1.20.1
docker.io/library/nginx:1.20.1
```

可以看到，镜像文件由很多**层`layer`**构成，前面的一串`b380bbd43752`表示这一层的唯一ID，当**不同的镜像包括相同的层时，本地仅存储层的一份内容**，减小了需要的存储空间。

#### 2、查看镜像

查看镜像的命令有如下几个：

- **查看下载的镜像**

```sh
docker images
# -a 显示所有镜像，包括临时镜像
# --no-trunc 此参数可以完全显示镜像信息（默认情况下会截断显示内容）
# -q 仅输出ID信息
```

demo：此处做示例

```shell
[root@slave1 ~]# docker images --no-trunc
REPOSITORY   TAG       IMAGE ID                                                                  CREATED       SIZE
mysql        5.7       sha256:938b57d64674c4a123bf8bed384e5e057be77db934303b3023d9be331398b761   4 weeks ago   448MB
nginx        1.20.1    sha256:c8d03f6b8b915209c54fc8ead682f7a5709d11226f6b81185850199f18b277a2   5 weeks ago   133MB
```

可见，`Image Id`是很长的串，只不过可以用前几位代替完整的ID

- **给镜像添加自定义名称**

命令：

```sh
docker tag name:[tag] my-name
# my-tag：表示自己可以自定义的名称
```

demo：下面是一个示例

```shell
[root@slave1 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
mysql        5.7       938b57d64674   4 weeks ago   448MB
nginx        1.20.1    c8d03f6b8b91   5 weeks ago   133MB
[root@slave1 ~]# docker tag nginx:1.20.1 my-nginx
[root@slave1 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
mysql        5.7       938b57d64674   4 weeks ago   448MB
my-nginx     latest    c8d03f6b8b91   5 weeks ago   133MB
nginx        1.20.1    c8d03f6b8b91   5 weeks ago   133MB
```

可以看到，`my-nginx`与`nginx`两个的ID值一样，说明两个指向同一个镜像，我们自定义的标签只是一个软连接而已

- 查看镜像详细信息

```
docker inspect name:tag
```

会返回一个json串，如果我们只想查看其中的部分数据可以加`-f`参数，例如这样（记着加`.`）

```
docker inspect my-nginx -f {{".Metadata"}}
```

#### 3、搜索镜像

```sh
docker search name
# -f 可以指定额外的参数
# -f stars=100 显示star数大于100的镜像
# --limit 5 只显示5条信息
```

demo：显示所有stars大于100的并且可以自动装配的nginx镜像

```sh
[root@slave1 ~]# docker search -f stars=100 nginx
NAME                          DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
jwilder/nginx-proxy           Automated Nginx reverse proxy for docker con…   2094                 [OK]
richarvey/nginx-php-fpm       Container running Nginx + PHP-FPM capable of…   818                  [OK]
tiangolo/nginx-rtmp           Docker image with Nginx using the nginx-rtmp…   145                  [OK]
jlesage/nginx-proxy-manager   Docker container for Nginx Proxy Manager        143                  [OK]
alfg/nginx-rtmp               NGINX, nginx-rtmp-module and FFmpeg from sou…   110                  [OK]
```

#### 4、删除镜像

```sh
docker rmi tag|id
# -f 强制删除，即使当前镜像已经启动了一些容器
```

注意：

- 如果参数指定为一个tag，那么此命令只会删除该镜像的一个标签而已（如果这个镜像只有一个标签，那么会被立即删除）

- 参数指定为image id的话，会先删除该镜像的所有标签，然后删除该镜像本身

示例：

```sh
[root@slave1 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
mysql        5.7       938b57d64674   4 weeks ago   448MB
my-nginx     latest    c8d03f6b8b91   5 weeks ago   133MB
nginx        1.20.1    c8d03f6b8b91   5 weeks ago   133MB
[root@slave1 ~]# docker rmi my-nginx
Untagged: my-nginx:latest
[root@slave1 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
mysql        5.7       938b57d64674   4 weeks ago   448MB
nginx        1.20.1    c8d03f6b8b91   5 weeks ago   133MB
[root@slave1 ~]# docker rmi nginx:1.20.1
Untagged: nginx:1.20.1
Untagged: nginx@sha256:a98c2360dcfe44e9987ed09d59421bb654cb6c4abe50a92ec9c912f252461483
Deleted: sha256:67a7407724b6c71e2355fc2236b5be27d1f03bf9cbdffdfbb97c1d2a326ccf94
...// 真正删除了所有的镜像
[root@slave1 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
mysql        5.7       938b57d64674   4 weeks ago   448MB
```

#### 5、查看镜像的占用空间

```sh
docker system df
```

输出docker目前占用的大小内容：

```sh
hynis@hynisVM:~$ docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          14        1         2.612GB   2.612GB (99%)
Containers      2         0         0B        0B
Local Volumes   3         0         9.208kB   9.208kB (100%)
Build Cache     0         0         0B        0B
```

### 容器相关命令

#### 1、创建与运行容器

1. 创建容器（基于一个镜像，创建一个容器）

```sh
docker create [参数] name:tag
# 会返回一个容器的ID Container ID
```

2. 启动容器

```sh
docker start id
```

3. 【**推荐**】创建并启动容器（建议直接使用`run`命令，等同于上述创建并启动容器）

```shell
docker run [options] image
# 其中`options`可选的参数很多，下面介绍几个常用的：
--name # 指定此容器的名字
-i # 让容器的标准输入保持打开（交互式模式运行容器），通常与-t一起使用
-t # 分配一个伪终端并绑定到容器的标准输出上
-d # 后台运行（守护态运行）
-P # 容器内随机映射一个端口
-p # 指定端口映射，比如 -p 3306:3306 就指定主机的3306端口与容器的3306端口进行映射，此时访问主机的3306端口，相当于访问容器的3306端口
```

demo：启动一个交互式的Ubuntu系统：

```sh
hynis@hynisVM:~$ docker run -it ubuntu /bin/bash
# 此处命令的意思是 run -it + 一个OS + shell
# 此处的61a30a155427就是容器号，想要退出使用命令exit
root@61a30a155427:/# exit
exit
```

想要退出当前容器，可以输入`exit`也可以直接按`ctrl + p + q`（注意：此按键并不会关闭容器）

> PS：补充run命令的执行顺序

当利用`docker run`来创建并启动容器时，Docker在后台运行的标准操作包括：

1. 检查**本地**是否存在指定的镜像，不存在就从**公有仓库**下载；
2. 利用镜像**创建**一个容器，并**启动**该容器；
3. **分配一个文件系统**给容器，并**在只读的镜像层外面挂载一层可读写层**；
4. 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中；
5. 从网桥的地址池配置一个IP地址给容器；
6. 执行用户指定的应用程序；
7. 执行完毕后容器被自动终止。

#### 2、查看容器

1. 查看所有的容器

```sh
docker ps -l
# -a 可以显示所有的容器，包括没有在运行的容器
# -l 显示最近创建的容器
# -n 显示最近n个创建的容器
# -q 静默模式，只显示容器编号
```

2. 查看容器的日志

```sh
docker logs [id]
```

#### 3、进入容器

进入**存活**的容器有两个命令：

```sh
# 进入正在进行的容器
docker attach [id]
docker exec -it [id] /bin/bash
```

两个有一些区别：

- `attach`直接进入容器启动命令的终端，不会启动新的进程，用`exit`会**直接退出**
- `exec`是在容器中打开新的终端，用`exit`不会退出【**推荐**】

#### 4、终止、重启与删除容器

1. 终止容器

首先向容器发送SIGTERM信号，**等待一段超时时间**（默认为10秒）后，再发送SIGKILL信号来终止容器

```sh
docker stop id
# -t 可以指定终止的时间
# 对于已经终止的容器 可以使用start命令
docker start id
```

2. 如果想要重启一个容器

```sh
docker restart id
```

3. 强制终止一个容器

会直接发送SIGKILL信号

```sh
docker kill id
```

4. 删除容器

只能删除非运行态的容器，除非加参数`-f`

```sh
docker rm id
# -f 强制删除
```

### 拷贝、导入与导出

1、从容器拷贝文件到主机

```sh
docker cp 容器ID:容器内路径 目的主机路径
# 在宿主机上敲此命令
# hynis@hynisVM:~$ docker cp d0367f6fb6c4:/a.txt ./
```

2、如果想要所有的数据，那么可以使用**导出命令**：导出容器的内容作为一个tar归档

```sh
docker export [id] > xxx.tar
# 将id为d0367f6fb6c4的容器的数据导出到当前目录下，并且打包为ubuntu1.tar
docker export d0367f6fb6c4 > ubuntu1.tar
```

3、导入一个tar归档文件

```sh
cat 归档.tar | docker import - 镜像用户/镜像:版本号
# 注意命令里有一个-
# 镜像用户就是包名
```

比如下面这个例子：我们使用第二步导出的数据恢复一个容器

```sh
hynis@hynisVM:~$ cat ubuntu1.tar | docker import hynis/ubuntu:latest
open hynis/ubuntu:latest: no such file or directory
hynis@hynisVM:~$ cat ubuntu1.tar | docker import - hynis/ubuntu:latest
sha256:95fad8b97bceb4cdb159d62694fe7f4f906729e61bc8528f00a9495b2287061a
hynis@hynisVM:~$ docker images
REPOSITORY                   TAG       IMAGE ID       CREATED         SIZE
hynis/ubuntu                 latest    95fad8b97bce   3 seconds ago   77.8MB
```

## 镜像的深入理解

镜像包括不同的层次，当**不同的镜像包括相同的层时，本地仅存储层的一份内容**，减小了需要的存储空间。其本质需要了解一下**联合文件系统**

### 联合文件系统

Unionfs 是2004年在stony brook大学开始的，它是一个可叠放的联合文件系统，它**能够联合多个目录（因此可称为分支）同时独立地保持它们的物理内容**。

所谓**联合**的意思是：联合在不同磁盘上的不同的文件系统到一个目录，或者把几张cd合并成一个统一的归档镜像。

允许任何ro（只读）和rw（可读可写）分支的 结合，同时允许在分支中修改和删除不使用的分支，（即具有复制可写功能的unionfs可以用来把ro和rw的文件系统合并起来），并且可以允许修改只读文件系统并把这些修改保存在可写文件系统。

### Docker镜像的加载原理

比如Docker上安装Ubuntu和CentOS，这是两种发行版，（如果我们使用VMWare安装，我们就需要这两者的镜像）他们都是Linux的操作系统，其实Linux操作系统由两部分构成：

- **BootFS**：Boot与Loader部分，Boot在主板的bios内，当电脑启动后，bios载入Boot程序，Boot载入Loader程序，loader程序将Linux内核载入到内存中，操作系统启动完毕。
- **RootFS**：`rootfs`则包含了一般系统上的常见目录结构，类似于`/dev, /proc, /bin`等等以及一些基本的文件和命令。发行版的区别在于此，即Ubuntu和Centos的区别在于此；

![Linux的两部分](http://img.yesmylord.cn//image-20230409133819626.png)

Docker的镜像加载原理就是基于UnionFS，复用同一套内核（即**使用宿主机的bootfs**），对不同的发行版再去下载不同的rootfs，这样不仅节省内存空间，还加快了速度。

注意：**镜像层都是只读的，只有容器层是可写的**

Docker的容器启动时，一个新的**可写层**会被加载到镜像的顶部，如下图所示。

![容器运行在镜像上](http://img.yesmylord.cn//image-20230409135550659.png)

### Commit命令

`docker commit`可以提交容器副本使之成为一个新的镜像

```sh
docker commit -m='提交的描述信息' -a='作者' 容器ID 新的镜像名:[标签名]
```

比如我们现在给基础的Ubuntu系统加一个Vim程序，然后使用Commit命令后，这个新的镜像一运行就可以直接使用vim命令

![commit镜像](http://img.yesmylord.cn//image-20230409144116869.png)



### 本地镜像推入私有库

![Docker云端](http://img.yesmylord.cn//image-20230409220551123.png)

如果不想让敏感数据存入阿里云或是Dockerhub，公司可以搭建自己的私有库**Docker Registry**

1、Docker拉取Docker Registry

```sh
docker pull registry
```

2、启动：此处用到了容器数据卷

```sh
docker run -d -p 5000:5000  -v /zzyyuse/myregistry/:/tmp/registry --privileged=true registry
# -d 后台运行
# -p 5000:5000 将本地的5000端口映射到5000端口
# -v 容器数据卷
# --privileged=true 特权模式开启：如果不开启特权模式可能会出现权限问题
```

Docker挂载主机目录访问如果出现`cannot open directory .: Permission denied`，记住加参数`--privileged=true`

3、可以查看以下本地的私有Docker库是否含有数据

```sh
hynis@hynisVM:~$ curl -XGET http://192.168.235.151:5000/v2/_catalog
{"repositories":[]}
```

4、开启http传输

```sh
# 修改daemon.json文件
sudo vim /etc/docker/daemon.json

# 加入以下内容，Docker的镜像配置也在这里，此处没有配置
{
  "insecure-registries": ["192.168.235.151:5000"]
}
```

注意：更改完成后，需要重启一下Docker，并重新跑一下Registry

5、将我们想要的一个镜像传输到本地私有库

```sh
# 更改一下tag
docker tag hynitu:1.0 192.168.235.151:5000/hynitu:1.0
# push
docker push 192.168.235.151:5000/hynitu:1.0
```

```sh
hynis@hynisVM:~$ docker push 192.168.235.151:5000/hynitu:1.0
The push refers to repository [192.168.235.151:5000/hynitu]
e3b19c432d29: Pushed 
b93c1bd012ab: Pushed 
1.0: digest: sha256:29f871c91b95fa69e13ebcc0013046ffbf32a7fc89ae2dee4e655f3ffa6281d9 size: 741
hynis@hynisVM:~$ curl -XGET http://192.168.235.151:5000/v2/_catalog
{"repositories":["hynitu"]}
```

6、拉取到本地运行

```sh
docker pull 192.168.235.151:5000/hynitu:1.0
```

```sh
hynis@hynisVM:~$ docker pull 192.168.235.151:5000/hynitu:1.0
1.0: Pulling from hynitu
Digest: sha256:29f871c91b95fa69e13ebcc0013046ffbf32a7fc89ae2dee4e655f3ffa6281d9
Status: Image is up to date for 192.168.235.151:5000/hynitu:1.0
192.168.235.151:5000/hynitu:1.0
```

## 容器数据卷

### 数据卷

在上一章我们使用Registry，将本地的镜像上传到了本地的私有仓库，使用到了如下命令

```sh
docker run -d -p 5000:5000 -v /zzyyuse/myregistry/:/tmp/registry --privileged=true registry
```

此处`-v`参数就添加了容器数据卷，这条命令的意思是，将宿主机的目录的`/zzyyuse/myregistry/`映射到容器内的`/tmp/registry`，两个目录资源共享

> **数据卷**：就是文件或目录，脱离于Docker容器的生命周期之外的，因此Docker不会在容器删除时删除其挂载的数据卷

数据卷本来的作用就是做**数据备份**或是**资源共享**的

- 数据备份：将容器数据备份到本地目录，防止Docker容器挂掉数据丢失
- 资源共享：数据卷可以在**容器之间共享或重用数据**

注意：对于数据卷的修改不会包含在镜像的更新中

### 相关命令

1、添加数据卷

```sh
docker run -it --privileged=true -v 宿主机目录:容器内目录[:权限] ubuntu
# 权限部分可以不写，默认即为rw即可读可写
# 权限部分有: rw 与 ro
# 如果为 ro 则容器不能对此目录进行修改，而宿主机可以
```

2、查看是否设置数据卷成功

```sh
docker inspect 容器ID

# 可以查看到Mount下有
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/tmp/temphynisdata",
                "Destination": "/tmp/tempdockerdata",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
```

3、此时目录`/tmp/temphynisdata`与`/tmp/tempdockerdata`关联，双向绑定

> 提问：如果此时将容器关掉，将数据卷位置的数据进行修改，再将容器重启动，那么容器内的数据卷会发生变化吗？

答案是会的，所以可以理解到，宿主机和容器的数据卷并不是双向同步，而是共享同一片地址

4、**容器卷可以进行继承**，假设现在我们需要另外一个容器，和之前的容器的规则相同，那么可以进行继承

```sh
#额外的参数，这样新建的容器就会继承父容器的卷规则
--volumes-from 父容器
```

# Docker实战

## Docker实现Mysql主从同步

1、Docker安装mysql5.7版本

```sh
docker pull mysql:5.7
```

2、先建立**主mysql**，需要`run`一主一从

```sh
# 主
docker run -p 3307:3306 --name mysql-master -v /mydata/mysql-master/log:/var/log/mysql -v /mydata/mysql-master/data:/var/lib/mysql -v /mydata/mysql-master/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
# -p 主3307 从3308
# -d 后台运行
# --name 命名
# -v 配置数据卷
```

需要在`/mydata/mysql-master/conf`下创建一个配置文件`my.cnf`内容为以下：

```sh
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=101
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql
## 开启二进制日志功能
log-bin=mall-mysql-bin
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
```

配置完成后重启mysql服务，还需要配置数据同步用户

```sh
# 重启mysql-master
docker restart mysql-master
# 进入master容器
docker exec -it mysql-master /bin/bash
# 在bash键入
mysql -uroot -proot
# 创建数据同步用户
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
# 授予REPLICATION权限
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
```

3、建立从mysql

```sh
# 从
docker run -p 3308:3306 --name mysql-slave -v /mydata/mysql-slave/log:/var/log/mysql -v /mydata/mysql-slave/data:/var/lib/mysql -v /mydata/mysql-slave/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
```

需要在`/mydata/mysql-slave/conf`下创建一个配置文件`my.cnf`内容为以下：

```sh
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=102
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql
## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用
log-bin=mall-mysql-slave1-bin
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
## relay_log配置中继日志
relay_log=mall-mysql-relay-bin
## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1
## slave设置为只读（具有super权限的用户除外）
read_only=1
```

配置完成后同样重启容器

```sh
docker restart mysql-slave
```

4、注意下面的操作需要区分主从了，不要键入错误

主mysql先看一下状态`show master status;`

```sh
主mysql> show master status;
# 主要看一下bin从哪里开始，这里是000004
+-----------------------+----------+--------------+------------------+-------------------+
| File                  | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-----------------------+----------+--------------+------------------+-------------------+
| mall-mysql-bin.000004 |      617 |              | mysql            |                   |
+-----------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

在从mysql中配置主从复制

```sh
# 命令根据自己的配置进行更改
change master to master_host='宿主机ip', master_user='slave', master_password='123456', master_port=3307, master_log_file='mall-mysql-bin.000001', master_log_pos=617, master_connect_retry=30;
```

```sh
从mysql> change master to master_host='192.168.235.151', master_user='slave', master_password='123456', master_port=3307, master_log_file='mall-mysql-bin.000004', master_log_pos=617, master_connect_retry=30;
Query OK, 0 rows affected, 2 warnings (0.01 sec)
```

顺便查看以下从机的主从复制配置`show slave status \G;`

```sh
mysql> show slave status \G; # \G 让结果以列展示
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.235.151
                  Master_User: slave
                  Master_Port: 3307
                Connect_Retry: 30
              Master_Log_File: mall-mysql-bin.000004
          Read_Master_Log_Pos: 617
               Relay_Log_File: mall-mysql-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mall-mysql-bin.000004
             Slave_IO_Running: No # 主要看此处，主从复制还没开始
            Slave_SQL_Running: No # 主要看此处，主从复制还没开始
```

从mysql启动主从同步`start slave`

```sh
mysql> start slave;
Query OK, 0 rows affected (0.01 sec)

mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.235.151
                  Master_User: slave
                  Master_Port: 3307
                Connect_Retry: 30
              Master_Log_File: mall-mysql-bin.000004
          Read_Master_Log_Pos: 1235
               Relay_Log_File: mall-mysql-relay-bin.000003
                Relay_Log_Pos: 325
        Relay_Master_Log_File: mall-mysql-bin.000004
             Slave_IO_Running: Yes # 此处已连接
            Slave_SQL_Running: Yes # 此处已连接
```

然后我们可以在主Mysql新建一个表，建一些数据测试一下，从数据库会得到同步

---

**排错提示**：我在第一次尝试的过程中，遇到的问题是`Slave_IO_Running: Connecting`，再确认其他配置正确的情况下，反复尝试，都是错误，关闭防火墙后重试，变为Yes

```sh
# 查看防火墙状态
sudo ufw status
# 关闭防火墙
sudo ufw disable
```

之后在从节点处输入命令，重启一下slave

```sh
stop slave;
start slave;
```

运行后查到双Yes

---

## 如何存储1~2亿的缓存数据？

> 现在我们有1~2亿条数据需要缓存，请问如何设计？

如此大量的数据我们单击Redis肯定完成不了，需要进行**分布式存储**

### 哈希取余分区

最简单的实现，就是模以N，来对用户的读写操作实现负载均衡

![哈希取余算法](http://img.yesmylord.cn//image-20230411092917092.png)

**优点**：简单易实现，流量会均匀打到各个节点上

**缺点**：当集群某一台挂掉，会导致一定的混乱。

比如说现在有三台机器，之前的读写操作都%3，分别打到0、1、2三台机器上，但是现在出现了一点问题，第三台机器挂了，那么此时读写操作%2，此时的流量会分别打到0、1上。

即使是没有机器宕机，Redis集群的扩容缩容也会存在，每次更改都会导致映射关系进行重新计算

### 一致性哈希算法

1997年，麻省理工提出的一致性哈希算法。

> 一致性哈希算法：为了当服务器个数发生变化的时候，尽量减少影响客户端到服务器的映射关系。

算法思想由三步骤组成：

1. 构造一个环，从`0~2^32-1`（哈希取余是根据服务器台数，此处是int的范围）
2. 对于不同的Redis服务器，对IP进行取余，分别落在这个环上的不同点
3. 当流量打入到环上后，会按顺时针的方向，落在第一个遇到的服务器上。

![一致性哈希算法](http://img.yesmylord.cn//image-20230411100725377.png)

如图所示，环由`0~2^32-1`个点构成，有三台Redis服务器，按照hash(IP)后，分布在环上的各个节点上。此时用户流量打入：User1的操作会按顺时针分别流入Redis1、Redis3，User2的操作会打入Redis2，User3的操作会打入Redis3。

> 【可容错】假如此时Redis2宕机会发生什么？

User2的流量会打入Redis3，所以整个环路中，出现某个结点挂机，只会影响两个节点（宕机的节点和宕机节点的下一个节点）的流量分布。

> 【可扩展】假如Redis2又恢复了，又会发生什么？

只需要将Redis1到Redis2之间的数据转移到Redis2即可，其他部分不受影响。

> 【可能出现**数据倾斜**】当节点较少的时候，或者数据hash高度一致的时候会有什么影响？

假如我们的节点很少（可以假设Redis2宕机，我们此时只有两个节点）那么可以看到Redis1到Redis3的距离，与Redis3到Redis1的距离差距很多，这样会导致大部分的数据打入到Redis3，Redis3可能会高度负载，而Redis2却没有负载。

所以总结一下：一致性哈希算法适合使用在Redis数量很多，流量也很多的情况下。

### 哈希槽分区

目前的绝对主流的使用方法。

> 哈希槽：本质是一个大小为`2^14-1`(即16384个)的**数组**
>
> 在**数据**和**服务器**节点之间又加入了一层，把这层称为**哈希槽（slot）**，用于处理数据和节点之间的关系。

![哈希槽分区](http://img.yesmylord.cn//image-20230411104332152.png)

如图所示，将哈希槽分为了4段，用不同的颜色表示。当流量打入时，会进行这样的运算：`slot = CRC16(key) % 16384`，先对数据求循环冗余码CRC，然后对循环冗余码模以16384，这样数据就会打入不同的Redis上。

如果某个节点出现了宕机，那么宕机节点应该承受的流量会由其他正常节点共同承担。（一致性哈希算法就出现了数据倾斜，哈希槽避免了这个问题）

> 为什么是16384个，也就是`2^14-1`个槽位？

如果槽位是65535个，那么Redis的心跳信息的消息头将会有8k，较为庞大。而且Redis集群中的主节点数不可能超过1000个，因此16384个完全够用了。

## Docker实现Redis三主三从

### Reids集群搭建

1、下载Redis镜像

```sh
docker pull redis:6.0.8
```

2、启动6个Redis容器，搭建一个三主三从的架构

```sh
docker run -d --name redis-node-1 --net host --privileged=true -v /data/redis/share/redis-node-1:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6381

docker run -d --name redis-node-2 --net host --privileged=true -v /data/redis/share/redis-node-2:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6382

docker run -d --name redis-node-3 --net host --privileged=true -v /data/redis/share/redis-node-3:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6383

docker run -d --name redis-node-4 --net host --privileged=true -v /data/redis/share/redis-node-4:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6384

docker run -d --name redis-node-5 --net host --privileged=true -v /data/redis/share/redis-node-5:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6385

docker run -d --name redis-node-6 --net host --privileged=true -v /data/redis/share/redis-node-6:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6386

# --net host 使用宿主的IP和端口
# --cluster-enabled yes 开启Redis集群
# --appendonly yes 开启持久化
```

3、配置主从结构

任意进入一个Redis容器

```sh
# 进入节点1
docker exec -it redis-node-1 /bin/bash
# 配置一主一从
redis-cli --cluster create 192.168.235.151:6381 192.168.235.151:6382 192.168.235.151:6383 192.168.235.151:6384 192.168.235.151:6385 192.168.235.151:6386 --cluster-replicas 1
# --cluster-replicas 1 表示为每个master创建一个slave节点
```

Redis会自动进行分配，比如我的运行结果就是：

```sh
>>> Performing hash slots allocation on 6 nodes... 
# 正在为6个节点分配哈希槽，分配情况如下面三行
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.235.151:6385 to 192.168.235.151:6381
Adding replica 192.168.235.151:6386 to 192.168.235.151:6382
Adding replica 192.168.235.151:6384 to 192.168.235.151:6383
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 910039156a5be3bd6798de420c6cb3771cccedd9 192.168.235.151:6381
   slots:[0-5460] (5461 slots) master
M: 68ddc85b453fc7f28933740f62a1b5721d1c606e 192.168.235.151:6382
   slots:[5461-10922] (5462 slots) master
M: 337571ef1e1d861189bcd3c72fb6f33b8cba64e0 192.168.235.151:6383
   slots:[10923-16383] (5461 slots) master
S: 2fb166b3d74833b37bc792fcb0810feac86c244e 192.168.235.151:6384
   replicates 337571ef1e1d861189bcd3c72fb6f33b8cba64e0
S: d57408e2bc1f968dae58cc155edcf0a326970bde 192.168.235.151:6385
   replicates 910039156a5be3bd6798de420c6cb3771cccedd9
S: 7278e520806f2f3b3c6fa25685a1d9c1e60a0e70 192.168.235.151:6386
   replicates 68ddc85b453fc7f28933740f62a1b5721d1c606e
Can I set the above configuration? (type 'yes' to accept): yes
```

可以看到，系统自己将6381、6382、6383三台机器作为Master，6385、6386、6384依次作为前者的Slave

4、进入Redis内部，查看集群信息

```sh
# 查看集群信息
root@hynisVM:/data# redis-cli -p 6381
127.0.0.1:6381> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_ping_sent:278
cluster_stats_messages_pong_sent:305
cluster_stats_messages_sent:583
cluster_stats_messages_ping_received:300
cluster_stats_messages_pong_received:278
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:583
```

```sh
# 查看节点信息
127.0.0.1:6381> cluster nodes
7278e520806f2f3b3c6fa25685a1d9c1e60a0e70 192.168.235.151:6386@16386 slave 68ddc85b453fc7f28933740f62a1b5721d1c606e 0 1681213675552 2 connected
2fb166b3d74833b37bc792fcb0810feac86c244e 192.168.235.151:6384@16384 slave 337571ef1e1d861189bcd3c72fb6f33b8cba64e0 0 1681213675000 3 connected
68ddc85b453fc7f28933740f62a1b5721d1c606e 192.168.235.151:6382@16382 master - 0 1681213676000 2 connected 5461-10922
337571ef1e1d861189bcd3c72fb6f33b8cba64e0 192.168.235.151:6383@16383 master - 0 1681213677574 3 connected 10923-16383
910039156a5be3bd6798de420c6cb3771cccedd9 192.168.235.151:6381@16381 myself,master - 0 1681213674000 1 connected 0-5460
d57408e2bc1f968dae58cc155edcf0a326970bde 192.168.235.151:6385@16385 slave 910039156a5be3bd6798de420c6cb3771cccedd9 0 1681213677000 1 connected
```

根据以上信息，我们可以构建下图：

![三主三从Redis集群](http://img.yesmylord.cn//image-20230411201140989.png)

### Redis集群的使用

如果我们进入随便一个主Redis，比如进入6381

```sh
root@hynisVM:/data# redis-cli -p 6381
127.0.0.1:6381> set k1 v1 # 存一个k1:v1的数据
(error) MOVED 12706 192.168.235.151:6383
```

报错了，这是因为key值`k1`经过运算后的hash值为12706，这不属于6381的范畴，所以报错，提示你应该在6383存入，可以使用参数`-c`来重定向

```sh
root@hynisVM:/data# redis-cli -p 6381 -c
127.0.0.1:6381> set k1 v1
-> Redirected to slot [12706] located at 192.168.235.151:6383
OK
192.168.235.151:6383> # 可以看到我们被自动切换到了6383
```

可以看到此时将12706重定向到6383上

此外，可以使用以下命令查看集群状态

```sh
redis-cli --cluster check 192.168.235.151:6381
# 出现以下信息
root@hynisVM:/data# redis-cli --cluster check 192.168.235.151:6381
192.168.235.151:6381 (91003915...) -> 1 keys | 5461 slots | 1 slaves.
192.168.235.151:6382 (68ddc85b...) -> 0 keys | 5462 slots | 1 slaves.
192.168.235.151:6383 (337571ef...) -> 1 keys | 5461 slots | 1 slaves.
[OK] 2 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.235.151:6381)
M: 910039156a5be3bd6798de420c6cb3771cccedd9 192.168.235.151:6381
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 7278e520806f2f3b3c6fa25685a1d9c1e60a0e70 192.168.235.151:6386
   slots: (0 slots) slave
   replicates 68ddc85b453fc7f28933740f62a1b5721d1c606e
S: 2fb166b3d74833b37bc792fcb0810feac86c244e 192.168.235.151:6384
   slots: (0 slots) slave
   replicates 337571ef1e1d861189bcd3c72fb6f33b8cba64e0
M: 68ddc85b453fc7f28933740f62a1b5721d1c606e 192.168.235.151:6382
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: 337571ef1e1d861189bcd3c72fb6f33b8cba64e0 192.168.235.151:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: d57408e2bc1f968dae58cc155edcf0a326970bde 192.168.235.151:6385
   slots: (0 slots) slave
   replicates 910039156a5be3bd6798de420c6cb3771cccedd9
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

### Redis容灾主从切换

假如此时我们将6381号停掉，那么他的slave应该会自动切换为master

```sh
# 关闭1号容器
docker stop redis-node-1
# 进入6382
docker exec -it redis-node-2 /bin/bash
```

```sh
root@hynisVM:/data# redis-cli --cluster check 192.168.235.151:6382
Could not connect to Redis at 192.168.235.151:6381: Connection refused
192.168.235.151:6382 (68ddc85b...) -> 0 keys | 5462 slots | 1 slaves.
192.168.235.151:6383 (337571ef...) -> 1 keys | 5461 slots | 1 slaves.
192.168.235.151:6385 (d57408e2...) -> 1 keys | 5461 slots | 0 slaves.
[OK] 2 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.235.151:6382)
M: 68ddc85b453fc7f28933740f62a1b5721d1c606e 192.168.235.151:6382
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 2fb166b3d74833b37bc792fcb0810feac86c244e 192.168.235.151:6384
   slots: (0 slots) slave
   replicates 337571ef1e1d861189bcd3c72fb6f33b8cba64e0
S: 7278e520806f2f3b3c6fa25685a1d9c1e60a0e70 192.168.235.151:6386
   slots: (0 slots) slave
   replicates 68ddc85b453fc7f28933740f62a1b5721d1c606e
M: 337571ef1e1d861189bcd3c72fb6f33b8cba64e0 192.168.235.151:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s
# 关键看这里，6385,变为了master
M: d57408e2bc1f968dae58cc155edcf0a326970bde 192.168.235.151:6385
   slots:[0-5460] (5461 slots) master 
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

即使我们现在将容器1恢复，也不会发生主从切换。

（为了之后叙述方便，我们将容器5再次stop、start，将Master换回容器1）

### Redis主从扩容

现在我们给系统中再加入一个Master-Slave，使之变成4主4从

1、 首先创建新的两个容器

```sh
docker run -d --name redis-node-7 --net host --privileged=true -v /data/redis/share/redis-node-7:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6387
docker run -d --name redis-node-8 --net host --privileged=true -v /data/redis/share/redis-node-8:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6388
```

2、进入6387容器节点内部

```sh
docker exec -it redis-node-7 /bin/bash
```

3、加入集群

```sh
redis-cli --cluster add-node 192.168.235.151:6387 192.168.235.151:6381
# 6387 就是将要作为master新增节点
# 6381 就是原来集群节点里面的领路人，相当于6387想进入组织需要一个介绍人
```

4、此时我们查看集群的情况：发现6387没有任何的插槽！

```sh
oot@hynisVM:/data# redis-cli --cluster check 192.168.235.151:6382
192.168.235.151:6382 (68ddc85b...) -> 0 keys | 5462 slots | 1 slaves.
# 主要看这里！！发现6387没有任何的插槽！
192.168.235.151:6387 (3e431906...) -> 0 keys | 0 slots | 0 slaves.
192.168.235.151:6383 (337571ef...) -> 1 keys | 5461 slots | 1 slaves.
192.168.235.151:6381 (91003915...) -> 1 keys | 5461 slots | 1 slaves.
[OK] 2 keys in 4 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.235.151:6382)
M: 68ddc85b453fc7f28933740f62a1b5721d1c606e 192.168.235.151:6382
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 2fb166b3d74833b37bc792fcb0810feac86c244e 192.168.235.151:6384
   slots: (0 slots) slave
   replicates 337571ef1e1d861189bcd3c72fb6f33b8cba64e0
M: 3e4319061f2943b3335406dbe369cc7e5cc1d9a5 192.168.235.151:6387
   slots: (0 slots) master
S: 7278e520806f2f3b3c6fa25685a1d9c1e60a0e70 192.168.235.151:6386
   slots: (0 slots) slave
   replicates 68ddc85b453fc7f28933740f62a1b5721d1c606e
M: 337571ef1e1d861189bcd3c72fb6f33b8cba64e0 192.168.235.151:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: d57408e2bc1f968dae58cc155edcf0a326970bde 192.168.235.151:6385
   slots: (0 slots) slave
   replicates 910039156a5be3bd6798de420c6cb3771cccedd9
M: 910039156a5be3bd6798de420c6cb3771cccedd9 192.168.235.151:6381
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

5、重新分配hash槽位

```sh
redis-cli --cluster reshard 192.168.235.151:6381
```

![重新分配槽位](http://img.yesmylord.cn//image-20230411214240735.png)

因为现在新加入了一台Master，所以`16384/4=4096`，我们输入4096个。并将插槽分配给6387

6、再次查看

```sh
root@hynisVM:/data# redis-cli --cluster check 192.168.235.151:6382
192.168.235.151:6382 (68ddc85b...) -> 0 keys | 4096 slots | 1 slaves.
192.168.235.151:6387 (3e431906...) -> 1 keys | 4096 slots | 0 slaves.
192.168.235.151:6383 (337571ef...) -> 1 keys | 4096 slots | 1 slaves.
192.168.235.151:6381 (91003915...) -> 0 keys | 4096 slots | 1 slaves.
[OK] 2 keys in 4 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.235.151:6382)
M: 68ddc85b453fc7f28933740f62a1b5721d1c606e 192.168.235.151:6382
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: 2fb166b3d74833b37bc792fcb0810feac86c244e 192.168.235.151:6384
   slots: (0 slots) slave
   replicates 337571ef1e1d861189bcd3c72fb6f33b8cba64e0
# 主要看这里！！！可以看到6387的槽位不是均匀的
M: 3e4319061f2943b3335406dbe369cc7e5cc1d9a5 192.168.235.151:6387
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
S: 7278e520806f2f3b3c6fa25685a1d9c1e60a0e70 192.168.235.151:6386
   slots: (0 slots) slave
   replicates 68ddc85b453fc7f28933740f62a1b5721d1c606e
M: 337571ef1e1d861189bcd3c72fb6f33b8cba64e0 192.168.235.151:6383
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
S: d57408e2bc1f968dae58cc155edcf0a326970bde 192.168.235.151:6385
   slots: (0 slots) slave
   replicates 910039156a5be3bd6798de420c6cb3771cccedd9
M: 910039156a5be3bd6798de420c6cb3771cccedd9 192.168.235.151:6381
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

6387的槽位是`slots:[0-1364],[5461-6826],[10923-12287] `，说明rehash的规则是从每一个Master的插槽，分出一部分给6387，使之凑成4096个！

7、为6387配置slave

```sh
redis-cli --cluster add-node 192.168.235.151:6388 192.168.235.151:6387 --cluster-slave --cluster-master-id 3e4319061f2943b3335406dbe369cc7e5cc1d9a5
# redis-cli --cluster add-node ip:新slave端口 ip:新master端口 --cluster-slave --cluster-master-id 新主机节点ID(此处即为6387的id)
```

此时我们就实现了4主4从的redis分布式集群，如下图所示：

![Redis主从扩容](http://img.yesmylord.cn//image-20230411220405617.png)

可以与之前的图对比查看

### Redis主从缩容

主从缩容与扩容同理，只不过在重新分配槽位的时候，如果想要槽位原路返回，那么需要分配三次，此处为了省事直接将6387的所有槽位分配给6381。

注意：缩容一定要先将slave删掉！

1、将slave节点删除

```sh
# 命令
redis-cli --cluster del-node 192.168.235.151:6388 2a7586688a28e8227d23f21c2c8c9575f00d5bfe

# 输出结果如下
root@hynisVM:/data# redis-cli --cluster del-node 192.168.235.151:6388 2a7586688a28e8227d23f21c2c8c9575f00d5bfe
>>> Removing node 2a7586688a28e8227d23f21c2c8c9575f00d5bfe from cluster 192.168.235.151:6388
>>> Sending CLUSTER FORGET messages to the cluster...
>>> Sending CLUSTER RESET SOFT to the deleted node.
```

2、将Master的所有槽位分配给其他Master

```sh
redis-cli --cluster reshard 192.168.235.151:6381
```

![主从缩容](http://img.yesmylord.cn//image-20230411221335692.png)

将6387的所有槽位分配给6381

3、删除节点6381

```sh
redis-cli --cluster del-node 192.168.235.151:6387 3e4319061f2943b3335406dbe369cc7e5cc1d9a5
```

4、查看集群状态

```sh
root@hynisVM:/data# redis-cli --cluster check 192.168.235.151:6382
192.168.235.151:6382 (68ddc85b...) -> 0 keys | 4096 slots | 1 slaves.
192.168.235.151:6383 (337571ef...) -> 1 keys | 4096 slots | 1 slaves.
192.168.235.151:6381 (91003915...) -> 1 keys | 8192 slots | 1 slaves.
[OK] 2 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.235.151:6382)
M: 68ddc85b453fc7f28933740f62a1b5721d1c606e 192.168.235.151:6382
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: 2fb166b3d74833b37bc792fcb0810feac86c244e 192.168.235.151:6384
   slots: (0 slots) slave
   replicates 337571ef1e1d861189bcd3c72fb6f33b8cba64e0
S: 7278e520806f2f3b3c6fa25685a1d9c1e60a0e70 192.168.235.151:6386
   slots: (0 slots) slave
   replicates 68ddc85b453fc7f28933740f62a1b5721d1c606e
M: 337571ef1e1d861189bcd3c72fb6f33b8cba64e0 192.168.235.151:6383
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
S: d57408e2bc1f968dae58cc155edcf0a326970bde 192.168.235.151:6385
   slots: (0 slots) slave
   replicates 910039156a5be3bd6798de420c6cb3771cccedd9
# 主要看这里！！6381的槽位
M: 910039156a5be3bd6798de420c6cb3771cccedd9 192.168.235.151:6381
   slots:[0-6826],[10923-12287] (8192 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

6381的槽位发生了变化，可以看到将原本6387的槽位全部合并到了6381

![主从缩容](http://img.yesmylord.cn//image-20230411222051250.png)

# DockerFile

## DockerFile

> DockerFile就是Docker的脚本文件：DockerFile的执行流程就相当于我们对镜像进行修改后，commit，然后不断重复这个过程。

[对于其中的关键字可以查看官网](https://docs.docker.com/engine/reference/builder/)

常见关键字：

- `FROM`：从哪一个源镜像构建而来，Dockerfile文件第一行就需要时FROM
- `MAINTAINER`：镜像维护者的姓名和邮箱地址
- `RUN`：在docker build时执行命令，有两种格式
  - shell格式：`RUN <命令>`
  - exec格式：`exec [命令, 参数, 参数]`
- `EXPOSE`：对外暴露出端口
- `WORKDIR`：创建容器后，终端默认登录进来的工作目录
- `USER`：以什么样的用户去执行命令，默认为`root`
- `ENV`：在构建镜像过程中设置环境变量
  - ENV指定的环境变量，在RUN指令中可以使用
- `ADD`：将宿主机目录下的文件拷贝进镜像而且会自动处理URL和解压tar压缩包。
- `COPY`：与ADD相似，拷贝文件到目录。ADD=COPY+解压
- `VOLUME`：数据卷
- `CMD`：指定容器启动后要执行的命令，格式与RUN一样
  - **注意**：Dockerfile可以有多个CMD指令，但是只有最后一个会生效，CMD会被`docker run`指令之后的参数替换
  - 与RUN的区别在于：**RUN在docker build时执行；CMD在docker run时执行**
- `ENTRYPOINT`：用来指定一个容器启动时要执行的命令
  - **注意**：Dockerfile可以有多个ENTRYPOINT指令，但是只有最后一个会生效
  - 搭配CMD命令使用的话，CMD命令可以给ENTRYPOINT传递参数

这里举一个小例子：

假设已通过Dockerfile构建了`nginx:test`

```dockerfile
FROM nginx

ENTRYPOINT ["nginx", "-c"]
CMD ["/etc/nginx/nginx.conf"]
```

如果要运行上述的`dockerfile`文件的话，输入

```sh
docker run nginx:test
# 实际上执行的命令是
nginx -c /etc/nginx/nginx.conf

# 如果在执行中自己指定了参数
docker run nginx:test -c /etc/nginx/temp.conf
# 那么实际上会替换原有的参数
nginx -c /etc/nginx/temp.conf
```

## 一个Dockerfile的实例

现在使用Dockerfile构建一个可以使用`vim + ifconfig + JDK8`的ubuntu镜像

1、创建一个Dockerfile文件

```dockerfile
FROM ubuntu
MAINTAINER hynis
 
ENV MYPATH /usr/local
WORKDIR $MYPATH

#更新索引
RUN apt-get update
#安装vim编辑器
RUN apt-get install -y vim
#安装ifconfig命令查看网络IP
RUN apt-get install -y net-tools
#安装java8及lib库
#RUN apt-get install -y  glibc.i686
RUN mkdir /usr/local/java
#ADD 是相对路径jar,把jdk-8u171-linux-x64.tar.gz添加到容器中,安装包必
须要和Dockerfile文件在同一位置
ADD jdk-8u171-linux-x64.tar.gz /usr/local/java/
#配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH

EXPOSE 80

CMD echo $MYPATH
CMD echo "success--------------ok"
CMD /bin/bash
```

2、build镜像

```sh
docker build -t hynix:1.0 .
```

3、进入镜像查看软件是否安装成功

```sh
docker run -it hynix:1.0
```

## 虚悬镜像

> 什么是**虚悬镜像**？

虚悬镜像指：仓库名、标签都是`<none>`的镜像，英文名为`dangling image`

Docker在拉取镜像的时候可能会出错，虚悬镜像没有任何用处，可以删除

# Docker网络

> 在Docker启动后，Linux系统中会多一个docker0的地址，这个docker0就是Docker自己配置的一个网络

## VMware的网络配置

在介绍Docker的网络配置前，有必要先介绍一下VMware中的模式：

[可以参考此篇知乎](https://zhuanlan.zhihu.com/p/32948325)

- **桥接模式**：虚拟机的虚拟网卡与主机的物理网卡直接交接；直白一点的说法是，网络中出现了一台可以完全独立的计算机，他可以与网络中的其他终端进行交互。
- **NAT模式**：网络地址转换（Network Address Translation），主机会给所有的虚拟机建一个网络，虚拟机内部可以直接交互，虚拟机向外部的通信需要通过主机转发
- **仅主机模式**：一种比NAT模式更加封闭的的网络连接模式，它将创建完全包含在主机中的专用网络。仅主机模式的虚拟网络适配器仅对主机可见，并在虚拟机和主机系统之间提供网络连接

## Docker的网络模式

此处略微介绍，下面会详细阐述

- `bridge`：为一个容器分配、设置IP，并将容器连接到一个`docker0`
- `host`：容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口
- `none`：【了解】容器有独立的Network namespace，但并没有对其进行任何的网络设置
- `container`：【了解】新创建的容器不会创建自己的网卡和配置自己的IP，而是和一个指定的容器共享IP、端口范围。

启动容器时，使用参数`--net`就可以指定模式，例如启动一个host容器

```sh
docker run -it --net host hynix:1.0 /bin/bash
```

我们可以通过`docker inspect 容器ID`来查看这个容器的网络状况：

```sh
"Networks": {
    "bridge": {
        "IPAMConfig": null,
        "Links": null,
        "Aliases": null,
        "NetworkID": "e9a47bb319e6523eb731752a4be2a495e757aa8d22e8ed46e7c0e3039fc373a0",
        "EndpointID": "e9b47c4014b839ac36b2197ebbda365314451acf7742c58363efd1fe3df954ec",
        "Gateway": "172.17.0.1",
        "IPAddress": "172.17.0.2",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "MacAddress": "02:42:ac:11:00:02",
        "DriverOpts": null
    }
}
```

可以看到我们的网络模式是bridge的，并且我们的IP地址和网关都有分配

注意：**容器的IP可能是会变化的**

## bridge模式

为一个容器分配、设置IP，并将容器连接到一个`docker0`

Docker服务默认会创建一个 docker0网桥（其上有一个 **docker0 内部接口**），该桥接网络的名称为docker0，它在**内核层连通了其他的物理或虚拟网卡**，这就将所有容器和本地主机都放到同一个物理网络。

Docker 默认指定了 docker0 接口 的 IP 地址和子网掩码，让主机和容器之间可以通过网桥相互通信。

如图所示：

![Docker宿主机与容器的网络关系](http://img.yesmylord.cn//image-20230412192012012.png)

- 宿主机网卡eth0（即网卡ens33）
- Docker运行后，默认构建一个桥接的网络docker0，其上有很多接口`veth`（虚拟eth）
- Docker会给每一个容器分配一个docker0网段内的IP地址（称为container_ip），而且每个容器都有网卡`eth0`与`veth`相互配对（这样的一对称为`veth pair`）
- docker0就相当于一个交换机，所有容器通过docker0进行通信
- 宿主机与容器的交互也需要通过docker0

使用`ip addr`就可以看到docker0的信息

```sh
docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
# BROADCAST 网卡可以发广播包
# MULTICAST 网卡可以发多播包
# UP 表示网卡处于启动状态
# LOWER_UP L1处于启动状态（L1指第一层，即物理层，即插着网线）
    link/ether 02:42:71:5f:92:b8 brd ff:ff:ff:ff:ff:ff
	# link/ether MAC地址
	# IPv4地址
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    # IPv6地址
    inet6 fe80::42:71ff:fe5f:92b8/64 scope link 
       valid_lft forever preferred_lft forever
```

我们也可以启动一个容器，来验证：

```sh
# 容器 ip addr查询
# 注意看下面 if36
35: eth0@if36: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

```sh
# 宿主机 ip addr查询
36: vethf7d2553@if35: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 5e:f8:66:59:05:e4 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::5cf8:66ff:fe59:5e4/64 scope link 
```

可以看到宿主机提供了`veth@if35` 与容器内的`veth@if36`配对，达成内部通信

## host模式

容器使用宿主机的IP与端口

![host模式](http://img.yesmylord.cn//image-20230412194418493.png)

如果启动一个容器以`--net host`的方式启动，那么它的IP与端口就与宿主机的是同一份

## none模式

【不常用】此种模式将只有`lo`回环地址，需要自己进行配置

## container模式

此种模式的容器将会与一个指定的容器的IP与端口一致，如图所示：

![container模式](http://img.yesmylord.cn//image-20230412195411130.png)

可以看到有一个容器没有创建自己的网卡，而是直接使用另一个容器的网卡。

## 自定义网络模式

前面我们说到：**容器的IP可能是会变化的**

这意味着，容器上面的服务如果我们使用IP去连接的话，有时候会出现错误，此时我们可以进行自定义网络，保证服务调用正常

1、创建docker网络

```sh
docker network create hynisNet
# 此时的网络默认驱动还是bridge模式
```

2、将容器使用该网络进行创建

```sh
docker run -d -p 8081:8080 --network hynisNet --name service1 hynix
docker run -d -p 8082:8080 --network hynisNet --name service2 hynix
```

使用`--network`就可以在指定网络中创建容器了

3、测试使用域名访问服务，是可以相互ping通的

```sh
ping service1
ping service2
```

> 为什么默认的桥接模式IP和主机名关系有时候会改变，但是自定义模式却能维护呢？

因为自定义网络自身维护好了IP与主机名的对应关系

## Docker整体的网络架构

如图所示：

![Docker网络架构](http://img.yesmylord.cn//image-20230412203858417.png)

Docker 是一个 **C/S 模式**的架构，运行的基本流程为：

1.  用户使用`Docker Client`与`Docker Daemon`建立通信
2. `Docker Daemon` 作为 Docker 架构中的**主体部分**，首先提供 `Docker Server` 的功能使其可以接受 Docker Client 的请求。
3. `Docker Engine` 执行 Docker 内部的一系列工作Job
4. Job 的运行过程中，当需要容器镜像时，则从`Docker Registry`中下载镜像，并通过**镜像管理驱动Graph driver**将下载镜像以`Graph`的形式存储。
5. 当需要为 Docker 创建网络环境时，通过**网络管理驱动 Network driver** 创建并配置 Docker 容器网络环境。
6. 当需要限制 Docker 容器运行资源或执行用户指令等操作时，则通过 `Execdriver`来完成。
7. `Libcontainer`是一项独立的容器管理包，`Network driver`以及`Execdriver`都是通过`Libcontaine`r来实现具体对容器进行的操作。

# Docker-compose

## Docker-compose

类似于最近很流行的k8s，是docker官方提供的容器编排工具，适用于对大量的容器进行管理。

Compose允许用户通过一个单独的`docker-compose.yml`模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）

一个简单的`docker-compose.yml`文件如下：这个文件，我们构建了搭建微服务的基本结构，首先我们启动了redis、mysql、以及测试的项目，并将三个容器放在了hynisNet网络中，只需要使用`docker-compose up`就可以一键启动整个微服务项目

```yml
version: "3"
services:
  microService:
    image: dockerTest:1.6
    container_name: ms01
    ports:
      - "6001:6001"
    volumes:
      - /app/microService:/data
    networks: 
      - atguigu_net 
    depends_on: # 此处依赖了redis与mysql
      - redis
      - mysql
  redis:
    image: redis:6.0.8
    ports:
      - "6379:6379"
    volumes:
      - /app/redis/redis.conf:/etc/redis/redis.conf
      - /app/redis/data:/data
    networks: 
      - hynisNet
    command: redis-server /etc/redis/redis.conf
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: '123456'
      MYSQL_ALLOW_EMPTY_PASSWORD: 'no'
      MYSQL_DATABASE: 'db2021'
      MYSQL_USER: 'zzyy'
      MYSQL_PASSWORD: 'zzyy123'
    ports:
       - "3306:3306"
    volumes:
       - /app/mysql/db:/var/lib/mysql
       - /app/mysql/conf/my.cnf:/etc/my.cnf
       - /app/mysql/init:/docker-entrypoint-initdb.d
    networks:
      - hynisNet
    command: --default-authentication-plugin=mysql_native_password #解决外部无法访问
networks: 
   hynisNet: 
```



















