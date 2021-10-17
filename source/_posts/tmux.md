---
title: tmux教程
comments: true
mathjax: true
tags:
  - tmux
  - Linux
categories:
  - Linux 基础
date: 2021-10-02 22:28:36
---
#### 音乐小港
{% meting "29922545" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
## Tmux教程

### 介绍

Tmux 是一个终端复用器（terminal multiplexer），非常有用，属于常用的开发工具。

### 功能

> （1）它允许在单个窗口中，同时访问多个会话。这对于同时运行多个命令行程序很有用。
>
>（2） 它可以让新窗口"接入"已经存在的会话。
>
>（3）它允许每个会话有多个连接窗口，因此可以多人实时共享会话。
>
>（4）它还支持窗口任意的垂直和水平拆分。
>
> （5） 允许断开Terminal连接后，继续运行进程。

### 结构

一个tmux可以包含多个session，一个session可以包含多个window，一个window可以包含多个pane。
```
实例：
        tmux:
            session 0:
                window 0:
                    pane 0
                    pane 1
                    pane 2
                    ...
                window 1
                window 2
                ...
            session 1
            session 2
            ...
```

### 安装

```
# Ubuntu 或 Debian
$ sudo apt-get install tmux

# CentOS 或 Fedora
$ sudo yum install tmux

# Mac
$ brew install tmux
```

### 操作

1. `tmux`：新建一个session，其中包含一个window，windo包含一个pane，pane里打开了一个shell对话框。
2. 按下`ctrl + a`后手指松开，然后按%：将当前pane左右平两个pane。
3. 按下`ctrl + a`后手指松开，然后按"（注意是双引号"）当前pane上下平分成两个pane。
4. `ctrl + d`：关闭当前pane；如果当前window的所有pane关闭，则自动关闭window；如果当前session的所有window均闭，则自动关闭session。
5. 鼠标点击可以选pane。
6. 按下`ctrl + a`后手指松开，然后按方向键：选择相pane。
7. 鼠标拖动pane之间的分割线，可以调整分割线的位置。
8. 按住`ctrl + a`的同时按方向键，可以调整pane之间分割位置。
9. 按下`ctrl + a`后手指松开，然后按z：将当前pane全屏/全屏。
10. 按下`ctrl + a`后手指松开，然后按d：挂起当前session。
11. `tmux a`：打开之前挂起的session。
12. 按下`ctrl + a`后手指松开，然后按s：选择其它session。
    方向键 —— 上：选择上一项 session/window/pane
    方向键 —— 下：选择下一项 session/window/pane
    方向键 —— 右：展开当前项 session/window
    方向键 —— 左：闭合当前项 session/window
13. 按下`ctrl + a`后手指松开，然后按`c`：在当前session中一个新的window。
14. 按下`ctrl + a`后手指松开，然后按`w`：选择其他window作方法与1.完全相同。
15. 按下`ctrl + a`后手指松开，然后按`PageUp`：翻阅当前pan的内容。
16. 鼠标滚轮：翻阅当前pane内的内容。
17. 在tmux中选中文本时，需要按住shift键。（仅支持Windo和Linux，不支持Mac，不过该操作并不是必须的，因此影响不大）
18. tmux中复制/粘贴文本的通用方式：
    1. 按下`ctrl + a`后松开手指，然后按`[`
    2. 用鼠标选中文本，被选中的文本会被自动复制到tmux 贴板
    3. 按下`ctrl + a`后松开手指，然后按`]`，会将剪贴板 容粘贴到光标处

> e.g 原始tmux的前缀键盘是`ctrl + b`
>
> 经修改的tmux的配置内容在下方给出

### tmux配置
`.tmux.conf`
```
set-option -g status-keys vi
setw -g mode-keys vi

setw -g monitor-activity on

# setw -g c0-change-trigger 10
# setw -g c0-change-interval 100

# setw -g c0-change-interval 50
# setw -g c0-change-trigger  75


set-window-option -g automatic-rename on
set-option -g set-titles on
set -g history-limit 100000

#set-window-option -g utf8 on

# set command prefix
set-option -g prefix C-a
unbind-key C-b
bind-key C-a send-prefix

bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D

bind < resize-pane -L 7
bind > resize-pane -R 7
bind - resize-pane -D 7
bind + resize-pane -U 7


bind-key -n M-l next-window
bind-key -n M-h previous-window



set -g status-interval 1
# status bar
set -g status-bg black
set -g status-fg blue


#set -g status-utf8 on
set -g status-justify centre
set -g status-bg default
set -g status-left " #[fg=green]#S@#H #[default]"
set -g status-left-length 20


# mouse support
# for tmux 2.1
# set -g mouse-utf8 on
set -g mouse on
#
# for previous version
#set -g mode-mouse on
#set -g mouse-resize-pane on
#set -g mouse-select-pane on
#set -g mouse-select-window on


#set -g status-right-length 25
set -g status-right "#[fg=green]%H:%M:%S #[fg=magenta]%a %m-%d #[default]"

# fix for tmux 1.9
bind '"' split-window -vc "#{pane_current_path}"
bind '%' split-window -hc "#{pane_current_path}"
bind 'c' new-window -c "#{pane_current_path}"

# run-shell "powerline-daemon -q"

# vim: ft=conf

```

---
来源链接：[yxc](https://www.acwing.com/file_system/file/content/whole/index/content/2855620/)