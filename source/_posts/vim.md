---
title: vim教程
comments: true
mathjax: true
tags:
  - Vim
  - Linux
categories:
  - Linux 基础
date: 2021-10-02 22:56:51
---
#### 音乐小港
{% meting "411315504" "netease" "song" "theme:#555" "mutex:true" "listmaxheight:340px" "preload:auto" %}

---
## Vim教程

### 介绍

Vim是从 vi 发展出来的一个文本编辑器。代码补完、编译及错误跳转等方便编程的功能特别丰富，在程序员中被广泛使用。

简单的来说， vi 是老式的字处理器，不过功能已经很齐全了，但是还是有可以进步的地方。 vim 则可以说是程序开发者的一项很好用的工具。

连 vim 的官方网站 (http://www.vim.org) 自己也说 vim 是一个程序开发工具而不是文字处理软件。

### 功能：

1. 命令行模式下的文本编辑器。
1. 根据文件扩展名自动判别编程语言。支持代码缩进、代码高亮等功能。
1. 使用方式：vim filename
    1. 如果已有该文件，则打开它。
    1. 如果没有该文件，则打开个一个新的文件，并命名为filename

### 模式：

1. 一般命令模式
    默认模式。命令输入方式：类似于打游戏放技能，按不符，即可进行不同操作。可以复制、粘贴、删除文本等。
1. 编辑模式
    在一般命令模式里按下i，会进入编辑模式。
    按下ESC会退出编辑模式，返回到一般命令模式。
1. 命令行模式
    在一般命令模式里按下:/?三个字母中的任意一个，会进令行模式。命令行在最下面。
    可以查找、替换、保存、退出、配置编辑器等。

操作：

1. `i`：进入编辑模式
1. `ESC`：进入一般命令模式
1. `h` 或 `左箭头键`：光标向左移动一个字符
1. `j` 或 `向下箭头`：光标向下移动一个字符
1. `k` 或 `向上箭头`：光标向上移动一个字符
1. `l` 或 `向右箭头`：光标向右移动一个字符
1. `n<Space>`：n表示数字，按下数字后再按空格，光标会向右这一行的n个字符
1. `0` 或 `功能键[Home]`：光标移动到本行开头
1. `$` 或 `功能键[End]`：光标移动到本行末尾
1. `G`：光标移动到最后一行
1. `:n` 或 `nG`：n为数字，光标移动到第n行
1. `gg`：光标移动到第一行，相当于1G
1. `n<Enter>`：n为数字，光标向下移动n行
1. `/word`：向光标之下寻找第一个值为word的字符串。
1. `?word`：向光标之上寻找第一个值为word的字符串。
1. `n`：重复前一个查找操作
1. `N`：反向重复前一个查找操作
1. `:n1,n2s/word1/word2/g`：n1与n2为数字，在第n1行与n2间寻找word1这个字符串，并将该字符串替换为word2
1. `:1,$s/word1/word2/g`：将全文的word1替换为word2
1. `:1,$s/word1/word2/gc`：将全文的word1替换为word2，替换前要求用户确认。
1. `v`：选中文本
1. `d`：删除选中的文本
1. `dd`: 删除当前行
1. `y`：复制选中的文本
1. `yy`: 复制当前行
1. `p`: 将复制的数据在光标的下一行/下一个位置粘贴
1. `u`：撤销
1. `Ctrl + r`：取消撤销
1. `大于号 >`：将选中的文本整体向右缩进一次
1. `小于号 <`：将选中的文本整体向左缩进一次
1. `:w` 保存
1. `:w!` 强制保存
1. `:q` 退出
1. `:q!` 强制退出
1. `:wq` 保存并退出
1. `:set paste` 设置成粘贴模式，取消代码自动缩进
1. `:set nopaste` 取消粘贴模式，开启代码自动缩进
1. `:set nu` 显示行号
1. `:set nonu` 隐藏行号
1. `gg=G`：将全文代码格式化
1. `:noh` 关闭查找关键词高亮
1. `Ctrl + q`：当vim卡死时，可以取消当前正在执行的命令

### 异常处理：

>每次用 vim 编辑文件时，会自动创建一个 .filename.swp 的临时文件。
>
>如果打开某个文件时，该文件的swp文件已存在，则会报错。此决办法有两种：
>    1. 找到正在打开该文件的程序，并退出
>    1. 直接删掉该swp文件即可

> 经自定义设置的vim的置内容在下方给出
### tmux配置
`.vimrc`
```
" An example for a vimrc file.
"
" To use it, copy it to
"     for Unix and OS/2:  ~/.vimrc
"	      for Amiga:  s:.vimrc
"  for MS-DOS and Win32:  $VIM\_vimrc
"	    for OpenVMS:  sys$login:.vimrc

" When started as "evim", evim.vim will already have done these settings.
if v:progname =~? "evim"
  finish
endif

" Use Vim settings, rather then Vi settings (much better!).
" This must be first, because it changes other options as a side effect.
set nocompatible

" allow backspacing over everything in insert mode
set backspace=indent,eol,start

if has("vms")
  set nobackup		" do not keep a backup file, use versions instead
else
  set backup		" keep a backup file
endif
set history=50		" keep 50 lines of command line history
set ruler		" show the cursor position all the time
set showcmd		" display incomplete commands
set incsearch		" do incremental searching
"==========================================================================
"My Setting-sunshanlu
"==========================================================================
vmap <leader>y :w! /tmp/vitmp<CR>
nmap <leader>p :r! cat /tmp/vitmp<CR>

"语法高亮
syntax enable
syntax on
"显示行号
set nu

"修改默认注释颜色
"hi Comment ctermfg=DarkCyan
"允许退格键删除
"set backspace=2
"启用鼠标
set mouse=a
set selection=exclusive
set selectmode=mouse,key
"按C语言格式缩进
set cindent
set autoindent
set smartindent
set shiftwidth=4

" 允许在有未保存的修改时切换缓冲区
"set hidden

" 设置无备份文件
set writebackup
set nobackup

"显示括号匹配
set showmatch
"括号匹配显示时间为1(单位是十分之一秒)
set matchtime=5
"显示当前的行号列号：
set ruler
"在状态栏显示正在输入的命令
set showcmd

set foldmethod=syntax
"默认情况下不折叠
set foldlevel=100
" 开启状态栏信息
set laststatus=2
" 命令行的高度，默认为1，这里设为2
set cmdheight=2


" 显示Tab符，使用一高亮竖线代替
set list
"set listchars=tab:\|\ ,
set listchars=tab:>-,trail:-


"侦测文件类型
filetype on
"载入文件类型插件
filetype plugin on
"为特定文件类型载入相关缩进文件
filetype indent on
" 启用自动补全
filetype plugin indent on 


"设置编码自动识别, 中文引号显示
filetype on "打开文件类型检测
"set fileencodings=euc-cn,ucs-bom,utf-8,cp936,gb2312,gb18030,gbk,big5,euc-jp,euc-kr,latin1
set fileencodings=utf-8,gb2312,gbk,gb18030
"这个用能很给劲，不管encoding是什么编码，都能将文本显示汉字
"set termencoding=gb2312
set termencoding=utf-8
"新建文件使用的编码
set fileencoding=utf-8
"set fileencoding=gb2312
"用于显示的编码，仅仅是显示
set encoding=utf-8
"set encoding=utf-8
"set encoding=euc-cn
"set encoding=gbk
"set encoding=gb2312
"set ambiwidth=double
set fileformat=unix


"设置高亮搜索
set hlsearch
"在搜索时，输入的词句的逐字符高亮
set incsearch

" 着色模式
set t_Co=256
"colorscheme wombat256mod
"colorscheme gardener
"colorscheme elflord
colorscheme desert
"colorscheme evening
"colorscheme darkblue
"colorscheme torte
"colorscheme default

" 字体 && 字号
set guifont=Monaco:h10
"set guifont=Consolas:h10

" :LoadTemplate       根据文件后缀自动加载模板
"let g:template_path='/home/ruchee/.vim/template/'

" :AuthorInfoDetect   自动添加作者、时间等信息，本质是NERD_commenter && authorinfo的结合
""let g:vimrc_author='sunshanlu'
""let g:vimrc_email='sunshanlu@baidu.com'
""let g:vimrc_homepage='http://www.sunshanlu.com'
"
"
" Ctrl + E            一步加载语法模板和作者、时间信息
""map <c-e> <ESC>:AuthorInfoDetect<CR><ESC>Gi
""imap <c-e> <ESC>:AuthorInfoDetect<CR><ESC>Gi
""vmap <c-e> <ESC>:AuthorInfoDetect<CR><ESC>Gi



" ======= 引号 && 括号自动匹配 ======= "
"
":inoremap ( ()<ESC>i

":inoremap ) <c-r>=ClosePair(')')<CR>
"
":inoremap { {}<ESC>i
"
":inoremap } <c-r>=ClosePair('}')<CR>
"
":inoremap [ []<ESC>i
"
":inoremap ] <c-r>=ClosePair(']')<CR>
"
":inoremap < <><ESC>i
"
":inoremap > <c-r>=ClosePair('>')<CR>
"
"":inoremap " ""<ESC>i
"
":inoremap ' ''<ESC>i
"
":inoremap ` ``<ESC>i
"
":inoremap * **<ESC>i

" 每行超过80个的字符用下划线标示
""au BufRead,BufNewFile *.s,*.asm,*.h,*.c,*.cpp,*.java,*.cs,*.lisp,*.el,*.erl,*.tex,*.sh,*.lua,*.pl,*.php,*.tpl,*.py,*.rb,*.erb,*.vim,*.js,*.jade,*.coffee,*.css,*.xml,*.html,*.shtml,*.xhtml Underlined /.\%81v/
"
"
" For Win32 GUI: remove 't' flag from 'guioptions': no tearoff menu entries
" let &guioptions = substitute(&guioptions, "t", "", "g")

" Don't use Ex mode, use Q for formatting
map Q gq

" This is an alternative that also works in block mode, but the deleted
" text is lost and it only works for putting the current register.
"vnoremap p "_dp

" Switch syntax highlighting on, when the terminal has colors
" Also switch on highlighting the last used search pattern.
if &t_Co > 2 || has("gui_running")
  syntax on
  set hlsearch
endif

" Only do this part when compiled with support for autocommands.
if has("autocmd")

  " Enable file type detection.
  " Use the default filetype settings, so that mail gets 'tw' set to 72,
  " 'cindent' is on in C files, etc.
  " Also load indent files, to automatically do language-dependent indenting.
  filetype plugin indent on

  " Put these in an autocmd group, so that we can delete them easily.
  augroup vimrcEx
  au!

  " For all text files set 'textwidth' to 80 characters.
  autocmd FileType text setlocal textwidth=80

  " When editing a file, always jump to the last known cursor position.
  " Don't do it when the position is invalid or when inside an event handler
  " (happens when dropping a file on gvim).
  autocmd BufReadPost *
    \ if line("'\"") > 0 && line("'\"") <= line("$") |
    \   exe "normal g`\"" |
    \ endif

  augroup END

else

  set autoindent		" always set autoindenting on

endif " has("autocmd")

" 增加鼠标行高亮
set cursorline
hi CursorLine  cterm=NONE   ctermbg=darkred ctermfg=white

" 设置tab是四个空格
set ts=4
set expandtab

" 主要给Tlist使用
let Tlist_Exit_OnlyWindow = 1
let Tlist_Auto_Open = 1

```

---
来源链接：[yxc](https://www.acwing.com/file_system/file/content/whole/index/content/2855620/)