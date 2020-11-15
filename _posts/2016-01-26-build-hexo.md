---
title: 利用Hexo和Github Pages搭建个人博客
date: 2016-01-26 23:29:28
tags: [Hexo, 博客, Github Pages]
categories: [Web]
---

### 简介
- Github Pages可以理解为一个用户编写的、托管在Github上的静态网页，它提供了300M的免费空间。因为是Github出品，所以足够稳定，这对于一个博客来说足够了。
- Hexo是一个快速、简洁且高效的博客框架。Hexo使用Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

### 配置Github Pages
这里假设你已经安装了git，拥有了Github的帐号，配置好了ssh key。

##### 在Github上建立仓库
登录Github之后点击首页的New repository创建一个新的仓库。

- 需要注意的是创建仓库的时候Repository name是特定的，不能随意起名字。比如我的账户名是skx926，这个仓库的名字就只能叫skx926.github.io，也就是说一个帐号只能创建一个Github Pages页面。

创建完成之后我们可以把这个空的仓库克隆到本地自己喜欢的任意目录。

### 使用Hexo

##### 安装Hexo
要安装Hexo必须先安装nodejs，这里假设你已经安装好了nodejs。使用下面的命令安装Hexo。

``` bash
$ npm install -g hexo
```

##### 创建博客
在终端输入下面的命令会在当前目录下创建一个名为skx926的文件夹，比并且Hexo会在此目录生成建立博客的所有文件。

``` bash
$ hexo init skx926
```

输入下面的命令就会运行起刚刚创建好的博客，在浏览器输入[http://0.0.0.0:4000](http://0.0.0.0:4000) 预览一下你的博客。

``` bash
$ hexo server
```

##### 下载主题
我们刚才创建好的博客里面包含了默认的主题landscape，你可以在/yourblog/themes文件夹下看到。不过这个主题我个人感觉不好看，好在Hexo有大量丰富的[Themes](https://hexo.io/themes/)可供我们选择，我个人比较喜欢[Maupassant](https://github.com/tufu9441/maupassant-hexo)这个主题。确保终端现在位于博客的根目录下，然后使用下面的命令将主题下载到博客的themes目录下。

``` bash
$ git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant
```

##### 配置博客
打开_config.yml文件，这个文件就是整个博客的配置文件，你可以修改里面的参数来对你的博客进行个性化的设置，比如设置博客的名称、主题什么的。我现在将主题修改成刚才下载的主题的名字maupassant，然后使用下面的命令预览一下新的主题。

``` bash
$ hexo server --watch
```
你可能注意到这里的命令比上面的多了个参数--watch，这个命令的作用就是监听文件的变化，只要有变动，就会自动重新生成博客的页面，就不用每次改一点东西都要手动运行一下hexo server命令了。

- 其实除了根目录的_config.yml文件以外每个主题的文件夹里都会有一个_config.yml文件，这些文件是对它所对应的主题的一些设置。

##### 发表博文
使用下面的命令新建一个博文。

``` bash
$ hexo new "我的第一篇博文"
```

执行完命令你会发现 sources/\_posts 文件夹内多了一个 "我的第一篇博文.md" 文件，这个就是文章的[markdown](https://daringfireball.net/projects/markdown/)格式的源文件。你也可以直接进入 sources/\_posts 目录新建一个文件。这两种方式的区别就是前者生成的文件中会自动配置好一些文章的信息，比如名称、时间等。下面就是一篇新建的博文的示例。

```
title: 我的第一篇博文

date: 2016-01-26 23:44:09

tags: [博客, Hexo]

categories: Web
---

如何新建一个博客呢？
```

tags是这篇博文的标签，categories是文章的分类，分类和标签都可以添加多个，用中括号括起来，中间用英文的逗号隔开。---的后面就是博文的正文部分了。

##### 部署到Github Pages
文章写好了，怎么把它发布到网站上去呢？
比较传统的方法是先用下面的命令生成博客的静态页面文件。

``` bash
$ hexo generate
```

你会发现多了一个public文件夹，这个文件夹里就是生成的目标文件，把文件复制到我们一开始克隆到本地的仓库中去，然后再通过git的各种命令提交到Github远程仓库。

你可能会想：这也太麻烦了吧？一点也不优雅。是的，hexo已经为我们解决了这个问题，使用deploy命令就可以一键吧pulic文件夹呢的静态页面文件提交到远程Github仓库里。但是使用这个命令之前我们需要做一些配置，打开_config.yml文件，在里面添加下面的内容，如果已经有的话就只需要修改一下。

```
deploy:
  type: git
  repository: https://github.com/skx926/skx926.github.io.git
  branch: master
```

repository就是我们一开始创建的Github仓库的地址。修改完成然后保存一下。由于新版的Hexo已经将deploy分离出来，所以我们需要安装一下它，执行下面的命令安装。

``` bash
$ npm install hexo-deployer-git --save
```

安装完之后使用下面的命令进行部署。

``` bash
$ hexo d -g
```

上面的命令其实是做了两件事，第一件事通过generate命令生成页面，紧接着通过deploy命令将其部署到Github。顺利的话打开浏览器输入你的仓库的名称就可以看到博客里，例如我的是[skx926.github.io](http://skx926.github.io)。

##### 版本管理
其实到这里搭建博客的整个过程已经基本上完成了，但是有个问题，就是：我们把博客的页面部署到了Github，但是博客的源文件还在本地，万一哪天电脑坏了，岂不是没了？其实这个问题很简单，直接在Github上再新建一个仓库来存储博客的源文件不就完了？处女座的人表示无法忍受一个博客两个仓库这种事，不过这个只是我自己的解决办法，不一定是最好的解决方案，但是能解决问题，先凑合着用吧。

### 绑定域名
Github Pages本身就已经给你免费提供了一个二级域名，但是很多人想我一样不喜欢这个域名，想要绑定自己的域名，那该怎么办呢？其实很简单，首先你得有个域名。关于注册域名，网上有很多教程，这里就不说了，有一个域名比价的网站我觉得挺不错的，地址是https://www.domcomp.com。
这里假设你已经有一个自己域名了。

1. 在sources文件夹下新建一个叫CNAME的文件，文件的内容就写你的域名地址，比如我的androiosx.com。
2. 在你的域名服务商提供的DNS管理页面添加相应的纪录或者使用DNSPod进行解析，我推荐使用DNSPod。

##### 使用DNSPod解析域名

首先你需要注册一个[DNSPod](https://www.dnspod.cn)帐号，登录成功后在域名解析模块添加一个域名。添加完成打开这个域名你会发现有两条默认的NS纪录，我们需要添加两条CNAME类型的纪录，主机纪录分别写@和www，纪录值就写Github为你提供的域名的地址，比如我的[skx926.github.io](http://skx926.github.io)，其他的都保持默认。

添加完成之后分别复制那两条默认的NS纪录后面的纪录值f1g1ns1.dnspod.net.和f1g1ns2.dnspod.net.。然后进入你的域名提供商的管理面板，找到域名的NameServer管理，使用刚才复制的DNSPod提供的NameServer替换掉默认的NameServer，替换完成后保存就大功告成了。
