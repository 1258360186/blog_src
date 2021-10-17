---
title: 管道、环境变量与常用命令
comments: true
mathjax: true
tags:
  - 常用命令
  - Linux
categories:
  - Linux 基础
date: 2021-10-13 15:20:52
---
#### 音乐小港
{% meting "1814971662" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# 管道、环境变量与常用命令
- 管道
- 环境变量
- 常用命令

## 管道
### 概念
管道类似于文件重定向，可以将前一个命令的`stdout`重定向到下一个命令的`stdin`。

### 要点
- 管道命令仅处理`stdout`，会忽略`stderr`。
- 管道右边的命令必须能接受`stdin`。
- 多个管道命令可以串联。
### 与文件重定向的区别
文件重定向左边为命令，右边为文件。
管道左右两边均为命令，左边有`stdout`，右边有`stdin`。
### 举例
统计当前目录下所有python文件的总行数，其中`find`、`xargs`、`wc`等命令可以参考**常用命令**这一节内容。
```
find . -name '*.py' | xargs cat | wc -l
```

## 环境变量
### 概念
Linux系统中会用很多环境变量来记录**配置信息**。

环境变量类似于全局变量，可以被各个进程访问到。我们可以通过修改环境变量来方便地修改系统配置。

### 查看
列出当前环境下的所有环境变量：
```
env  # 显示当前用户的变量
set  # 显示当前shell的变量，包括当前用户的变量;
export  # 显示当前导出成用户变量的shell变量
```
输出某个环境变量的值：
```
echo $PATH
```
### 修改
环境变量的定义、修改、删除操作可以参考**shell语法——变量**这一节的内容。

为了将对环境变量的修改应用到未来所有环境下，可以将修改命令放到`~/.bashrc`文件中。

修改完`~/.bashrc`文件后，记得执行`source ~/.bashrc`，来将修改应用到当前的`bash`环境下。

为何将修改命令放到`~/.bashrc`，就可以确保修改会影响未来所有的环境呢？

- 每次启动`bash`，都会先执行`~/.bashrc`。
- 每次`ssh`登陆远程服务器，都会启动一个`bash`命令行给我们。
- 每次`tmux`新开一个`pane`，都会启动一个`bash`命令行给我们。
- 所以未来所有新开的环境都会加载我们修改的内容。
### 常见环境变量
1. `HOME`：用户的家目录。
1. `PATH`：可执行文件（命令）的存储路径。路径与路径之间用:分隔。当某个可执行文件同时出现在多个路径中时，会选择从左到右数第一个路径中的执行。**下列所有存储路径的环境变量，均采用从左到右的优先顺序。**
1. `LD_LIBRARY_PATH`：用于指定动态链接库(.so文件)的路径，其内容是以冒号分隔的路径列表。
1. `C_INCLUDE_PATH`：C语言的头文件路径，内容是以冒号分隔的路径列表。
1. `CPLUS_INCLUDE_PATH`：CPP的头文件路径，内容是以冒号分隔的路径列表。
1. `PYTHONPATH`：Python导入包的路径，内容是以冒号分隔的路径列表。
1. `JAVA_HOME`：jdk的安装目录。
1. `CLASSPATH`：存放Java导入类的路径，内容是以冒号分隔的路径列表。

## 常用命令
### 简介
Linux命令非常多，本节讲解几个常用命令。其他命令依赖于大家根据实际操作环境，边用边查。

### 系统状况
1. `top`：查看所有进程的信息（Linux的任务管理器）
    - 打开后，输入`M`：按使用内存排序
    - 打开后，输入`P`：按使用CPU排序
    - 打开后，输入`q`：退出
1. `df -h`：查看硬盘使用情况
1. `free -h`：查看内存使用情况
1. `du -sh`：查看当前目录占用的硬盘空间
1. `ps aux`：查看所有进程
1. `kill -9 pid`：杀死编号为`pid`的进程
    - 传递某个具体的信号：`kill -s SIGTERM pid`
1. `netstat -nt`：查看所有网络连接
1. `w`：列出当前登陆的用户
1. `ping www.baidu.com`：检查是否连网
### 文件权限
1. `chmod`：修改文件权限
    - `chmod +x xxx`：给`xxx`添加可执行权限
    - `chmod -x xxx`：去掉`xxx`的可执行权限
    - `chmod 777 xxx`：将`xxx`的权限改成777。
### 文件检索
1. `find /path/to/directory/ -name '*.py'`：搜索某个文件路径下的所有`*.py`文件
1. ` grep xxx`：从stdin中读入若干行数据，如果某行中包含`xxx`，则输出该行；否则忽略该行。
1. `wc`：统计行数、单词数、字节数
    - 既可以从stdin中直接读入内容；也可以在命令行参数中传入文件名列表；
    - wc -l：统计行数
    - wc -w：统计单词数
    - wc -c：统计字节数
1. `tree`：展示当前目录的文件结构
    - `tree /path/to/directory/`：展示某个目录的文件结构
    - `tree -a`：展示隐藏文件
1. `ag xxx`：搜索当前目录下的所有文件，检索`xxx`字符串
1. `cut`：分割一行内容
    - 从`stdin`中读入多行数据
    - `echo $PATH | cut -d ':' -f 3,5`：输出`PATH`用:分割后第3、5列数据
    - `echo $PATH | cut -d ':' -f 3-5`：输出`PATH`用:分割后第3-5列数据
    - `echo $PATH | cut -c 3,5`：输出`PATH`的第3、5个字符
    - `echo $PATH | cut -c 3-5`：输出`PATH`的第3-5个字符
1. `sort`：将每行内容按字典序排序
    - 可以从`stdin`中读取多行数据
    - 可以从命令行参数中读取文件名列表
1. `xargs`：将`stdin`中的数据用空格或回车分割成命令行参数
    - `find . -name '*.py' | xargs cat | wc -l`：统计当前目录下所有python文件的总行数
### 查看文件内容
1. `more`：浏览文件内容
    - 回车：下一行
    - 空格：下一页
    - `b`：上一页
    - `q`：退出
2. `less`：与`more`类似，功能更全
    - 回车：下一行
    - `y`：上一行
    - `Page Down`：下一页
    - `Page Up`：上一页
    - `q`：退出
3. `head -3 xxx`：展示`xxx`的前3行内容
    - 同时支持从`stdin`读入内容
4. `tail -3 xxx`：展示`xxx`末尾3行内容
    - 同时支持从`stdin`读入内容
### 用户相关
1. `history`：展示当前用户的历史操作。内容存放在`~/.bash_history`中
### 工具
1. `md5sum`：计算`md5`哈希值
    - 可以从`stdin`读入内容
    - 也可以在命令行参数中传入文件名列表；
1. `time command`：统计`command`命令的执行时间
1. `ipython3`：交互式python3环境。可以当做计算器，或者批量管理文件。
    - `! echo "Hello World"`：!表示执行shell脚本
1. `watch -n 0.1 command`：每0.1秒执行一次`command`命令
1. `tar`：压缩文件
    - `tar -zcvf xxx.tar.gz /path/to/file/*`：压缩
    - `tar -zxvf xxx.tar.gz`：解压缩
1. `diff xxx yyy`：查找文件`xxx`与`yyy`的不同点
### 安装软件
1. `sudo command`：以`root`身份执行`command`命令
1. `apt-get install xxx`：安装软件
1. `pip install xxx --user --upgrade`：安装python包

---
来源链接：[yxc](https://www.acwing.com/file_system/file/content/whole/index/content/3030391/)