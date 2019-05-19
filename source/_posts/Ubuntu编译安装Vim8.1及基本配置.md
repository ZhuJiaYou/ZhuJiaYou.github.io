---
title: Ubuntu编译安装Vim8.1及基本配置
date: 2019-05-18 12:20:28
tags:
  - VIM
  - Linux
  - Completor
  - Flake8
categories: Linux 技术
---
## 一 引言
1. Vim是一款强大简洁的代码编辑器；
2. 不过想要它成为开发的千里马，还需要一番调教；
3. 这里针对本人的日常使用经验，提供一份详细简约的Vim8.1基础配置方案。
<!--more-->

## 二 Vim编译安装
为什么要编译安装呢？主要是因为从操作系统源使用apt安装别人编译好的版本，大多没有python等支持，在使用一些很方便的自动补全等插件时，会出问题。
1. 安装依赖库：
(1) 安装libncurses5-dev，否则编译时会报`no terminal library found`错误：
`sudo apt install libncurses5-dev`
(2) 由于这里要添加python支持，所以要装python3-dev(或者python-dev，对于python2用户)，否则编译时报`Python.h: No such file or directory`错误：
`sudo apt-get install python3-dev`
2. 克隆Vim源代码，并进入目录：
`git clone https://github.com/vim/vim`
`cd vim`
3. 配置Vim编译选项：
```
./configure --with-features=huge \
                  --enable-multibyte \
                  --enable-rubyinterp=yes \
                  --enable-python3interp=yes \
                  --with-python3-config-dir=/usr/lib/python3.5/config-3.5m-x86_64-linux-gnu \
                  --enable-pythoninterp=yes \
                  --with-python-config-dir=/usr/lib/python2.7/config-x86_64-linux-gnu \
                  --enable-perlinterp=yes \
                  --enable-luainterp=yes \
                  --enable-gui=auto \
                  --enable-cscope \
                  --prefix=/usr
```
参数解释如下：
--with-features=huge：支持最大特性
--enable-rubyinterp：打开对ruby编写的插件的支持
--enable-pythoninterp：打开对python编写的插件的支持
--enable-python3interp：打开对python3编写的插件的支持
--enable-luainterp：打开对lua编写的插件的支持
--enable-perlinterp：打开对perl编写的插件的支持
--enable-multibyte：打开多字节支持，可以在Vim中输入中文
--enable-cscope：打开对cscope的支持
--with-python-config-dir=/usr/lib/python2.7/config-x86_64-linux-gnu/ 指定python config路径
--with-python-config-dir=/usr/lib/python3.5/config-3.5m-x86_64-linux-gnu/ 指定python3 config路径(**根据自己系统实际情况配置**)
--prefix=/usr：指定将要安装到的路径(可自行创建)
--enable-gui：GUI支持，可用auto、gtk2或者gnome
4. 编译安装：
`make`
`sudo make install`


## 三 基本配置
可以参考我的.vimrc文件：
`https://github.com/ZhuJiaYou/ZJYVimConfig/blob/master/.vimrc`


## 四 自动补全插件Completor
1. 安装自动补全插件Completor：
`mkdir -p ~/.vim/pack/completor/start`
`cd ~/.vim/pack/completor/start`
`git clone https://github.com/maralla/completor.vim.git`
2. python补全支持：
(1)安装pip3：
`sudo apt install python3-pip`
(2)安装jedi：
`pip3 install jedi`
.vimrc中加入安装jedi的python路径(上三中基本配置中已有)：
`let g:completor_python_binary = '/path/to/python/with/jedi/installed'`
3. C/C++补全支持：
(1)安装clang：
`sudo apt install clang`
在.vimrc中添加(已有)：
`let g:completor_clang_binary = '/path/to/clang'`
(2)C++11 支持：
`vim ~/.vim/pack/completor/start/completor.vim/.clang_complete`
写入：
`-std=c++11`
还可添加其它配置，可[参考](https://github.com/maralla/completor.vim)
(3)去掉C/C++补全自动占位符，在.vimrc中添加(已有)：
`let g:completor_clang_disable_placeholders = 1`


## 五 代码格式检查插件flake8
1. 安装：
```
mkdir -p ~/.vim/pack/flake8/start/
cd ~/.vim/pack/flake8/start/
git clone https://github.com/nvie/vim-flake8.git
```
2. 按F7可代码检查。
3. 详细配置请[参考](https://github.com/nvie/vim-flake8)。


