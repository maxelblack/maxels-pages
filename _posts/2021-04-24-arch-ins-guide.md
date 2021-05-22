---
layout: post
title: "Arch Linux 安装指引"
author: maxelbk
categories: [ "Arch Linux", "技术", "教程" ]
image: r/pic/SCS_ARCHISO_202104.png
---

At first,ARCH IS THE BEST!

按照 [Arch Wiki](https://wiki.archlinux.org/title/Arch_Linux){:target="_blank"} 的说法，Arch Linux 是一个简洁、现代、实用且以用户为中心的 GNU/Linux 发行版，采用滚动更新模式，尽全力提供最新的稳定版软件。<!--MORE-->

鉴于 Arch Wiki 的安装指引有些许复杂，于是就写了这一篇指引，帮助想要一个完全自定义化的系统的 Linux 入门级爱好者更快配置一个可以满足日常使用的 Arch Linux 。（大佬就不要看这篇指引了，直接看 [Arch Wiki](https://wiki.archlinux.org/title/Installation_guide){:target="_blank"} 更加合适）

顺带一提，这篇指引中几乎所有 Arch Wiki 外链都是英文原版，若有需要请自行切换语言（PC端左侧/移动端页末）

## 准备工作

你需要：

- **一个好用的脑子**（没错这是最重要的）
- 一个标称容量大于或等于 1 GB 的U盘（用来装 Live CD）
- 足够的时间（安装基本需要全程守着，用时取决于设备配置和你的运气，一般 1h 左右）
- 一台能联网的 x86_64 架构设备（现在的个人电脑基本都是这个架构）和一个稳定的网络（Arch是联网安装的）

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

可以 ping 百度来测试一下互联网连接：

```shell
ping www.baidu.com
```

若在中国大陆以外的区域也可以 ping Google：

```shell
ping www.google.com
```

如果一直输出正常的数据包返回信息，则表示网络连接没有问题，使用 Ctrl + C 退出 ping 测试。

#### 调整时间

使用 `timedatectl` 确保系统时间准确：

```shell
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

```shell
mkfs.fsname diskX
```

##### 挂载分区

使用 `mount` 命令来挂载分区，其中 `mountpoint` 是挂载点（即一个实际存在的目录）：

```shell
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

#### 配置镜像源

在安装之前记得看一下 `/etc/pacman.d/mirrorlist` 文件，将你想用的镜像服务器放在最上方。这里仍旧是推荐使用 [OpenTUNA](https://opentuna.cn/){:target="_blank"} 镜像源，在文件最上方添加：

```
Server = https://opentuna.cn/archlinux/$repo/os/$arch
```

然后保存即可。

#### 安装基本系统

挂载根目录并配置好镜像源之后，就可以正式开始安装了。

（如果你的根目录不是挂载在 `/mnt`，则这一步中所有的 `/mnt` 都需要换成实际的根目录挂载位置）

首先使用 `pacstrap` 脚本安装基本系统组件：

```shell
pacstrap /mnt base linux linux-firmware
```

P.S.: `linux` 软件包即 Arch Linux 官方的系统内核，这里可以替换为社区的其他内核软件包（比如 `linux-zen` ）以获得不同的特性。若是虚拟机可以去掉 `linux-firmware` 软件包，这个软件包主要为硬件设备提供基本的驱动程序。

#### 生成 Fstab

使用 `genfstab` 脚本生成 `fstab` 文件，这个文件记录了分区的默认挂载配置：

```shell
genfstab -U /mnt >> /mnt/etc/fstab
```

强烈建议生成之后查看一下文件 `/mnt/etc/fstab` ，确认配置无误后再继续操作。若配置有误，Arch Linux 可能无法启动。

#### 安装引导加载程序

Linux 下的引导加载程序有很多，这里只介绍 GRUB 的安装方法（~~当然是因为我只会用GRUB~~）。

首先，安装 `grub` 软件包：

```shell
pacman -S grub
```

然后向硬盘安装 GRUB ，这里根据引导方式有不同的安装方法。

##### UEFI 引导

```shell
grub-install --target=x86_64-efi --efi-dircetory=/efi --boot-directory=/mnt/boot --bootloader-id="Arch Linux"
```

其中 `/efi` 需要替换成你的 EFI 分区挂载位置（若按照上文建议的分区布局挂载则无需替换）。

##### 传统引导

```shell
grub-install --target=i386-pc --boot-directory=/mnt/boot disk
```

其中 `disk` 即上文分区时提到的硬盘，也就是根目录分区所在的硬盘（例如，根目录分区为 `/dev/sda1` ，则这里的 `disk` 就是 `/dev/sda` ）。

## 后续配置

首先进入 chroot 环境（可以近似看作直接在 LiveCD 进入新系统的环境）：

```shell
arch-chroot /mnt
```

其中 `/mnt` 也需要换成你的根目录挂载位置（如果你不是挂载在 `/mnt` 的话）。

#### 时区

将 `/usr/share/zoneinfo` 目录中的时区配置软链接到 `/etc/localtime` ，例如：

```shell
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime #将时区设置为上海
```

运行 hwclock 同步硬件时间并生成 `/etc/adjtime` 文件：

```shell
hwclock --systohc
```

P.S.: 大部分操作系统都将硬件时间作为 [UTC 时间](https://zh.wikipedia.org/wiki/%E5%8D%8F%E8%B0%83%E4%B8%96%E7%95%8C%E6%97%B6){:target="_blank"} ，Linux 也不例外。但 Windows 比较特殊，它默认将硬件时间视为本地时间，因此如果你的电脑上有 Windows 系统，请参考[这里](https://wiki.archlinux.org/title/System_time#UTC_in_Microsoft_Windows){:target="_blank"}（[中文页面](https://wiki.archlinux.org/title/System_time_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29#Windows_%E7%B3%BB%E7%BB%9F%E4%BD%BF%E7%94%A8_UTC){:target="_blank"}）将 Windows 的硬件时间设置为 UTC 时间并关闭 Windows 的自动时间同步。

#### 本地化（语言和区域）

编辑 `/etc/locale.gen` ，移除想要启用的区域前的 `#` 符号（简体中文用户建议启用 `zh_CN.UTF-8` 和 `en_US.UTF-8` ，同时不建议在 Linux 启用任何不是 `UTF-8` 编码的语言）。

保存后运行 `locale-gen` 生成 locale 信息。

然后在 `/etc/locale.conf` 文件中设定 `LANG` 变量：

```shell
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```

#### 网络

使用 `/etc/hostname` 设定主机名：

```shell
echo "myhostname" >> /etc/hostname
```

其中 `myhostname` 可以替换成任意你喜欢的字符串，当然只能包括英文字母(A~Z,a~z)、数字(0~9)和连词符(-)，这个名字就是你的电脑在网络上显示的名字，有时也会显示在 shell 的命令提示等地方。

然后编辑 `/etc/hosts` 文件，在其中添加以下内容：

```conf
127.0.0.1   localhost
::1         localhost
127.0.1.1   myhostname.localdomain myhostname
```

同样， `myhostname` 需要替换为上述主机名，而 `localdomain` 则是本地域名。

同时需要安装并启用一个网络服务，用于管理网络连接，这里推荐使用 NetworkManager ：

```shell
pacman -S networkmanager
systemctl enable NetworkManager
```

P.S.: `localdomain` 若无特殊需求可以直接写为 `localdomain` ，通常不需要替换为其他内容。

#### 常用软件

Arch Linux 使用 Pacman 管理软件包，可以通过 `pacman -S package` 来安装软件，其中 `package` 是软件包名或包组名（可以通过直接安装包组来同时安装多个软件包）。

常用的软件包包括 `vim`, `nano`, `base-devel`, `pulseaudio` 等。

#### 用户

首先设置 root 用户的密码：

```shell
passwd
```

输入密码时是没有回显字符的，所以大可不必怀疑你的键盘坏了，放心输入就好。

当然也可以不设置密码，但这样会使 root 用户不可用。不建议禁用 Arch Linux 的 root 用户，这样在系统发生故障无法正常启动时就麻烦大了。

在 Linux 中，未启用 SELinux 时，root 用户具有无视一切限制的特殊权限，因此十分危险（比如不小心手残输入了 `rm -rf /*` ，系统直接就没了），日常使用不建议直接以 root 用户身份登录。这样就需要新建一个普通用户用于日常使用：

```shell
useradd -m -G wheel username
```

其中 `username` 是用户名，只能包含小写字母(a~z)、下划线(_)； `-G wheel` 表示将这个用户加入 `wheel` 用户组，以便通过 `sudo command` 的形式直接以 root 用户的身份执行指令。

设置这个用户的密码：

```shell
passwd username
```

使用 `visudo` 编辑 `sudoers` 文件，启用 `wheel` 用户组的 sudo 权限：

```shell
EDITOR=vim visudo # 其中 vim 可以替换为其他已安装的编辑器
```

#### GRUB 配置文件

如果安装了 GRUB ，则需要在 `grub` 目录中生成配置文件：

```shell
grub-mkconfig -o /boot/grub/grub.cfg
```

## 完成

至此，一个完整的 Arch Linux 就安装完成了，使用 `exit` 退出 chroot 环境并重启，你就可以享受一个崭新的完全可定制的 Arch Linux 了！

使用 `restart` 重启，使用 `shutdown now` 关机。
