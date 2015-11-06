title: Vundle安装
comment: true
toc: true
share: true
date: 2015-10-08 14:19:26
tags: [linux, vim]
---

   轻度vimer,一直想好好使用vim，无耐win已经把我养成了一个懒人，好几次都有冲动想好好使用vim，把它用好，所有的文本操作都在上面进行。vim的学习曲线比较强，而且很多东西，过一段时间不用就忘了，所以，每次都学了一些，然后又忘了一些。现在的态度就是，能学多少学多少，也不刻意去学了，该用的时候就用。

   vim的插件众多，玩过vim的同学都知道，vim的插件基本都是用vim那个脚本写出来的，一般都是下载`.vim`结尾的文件，然后放到vimfiles下面。这种方式带来的不好之处就在于，如果你换了一台电脑，那么你的插件就得重新下载。当然，也可以从一台电脑拷到另一台电脑。

   vim做为神一样的编辑器，发展了这么多年，它的插件管理(器)自然也毫不逊色。常用的插件管理器有[Vbundle](https://github.com/VundleVim/Vundle.vim)和[Pathogen](http://www.vim.org/scripts/script.php?script_id=2332)。在这里简单介绍一下`Vundle`。


<!-- more -->

 1. 前提准备  

   你的电脑需要安装git和curl。如果是linux或者mac，那么就很好办。
   在centos下，执行`yum install git curl`就可以。
   在ubuntu下，执行`sudo apt-get install git curl`就可以。
   在mac下，不需要做什么。。。
   在windows下，那就比较麻烦，建议安装[msysgit](https://git-for-windows.github.io/)，里面带有curl，具体怎么使用，可自行百度、google。

2. 下载Vundle
   `git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim`

3. 配置插件
   
   在你的`.vimrc`文件中添加如下配置，使得vundle能够工作。
   ``` vim
   set nocompatible              " be iMproved, required
   filetype off                  " required

   " set the runtime path to include Vundle and initialize   
   set rtp+=~/.vim/bundle/Vundle.vim
   call vundle#begin()

   " let Vundle manage Vundle, required
   Plugin 'VundleVim/Vundle.vim'

   " own plugin
   Plugin 'https://github.com/Lokaltog/vim-powerline.git'
   Plugin 'The-NERD-tree'
   Plugin 'taglist.vim'
   Plugin 'OmniCppComplete'
   Plugin 'winmanager'
   Plugin 'altercation/vim-colors-solarized'
                                                                                                   
   " All of your Plugins must be added before the following line
   call vundle#end()            " required
   filetype plugin indent on    " required
   ```

   中间有几个插件是我自己用的插件，不需要的可以去掉，但是，其中的`Plugin 'VundleVim/Vundle.vim'`必须要。
   你也可以将自己需要的插件放在`call vundle#end()`之前。

4. 安装插件

   将插件在`.vimrc`中添加之后，只需要在vim中运行`:PluginInstall`就可以自动安装插件了。

5. 其它操作

   列出所有插件:`PluginList`

   删除插件：在`.vimrc`删掉，保存退出，重新打开vim，执行`PluginClean`

有了Vundle，以后走到哪都只要带着.vimrc就行了，然后只要`PluginInstall`一下，插件就可以自动安装了。

   你可以输入`:h vundle`查看Vundle的Vimdoc获得更多得操作。

