---
title: 租云服务器及配docker环境
comments: true
mathjax: true
tags:
  - docker
  - Linux
  - 常用命令
categories:
  - Linux 基础
date: 2021-10-17 16:59:29
---
#### 音乐小港
{% meting "1451525481" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# 租云服务器及配docker环境
## 概述
平台的作用:

1. 存放我们的docker容器，让计算跑在云端。
1. 获得公网IP地址，让每个人可以访问到我们的服务。

任选一个云平台即可，推荐配置：

1. 1核 2GB（后期可以动态扩容，前期配置低一些没关系）
1. 网络带宽采用按量付费，最大带宽拉满即可（费用取决于用量，与最大带宽无关）
1. 系统版本：ubuntu 20.04 LTS（推荐用统一版本，避免后期出现配置不兼容的问题）

## 租云服务器及安装docker（以阿里云为例）
### 阿里云

阿里云地址：[点我去](https://www.aliyun.com/)

1. 创建工作用户`acs`并赋予`sudo`权限 （名字可自定义）

登录到新服务器。打开 `Terminal`，然后：
```
ssh root@xxx.xxx.xxx.xxx  # xxx.xxx.xxx.xxx替换成新服务器的公网IP
```
创建`acs`用户：
```
adduser acs  # 创建用户acs
usermod -aG sudo acs  # 给用户acs分配sudo权限
```
2. 配置免密登录方式

退回 `Terminal`，然后配置`acs`用户的别名和免密登录，可以参考 [ssh节的ssh登录]。

3. 配置新服务器的工作环境

将 `Terminal`的配置传到新服务器上：
```
scp .bashrc .vimrc .tmux.conf server_name:  # server_name需要换成自己配置的别名
```
4. 安装tmux和docker

登录自己的服务器，然后安装`tmux`：
```
sudo apt-get update
sudo apt-get install tmux
```
打开`tmux`。（养成好习惯，所有工作都在tmux里进行，防止意外关闭终端后，工作进度丢失）

然后在`tmux`中根据 [docker安装教程](https://docs.docker.com/engine/install/ubuntu/) 安装`docker`即可。

> **主要命令**
> ```
> sudo apt-get update
>
> sudo apt-get install \
>    apt-transport-https \
>    ca-certificates \
>    curl \
>    gnupg \
>    lsb-release
>
> curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
>
> echo \
>  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
>  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
>
>  sudo apt-get update
>
>  sudo apt-get install docker-ce docker-ce-cli containerd.io
> ```
> 查看docker
> ```
> docker --version
> ```

## docker教程
### 将当前用户添加到docker用户组
为了避免每次使用`docker`命令都需要加上`sudo`权限，可以将当前用户加入安装中自动创建的`docker`用户组(可以参考[官方文档](https://docs.docker.com/engine/install/linux-postinstall/))：
```
sudo usermod -aG docker $USER
```

### 镜像（images）
1. `docker pull ubuntu:20.04`：拉取一个镜像
1. `docker images`：列出本地所有镜像
1. `docker image rm ubuntu:20.04` 或 `docker rmi ubuntu:20.04`：删除镜像`ubuntu:20.04`
1. `docker [container] commit CONTAINER IMAGE_NAME:TAG`：创建某个`container`的镜像
1. `docker save -o ubuntu:20.04.tar ubuntu:20.04`：将镜像`ubuntu:20.04`导出到本地文件`ubuntu:20.04.tar`中
1. `docker load -i ubuntu:20.04.tar`：将镜像`ubuntu:20.04`从本地文件`ubuntu:20.04.tar`中加载出来

### 容器(container)
1. `docker [container] create -it ubuntu:20.04`：利用镜像`ubuntu:20.04`创建一个容器。
1. `docker ps -a`：查看本地的所有容器
1. `docker [container] start CONTAINER`：启动容器
1. `docker [container] stop CONTAINER`：停止容器
1. `docker [container] restart CONTAINER`：重启容器
1. `docker [contaienr] run -itd ubuntu:20.04`：创建并启动一个容器
1. `docker [container] attach CONTAINER`：进入容器
    -  先按`Ctrl+p`，再按`Ctrl+q`可以挂起容器  
1. `docker [container] exec CONTAINER COMMAND`：在容器中执行命令
1. `docker [container] rm CONTAINER`：删除容器
1. `docker container prune`：删除所有已停止的容器
1. `docker export -o xxx.tar CONTAINER`：将容器`CONTAINER`导出到本地文件`xxx.tar`中
1. `docker import xxx.tar image_name:tag`：将本地文件`xxx.tar`导入成镜像，并将镜像命名为`image_name:tag`
1. `docker export/import`与`docker save/load`的区别：
    -  `export/import`会丢弃历史记录和元数据信息，仅保存容器当时的快照状态
    -  `save/load`会保存完整记录，体积更大
1. `docker top CONTAINER`：查看某个容器内的所有进程
1. `docker stats`：查看所有容器的统计信息，包括CPU、内存、存储、网络等信息
1. `docker cp xxx CONTAINER:xxx` 或 `docker cp CONTAINER:xxx xxx`：在本地和容器间复制文件
1. `docker rename CONTAINER1 CONTAINER2`：重命名容器
1. `docker update CONTAINER --memory 500MB`：修改容器限制

### 实战
进入`AC Terminal`，然后：
```
scp /var/lib/acwing/docker/images/docker_lesson_1_0.tar server_name:  # 将镜像上传到自己租的云端服务器
ssh server_name  # 登录自己的云端服务器

docker load -i docker_lesson_1_0.tar  # 将镜像加载到本地
docker run -p 20000:22 --name my_docker_server -itd docker_lesson:1.0  # 创建并运行docker_lesson:1.0镜像
#将容器的22端口映射为本机的20000端口 重命名为my_docker_server 

docker attach my_docker_server  # 进入创建的docker容器
passwd  # 设置root密码
```
去云平台控制台中修改安全组配置，放行端口·。

返回`Terminal`，即可通过`ssh`登录自己的`docker`容器：
```
ssh root@xxx.xxx.xxx.xxx -p 20000  # 将xxx.xxx.xxx.xxx替换成自己租的服务器的IP地址
```
然后，可以仿照上节课内容，创建工作用户`acs`。

最后，可以参考[ssh节的ssh登录]配置`docker`容器的别名和免密登录。

## 小Tips
如果apt-get下载软件速度较慢，可以参考[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)中的内容，修改软件源。

---
来源链接：[yxc](https://www.acwing.com/file_system/file/content/whole/index/content/3074146/)