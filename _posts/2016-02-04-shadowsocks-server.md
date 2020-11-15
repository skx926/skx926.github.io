---
title: 搭建自己的Shadowsocks翻墙服务器
date: 2016-02-04 17:40:35
tags: 工具
---

其实现在网上已经有很多类似的文章了，这种教程对于大多数人来说可能只需要看一次就行了，但也有可能过一段时间服务器被墙了或者速度不行ping值太高需要重新在新的服务器上搭建。所以我在此把我搭建以及优化的过程纪录一下，以便不时之需。

### 为什么要翻墙
我已经想不起来我是什么时候开始翻墙了，其实我一开始翻墙就是为了多了解一些关于我党的黑历史，希望上Twitter、facebook、Google、Youtube能发现一些普通人所不知道的事情。一开始确实如我所愿，但是知道这些事情并不能改变什么，渐渐的对这些事情失去了兴趣。后来随着技术的学习，对国外的产品和技术产生了一定的崇拜，尤其是Google，但是万恶的GFW阻断了我和这些优秀的产品和技术之间的联系，所以我不得不翻墙来继续保持我和它们的联系。

### 为什么要搭建自己的翻墙服务器
翻墙有很多的渠道和方法，大多数人嫌麻烦，购买一个商业的VPN就行了，这个确实不失为一个好方法。但是我们伟大的墙随着时间的推移变得越来越智能，它已经能够通过对数据包的深度监测发现你是否在翻墙，从而阻断你的连接。常见的例如PPTP、L2TP之类的VPN协议特征比较明显，很容易被墙发现。根据我的使用体验，基本上连上几分钟后就断开了，有时候甚至连接不上。而Shadowsocks是基于一种自定义的协议，相对而言墙更难检测，所以也较为稳定。

其实现在也有很多网站提供Shadowsocks服务，但是鱼龙混杂，安全性无法得到保障，你的所有数据包都会暴漏给Shadowsocks服务器，很容易泄漏一些账号密码之类的敏感信息。其次是速度和稳定性的问题，有一些免费的账号使用人数太多，速度和稳定性都无法得到保障。即便是付费购买的账号，由于VPS提供商对于流量和带宽的限制，而商家需要从中获利，所以永远没有自己搭建的服务器实在。

### 选择和购买VPS服务器
>作为一个完美主义者，一旦使用了VPS，就会掉进追求更快速度和最低延迟的坑。

