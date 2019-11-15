---
title: "Vim使用"
date: 2019-10-31T09:27:51+08:00
draft: false
categories: [tools]
tags: [Mac,vim,plugin]
---

# 1. Mac Vim设置
## 1.1 vim基础配置
* 修改配置文件`~/.vimrc`,添加以下配置
``` shell
" 显示行号
set number
" 显示标尺
set ruler
" 历史纪录
set history=1000
" 输入的命令显示出来，看的清楚些
set showcmd
" 状态行显示的内容
set statusline=%F%m%r%h%w\ [FORMAT=%{&ff}]\ [TYPE=%Y]\ [POS=%l,%v][%p%%]\ %{strftime(\"%d/%m/%y\ -\ %H:%M\")}
" 启动显示状态行1，总是显示状态行2
set laststatus=2
" 语法高亮显示
syntax on
set fileencodings=utf-8,gb2312,gbk,cp936,latin-1
set fileencoding=utf-8
set termencoding=utf-8
set fileformat=unix
set encoding=utf-8
" 配色方案
colorscheme desert
" 指定配色方案是256色
set t_Co=256

set wildmenu

" 去掉有关vi一致性模式，避免以前版本的一些bug和局限，解决backspace不能使用的问题
set nocompatible
set backspace=indent,eol,start
set backspace=2

" 启用自动对齐功能，把上一行的对齐格式应用到下一行
set autoindent

" 依据上面的格式，智能的选择对齐方式，对于类似C语言编写很有用处
set smartindent

" vim禁用自动备份
set nobackup
set nowritebackup
set noswapfile

" 用空格代替tab
set expandtab

" 设置显示制表符的空格字符个数,改进tab缩进值，默认为8，现改为4
set tabstop=4

" 统一缩进为4，方便在开启了et后使用退格(backspace)键，每次退格将删除X个空格
set softtabstop=4

" 设定自动缩进为4个字符，程序中自动缩进所使用的空白长度
set shiftwidth=4

" 设置帮助文件为中文(需要安装vimcdoc文档)
set helplang=cn

" 显示匹配的括号
set showmatch

" 文件缩进及tab个数
au FileType html,python,vim,javascript setl shiftwidth=4
au FileType html,python,vim,javascript setl tabstop=4
au FileType java,php setl shiftwidth=4
au FileType java,php setl tabstop=4
" 高亮搜索的字符串
set hlsearch

" 检测文件的类型
filetype on
filetype plugin on
filetype indent on

" C风格缩进
set cindent
set completeopt=longest,menu

" 功能设置

" 去掉输入错误提示声音
set noeb
" 自动保存
set autowrite
" 突出显示当前行 
set cursorline
" 突出显示当前列
set cursorcolumn
"设置光标样式为竖线vertical bar
" Change cursor shape between insert and normal mode in iTerm2.app
"if $TERM_PROGRAM =~ "iTerm"
let &t_SI = "\<Esc>]50;CursorShape=1\x7" " Vertical bar in insert mode
let &t_EI = "\<Esc>]50;CursorShape=0\x7" " Block in normal mode
"endif
" 共享剪贴板
set clipboard+=unnamed
" 文件被改动时自动载入
set autoread
" 顶部底部保持3行距离
set scrolloff=3
```

## 1.2 插件管家Vundle配置
* 相比VIM7，VIM8最强大的地方是给我们带来了期待已久的异步任务机制
* 异步任务机制的强大之处在于它允许各种插件创建异步任务而不会阻塞当前的编辑
### 1.2.1下载plug.vim插件管理器
``` shell
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```
### 1.2.2 安装 GNU Global和 Universal ctags
``` shell
git clone https://github.com/universal-ctags/ctags.git
cd ctags
sh autogen.sh
./configure
make -j
sudo make install

wget http://tamacom.com/global/global-6.6.3.tar.gz --no-check-certificate
tar xf global-6.6.3.tar.gz
cd global-6.6.3
sed -i "s/(int i = 0;/(i = 0;/g" gtags-cscope/find.c
sed -i "/regex_t reg;/a\int i;" gtags-cscope/find.c
./configure
make -j
sudo make install

pip install pygments  # 这个一定要安装！！！
```
### 1.2.3 配置vim8的插件系统
* 先配置一下`plug.vim`，然后让`plug.vim`帮助我们自动管理插件，编辑~/.vimrc文件．
``` shell
" Specify a directory for plugins
" - For Neovim: ~/.local/share/nvim/plugged
" - Avoid using standard Vim directory names like 'plugin'
call plug#begin('~/.vim/plugged')

" All of your Plugs must be added before the following line
call plug#end()              " required
```

### 1.2.4 更新配置文件之后调用 vim +PlugInstall +qall 安装全部插件，执行这个命令Plug会自动下载全部的插件到vim插件目录下。
``` shell
vim +PlugInstall +qall
```

### 1.2.5 常用插件
### NERD Tree
* `NERD Tree`是一个树形目录插件，方便浏览当前目录有哪些目录和文件。
* 在~/.vimrc文件中配置NERD Tree，设置一个启用或禁用NERD Tree的键映射
  ``` java
Plug 'scrooloose/nerdtree', { 'on':  'NERDTreeToggle' }
nmap <F5> :NERDTreeToggle<cr>
  ```
所以你只需按F5键就能启用或禁用NERD Tree，NERD Tree提供一些常用快捷键来操作目录：
  通过hjkl来移动光标
  o打开关闭文件或目录，如果想打开文件，必须光标移动到文件名
  t在标签页中打开
  s和i可以水平或纵向分割窗口打开文件
  p到上层目录
  P到根目录
  K到同目录第一个节点
  P到同目录最后一个节点
### 快捷键
* ctrl+w+h: 光标focus左侧树形目录
* ctrl+w+l: 光标focus右侧文件显示窗口

### 参考
    - [文章1](https://kernelgo.org/vim8.html)
    - [文章2](http://www.skywind.me/blog/archives/2084)

# 2. Vim快捷键
## 2.1 一图尽览
![vim_keys_1.png](/img/03_vim_tips/01.jpg)
