---
title: 前端工程师使用 Deepin 笔记
date: 2018-11-02 21:25:42
categories: Linux
---
![_ _20180704114914](https://user-images.githubusercontent.com/25274581/42255823-4ef7f7f0-7f80-11e8-8f0c-d553a2f84db1.png)

笔者是一枚前端开发，在学习 Linux 的时候碰到了一个问题 —— 怎么练手？因为自己电脑上面装的是 Windows 系统，所以学习 Linux 的时候没办法进行练习，而敲指令是学习 Linux 最高效的途径，这就需要我装一个 Linux 虚拟机或者双系统了。最开始的时候我用 VMWare 的虚拟机装了个 Ubuntu，后来觉得 Linux 好像确实好用，虚拟机又太耗资源，再加上我的 Windows 越来越卡顿，我决定装一个双系统。
<!-- more -->
一开始的时候，我还是安装的 Ubuntu 系统，后来发现真的用 Ubuntu 系统进行工作学习的话，好像还是有很多的困难的，首先是 QQ 官方停止了对 Linux 平台的开发支持，在 Ubuntu 上面很难装 QQ；其次，Ubuntu 的字体什么的对于中文支持都还不够完善；再者，网易云音乐在 Ubuntu 上的表现也不是很好（启动都需要使用 `sudo netease-cloud-music` 命令来实现，不然点击图标都没得反应）。总之，Ubuntu 对于中国用户的日常使用而言，不太合适（当然人家可能压根就不是为了日常使用而开发的，更不是为了中国人开发的）。为了更好的学习 Linux，在经过了解之后，我决定安装 Deepin 作为我的日常使用系统。这是由武汉深之度科技公司开发的针对中国用户量身定制的 Linux 系统，预装搜狗输入法、QQ、网易云音乐等常用软件，应用商店也是应有尽有（甚至还有 Steam……虽然我没用过，详情见下图），用起来十分舒服。

![_deepin-appstore_20180704114634](https://user-images.githubusercontent.com/25274581/42255763-ef5d2eaa-7f7f-11e8-9697-0c3ed4e1b7b9.png)

## 一、怎么安装 Deepin？

好了，讲了这么多，那我们到底该怎么安装 Deepin 双系统呢？

以 Windows7 为例，大概包括以下几个步骤：

*   （1）磁盘管理，划分出一个大概 60G 的磁盘空间，不需要分配盘符。至于怎么划分磁盘，参考[教程](https://www.jb51.net/os/windows/41969.html)。
*   （2）在 Deepin 官网下载 [iso 镜像](https://www.deepin.org/download/) 以及[启动盘制作工具](https://www.deepin.org/original/deepin-boot-maker/)。
*   （3）将启动盘插入，重启电脑，进入 BIOS，选择从启动盘启动。
*   （4）按照指引完成安装，记得选择安装在之前划分出来的磁盘，可以选择安装之前将其格式化。
*   （5）安装完成，Enjoy it！

这篇文章在 Deepin 系统中完成，所以没办法重温安装过程，只能讲一个安装的大概了。如果需要了解详细，可以参考 [Deepin 的官方安装教程](https://www.deepin.org/installation/)，其中还包含视频演示。


## 二、安装之后要做的事情

### 1. 修改启动项

安装系统完成之后，对计算机进行重启，开机时会进入引导界面。进入引导界面之后，可以看到前三个都是 Deepin 的选项，第四个叫做 `system setup`。选中这一项时，系统会报错，因为这一项是为启动 Windows 做准备的；可能由于 Deepin 的 Bug 问题，一开始是没有 Windows选项的，需要我们进入 Deepin 操作系统之后，在控制中心进行修改。

说是修改，其实也不用做什么。进入 Deepin 之后，点击“控制中心”，右侧边栏会弹出设置界面。然后选择“系统信息”，拉倒最底下可以看到“启动菜单”。随便动一动就好了，比如把一个开关打开然后关掉……这样就行了。再次重启时就可以发现引导界面的最后一项可以正确的显示 Windows 了。

### 2. 搭建开发环境

作为前端开发，我最基本的开发环境包括 VS Code、Git、Node、Python、Vim 等等。现在先安装这几个软件。

安装 Git 和 Vim 比较简单，使用 

```bash
    $ sudo apt-get install git vim
``` 

就阔以了。

安装 VS Code 有两种方法，一种是在深度商店安装，一种是在 VS Code 官网下载 .deb 包，然后使用 

```bash
    $ sudo dpkg -i 包名
```

安装就可以了。两者的区别是官网下载的是最新版，深度商店的版本要落后于官网的版本。

安装 Node 也有两种方式。一种是通过包管理器安装、一种是官网源码安装。两者的区别是包管理器安装之后包名叫做 `nodejs` 而非 `node`，运行脚本时也是 `nodejs` 命令，很不习惯，如果要修改包名还需要使用其它命令更改。

我使用的是源码安装方式。首先在 [Node.js 中文网](http://nodejs.cn/download/) 下载源代码。下载之后使用 

```bash
    $ tar -zxf node-vxx.x.x.tar.gz
```

解压源码，然后使用

```bash
    $ sudo apt-get install g++
```

安装 gcc 源码编译器。

接下来进入解压后的源码文件夹：

```bash
    $ cd node-vxx.x.x
```

运行配置文件：

```bash
    $ ./configure
```

然后开始编译：

```bash
    $ make
```

编译后开始安装：

```bash
    $ make install
```

安装完成之后就可以通过 `node -v` 查看所安装的 Node 版本是否正确了。

Python 的话，Deepin 本身就安装了 Python，而且 2 和 3 两个版本都有。如果要使用 Python 3.x 运行脚本，需要使用 `python3` 命令。切记不要卸载系统本身自带的 2 版本的 Python！另外如果要在 VS Code 中调试 Python 代码，配置文件的写法请参考[我的另一篇博客](https://www.cnblogs.com/DM428/p/9248342.html)。

另外推荐一个清理垃圾的软件，叫做 BleachBit，可以在深度商店直接安装，截图如下：

![_deepin-appstore_20180704114413](https://user-images.githubusercontent.com/25274581/42255796-2b0ef708-7f80-11e8-978d-065e7aacfced.png)
