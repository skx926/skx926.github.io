---
title: 打造一台你的专属黑苹果
date: 2016-11-28 23:22:06
tags: [黑苹果]
categories: [Mac]
---

要打造一台你的专属黑苹果需要经过以下四个大的步骤：

1. 配件购买
2. 组装机器
3. 安装系统
4. 安装驱动

### 1.配件购买

对于Mac的兼容配件的购买可以查看tonymacx86的[Buyer's Guide](https://www.tonymacx86.com/buyersguide/november/2016)，里面已经很详细的列出了很多推荐的配件和购买的链接。美中不足的是里面给出的链接都是国外的网站，不支持国内的邮寄地址。国内要想买就得通过海淘过程比较繁琐，有些转运公司也不支持电脑配件的邮寄，邮寄的周期也比较长，一般都得半个月。我自己以前没有海淘过东西，所以不推荐，当然以前如果有过海淘经历的朋友可以根据自己的情况来定。我个人推荐京东和天猫，京东相对来说更加正规而且送货快（这个很重要）。

tonymac上列出了组装一台电脑所有需要的配件，其实要与Mac系统兼容，最主要的就是主板和显卡，基本这两个兼容了就没啥大问题。因为主板上包含网卡、声卡、电源管理等芯片。像机箱、电源、硬盘、内存等等这些配件基本上没有兼容性的问题（前提是接口相同），这些东西你都可以根据自己的喜好进行购买。

购买配件之前首先要确定你想要哪种类型的主板，主板根据面积从大到小可以分为ATX、mATX、iTX。最常见的就是ATX类型的主板，也是体积最大的。不同大小的主板就意味着需要选用不同形状的机箱，不同的接口，这个务必要搞清楚。

下面是我购买的配件列表：

[英特尔（Intel）酷睿四核 i7-6700k 1151接口 盒装CPU处理器](https://item.jd.com/1748176.html)
[技嘉（GIGABYTE）Z170X-Gaming 5主板 (Intel Z170/LGA 1151)](https://item.jd.com/1791949.html)
[影驰（Galaxy) GTX 970 名人堂海外版 1178MHz 4GB D5独立游戏显卡](https://item.jd.com/1533984592.html)
[英睿达(Crucial)铂胜运动LT系列DDR4 2400 16G（8Gx2 套装）台式机内存](https://item.jd.com/3159710.html)
[振华（SUPER FLOWER）额定650W 冰山金蝶GX650 电源](https://item.jd.com/600637.html)
[九州风神（DEEPCOOL） 玄冰400 CPU散热器](https://item.jd.com/598827.html)
[先马（SAMA）坦克 黑色 中塔式电脑机箱](https://item.jd.com/1579807.html)
[TP-LINK TL-WDN4800 450M双频无线PCI-E网卡](https://item.jd.com/968915.html)

其中的显卡是在闲鱼上买了个二手的，这些配件总共加起来7000+，再加一台Dell的4K显示器P2415Q总共10000+，性能跟2015年的iMac 5K差不多，价格却只有它的一半。

### 2.组装机器

这个过程其实没什么说的。如果你是第一次组装电脑，可能会无从下手。我建议你找个装机教程，花点时间一步一步自己动手来做。其实装机的过程也是蛮有趣的，把一堆奇形怪状的零件组装在一起，点亮的那一刻还是很爽的。

需要注意的CPU的安装要特别小心，一定要方向和位置对准了再安装。主板上的CPU卡槽那里的成百上千根针很细很脆弱，一旦把压弯了就很麻烦了。另外一个需要注意的点是现在很多机箱都支持走背线，比如我买的这一款先马坦克。走背线可以让机箱里的走线清晰明了，看着舒服很多。装完电脑各种配件的保修卡、发票等最好保留好，日后也许用得到。

晒一下我的成果：

![]({{site.url}}/assets/img{{page.id}}/case.jpg)

### 3.安装系统

安装的具体过程我在这里就不细说了，你可以根据这个[教程](https://www.tonymacx86.com/threads/unibeast-install-macos-sierra-on-any-supported-intel-based-pc.200564/)来安装。

我说一下过程中需要注意到以下两点：

1. 教程里说安装Sierra需要16G或以上的U盘，其实理论上8G的U盘就够了。为什么教程里需要16G呢？正常来说8G的U盘可用容量肯定不到8G，但是也不会差多少。但是有些U盘格式化之后的容量却只有7G多一点，这样可能写入系统再加上[MultiBeast](https://www.tonymacx86.com/resources/categories/tonymacx86-downloads.3/)等一些工具就不够了。
2. BIOS的设置可以说是非常重要，它关系到你能不能进入到系统安装界面。如果不能顺利进入系统安装界面，这无疑是对你的重大打击。BIOS中需要重点照顾的选项包括：
 - 设置 Extreme Memory Profile(X.M.P) 为 Profile1
 - 设置 Secure Boot 为 Disabled
 - 设置 XHCI Hand-off 为 Enabled
 - 设置 Port 60/64 Emulation 为 Disabled
 - 设置 Super IO Configuration 里的 Serial Port 1 为 Disabled
 - 设置 VT-d 为 Disabled
 - 设置 IOAPIC 24-119 Entries 为 Disabled
 - 设置 Wake on LAN 为 Disabled
 - 设置 Legacy USB Support 为 Auto

有一些BIOS的设置对于安装系统来说不是必须的，但是可能会导致一些Bug的产生，后面的问题解决部分我会进行详细的说明。

### 4.安装驱动

前面的[教程](https://www.tonymacx86.com/threads/unibeast-install-macos-sierra-on-any-supported-intel-based-pc.200564/)其实已经包括了使用[MultiBeast](https://www.tonymacx86.com/resources/categories/tonymacx86-downloads.3/)来安装驱动，我为什么要把这一块单独拿出来讲呢？其实主要是觉得[教程](https://www.tonymacx86.com/threads/unibeast-install-macos-sierra-on-any-supported-intel-based-pc.200564/)里说的还不够细，你并不知道你的声卡、网卡等是什么型号，所以用[MultiBeast](https://www.tonymacx86.com/resources/categories/tonymacx86-downloads.3/)也就不知道按选择哪个驱动来进行安装。这个问题其实不难解决，你可以通过搜索跟你的主板相同或相近的人的配置来选择你的驱动，比如我的MultiBeast的配置就如下图所示：

![]({{site.url}}/assets/img{{page.id}}/multibeast.png)

安装N卡的驱动需要到NVIDIA的网站下载[Web Driver](http://www.nvidia.com/Download/index.aspx?lang=en-us)进行安装，安装完成之后需要配置Clover的config.list文件才能够启用NVIDIA的显卡驱动，10.12以前的系统需要设置在Boot里的Arguments里面添加nvda_drv=1，10.12以后的系统需要在SystemParameters中添加NvidiaWeb属性并设置为YES。config.plist文件位于EFI分区的/EFI/CLOVER/目录中，EFI分区默认是没有挂载的，要对config.plist文件进行操作，首先得挂在EFI分区，我建议使用[Clover Configurator](https://www.tonymacx86.com/resources/clover-configurator.328/)来进行对EFI分区的挂载和对config.plist文件的操作。这个软件长这样：

![]({{site.url}}/assets/img{{page.id}}/configurator.png)

安装完驱动重启以下你的机器基本上就可以正常的运行了。

### 安装过程中遇到的问题和解决方案

[教程](https://www.tonymacx86.com/threads/unibeast-install-macos-sierra-on-any-supported-intel-based-pc.200564/)的最后面包含了一些比较常见的问题的解决方案。

这里还有一片帖子[Common (some unsolved) Problems in 10.12 Sierra](https://www.tonymacx86.com/threads/readme-common-some-unsolved-problems-in-10-12-sierra.202316/)里面收集了许多用户在安装最新系统10.12 Sierra遇到的问题，很有参考意义。

我在这里着重说一下我的这套配置在安装过程中所遇到的问题：

##### 无法进入安装界面

这个问题其实是由BIOS的某一项设置没有设置正确所导致的，具体是哪一项我也记不清楚了。只要做到我上面列出的那些设置，一般应该问题不到。

##### 屏幕左上角有个黑色的条在不停的闪动

这个问题跟英特尔第六代CPU的核显HD530有关系，因为我一开始还没有安装独立显卡，所以会遇到这个问题，如果你使用独立显卡，就不会有这个问题。解决办法其实在config.plist文件里添加一些东西，具体添加的内容可以看这个帖子[Skylake Intel HD 530 Graphics Glitch Fix](https://www.tonymacx86.com/threads/skylake-intel-hd-530-graphics-glitch-fix.206410/)。

##### 关机之后会自动重启

这个问题的解决办法就是使用[Clover Configurator](https://www.tonymacx86.com/resources/clover-configurator.328/)打开config.plist文件并勾选Acpi部分的FixShutdown选项。

##### WiFi使用一段时间之后就会无法联网

这个问题同样跟BIOS的设置有关，设置 Wake on LAN 为 Disabled就可以解决。

##### 睡眠之后无法唤醒屏幕

解决这个问题需要在config.plist中设置darkwake=0，意思就是禁用Mac的[Power Nap](https://support.apple.com/zh-cn/HT204032)功能。对于darkwake这个参数的其他值的意义可以查看这篇文章[Power Nap功能与Darkwake参数](http://www.yekki.me/power-nap-and-darkwake-argument/)。

##### 睡眠之后又会立刻唤醒

这个问题其实跟USB有关，当我选择睡眠之后拔掉鼠标和键盘等USB设备之后就能正常的睡眠了。我之前
BIOS中设置Legacy USB Support 为 enabled，当我尝试把它改成Auto之后睡眠的问题就解决了。

##### 睡眠之后声卡驱动失效

下载[CodecCommander.kext](https://www.tonymacx86.com/attachments/codeccommander-kext-zip.146742/)并使用[KextBeast](https://www.tonymacx86.com/resources/kextbeast.32/)安装kext文件就可以解决这个问题。

##### 系统可以正常进入但是无法启动Recovery HD

下载[FakeSMC.kext](https://www.tonymacx86.com/resources/fakesmc.325/)并放到EFI分区的/EFI/CLOVER/kexts/Other目录下。FakeSMC.kext的作用欺骗macOS让它以为你的电脑是一台白苹果，从而顺利安装和启动。

***

到这里我的黑苹果已经折腾的差不多了，可以愉快的玩耍了！如果安装过程中遇到任何问题，欢迎到下面留言与我交流。

最后再送上美图一张。

![]({{site.url}}/assets/img{{page.id}}/computer.jpg)
