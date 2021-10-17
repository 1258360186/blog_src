---
title: Ubuntu 安装 Thrift 以及常见问题
comments: true
mathjax: true
tags:
  - thrift
  - Linux
categories:
  - Linux 基础
date: 2021-10-11 18:45:30
---
#### 音乐小港
{% meting "563733091" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# Ubuntu 安装 Thrift 以及常见问题
**本文演示ubuntu20.04下安装Thrift 0.15.0并配置CPP和Python3的使用环境**
**AC Terminal 中的 thrift为 0.16.0**

官方教程链接:[Ubuntu/Debian install](https://thrift.apache.org/docs/install/debian.html),[Building From Source](https://thrift.apache.org/docs/BuildingFromSource)

## 先安装好 g++ 和 python3
```
sudo apt update
sudo apt install g++
sudo apt install python3
```
## 安装 Thrift
### 安装相关依赖包
```
sudo apt-get install automake bison flex g++ git libboost-all-dev libevent-dev libssl-dev libtool make pkg-config
```
### 安装python packages
```
sudo apt install python-all python-all-dev python-all-dbg
```
### 下载 Thrift 并解压
```
wget https://dlcdn.apache.org/thrift/0.15.0/thrift-0.15.0.tar.gz
tar -xf thrift-0.15.0.tar.gz
```
### 执行命令
```
cd thrift-0.15.0/
./configure
```
执行完后最后的输出内容如下，yes即代表将支持的语言

![thrift](https://ucc.alicdn.com/pic/developer-ecology/1d63a3f72b13403888730c2950bf062c.png)

### 执行命令
```
sudo make //此步骤花费时间稍长
sudo make install
thrift -version //若正常输出Thrift的版本则证明安装完成
```
## 常见问题
### 找不到动态链接库
- 报错类似 
  ```
  ./main: error while loading shared libraries: libthrift-0.15.0.so: cannot open shared object file: No such file or directory
  ```
- 配置 `/etc/ld.so.conf` 文件，否则可能会报找不到动态链接库等错误
- 执行命令
  ```
  vim /etc/ld.so.conf
  ```
- 添加内容 `/usr/local/lib`，添加后文件内容如下
 ```
 include /etc/ld.so.conf.d/*.conf /user/local/lib
 ```
- 执行命令使添加的内容生效
```
sudo /sbin/ldconfig
```
### python找不到thrift模块
- 报错内容类似：
  ```
  ModuleNotFoundError: No module named ‘thrift’
  ```
  ![thrift](https://ucc.alicdn.com/pic/developer-ecology/cd12041534d84b88864fa886b7172247.png)
- 可通过pip安装thrift解决，若未安装pip，先执行安装pip的命令  
  ```
  sudo apt install python3-pip
  ```
- 然后执行  
  ```
  sudo pip install thrift
  ```
- 即可解决找不到thrift模块的问题

---
来源链接：[congee](https://www.acwing.com/file_system/file/content/whole/index/content/3034732/#comment_136807/)
