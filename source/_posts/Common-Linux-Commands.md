---
title: Linux常用命令
comments: true
mathjax: true
tags:
  - Linux
  - 常用命令
categories:
  - Linux 基础
date: 2021-10-02 21:18:01
---
#### 音乐小港
{% meting "1396940451" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
## 常用命令介绍
---
```
1. ctrl c: 取消命令，并且换行

2. ctrl u: 清空本行命令

3. tab键：可以补全命令和文件名，如果补全不了快速按两下tab键，可以显示备选选项

4. ls: 列出当前目录下所有文件，蓝色的是文件夹，白色的是普通文件，绿色的是可执行文件

5. pwd: 显示当前路径

6. cd XXX: 进入XXX目录下, cd .. 返回上层目录

7. cp XXX YYY: 将XXX文件复制成YYY，XXX和YYY可以是一个路径，比如../dir_c/a.txt，表示上层目录下的dir_c文件夹下的文件a.txt

8. mkdir XXX: 创建目录XXX

9. rm XXX: 删除普通文件;  rm XXX -r: 删除文件夹

10. mv XXX YYY: 将XXX文件移动到YYY，和cp命令一样，XXX和YYY可以是一个路径；重命名也是用这个命令

11. touch XXX: 创建一个文件

12. cat XXX: 展示文件XXX中的内容

13. 复制文本
    windows/Linux下：Ctrl + insert，Mac下：command + c

14. 粘贴文本
    windows/Linux下：Shift + insert，Mac下：command + v
```
来源链接：[yxc](https://www.acwing.com/file_system/file/content/whole/index/content/2855530/)

---
## 基础命令介绍
---
### 目录相关命令：
---

>`pwd`：绝对路径

>`ls`：`-a` 全部文件 `-l` 更多信息

>`ll`：等于 `ls -al`

>`cd`：相对路径或绝对路径 `~` 为home目录 `..` 上一级 `.` 当前目录 `-` 上一目录

>`mkdir`:

语法：`mkdir` [选项] 目录名称  

功能：创建文件夹  

示例：

- `mkdir test`
- `-p` 创建多层目录 
  - `mkdir -p test/test1`

>`rmdir`：

语法：`rmdir` [选项] 目录名称  

功能：删除文件夹  

示例：

- `rmdir test`
- `-p` 删除多层目录 
  - `rmdir -p test/test1`

>`cp`：

语法：`cp` 源目录或文件 目标目录或文件  

功能：复制文件，复制文件夹必须加 `-r`  

示例：

- `cp test test.txt` 
  - 将 test 复制为 test.txt ，复制时重命名）  
- `-r` 递归复制整个文件夹 
  - `cp -r test test1` 
  - 将所有 test 目录内容复制到 test1 目录（自动创建 test1 目录））

>`mv`：

语法：`mv` [选项] 源目标  

功能描述：移动文件或重命名文件  

示例：

- 命名： `mv test test.txt` 
  - 将 test 文件重命名为 test.txt
- 移动文件： `mv a.txt ./user/b.txt` （相对路径移动并重命名文件）
- 重命名文件夹： `mv aaa/ bbb`

>`rm`：

语法：`rm` [选项] 文件

功能描述：删除文件及目录

选项： 
- `-f`：force 强制执行 
- `-r`：recursive 递归执行

示例： 
- `rm -rf test2`
---
### 文件相关命令：
---
>`touch`：

语法：`touch` [选项] 文件名

功能描述：新建文件  
示例： 
- `touch test.txt`

>`echo`：

语法：`echo` 字符串或变量

功能描述：输出字符串或变量值，还可以搭配从定向符将内容存储到文件

示例：

- `echo hello`
- `echo $SHELL`
- `echo this is echo >> test.txt`

>`cat`：

语法：`cat` [选项] 文件名

功能描述：查看文件内容，从第一行开始显示

选项：

- `-A`：列出特殊字符而非空白
- `-b`：列出行号，空白行不算行号
- `-n`：列出行号，空白行也会有行号
- `-v`：列出一些看不出来的特殊字符

示例：

- `cat -n test.txt`

>`more`：

语法：`more` [选项] 文件

功能描述：查看文件内容，一页一页的显示

使用说明：

- `空格键（space）`：向下翻一页
- `enter`：向下翻一行
- `q`：退出more，不在显示文件内容
- `ctrl+f`：向下滚动一屏
- `ctrl+b`：返回上一屏
- `=`：输出当前行的行号
- `:f`：输出文件名和当前行号

>`head`：

语法：`head` [选项] 文件

功能描述：查看文件内容，只看头几行

选项： 
- `-n`：查看头n行

示例：
- `head -n 2 test.txt`
  - 只看头两行

>`tail`：

语法：`tail` [选项] 文件

功能描述：查看文件内容，只查看文件末尾几行

选项：

- `-n`：末尾几行
- `-f`：follow输出文件修改的内容，用于追踪文件修改

示例：

- `tail -n 2 test.txt`

>`wget`：

语法：`wget` [参数] [url地址]

功能：下载网络文件

参数：

- `-b`：background后台下载
- `-P`：directory-prefix下载到指定目录
- `-t`：tries 最大尝试次数
- `-c`：continue断点续传clear
- `-p`：page-requisites下载页面所有内容，包括图片、视频等
- `-r`：recursive递归下载

示例：
- 下载百度logo 
  - `wget https://www.baidu.com/img/bd_logo1.png`
---
### 查找命令：
---
>`find`：

语法：`find` [搜索范围] [参数] [匹配条件]

功能描述：查找文件或目录

参数说明：

- `-name`：按文件名称查找
- `-user`：按文件拥有者查找
- `-size`：根按文件大小查找文件
  - `+n` 大于
  - `-n` 小于
  - `n` 等于

示例：

- `find test/ -name test1.txt`
- `find test/ -user root`
- `find test/ -size -102400`

>`grep`：

语法：`grep` [参数] 查找内容 源文件

功能描述：在文件内搜索字符串匹配的行并输出

参数：

- `-c`：count只输出匹配行的计数
- `-n`：line-number

示例： 
- `grep -n James test.txt`
---
### 压缩解压：
---
>`tar`：

语法：`tar` [参数] 包名.tar.gz 待打包的内容

功能描述：打包目录，压缩后的文件格式为.tar.gz

参数：

- `-c`：create生成.tar打包文件
- `-x`：extract解包.tar文件
- `-v`：verbose显示详细信息
- `-f`：file指定压缩后的文件名
- `-z`：打包同时压缩
- `-C`：解压到指定目录

常用：

- `tar -cvf 打包名.tar 源文件名1 源文件名2 ... `
  - 打包文件
- `tar -xvf 打包文件名` 
  - 解包
- `tar -zcvf 压缩名.tar.gz 源文件名1 源文件名2 ...` 
  - 打包并压缩
- `tar -zxvf 压缩文件名`
  - 解压缩并解打包

示例：

- `tar -cvf abc.tar a.txt b.txt c.txt`
- `tar -xvf abc.tar`
- `tar -zcvf abc.tar.gz a.txt b.txt c.txt`
- `tar -zxvf abc.tar.gz`
- `tar -zcvf abc.tar.gz test/` 
  - 压缩整个目录


>`zip`和`unzip`：

语法：

压缩：`zip` [参数] 包名.zip

解压：`unzip` 包名.zip

功能描述：压缩文件和目录，windows和linux通用且可以压缩目录并保留源文件

参数： `-r`：recurse-paths递归压缩目录

示例：

- `zip test.zip test.txt test1.txt`
- `unzip test.zip`
---
### 进程线程命令：
---
>`ps`：

语法：`ps` [选项]

功能描述：查看系统中所有进程

参数：

- `-a`：all 显示现行终端机下的所有程序，包括其他用户的程序（比如多克隆几个会话执行不同命令，也会列出来）
- `-u`：userlist 以用户为主的格式来显示程序状况
- `-x`：显示所有程序，不以终端机来区分（前面讲过终端有很多类型，不仅显示当前终端）

示例：

- `ps -aux`
  - 配合管道命令： `ps -aux | grep xxx`

>`kill`：

语法：`kill` [选项] 进程id

功能描述：终止某个指定pid的服务进程

选项： 
- `-9`：强迫进程立即停止

示例： 
- `kill -9 65482`
---
### 系统状态检测命令：
---
>`netstat`:

语法：`netstat` [参数]

功能描述：显示整个系统目前网络情况，比如目前的链接、数据包传递数据、路由表内容等

示例：
- `netstat`

>`free`：

语法：`free` [选项]

功能描述：显示当前系统中内存的使用信息

选项：

- `-m`：megabytes以兆字节显示
- `-h`：human带单位输出
---
### 关机：
---
>`shutdown`：

语法：`shutdown` [选项] [关机时间] [提示内容]

功能描述：关机

选项：

- `-h`：关机
- `-r`：重启

---
来源链接：[女娲补 Linux 指令](https://www.acwing.com/blog/content/4756/)