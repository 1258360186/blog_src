---
title: git教程
comments: true
mathjax: true
tags:
  - git
  - Linux
  - 常用命令
categories:
  - Linux 基础
date: 2021-10-13 15:07:08
---
#### 音乐小港
{% meting "1467976888" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
# git教程
## 代码托管平台：
- [gitee.com](https://gitee.com/)
- [github](https://github.com/)
- [git.acwing.com](http://git.acwing.com/)

## git基本概念
- 工作区：仓库的目录。工作区是独立于各个分支的。
- 暂存区：数据暂时存放的区域，类似于工作区写入版本库前的缓存区。暂存区是独立于各个分支的。
- 版本库：存放所有已经提交到本地仓库的代码版本
- 版本结构：树结构，树中每个节点代表一个代码版本。

## git常用命令
1. `git config --global user.name xxx`：设置全局用户名，信息记录在`~/.gitconfig`文件中
1. `git config --global user.email xxx@xxx.com`：设置全局邮箱地址，信息记录在`~/.gitconfig`文件中
1. `git init`：将当前目录配置成git仓库，信息记录在隐藏的`.git`文件夹中
1. `git add XX`：将XX文件添加到暂存区
    -  `git add .`：将所有待加入暂存区的文件加入暂存区
1. `git rm --cached XX`：将文件从仓库索引目录中删掉
1. `git commit -m` "给自己看的备注信息"：将暂存区的内容提交到当前分支
1. `git status`：查看仓库状态
1. `git diff XX`：查看XX文件相对于暂存区修改了哪些内容
1. `git log`：查看当前分支的所有版本
1. `git reflog`：查看HEAD指针的移动历史（包括被回滚的版本）
1. `git reset --hard HEAD^` 或 `git reset --hard HEAD~`：将代码库回滚到上一个版本
1. `git reset --hard HEAD^^`：往上回滚两次，以此类推
1. `git reset --hard HEAD~100`：往上回滚100个版本
1. `git reset --hard 版本号`：回滚到某一特定版本
1. `git checkout — XX`或`git restore XX`：将XX文件尚未加入暂存区的修改全部撤销
1. `git remote add origin git@git.acwing.com:xxx/XXX.git`：将本地仓库关联到远程仓库
1. `git push -u (第一次需要-u以后不需要)`：将当前分支推送到远程仓库
    -  `git push origin branch_name`：将本地的某个分支推送到远程仓库
1. `git clone git@git.acwing.com:xxx/XXX.git`：将远程仓库XXX下载到当前目录下
1. `git checkout -b branch_name`：创建并切换到`branch_name`这个分支
1. `git branch`：查看所有分支和当前所处分支
1. `git checkout branch_name`：切换到`branch_name`这个分支
1. `git merge branch_name`：将分支`branch_name`合并到当前分支上
1. `git branch -d branch_name`：删除本地仓库的`branch_name`分支
1. `git branch branch_name`：创建新分支
1. `git push --set-upstream origin branch_name`：设置本地的`branch_name`分支对应远程仓库的      branch_name分支
1. `git push -d origin branch_name`：删除远程仓库的`branch_name`分支
1. `git pull`：将远程仓库的当前分支与本地仓库的当前分支合并
1. `git pull origin branch_name`：将远程仓库的`branch_name`分支与本地仓库的当前分支合并
1. `git branch --set-upstream-to=origin/branch_name1 branch_name2`：将远程的`branch_name1`分支与本地的`branch_name2`分支对应
1. `git checkout -t origin/branch_name` 将远程的`branch_name`分支拉取到本地
1. `git stash`：将工作区和暂存区中尚未提交的修改存入栈中
1. `git stash apply`：将栈顶存储的修改恢复到当前分支，但不删除栈顶元素
1. `git stash drop`：删除栈顶存储的修改
1. `git stash pop`：将栈顶存储的修改恢复到当前分支，同时删除栈顶元素
1. `git stash list`：查看栈中所有元素

> `git restore –staged XX` 表示将某文件从暂存区中拿出，但还是会对其进行管理，当其内容相对于上个版本发生改变时，git status后能看到其显示为红色  
> `git rm –cached XX` 表示不对XX文件进行管理（即使其内容发生改变，在git status后，我们也可以发现其显示的是绿色）  
> `git log –pretty=oneline` 将历史版本在一行显示  
> `git diff XX`会先用工作区中的XX与缓存区中的XX进行比较，若缓存区中没有XX，再用工作区中的XX与当前head指向的版本中的XX进行比较。

[点我去](https://learngitbranching.js.org/?locale=zh_CN)一个git可视化的学习网站（趣味性挺强的，像闯关游戏)

---
来源链接：[yxc](https://www.acwing.com/file_system/file/content/whole/index/content/2897078/)