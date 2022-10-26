官网：https://docs.docker.com/

教程1：https://cloud.tencent.com/developer/article/1885678

教程2：https://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html

教程3：https://yeasy.gitbook.io/docker_practice/



# 概念

Docker是一个虚拟环境容器，可以将你的开发环境、代码、配置文件等一并打包到这个容器中，并发布和应用到任意平台中。



Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离。或者说，在正常进程的外面套了一个[保护层](https://opensource.com/article/18/1/history-low-level-container-runtimes)。对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。



# 安装

Linux：https://docs.docker.com/engine/install/

```shell
docker version
docker --help
sudo service docker start 
```



windows：https://zhuanlan.zhihu.com/p/441965046

安装问题：https://superuser.com/questions/1584710/docker-wsl-2-installation-is-incomplete

操作同linux ，也是使用命令行



# Image

镜像（Image）：是一个包含有文件系统的面向Docker引擎的只读模板。

镜像仓库（Repository）：仓库是存放镜像的地方，一般每个仓库存放一类镜像，每个镜像利用tag进行区分。

注册服务器（Registry）：注册服务器是存放仓库的地方。



docker 镜像网址 https://hub.docker.com/

```shell
docker search centos    # 查看centos镜像是否存在
docker pull centos    # 利用pull命令获取镜像
docker images    # 查看当前系统中的images信息
docker rmi image_name/image_id	#删除镜像
docker save -o centos.tar xianhu/centos:git    # 保存镜像
docker load -i centos.tar    # 加载镜像
```

自建镜像（容器转化为镜像）

```shell
docker commit -m "centos with git" -a "qixianhu" 72f1a8a0e394 xianhu/centos:git
#-m指定说明信息
#-a指定用户信息
#72f1a8a0e394代表容器的id
#xianhu/centos:git指定目标镜像的用户名、仓库名和 tag 信息
```

自建镜像（Dockerfile）

```shell
# 说明该镜像以哪个镜像为基础
FROM centos:latest

# 构建者的基本信息
MAINTAINER xianhu

# 在build这个镜像时执行的操作
RUN yum update
RUN yum install -y git

# 拷贝本地文件到镜像中
COPY ./* /usr/share/gitdir/
```

```shell
docker build -t="xianhu/centos:gitdir" .
#-t用来指定新镜像的用户信息、tag
#点是Dockerfile路径
```

镜像更新：更新Dockerfile



# Container

容器（Container）：容器是镜像创建的应用实例，可以创建、启动、停止、删除容器，各个容器之间是是相互隔离的，互不影响。每个容器就是一个系统，一般包含一个应用。



```shell
docker run -it centos:latest /bin/bash    # 创建并启动一个容器
#exit退出，则容器的状态处于Exit。如果想让容器一直运行，可以使用快捷键 ctrl+p+q 退出
```

```shell
docker ps -a			#查看运行中的容器
docker rm container_name/container_id	#删除容器
```

```shell
docker start container_name/container_id	#启动容器
docker stop container_name/container_id		#停止容器
docker container kill [containID]			#停止容器
docker restart container_name/container_id	#重启容器
docker attach container_name/container_id	#进入容器
# 类似创建一个ssh终端访问容器，exit不会退出容器
docker container exec -it [containerID] /bin/bash #进入容器
docker container ls --all					#列出容器
```

```shell
#用于从正在运行的 Docker 容器里面，将文件拷贝到本机
docker container cp [containID]:[/path/to/file] .  
```



# 仓库

Docker官方维护了一个DockerHub的公共仓库。除了从上边下载镜像之外，我们也可以将自己自定义的镜像发布（push）到DockerHub上。



```shell
# 问https://hub.docker.com/ 注册
# 登录DockerHub，输入用户名、密码
docker login
docker push xianhu/centos:git    # 推送镜像
```











