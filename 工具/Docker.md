## 基本概念

### 镜像

 **Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）** 

 镜像不包含任何动态数据，其内容在构建之后也不会被改变。 

### 容器

 镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 实例 一样，镜像是静态的定义，**容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等** 。

 **容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。前面讲过镜像使用的是分层存储，容器也是如此。**  

### 仓库

**集中存放镜像的地方**

 镜像构建完成后，可以很容易的在当前宿主上运行，但是， **如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，Docker Registry 就是这样的服务。** 

## 常用命令
### 常用
```bash
docker version # 查看docker版本
docker images # 查看所有已下载镜像，等价于：docker image ls 命令
docker container ls #    查看所有容器
docker ps #查看正在运行的容器
docker image prune # 清理临时的、没有被使用的镜像文件。-a, --all: 删除所有没有用的镜像，而不仅仅是临时文件；
```
### 镜像操作
```bash
docker search mysql # 查看mysql相关镜像
docker pull mysql:5.7 # 拉取mysql镜像
docker image ls # 查看所有已下载镜像 
docker rmi [image] #删除镜像 名字或者id

#通过 docker inspect 命令，我们可以获取镜像的详细信息，其中，包括创建者，各层的数字摘要等。
docker inspect docker.io/mysql:5.7
```

## 容器操作

```bash
docker run -itd --name redis-test -p 6379:6379 redis
```

## 重启Docker

```bash
systemctl daemon-reload 
systemctl restart docker
```



## 卸载和安装docker

```bash
yum list installed|grep docker
rpm -qa|grep docker
yum list installed | grep docker
yum –y remove docker.x86_64
rm -rf /var/lib/docker
dokcer 

sudo yum install docker-ce docker-ce-cli containerd.io

1、Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 。

通过 uname -r 命令查看你当前的内核版本

 $ uname -r
2、使用 root 权限登录 Centos。确保 yum 包更新到最新。

$ sudo yum update
3、卸载旧版本(如果安装过旧版本的话)

$ sudo yum remove docker  docker-common docker-selinux docker-engine
4、安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
5、设置yum源

$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
 

6、可以查看所有仓库中所有docker版本，并选择特定版本安装

$ yum list docker-ce --showduplicates | sort -r


7、安装docker

$ sudo yum install docker-ce  #由于repo中默认只开启stable仓库，故这里安装的是最新稳定版17.12.0
$ sudo yum install <FQPN>  # 例如：sudo yum install docker-ce-17.12.0.ce
 

8、启动并加入开机启动

$ sudo systemctl start docker
$ sudo systemctl enable docker
9、验证安装是否成功(有client和service两部分表示docker安装启动都成功了)

$ docker version


 

 二、问题
1、因为之前已经安装过旧版本的docker，在安装的时候报错如下：

复制代码
复制代码
Transaction check error:
  file /usr/bin/docker from install of docker-ce-17.12.0.ce-1.el7.centos.x86_64 conflicts with file from package docker-common-2:1.12.6-68.gitec8512b.el7.centos.x86_64
  file /usr/bin/docker-containerd from install of docker-ce-17.12.0.ce-1.el7.centos.x86_64 conflicts with file from package docker-common-2:1.12.6-68.gitec8512b.el7.centos.x86_64
  file /usr/bin/docker-containerd-shim from install of docker-ce-17.12.0.ce-1.el7.centos.x86_64 conflicts with file from package docker-common-2:1.12.6-68.gitec8512b.el7.centos.x86_64
  file /usr/bin/dockerd from install of docker-ce-17.12.0.ce-1.el7.centos.x86_64 conflicts with file from package docker-common-2:1.12.6-68.gitec8512b.el7.centos.x86_64
复制代码
复制代码
2、卸载旧版本的包

$ sudo yum erase docker-common-2:1.12.6-68.gitec8512b.el7.centos.x86_64


3、再次安装docker

$ sudo yum install docker-ce
 

⚠️：国外镜像一般很难访问，建议配置阿里云镜像。
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

## Redis

```bash
docker pull redis:5.0.7
docker run -itd --name redis -p 6379:6379 redis:5.0.7
docker exec -it redis /bin/bash

docker start redis
默认没有配置文件
/root/redis/redis01/conf/redis.conf 中daemonize=NO。非后台模式，如果为YES 会的导致 redis 无法启动，因为后台会导致docker无任务可做而退出。

docker run -p 6379:6379 
--name myredis 
-v /usr/local/docker/redis.conf:/etc/redis/redis.conf 
-v /usr/local/docker/data:/data 
-d redis redis-server /etc/redis/redis.conf 
--appendonly yes  #开启持久化

systemctl stop firewalld.service
```

