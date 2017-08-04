---
title: 自用Vim配置注释版
categories: 工具
tags:
  - vim
---
```
set nocompatible              " 去除VI一致性,必须要添加
filetype off                  " 必须要添加

" 设置包括vundle和初始化相关的runtime path
" 使用vundle需要先git clone到本地
" 1、git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
" 如果目录不存在，需要那就执行如下命令新建
" cd ~
" mkdir .vim
" cd .vim
" mkdir bundle
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" 另一种选择, 指定一个vundle安装插件的路径
"call vundle#begin('~/some/path/here')

" 让vundle管理插件版本,必须
Plugin 'VundleVim/Vundle.vim'

" 以下范例用来支持不同格式的插件安装.
" 请将安装插件的命令放在vundle#begin和vundle#end之间.
" Github上的插件
" 格式为 Plugin '用户名/插件仓库名'
" Plugin 'tpope/vim-fugitive'
" 来自 http://vim-scripts.org/vim/scripts.html 的插件
" Plugin '插件名称' 实际上是 Plugin 'vim-scripts/插件仓库名' 只是此处的用户名可以省略
" Plugin 'L9'
" 由Git支持但不再github上的插件仓库 Plugin 'git clone 后面的地址'
" Plugin 'git://git.wincent.com/command-t.git'
" 本地的Git仓库(例如自己的插件) Plugin 'file:///+本地插件仓库绝对路径'
" Plugin 'file:///home/gmarik/path/to/plugin'
" 插件在仓库的子目录中.
" 正确指定路径用以设置runtimepath. 以下范例插件在sparkup/vim目录下
" Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" 安装L9，如果已经安装过这个插件，可利用以下格式避免命名冲突
" Plugin 'ascenator/L9', {'name': 'newL9'}

" 你的所有插件需要在下面这行之前
call vundle#end()            " 必须
filetype plugin indent on    " 必须 加载vim自带和插件相应的语法和文件类型相关脚本
" 忽视插件改变缩进,可以使用以下替代:
"filetype plugin on
"
" 常用的命令
" :PluginList       - 列出所有已配置的插件
" :PluginInstall     - 安装插件,追加 `!` 用以更新或使用 :PluginUpdate
" :PluginSearch foo - 搜索 foo ; 追加 `!` 清除本地缓存
" :PluginClean      - 清除未使用插件,需要确认; 追加 `!` 自动批准移除未使用插件
"
" 查阅 :h vundle 获取更多细节和wiki以及FAQ
" 将你自己对非插件片段放在这行之后





" 自定义插件
Plugin 'scrooloose/nerdtree'
"Plugin 'Lokaltog/vim-powerline'

" 插件设置
" NERDTree
map <F2> :NERDTreeToggle<CR>

"powerline{
set guifont=PowerlineSymbols\ for\ Powerline
set nocompatible
set t_Co=256
let g:Powerline_symbols = 'fancy'
"}



" 自定义系统环境
"set cursorline " 突出显示当前行
set nu         " 显示行号

set cursorline  " 高亮当前行 
hi CursorLine   cterm=NONE ctermbg=darkred ctermfg=white guibg=darkred guifg=white " 当前行颜色

" set cursorcolumn   " 高亮当前列
" hi CursorColumn cterm=NONE ctermbg=darkred ctermfg=white guibg=darkred guifg=white  " 当前列颜色

set shiftwidth=4 "这个量是每行的缩进深度，一般设置成和tabstop一样的宽度"
set tabstop=4 "设置Tab显示的宽度，Python建议设置成4"

"刚才说过Tab和空格是不同的，虽然你可以在自己的代码中全部使用Tab"
""但是如果你将你的代码分享给使用空格的朋友，就会带来很多麻烦"
"那么设置下面这行就可以将Tab自动展开成为空格"
"set expandtab
""如果只想在Python文件中将Tab展开成空格，就改换成下面这句"
autocmd FileType python set expandtab

"上面的一些配置已经可以让你避免编译出现错误的问题了"
"
""不过下面还有一些配置是建议同学们根据需要加上的"
set smartindent "智能缩进"
set cindent "C语言风格缩进"
set autoindent "自动缩进"
```