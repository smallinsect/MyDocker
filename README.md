[TOC]

# MyDocker

学习网址

https://www.runoob.com/docker/ubuntu-docker-install.html

http://c.biancheng.net/view/3118.html

# 3. 安装

1. yum 更新最新的包

```
yum update
```

2. 安装需要的软件包，yum-util 提供yum-config-manager功能，例外两个是devicemapper驱动依赖

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

3. 设置yum源为阿里云

```
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

4. 安装docker

```
yum -y install docker-ce
```

5. 查看docker版本

```
docker -v
```

## 3.2 设置ustc的镜像

 	ustc是老牌的linux镜像服务提供者了，还在遥远的Ubuntu 5.04版本的时候就在用。ustc的docker镜像加速度很快。ustc docker mirror的优势之一就是不需要注册，是真正的公共服务。

​	https://lug.ustc.edu.cn/wiki/mirrors/help/docker/

编辑该文件

```
mkdir -p /etc/docker
vim /etc/docker/daemon.json
```

在该文件中输入如下内容

```
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

或者

````
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
````

## 3.3 Docker启动与停止

启动docker

```
systemctl start docker
```

停止docker

```
systemctl stop docker
```

查看docker状态

```
systemctl status docker
看到
Active: active (running)  表示正在运行
```

查看docker信息

```
docker info
```

查看docker操作文档

```
docker --help
```

开机自动启动docker

```
systemctl enable docker
```

# 4. 常用命令

## 4.1 镜像相关命令

### 4.1.1 查看镜像

```
docker images
```

- `REPOSITORY`: 镜像名

- `TAG`:  镜像标签
- `IMAGE ID`: 镜像ID
- `CREATED`: 镜像的创建日期（不是获取该镜像的日期）
- `SIZE`: 镜像大小

这些镜像都是存储在Docker宿主机的`/var/lib/docker`目录下

### 4.1.2 搜索镜像

如果你需要从网络中查找需要的镜像，可以通过以下命令搜索

```
docker search 镜像名
```

- `NAME`: 仓库名
- `DESCRIPTION`: 镜像描述
- `STARS`: 用户评价，反应一个镜像的受欢迎程度
- `OFFICIAL`:  是否官方
- `AUTOMATED`: 自动构建，表示该镜像有Docker Hub自动构建流程创建的

### 4.1.3 拉取镜像

拉取镜像就是从中央仓库中下载镜像到本地

```
docker pull 镜像名称
```

例如，我要下载centos7镜像

```
docker pull centos:7
```

### 4.1.4 删除镜像

按镜像ID删除镜像

```
docker rmi 镜像ID
```

删除所有镜像

```
docker rmi `docker images -q`
```

## 4.2 容器相关命令

### 4.2.1 查看容器

查看正在运行的容器

```
docker ps
```

查看所有容器

```
docker ps -a
```

查看最后一次运行的容器

```
docker ps -l
```

查看停止的容器

```
docker ps -f status=exited
```

### 4.2.2 创建与启动容器

创建容器常用的参数说明：

创建容器命令：

```
docker run
```

`-i`:表示运行容器

`-t`:表示容器启动后进入其命令行。加入这两个参数后，容器创建就能登录进去。即分配一个伪终端。

`--name`:为创建的容器命令

`-v`:表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个-v做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上。

`-d`:在run后面加上-d参数，则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加-i-t两个参数，创建后就会自动进去容器）

`-p`:表示端口映射，前者是宿主机端口，后者是容器内的映射端口，可以使用多个p做多个端口映射

1. 交互式方式创建容器

```
docker run -it --name=容器名称 镜像名称：标签/bin/bash
```

这时我们通过ps命令查看，发现可以看到启动的容器，状态启动

退出当前容器

```
exit
```

创建一个centos:7系统 守护式 容器，名称mycentos

```
docker run -di --name=mycentos centos:7
```

进入容器

```
docker exec -it mycentos /bin/bash
```

启动容器

```
docker start 容器ID
```

停止容器

```
docker stop 容器ID
```

删除容器

```
docker rm 镜像ID
```

拷贝文件

本地文件拷贝到容器中

```
docker cp 本地文件 容器ID:/root
```

容器中的文件拷贝到本地种

```
docker cp 容器名称:/文件路径/文件名 本地文件名
```

### 4.2.5 目录挂载

我们可以在创建容器的时候，将宿主的目录与容器内的目录进行映射，这样我们就可以通过修改宿主主机某个目录的文件从而去影响容器

创建容器 添加 -v 参数后边为宿主目录：容器目录，例如：

```
docker run -di -v /user/local/myhtml:/user/local/myhtml --name=mycentos3  centos:7
```

如果你共享的是多级的目录，可能会出现权限不足的提示

这是因为CentOS7中的安全模块selinux把权限禁掉了，我们需要添加参数`--privileged=true`来解决挂载的目录没有权限的问题

### 4.2.6 查看容器IP地址

