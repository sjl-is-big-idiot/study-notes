# 1. vim简单配置

```vimrc
set nocompatible
syntax on
colorscheme evening
set showmode
set showcmd
set mouse=a
set encoding=utf-8  
set t_Co=256
filetype indent on
set autoindent
set tabstop=2
set shiftwidth=4
set expandtab
set softtabstop=2
set number
set relativenumber
set cursorline
set textwidth=80
set wrap
set nowrap
set linebreak
set wrapmargin=2
set scrolloff=5
set sidescrolloff=15
set laststatus=2
set  ruler
set showmatch
set hlsearch
set incsearch
set ignorecase
set smartcase
set spell spelllang=en_us
set nobackup
set noswapfile
set undofile
set backupdir=~/.vim/.backup//  
set directory=~/.vim/.swp//
set undodir=~/.vim/.undo// 
set autochdir
set noerrorbells
set visualbell
set history=1000
set autoread
set listchars=tab:»■,trail:■
set list
set wildmenu
set wildmode=longest:list,full
```



## vim配色方案

参考文章：[Vim设置colorscheme小技巧](https://cloud.tencent.com/developer/article/2146904)

Vim的颜色主题在`/usr/share/vim/vim74/colors`文件夹里。打开vim后在normal模式下输入`:colorscheme`查看当前的主题，修改主题使用命令`:colorscheme mycolor`，其中mycolor是你`/usr/share/vim/vim74/colors`文件夹包含的文件名。也可以把这个命令写入`~/.vimrc`配置文件中，这样每次打开Vim都是你设定的主题。

<font color="red">注意：不同的OS版本可能vim的文件夹不一样，有的可能是`/usr/share/vim/vim73/colors`，需要根据自己的系统进行调整。</font>

Q. tab转为2个空格

```bash
:set ts=2
:set expandtab
:%rerab!
```

# VIM问题

## linux中用xshell工具vim编辑文件不能鼠标右键粘贴复制问题

**解决方法**：

```bash
:set mouse=c
```



# 参考

[]

[2] [Vim设置colorscheme小技巧](https://cloud.tencent.com/developer/article/2146904)