# Golang环境配置

1. Golang下载地址：https://golang.org/dl/go1.13.12.linux-amd64.tar.gz
2. 解压go1.13.12.linux-amd64.tar.gz至/usr/local/go
3. 将/usr/local/go/bin/go和/usr/local/go/bin/gofmt拷贝至/usr/bin/
4. 创建go开发路径$GOPATH：/data/go-env-nice
5. 配置/etc/profile

```
# Enable the go modules feature
export GO111MODULE=on
# Set the GOPROXY environment variable
export GOPROXY=https://goproxy.io  #https://gocenter.io
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
export GOPATH=/data/go-env-nice
export PATH=$PATH:$GOPATH/bin
```

6. 查看go版本

```
go version
go version go1.13.12 linux/amd64
```

# vim环境配置

```
"==============================================================================
" 处理 Gnome 终端不能使用 alt 快捷键
"==============================================================================
let c='a'
while c <= 'z'
  exec "set <A-".c.">=\e".c
  exec "imap \e".c." <A-".c.">"
  let c = nr2char(1+char2nr(c))
endw

set timeout ttimeoutlen=50

"==============================================================================
" vim 内置配置 
"==============================================================================

" 设置 vimrc 修改保存后立刻生效，不用在重新打开
" 建议配置完成后将这个关闭，否则配置多了之后会很卡
" autocmd BufWritePost $MYVIMRC source $MYVIMRC

" 设置编码格式
set fileencodings=utf-8,gb2312,gbk,gb18030
set termencoding=utf-8
set encoding=utf-8

" 关闭兼容模式
set nocompatible

set number " 设置绝对行号
" set relativenumber " 设置相对行号
set cursorline "突出显示当前行
" set cursorcolumn " 突出显示当前列
set showmatch " 显示括号匹配
" 解决插入模式下delete/backspce键失效问题
set backspace=2

" tab的相关设置
set autoindent " 继承前一行的缩进方式，适用于多行注释
" 开启时，在行首按TAB将加入shiftwidth个空格，否则加入tabstop个空格
set smarttab
" 将输入的TAB自动展开成空格。开启后要输入TAB，需要Ctrl-V<TAB>
set expandtab
set tabstop=4
" 设定 << 和 >> 命令移动时的宽度为 4
set shiftwidth=4

" 自定义快捷键

" ==== 系统剪切板复制粘贴 ====
" v 模式下复制内容到系统剪切板
vmap <Leader>c "+yy
" n 模式下复制一行到系统剪切板
nmap <Leader>c "+yy
" n 模式下粘贴系统剪切板的内容
nmap <Leader>v "+p

" 修改默认的区域切换如ctrl+w+h 奇幻到左侧， 依次是 左右上下
nmap <M-h> <C-w>h
nmap <M-l> <C-w>l
nmap <M-k> <C-w>k
nmap <M-j> <C-w>j
" 作用和上面一样，只不过是使用的 Leader
nmap <Leader>h <C-w>h
nmap <Leader>l <C-w>l
nmap <Leader>k <C-w>k
nmap <Leader>j <C-w>j

nmap <Leader>nu :set relativenumber<CR>
nmap <Leader>nn :set norelativenumber<CR>

"开启实时搜索
set incsearch
" 搜索时大小写不敏感
set ignorecase
syntax enable
syntax on                    " 开启文件类型侦测
filetype plugin indent on    " 启用自动补全

" 退出插入模式指定类型的文件自动保存
au InsertLeave *.go,*.sh,*.cpp,*.c,*.py write

"==============================================================================
" 插件配置 
"==============================================================================

" 插件开始的位置
call plug#begin('~/.vim/plugged')

" Vim 中文文档
Plug 'yianwillis/vimcdoc'

" Shorthand notation; fetches https://github.com/junegunn/vim-easy-align
" 可以快速对齐的插件
Plug 'junegunn/vim-easy-align'

" 用来提供一个导航目录的侧边栏
Plug 'scrooloose/nerdtree'

" 可以使 nerdtree Tab 标签的名称更友好些
Plug 'jistr/vim-nerdtree-tabs'

" 可以在导航目录中看到 git 版本信息
Plug 'Xuyuanp/nerdtree-git-plugin'

" 查看当前代码文件中的变量和函数列表的插件，
" 可以切换和跳转到代码中对应的变量和函数的位置
" 大纲式导航, Go 需要 https://github.com/jstemmer/gotags 支持
Plug 'majutsushi/tagbar'

" 自动补全括号的插件，包括小括号，中括号，以及花括号
Plug 'jiangmiao/auto-pairs'

" Vim状态栏插件，包括显示行号，列号，文件类型，文件名，以及Git状态
Plug 'vim-airline/vim-airline'

" 有道词典在线翻译
Plug 'ianva/vim-youdao-translater'

" 代码自动完成，安装完插件还需要额外配置才可以使用
" Plug 'Valloric/YouCompleteMe'

" 可以在文档中显示 git 信息
Plug 'airblade/vim-gitgutter'

" 下面两个插件要配合使用，可以自动生成代码块
Plug 'SirVer/ultisnips'
Plug 'honza/vim-snippets'

" 配色方案
" colorscheme gruvbox 
Plug 'morhetz/gruvbox'
" colorscheme molokai
Plug 'crusoexia/vim-monokai'
" colorscheme one
Plug 'rakr/vim-one'
" colorscheme github
Plug 'acarapetis/vim-colors-github'
" colorscheme neodark
Plug 'KeitaNakamura/neodark.vim'

" markdown 插件
Plug 'iamcco/mathjax-support-for-mkdp'
Plug 'iamcco/markdown-preview.vim'

" 自动生成注释的插件
Plug 'scrooloose/nerdcommenter'

" vim-go 插件
Plug 'fatih/vim-go'
" vim-godef
Plug 'dgryski/vim-godef'

" 插件结束的位置，插件全部放在此行上面
call plug#end()

"==============================================================================
" 主题配色 
"==============================================================================

" 开启24bit的颜色，开启这个颜色会更漂亮一些
set termguicolors
" 配色方案, 可以从上面插件安装中的选择一个使用 
" colorscheme gruvbox " gruvbox " 主题
colorscheme molokai " 主题
set background=dark " 主题背景 dark-深色; light-浅色

"==============================================================================
" NERDTree 插件
"==============================================================================

" 打开和关闭NERDTree快捷键
map <F10> :NERDTreeToggle<CR>
nmap <M-m> :NERDTreeFind<CR>

" 显示行号
let NERDTreeShowLineNumbers=1
" 打开文件时是否显示目录
let NERDTreeAutoCenter=0
" 是否显示隐藏文件
let NERDTreeShowHidden=0
" 设置宽度
" let NERDTreeWinSize=31
" 忽略一下文件的显示
let NERDTreeIgnore=['\.pyc','\~$','\.swp']
" 打开 vim 文件及显示书签列表
let NERDTreeShowBookmarks=2

" 在终端启动vim时，共享NERDTree
" let g:nerdtree_tabs_open_on_console_startup=1

"==============================================================================
"  majutsushi/tagbar 插件
"==============================================================================

" majutsushi/tagbar 插件打开关闭快捷键
nmap <F9> :TagbarToggle<CR>
" 设置tagbar的窗口宽度
let g:tagbar_width=35
let g:tagbar_left = 1

"==============================================================================
"  nerdtree-git-plugin 插件
"==============================================================================
let g:NERDTreeIndicatorMapCustom = {
    \ "Modified"  : "✹",
    \ "Staged"    : "✚",
    \ "Untracked" : "✭",
    \ "Renamed"   : "➜",
    \ "Unmerged"  : "═",
    \ "Deleted"   : "✖",
    \ "Dirty"     : "✗",
    \ "Clean"     : "✔︎",
    \ 'Ignored'   : '☒',
    \ "Unknown"   : "?"
    \ }

let g:NERDTreeShowIgnoredStatus = 1
nmap <Leader>pwd :NERDTreeCWD<CR>

"==============================================================================
"  Valloric/YouCompleteMe 插件
"==============================================================================

" make YCM compatible with UltiSnips (using supertab)
" let g:ycm_key_list_select_completion = ['<M-j>', '<DOWN>']
" let g:ycm_key_list_previous_completion = ['<M-k>', '<Up>']
" let g:SuperTabDefaultCompletionType = '<M-j>'

" 关闭了提示再次触发的快捷键
" let g:ycm_key_invoke_completion = '<Leader>,'

"==============================================================================
" UltiSnips 插件
"==============================================================================
" better key bindings for UltiSnipsExpandTrigger
let g:UltiSnipsExpandTrigger = "<tab>"
let g:UltiSnipsJumpForwardTrigger = "<tab>"
let g:UltiSnipsJumpBackwardTrigger = "<s-tab>"

"==============================================================================
"  其他插件配置
"==============================================================================

" markdwon 的快捷键
map <silent> <F5> <Plug>MarkdownPreview
map <silent> <F6> <Plug>StopMarkdownPreview

" tab 标签页切换快捷键
:nn <Leader>1 1gt
:nn <Leader>2 2gt
:nn <Leader>3 3gt
:nn <Leader>4 4gt
:nn <Leader>5 5gt
:nn <Leader>6 6gt
:nn <Leader>7 7gt
:nn <Leader>8 8gt
:nn <Leader>9 8gt
:nn <Leader>0 :tablast<CR>

" 自动注释的时候添加空格
let g:NERDSpaceDelims=1

" 有道词典插件
map <M-t> :Ydc<CR>

" 自动切换到当前文件所在的目录 cdpath
map <Leader>cd :cd %:h<CR>

"==============================================================================
" GVim 的配置
"==============================================================================
" 如果不使用 GVim ，可以不用配置下面的配置
if has('gui_running')
	" colorscheme molokai
	colorscheme molokai
	set background=dark " 主题背景 dark-深色; light-浅色

	" 设置启动时窗口的大小
	set lines=999 columns=999 linespace=4

	" 设置字体及大小
    set guifont=Roboto\ Mono\ 13

	set guioptions-=m " 隐藏菜单栏
	set guioptions-=T " 隐藏工具栏
	set guioptions-=L " 隐藏左侧滚动条
	set guioptions-=r " 隐藏右侧滚动条
	set guioptions-=b " 隐藏底部滚动条

	" 在 gvim 下不会和 terminal 的 alt+数字的快捷键冲突，
	" 所以将 tab 切换配置一份 alt+数字的快捷键
	:nn <M-1> 1gt
	:nn <M-2> 2gt
	:nn <M-3> 3gt
	:nn <M-4> 4gt
	:nn <M-5> 5gt
	:nn <M-6> 6gt
	:nn <M-7> 7gt
	:nn <M-8> 8gt
    :nn <M-9> 9gt
    :nn <M-0> :tablast<CR>

endif

"==============================================================================
" 光标随着插入模式改变形状
"==============================================================================
if has("autocmd")
	au VimEnter,InsertLeave * silent execute '!echo -ne "\e[2 q"' | redraw!
	au InsertEnter,InsertChange *
		\ if v:insertmode == 'i' |
		\   silent execute '!echo -ne "\e[6 q"' | redraw! |
		\ elseif v:insertmode == 'r' |
		\   silent execute '!echo -ne "\e[4 q"' | redraw! |
		\ endif
	au VimLeave * silent execute '!echo -ne "\e[ q"' | redraw!
endif

"==============================================================================
" vim 的 session 管理
"==============================================================================
"
" 自动保存 session
autocmd VimLeave * mks! ~/.vim/session.vim
" 加载 session 的快捷键
nmap <Leader>his :source ~/.vim/session.vim<CR>

"==============================================================================
" vim-go 插件
"==============================================================================

" go 语言快捷键支持
" gd 快速打开:GoDef，GoDef支持代码内跳转到指定函数
" gr 快速执行:GoRun , 运行go程序
au FileType go cnoremap gd :GoDef<CR>
au FileType go cnoremap gr :GoRun<CR>

let g:go_fmt_command = "goimports"
let g:go_def_mode = 'gopls'
let g:go_info_mode='gopls'

let g:go_version_warning = 1
let g:go_highlight_types = 1
let g:go_highlight_fields = 1
let g:go_highlight_functions = 1
let g:go_highlight_function_calls = 1
let g:go_highlight_operators = 1
let g:go_highlight_extra_types = 1
let g:go_highlight_methods = 1
let g:go_highlight_generate_tags = 1

" godef跳转方式设置
let g:godef_split=2
```