我们可以通过以下命令查看容器运行的各种数据

```
docker inspect 容器名称（容器ID）
```

也可以直接执行下面的命令直接输出IP地址

```
docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称（容器ID）
```

# 5. 应用部署

## 5.1. MySQL部署

拉取MySQL镜像

 ```
docker pull mysql:5.7
 ```

创建MySQL容器

本地端口：容器端口 -e 直接默认密码

```
docker run -di --name=mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql:5.7
```

进入MySQL容器

```
docker exec -it mysql /bin/bash
```

连接MySQL

```
mysql -uroot -proot --default-character-set=utf8
```

## 5.2.Nginx部署

拉取Nginx镜像

```
docker pull nginx
```

创建Nginx容器

```
docker run -di --name=nginx -p 80:80 nginx
```

## 5.3.Redis部署

拉取Redis镜像

```
docker pull redis
```

创建Redis容器

```
docker run -di --name=redis -p 6379:6379 redis
```

## 5.4.RabbitMQ部署

拉取镜像

```
docker pull rabbitmq:3.7.12
```

创建镜像

```
docker run -di --name=rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq:3.7.12
```

进入镜像

```
docker exec -it rabbitmq /bin/bash
```

安装UI插件

```
rabbitmq-plugins enable rabbitmq_management
```

退出

```
exit
```

## 5.5.Elasticsearch部署

拉取镜像

```
docker pull elasticsearch:7.5.0
```

修改虚拟内存大小

```
sysctl -w vm.max_map_count=262144
```

创建容器

```

```

进入容器

```
docker exec -it elasticsearch /bin/bash
```

安装组件 elasticsearch-plugin

```

```

## 5.6.Zookeeper部署

拉取镜像

```
docker pull zookeeper:3.4.13
```

创建容器

```
docker run -di --name=zookeeper -p 2181:2181 zookeeper:3.4.13
```

# 6. 迁移和部署

## 6.1. 容器保存为镜像

我们可以通过以下命令将容器保存为镜像

```
docker commit redis myredis
```

## 6.2. 镜像备份

我们可以通过以下命令将镜像保存为tar文件

```
docker save -o myredis.tar myredis
```

## 6.3. 镜像恢复与迁移

首先我们先删除myredis镜像 然后执行此命令进行恢复

```
docker load -i myredis.tar
```

`-i` 输入的文件

执行后再次查看镜像，可以看到镜像已经恢复

# 7. Dockerfile

## 7.1. 什么是Dockerfile

Dockerfile是由一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新的镜像。

- 对于开发人员：可以为开发团队提供一个完全一致的开发环境
- 对于测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个新的镜像开始工作了
- 对于运维人员：在部署时，可以实现应用的无缝移植

## 7.2. 常用命令

| 命令                               | 作用                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| FROM image_name:tag                | 定义了使用哪个基础镜像启动构建流程                           |
| MAINTANIER user_name               | 声明镜像的创建者                                             |
| ENV key value                      | 设置环境变量（可以写多条）                                   |
| RUN command                        | 是Dockerfile的核心部件（可以写多条）                         |
| ADD source_dir/file dest_dir/file  | 将宿主机的文件复制到容器内，如果是一个压缩文件，将会在复制后自动解压 |
| COPY source_dir/file dest_dir/file | 和ADD相识，但是如果有压缩文件并不能解压                      |
| WORKDIR path_dir                   | 设置工作目录                      |

# 8. Docker私有仓库

## 8.1. 私有仓库搭配与配置

拉取私有仓库镜像

```
docker pull registry
```

启动私有仓库容器

```
docker run -di --name=registry -p 5000:5000 registry
```

打开浏览器输入地址http://宿主ip:5000/v2/_catalog看到`{"repositories":[]}`表示私有仓库搭建成功并且内容为空

修改daemon.json

```
vim /etc/docker/daemon.json
```

添加以下内容，保存退出

```
{
	"insecure-registries":["宿主ip:50000"]
}
```

此步用于让docker信任私有仓库地址

重启docker服务

打上标签

```
docker tag jdk1.8 192.168.10.101:5000/jdk1.8
```

查看镜像

```
docker images
```

把镜像上传到私有仓库

```
docker push 192.168.10.101:5000/jdk1.8
```

# 9. DockerMaven插件

微服务部署有两种方法：

- 手动部署：首先基于源码打包生成jar包（或war包），将jar包（或war包）上传至虚拟机并拷贝至JDK容器
- 通过Maven插件自动部署

对于数量众多的微服务，手动部署无疑是非常麻烦的做法，并且容器出错。所以我们可以学习如何自动部署，这也是企业实际开发中经常使用的方法。

Maven插件自动部署步骤：

## 9.1. 修改宿主机的docker配置，让其可以远程访问

```
vim /lib/systemd/system/docker.service
```

修改以ExecStart开头的行

```
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375
```

修改如下



刷新配置

```
systemctl daemon-reload
```

重启docker

```
systemctl restart docker
```




















