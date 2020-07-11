# centos7+vim8.2+python3.8.3

## python3.8.3安装流程

1. 安装依赖

```
#下载依赖包
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make

#运行这个命令添加epel扩展源
yum -y install epel-release

#安装pip
yum install python-pip
```

2. 下载Python3.8.3源码包

```
wget https://www.python.org/ftp/python/3.8.3/Python-3.8.3.tgz
```

3. 编译python3.8.3源码包

```
tar -vxf Python-3.8.3.tgz
```

4. 进入解压后的目录，依次执行下面命令进行手动编译

```
./configure prefix=/usr/local/python3 --enable-shared
make && make install
```

5. 添加软链接

```
#查看/usr/bin下的python可执行程序
ls /usr/bin/ | grep python
abrt-action-analyze-python
pmpython
python
python2
python2.7
python2.7-config
python2.7-futurize
python2.7-pasteurize
python2-config
python3.4
python3.4-config
python3.4m
python3.4m-config
python3.4m-x86_64-config
python-config


#将原来的链接备份
mv /usr/bin/python /usr/bin/python.bak

#添加python3的软链接
ln -s /usr/local/python3/bin/python3.8 /usr/bin/python

#拷贝动态链接库
cp /usr/local/python3/lib/libpython3.8.so.1.0 /usr/local/lib64/
cp /usr/local/python3/lib/libpython3.8.so.1.0 /usr/lib/
cp /usr/local/python3/lib/libpython3.8.so.1.0 /usr/lib64/

cp /usr/local/python3/bin/* /usr/local/bin/

#测试是否安装成功
python -V
```

6. 更改yum配置，因为其要用到python2才能执行，否则会导致yum不能正常使用

```
vi /usr/bin/yum
把#! /usr/bin/python修改为#! /usr/bin/python2

vi /usr/libexec/urlgrabber-ext-down
把#! /usr/bin/python 修改为#! /usr/bin/python2
```

## vim8.2安装流程

1. 安装依赖

```
配置epel源
yum install epel-release
安装vim8.2编译环境
yum install -y gcc ncurses-devel wget libzip bzip2 git
```

2. 下载vim8.2

```
wget https://ftp.nluug.nl/pub/vim/unix/vim-8.2.tar.bz2
tar -jxf vim-8.2.tar.bz2
cd vim82
```

3. 编译安装并支持python3

```
make clean
./configure --prefix=/usr/local/vim/ --enable-fail-if-missing --enable-python3interp --enable-multibyte --enable-fontset --with-features=huge
make
make install
```

4. 更新vim环境配置信息

```
whereis vim
vim: /usr/bin/vim /usr/local/bin/vim /usr/local/vim /usr/share/man/man1/vim.1.gz
cp -d /usr/local/vim/bin/* /usr/bin/
cp -d /usr/local/vim/bin/* /usr/local/bin/
```

5. 查看是否安装成功

```
vim --verison
VIM - Vi IMproved 8.2 (2019 Dec 12, compiled Jul  4 2020 19:51:44)
编译者 root@yf_20_203
巨型版本 无图形界面。  可使用(+)与不可使用(-)的功能:
+acl               -farsi             -mouse_sysmouse    -tag_old_static
+arabic            +file_in_path      +mouse_urxvt       -tag_any_white
+autocmd           +find_in_path      +mouse_xterm       -tcl
+autochdir         +float             +multi_byte        +termguicolors
-autoservername    +folding           +multi_lang        +terminal
-balloon_eval      -footer            -mzscheme          +terminfo
+balloon_eval_term +fork()            +netbeans_intg     +termresponse
-browse            +gettext           +num64             +textobjects
++builtin_terms    -hangul_input      +packages          +textprop
+byte_offset       +iconv             +path_extra        +timers
+channel           +insert_expand     -perl              +title
+cindent           +job               +persistent_undo   -toolbar
-clientserver      +jumplist          +popupwin          +user_commands
-clipboard         +keymap            +postscript        +vartabs
+cmdline_compl     +lambda            +printer           +vertsplit
+cmdline_hist      +langmap           +profile           +virtualedit
+cmdline_info      +libcall           -python            +visual
+comments          +linebreak         +python3           +visualextra
+conceal           +lispindent        +quickfix          +viminfo
+cryptv            +listcmds          +reltime           +vreplace
+cscope            +localmap          +rightleft         +wildignore
+cursorbind        -lua               -ruby              +wildmenu
+cursorshape       +menu              +scrollbind        +windows
+dialog_con        +mksession         +signs             +writebackup
+diff              +modify_fname      +smartindent       -X11
+digraphs          +mouse             -sound             -xfontset
-dnd               -mouseshape        +spell             -xim
-ebcdic            +mouse_dec         +startuptime       -xpm
+emacs_tags        -mouse_gpm         +statusline        -xsmp
+eval              -mouse_jsbterm     -sun_workshop      -xterm_clipboard
+ex_extra          +mouse_netterm     +syntax            -xterm_save
+extra_search      +mouse_sgr         +tag_binary
     系统 vimrc 文件: "$VIM/vimrc"
     用户 vimrc 文件: "$HOME/.vimrc"
 第二用户 vimrc 文件: "~/.vim/vimrc"
      用户 exrc 文件: "$HOME/.exrc"
       defaults file: "$VIMRUNTIME/defaults.vim"
         $VIM 预设值: "/usr/local/vim/share/vim"
编译方式: gcc -std=gnu99 -c -I. -Iproto -DHAVE_CONFIG_H     -g -O2 -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=1
链接方式: gcc -std=gnu99   -L/usr/local/lib -Wl,--as-needed -o vim        -lm -ltinfo -lnsl  -lselinux  -ldl     -L/usr/local/lib/python3.8/config-3.8-x86_64-linux-gnu -lpython3.8 -lcrypt -lpthread -ldl -lutil -lm -lm
```

