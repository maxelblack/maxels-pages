---
layout: post
title: "Arch Linux 安装指引"
author: maxelbk
categories: [ "Arch Linux", "技术", "教程" ]
image: r/pic/SCS_ARCHISO_202104.png
---

At first,ARCH IS THE BEST!

按照 [Arch Wiki](https://wiki.archlinux.org/title/Arch_Linux){:target="_blank"} 的说法，Arch Linux 是一个简洁、现代、实用且以用户为中心的 GNU/Linux 发行版，采用滚动更新模式，尽全力提供最新的稳定版软件。

鉴于 Arch Wiki 的安装指引有些许复杂，于是就写了这一篇指引，帮助想要一个完全自定义化的系统的 Linux 入门级爱好者更快配置一个可以满足日常使用的 Arch Linux 。（大佬就不要看这篇指引了，直接看 [Arch Wiki](https://wiki.archlinux.org/title/Installation_guide){:target="_blank"} 更加合适）

顺带一提，这篇指引中几乎所有 Arch Wiki 外链都是英文原版，若有需要请自行切换语言（PC端左侧/移动端页末）

## 准备工作

你需要：

- **一个好用的脑子**（没错这是最重要的）
- 一个标称容量大于或等于 1 GB 的U盘（用来装 Live CD）
- 足够的时间（安装基本需要全程守着，用时取决于设备配置和你的运气，一般 1h 左右）
- 一台能联网的设备和一个稳定的网络（Arch是联网安装的）

另外，Arch安装需要**手动输入大批量指令**（尽管有个安装向导程序但功能十分弱），如果认为很麻烦或有指令恐惧症就不要尝试了。

## 开始

#### 下载镜像

Arch官方源服务器在国外，国内速度相当慢（每秒几十KB的节奏），故建议从镜像源下载，这里推荐 [OpenTUNA](https://opentuna.cn/){:target="_blank"} 。

右侧 获取下载链接 > Arch Linux > 直接点击版本号（一般只会显示最新版）即可下载。

当然其他镜像源也可以，具体方法请自行搜索。

下载之后建议校验一下文件是否完整（文件的Hash值一般会被放在镜像源同一目录下），当然如果不会也可以跳过这一步。

#### 制作可启动U盘

这里有两种方式可以选：**直接将ISO文件写入U盘** 或 **使用 [Ventoy](https://ventoy.net/) 引导U盘上的ISO文件** 。此处介绍第一种方式，有关第二种请参考 [Ventoy 官方文档](https://ventoy.net/doc_start.html){:target="_blank"} 。

首先下载 balenaEtcher 并打开。

![balenaEtcher](https://i.loli.net/2021/01/19/C48o7rlB3TIOfsu.png)

点击 **Flash from file** 并选择刚刚下载的ISO文件，然后点击 **Select target** 选择要写入的U盘，最后点击 **Flash!** 开始写入，等待进度完成即可。

#### 进入 Live 环境

首先确定你的主板是否支持 [UEFI](https://baike.baidu.com/item/%E7%BB%9F%E4%B8%80%E5%8F%AF%E6%89%A9%E5%B1%95%E5%9B%BA%E4%BB%B6%E6%8E%A5%E5%8F%A3){:target="_blank"} 引导 ，这里推荐三种方式：

- 通过系统的启动方式
  - Windows：Win + R 打开 “运行” 窗口，输入 msinfo32 并回车，在弹出的窗口中找到 “BIOS模式” 这一项，若为 “UEFI” 则说明支持 UEFI 引导，若为 “传统” 则不一定支持。
  - Linux：可以**在启动后**检查一下是否存在 /sys/firmware/efi/efivars 这个目录，若存在则支持 UEFI 引导，若不存在则不一定支持。
  - macOS：都有 macOS 了你还用啥 Linux？
  - 其他系统：请自行搜索。
- 通过主板的说明书（如果有说明书的话这个方法是最准确的）
- 通过直接搜索 你的主板**型号**+“UEFI”

关于虚拟化软件，VMware 在 Workstation Pro 15+ 中提供了完整支持，Virtual Box 6+ 及最新版的 QEMU 也有支持。上述软件均需手动在虚拟机配置中将引导方式修改为 UEFI 。

确定了引导方式后请记住，后面安装引导加载程序时会用到。

BIOS设置是老规矩——关快速启动、U盘设为第一引导项。

引导进入U盘的 LiveCD 系统，这里各种主板的方法五花八门，请自行搜索自己主板的方法（一般搜索 你的主板品牌+“U盘启动” 就能找到）。

P.S.: 这里有一个坑，你的主板也许同时支持 传统 和 UEFI 两种启动方式，在选择启动设备时同一个U盘可能会有两个选项，此时若有 UEFI 选项请 优先选择 UEFI 。

如果看到类似下图这样的一个界面，那么恭喜你已经进入 Arch Linux 的 Live 环境了。

![Arch Linux LiveCD](https://i.loli.net/2021/01/23/YKv7VfwnJkdhPtz.png)

## 安装

#### 连接互联网

- 有线网络（大多数时候是这种）：
  - Live 环境默认启用了 DHCP 功能，一般会自动连接，不需要其他配置，直接跳过连接互联网这一步即可。（当然如果网线没插的话我帮不了你）
- 无线网络（WiFi）：
  1. 输入 `iwctl` 并回车，此时会进入提示为 `[iwd]#` 的交互式命令行。
  2. 输入 `device list` 并回车，会列出无线网卡的信息， `name` 一栏的那个名字就是网卡的名字（将下文中 `wlan` 替换成这个名字）。
  3. 输入 `station wlan scan` 并回车扫描周围的无线网络，然后输入 `station wlan get-networks` 列出扫描到的网络，在其中找到你需要连接的 WiFi 名字。
  4. 输入 `station wlan connect SSID` 并回车即可连接，其中 `SSID` 为 WiFi 名字。若这个 WiFi 设置了密码会提示输入，此时请输入密码并回车。

[有关 iwd 的更多信息](https://wiki.archlinux.org/title/Iwd){:target="_blank"}

可以 ping 百度 来测试一下互联网连接：

```bash
ping www.baidu.com
```

如果一直输出正常的数据包返回信息，则表示网络连接没有问题，使用 Ctrl + C 退出 ping 测试。

#### 调整时间

使用 timedatectl 确保系统时间准确：

```bash
timedatectl set-ntp true
```

#### 硬盘分区

##### 找到你的目标设备（即要安装 Arch Linux 的硬盘）

在 Linux 中，硬盘若被系统识别到，就会被分配为一个[块设备](https://zh.wikipedia.org/wiki/%E8%AE%BE%E5%A4%87%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F#%E5%9D%97%E8%AE%BE%E5%A4%87){:target="_blank"}，类似 `/dev/sda` 、 `/dev/nvme0n1` 或 `/dev/mmcblk0` 等。

可以使用 `fdisk -l` 查看所有能被识别的硬盘设备，在其中找到你需要的并记下它对应的块设备文件路径，并将下文中的 `disk` 替换为这个名字。

硬盘上的每一个分区也都会被分配为一个单独的块设备，文件名一般是所在设备的文件名后面带一个表示分区编号的数字（有的设备数字前会有一个字母“p”），如 `/dev/sda1` 、 `/dev/nvme0n1p1` 等，将下文中 `diskX` 替换为特定分区的块设备文件名。

P.S.: 所有的设备文件路径均以 `/dev` 开头，若不以 `/dev` 开头则不是设备文件。

##### 创建和修改分区

**这一步一定要仔细仔细再仔细，否则你的数据很可能就全没了！**

根据你的引导方式来编辑分区表：

- UEFI
  - 需要使用 GPT 分区表
  - 需要一个 EFI 分区用于存放 EFI 引导加载程序
- 传统
  - GPT 或 MBR 分区表皆可（其中 GPT 分区表需要进行一些特殊操作）
  - 不需要任何额外的分区

其中，无论何种引导方式，都需要一个最小 6 GB 左右（建议 20 GB 以上）的分区用于安装 Arch Linux 系统。

Linux 下的分区软件有很多（小白建议使用 **cfdisk** ），这里不详细讲述分区方法，具体请自行搜索。

P.S.: Arch Wiki 有很多 Linux 分区软件的使用方法，如 [fdisk](https://wiki.archlinux.org/title/Fdisk) 、 [parted](https://wiki.archlinux.org/title/Parted){:target="_blank"} 等，[这里](https://wiki.archlinux.org/title/Partitioning#Partitioning_tools){:target="_blank"}（[中文页面](https://wiki.archlinux.org/title/Partitioning_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29#%E5%88%86%E5%8C%BA%E5%B7%A5%E5%85%B7){:target="_blank"}）甚至有一个基本上所有能用的分区工具列表。

##### 格式化分区（即创建文件系统）

使用 mkfs 创建文件系统，其中 `fsname` 为要创建的文件系统名：

```bash
mkfs.fsname diskX
```

##### 挂载分区

使用 mount 命令来挂载分区，其中 `mountpoint` 是挂载点（即一个实际存在的目录）：

```bash
mount diskX mountpoint
```

挂载之后，挂载点对应目录中的文件就会变成分区中存放的文件，这时我们就可以访问分区的文件系统了。

Arch Linux 的安装常将根目录分区挂载到 /mnt 文件夹。

P.S.: 挂载多个分区时，必须按照挂载点**由父目录到子目录的顺序**。比如，若需要将两个分区挂载到 `/mnt` 和 `/mnt/home` 这两个目录，则需要先挂载 `/mnt` 对应的分区，后挂载 `/mnt/home` 对应的分区，否则会报出类似“目录正忙”的错误信息。如果两个挂载点属于同级目录则无此需求。

##### 建议的分区布局

如果对 Linux 的文件分布不是很了解，推荐使用以下的分区布局（假设你有一整个空闲的硬盘用来安装），表格由上到下的顺序即为分区在硬盘上建立的顺序。

**UEFI 引导 + GPT 分区表**

| 挂载点 | 大小 | 文件系统 | 用途 |
|----|----|----|----|
| /efi | 200 MB | fat32 | EFI 系统分区 |
| /mnt/boot | 2 GB | ext4 | 启动分区 |
| /mnt | 20 GB + | btrfs/ext4 | 根目录 |
| /mnt/home | 剩余空间 | xfs/ext4 | 用户文件分区 |

**传统引导 + MBR 分区表**

| 挂载点 | 大小 | 文件系统 | 用途 |
|----|----|----|----|
| /mnt | 20 GB + | ext4 | 根目录/启动分区 |
| /mnt/home | 剩余空间 | xfs/ext4 | 用户文件分区 |

P.S.: 如果内存比较小，或者单纯硬盘空闲空间太多了，可以另外添加一个空分区以便之后配置**交换分区**。当然这不是必要的，因为有无交换分区并不影响系统的功能，而且交换空间也可以放在一个文件而不是单独的分区上。

*未完待续…*
