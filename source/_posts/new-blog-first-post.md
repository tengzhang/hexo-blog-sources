title: '新博客的第一篇博文'
date: 2014-07-06 20:59:23
tags:
---
以前在github上用octopress挂了一个博客，但是用它还得装ruby，感觉比较麻烦（主要还是不懂ruby）。  
前段时间，看到有人用hexo搭建博客，我也试了试。hexo是基于node，比octopress方便多了，就转了过来。
这是他的官网[http://hexo.io/](http://hexo.io/)，关于具体怎么搭建，我就不多说了，官网写得和详细，直接上官方文档的链接[http://hexo.io/docs/index.html](http://hexo.io/docs/index.html)。
说说我遇到的问题吧，按照官网说的做了，我用`hexo deploy`，一直deploy上去，也不知道为啥，可能主要还是我对node和git都不熟悉吧。
最后我是手动deploy的。具体命令如下：
``` bash
$ hexo clean
$ hexo deploy --generate
$ cd .deploy
$ git push -u origin master
```
命令都是在hexo博客的目录下执行的。
另外奉上我使用的主题地址:[http://yangjian.me/pacman/hello/introducing-pacman-theme/](http://yangjian.me/pacman/hello/introducing-pacman-theme/)