##### 一些VPS服务提供商的介绍
###### Bandwagon
一开始我使用的是[Bandwagon](https://bandwagonhost.com/cart.php?a=add&pid=19)(搬瓦工)的3.99美元一年的VPS，这个VPS的特点就是便宜，但是它的配置很低，只有64M的内存，不过这个对于搭建一个个人使用的Shadowsocks服务器也够用了。搬瓦工的VPS的另一个好处是它的控制面板上自带了一键配置Shadowsocks的功能，这对于新手来说很好。如果你舍不得花钱，这是个不错的选择，但是不幸的是，由于卖的太火爆，已经没有存货了。

###### Linode
搬瓦工便宜，但是速度不怎么用，用它来上上Google搜索一下资料倒是没有什么问题，但是如果想看Youtube的视频那就算了吧。于是乎我又掉进了[Linode](https://manager.linode.com/linodes/add?group=)的大坑。Linode在全球有多个数据中心，其中日本的机房对于中国大陆的用户来说最爽，延时大概是在70ms左右，新加坡机房也不错，延时在140ms左右。比较悲剧的是由于日本机房太抢手，新注册的账号现在只能购买除日本机房以外的机房的VPS。延时多那么几十号秒其实影响不大，对于看视频来说最主要的还是带宽和稳定性，所以新加坡机房也挺不错，由于我注册上时间也比较晚，所以只好购买新加坡机房的VPS。根据我和朋友使用的效果来看，只要你的带宽足够，看Youtube 1080p的视频不是问题，甚至4k的视频也是可以的！但是这个是建立在你在服务器上安装了[锐速](http://www.serverspeeder.com/)的基础之上，这个后面我会详说。Linode最便宜的服务器是10美元每月，对于个人来说太贵，找几个人合租是一个不错的选择。如果在朋友当中不好找，你可以到Google Plus上的Shadowsocks相关的社群里逛逛或者发帖留言，上面有很多活跃的分子。

###### Softlayer
不知道是不是墙越来越牛逼的缘故，Linode用着用着有时候会出现几分钟连接不上的问题。而且我现在用的这个新加坡机房的服务器在深圳移动4G的网络下怎么也连接不上，电信的WiFi下正常。所以我又有了第三次探索。偶然听朋友说[Softlayer](http://www.softlayer.com/)香港服务器不错，我就上网查了一下发现它被IBM收购了，既然是IBM出品，那质量应该有所保证。我通过[试用链接](http://www.softlayer.com/info/free-cloud)获取了一个月的试用机会，发现速度ping值确实低的离谱，很多时候只有几号秒的延时！可能因为机房在香港，我在深圳的缘故。速度也还不错，感觉没有Linode快，但是贵的离谱，最便宜的配置也需要27美金每月。如果你不是土豪的话，试用一个月就行了。

###### Digital Ocean和Vultr
[Digital Ocean](https://www.digitalocean.com/?refcode=98b3c0c5a987)和[Vultr](https://www.vultr.com/)都提供配置差不多的最低每月5美金的VPS，区别就是Vultr有日本的机房可以选择，但是根据我的使用体验，Vultr的日本机房的ping值也要200ms左右，但是稳定性不错，不会出现Linode那种间歇性连不上和移动4G连不上的问题，在使用锐速优化了之后看Youtube 720p的视频问题不大，1080p的就有些吃力了。很多网上的文章说Digital Ocean的速度不错，但是根据我个人的使用体验来说，即便是使用锐速优化过之后，也无法流畅的观看Youtube 720p的视频，甚至480p的也看不了。

##### 购买VPS
购买的过程我就不多说了，大体都是注册账号、绑定信用卡或者Paypal等支付方式、选择VPS、购买。

### 安装Shadowsocks
成功购买VPS之后你应该已经能够获取到你的VPS的IP地址，root账号的密码。打开终端，使用ssh登录服务器：

```bash
$ ssh root@你的VPS的IP地址
```

然后会提示你输入密码，密码不会显示，输入完成后回车即可。登录成功之后依次输入下面的命令：

```bash
// Cent OS使用下面的方法安装
$ yum install epel-release
$ yum update
$ easy_install pip
$ pip install shadowsocks

// Debian使用下面的命令
$ apt-get install python-pip
$ pip install shadowsocks
```

等待安装完成之后我们需要配置一下Shadowsocks的账号，使用下面的命令新建一个Shadowsocks的配置文件：

```bash
$ vi /etc/shadowsocks.json
```

这时点击i键进入编辑模式，然后将下面的内容复制粘贴过去：

```json
{
    "server":"0.0.0.0",
    "port_password": {
        "9990":"password0",
        "9989":"password1"
    },
    "timeout":600,
    "method":"aes-256-cfb"
}
```

- port_password就是端口和它所对应的密码，这里可以使用多个端口和密码，你可以把它分配给不同的人，每个人使用单独的端口。

复制粘贴完了之后按一下ESC键然后输入:wq对文件进行保存并退出vi编辑模式。然后我们通过下面的命令启动Shadowsocks服务：

```bash
$ ssserver -c /etc/shadowsocks.json -d start
```

如果你需要停止或者重启Shadowsocks服务，只需要把后面的start替换成stop或者restart。

这样我们的Shadowsocks服务器就搭建成功了。你可以在手机或电脑上使用上面IP、端口和密码连接感受一下速度。如果速度不咋地的话（一般来说肯定不咋地），那就需要安装锐速来优化一下速度。

### ~~使用锐速优化Shadowsocks~~
~~锐速是一款非常不错的TCP底层加速软件，可以非常方便快速地完成服务器网络的优化，配合 ShadowSocks 效果奇佳。目前锐速官方也出了永久免费版本，适用带宽20M的加速连接，个人使用是足够了。如果需要，先要在锐速官网注册个账户（这两天官网暂时关闭了注册和安装，不知道什么时候会恢复）。~~

- ~~搬瓦工的服务器是OpenVZ架构，无法修改内核参数，所以无法使用锐速，只有Linode、Digital Ocean和Vultr等KVM或XEN架构的服务器可以使用。~~

~~锐速只支持有限的Linux内核版本，安装之前最好查看一下自己的内核版本看是不是在这个[列表](http://my.serverspeeder.com/ls.do?m=availables)里面。如果没有，需要更换内核。~~

~~确定自己的内核在支持列表里面之后就可以使用下面的命令进行安装了：~~

```bash
$ wget http://my.serverspeeder.com/d/ls/serverSpeederInstaller.tar.gz
$ tar xzvf serverSpeederInstaller.tar.gz
$ bash serverSpeederInstaller.sh
```

~~输入在官网注册的账号和密码，参数设置直接回车默认即可，最后两项输入y设置为开机启动和立即启动。然后可以通过下面的命令查看锐速是否成功运行：~~

```bash
$ lsmod
```

~~如果能够看到一个叫appex0的模块在运行，就说明锐速已经安装并启动成功。~~

~~为了发挥锐速的最大效力，我们需要再修改一下锐速的几个参数，使用下面的命令打开锐速的配置文件：~~

```bash
$ vi /serverspeeder/etc/config
```

~~点击i进入编辑模式，然后将其中的rsc、advinacc和maxmode分别修改为1，然后点击ESC键退出编辑模式，然后输入:wq进行保存。保存之后使用下面的命令重启锐速:~~

```bash
service serverSpeeder restart
```

~~至此你可以畅快的刷推、看Youtube的视频了。Cheers！~~

### 开启 TCP-BBR 拥塞控制算法

我们都知道在国内访问国外的服务器最大的两个问题就是丢包率高和网络延迟高，而 TCP-BBR 致力于解决两个问题：
1. 在有一定丢包率的网络链路上充分利用带宽。
2. 降低网络链路上的 buffer 占用率，从而降低延迟。

因此可以有效的提高访问国外服务器的速度，具体它是怎么实现的可以参考知乎上的回答 [Linux Kernel 4.9 中的 BBR 算法与之前的 TCP 拥塞控制相比有什么优势？](https://www.zhihu.com/question/53559433)。

下面介绍一下在Digital Ocean和Vultr的CentOS7系统的VPS上开启BBR的方法：

#### 更新内核
更新系统
```bash
$ yum update -y
```

安装内核，目前最新内核版本为：`4.12.9`
```bash
$ rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
$ rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
$ yum --enablerepo=elrepo-kernel install kernel-ml -y
```

检查内核是否安装成功
```bash
$ awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
```

如果安装成功就会出现类似下面的返回
```bash
0 : CentOS Linux 7 Rescue f16713269c69461db4addeffb3a94dc9 (4.12.9-1.el7.elrepo.x86_64)
1 : CentOS Linux (4.12.9-1.el7.elrepo.x86_64) 7 (Core)
2 : CentOS Linux (3.10.0-514.21.1.el7.x86_64) 7 (Core)
3 : CentOS Linux (0-rescue-de820036e9b44905be9e94bc0f95cdc7) 7 (Core)
```

把 CentOS Linux (4.12.9-1.el7.elrepo.x86_64) 设置成默认内核
```bash
$ grub2-set-default 0
```

重启系统之后查看内核是否更换成功
```bash
$ uname -r
```

#### 开启 BBR
编辑`/etc/sysctl.conf`，加入如下内容
```bash
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

保存生效
```bash
$ sysctl -p
```

查看当前内核TCP设置
```bash
$ sysctl net.ipv4.tcp_available_congestion_control
$ sysctl net.ipv4.tcp_congestion_control
```

如果结果都有bbr，则说明内核已开启BBR算法，执行：
```bash
$ lsmod | grep bbr
```

显示tcp_bbr说明BBR已正常启动


参考文章：[Digital Ocean开启TCP-BBR拥塞控制算法](http://w3cboy.com/post/2017/08/digital-ocean-bbr/)