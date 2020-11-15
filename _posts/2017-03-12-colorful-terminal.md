---
title: 让你的Mac终端“漂亮”起来
date: 2017-03-12 22:47:33
tags: Mac技巧
---

Mac自带的终端默认的配色看起来有些单调，如何让它变的“漂亮”一些呢？这里我们以`Tomorrow`配色方案为例来进行设置。

首先我们使用下面的命令来从`Github`上克隆[Tomorrow](https://github.com/chriskempson/tomorrow-theme)项目
```bash
$ git clone https://github.com/chriskempson/tomorrow-theme.git
```

## 窗口
进入刚才下载的项目的文件夹中的`OS X Terminal`文件夹，然后双击运行`Tomorrow Night.terminal`即可导入`Tomorrow Night`主题到终端。然后我们在终端的`Preferences`中的`Profiles`选项卡中选择`Tomorrow Night`并将它设置为默认。重启终端即可应用新的配色方案。

## Vim
打开`vim`文件夹中的`colors`文件夹并将`Tomorrow-Night.vim`文件拷贝到`~/.vim/colors/`文件夹下，然后在`~/.vimrc`文件中添加一些内容来开启vim语法高亮并设置配色方案为`Tomorrow Night`：

```bash
$ vi ~/.vimrc
syntax enable
colorscheme Tomorrow-Night 
```

## ls
你可能会发现我们安装了`Tomorrow-Night`主题之后重启终端之后`ls`并没有显示高亮，这个时候我们只需要使用下面的命令启用`ls`的高亮即可
```bash
$ vi ~/.bash_profile
export CLICOLOR=1
```

## 最终效果如下：
![]({{site.url}}/assets/img{{page.id}}/terminal1.png)
![]({{site.url}}/assets/img{{page.id}}/terminal.png)

