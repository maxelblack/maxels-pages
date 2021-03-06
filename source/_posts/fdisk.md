---
layout: post
title: "FDISK 常用功能详解"
categories: [ "Linux" ]
date: 2021-07-29
---

<div style="color:red">
    <p>WARNING: 关于 FDISK 的详细说明请直接使用 <code style="color:black" class="highlighter-rouge language-plaintext">man fdisk</code> 查看，本文不保证说明全面、不保证所有特性均为最新。</p>
    <p>WARNING: 数据无价，请谨慎操作，保存分区操作前请三思。</p>
</div>

下面进入正题。

fdisk 是一个命令行分区工具，常常在 Arch Linux 安装中 [硬盘分区](/linux/arch-inst#硬盘分区) 部分使用频繁。

在 Arch Linux 中，这个工具包含在 `util-linux` 包中。

对于这篇文章的说明:
- `disk` 表示某个硬盘对应的块设备文件(e.g.: `/dev/sda`, `/dev/nvme0n1` )
- `diskX` 表示某个分区对应的块设备文件(e.g.: `/dev/sda2`, `/dev/nvme0n1p2` )
- 若无特别说明，指令均在 root 用户下执行(可使用 `sudo` )。

### 查看分区表

使用 `fdisk -l disk` 来查看指定硬盘的分区表。

### 进入编辑模式

使用 `fdisk disk` 开始编辑指定硬盘。

这时会进入一个命令提示界面(CLI)，输入指令(区分大小写)并回车即可执行相应动作。

输入 m 并回车，fdisk 会输出一个指令列表，可以直接在这里查看各指令用法。

编辑结束后，使用 `w` 保存分区操作。

<p><font color="red">重要的事情说两遍: 数据无价，保存分区操作之前请三思！</font></p>

#### 基本操作

- `p` 显示分区表(相当于直接执行 `fdisk -l disk` )
- `d` 删除指定分区
  - 分区号(Partition number): 一个正整数，可以从块设备名看出，如 `/dev/sda4` 的分区编号是 4 。
- `n` 添加新分区
  - 分区号(Partition number): 任意给出范围内的正整数，默认即可(直接回车)。
  - 起始扇区(First sector): 一个不属于任何其他分区的**扇区**号，一般默认即可。
  - 结束扇区(Last sector): 一个不属于任何其他分区的**扇区**号，必须位于起始扇区的扇区号之后，新分区将建立在在这两个扇区之间(包括这两个扇区)。
    - 可以使用 `+number` 的形式通过指定分区大小让 fdisk 自动计算扇区号。
    - 可以使用 `-number` 的形式通过指定剩余空间大小让 fdisk 自动计算扇区号。
    - 有关 `+number` 和 `-number` 的写法将在[下文](#结束扇区特殊写法)介绍
- `F` 列出剩余空间
- `i` 显示指定分区信息
  - 分区号(Partition number): 一个正整数，同上文指令 `d` 中的分区号。
- `v` 检查分区表错误

#### 分区类型操作

- `l` 列出所有已知的分区类型
- `t` 更改指定分区的类型
  - 分区号(Partition number): 一个正整数，同上文指令 `d` 中的分区号。
  - 分区类型或别称: 其中分区类型是一个 UUID 编号，别称则是类似 `ntfs`, `ext4`, `xfs` 这样的一个名称。可以使用 `l` 查看支持的分区类型。

#### 分区表操作

<p><font color="red">下述操作均会直接覆盖当前分区表并创建一个空白的新分区表，如有必要请将当前分区表做好备份。</font></p>

- `g` 创建 [GPT 分区表](https://zh.wikipedia.org/wiki/GUID%E7%A3%81%E7%A2%9F%E5%88%86%E5%89%B2%E8%A1%A8)
- `o` 创建 [MBR 分区表](https://zh.wikipedia.org/wiki/%E4%B8%BB%E5%BC%95%E5%AF%BC%E8%AE%B0%E5%BD%95) (又称 DOS 分区表)
- `G` 创建 [IRIX/SGI 分区表](https://datacadamia.com/os/linux/disk/partition_table#irixsgi_type)
- `s` 创建 [BSD/SUN 分区表](https://datacadamia.com/os/linux/disk/partition_table#bsdsun_type)

其中后两种分区表现在已经很罕见了，以至于很多人从未听说过，使用 IRIX/SGI 分区表 的 [IRIX 操作系统](https://zh.wikipedia.org/wiki/IRIX) 已经停止开发，而 BSD/SUN 分区表 多用于企业服务且早已被淘汰，现代企业已基本普及 GPT 分区表。

#### 结束扇区特殊写法

这里的特殊写法即为 `+number` 或 `-number` 的写法。

其中 `number` 有两种写法:

1. 标准字节型大小
  - 基本写法为 `numX` ，其中 `num` 为一个数字，而 `X` 则是单位
  - 可以使用 `K`,`M`,`G`,`T`,`P` 来表示这个数据的单位，这五个单位依次是 **Kibibyte(KiB)**,**Mebibyte(MiB)**,**Gibibyte(GiB)**,**Tebibyte(TiB)**,**Pebibyte(PiB)** ，不写单位则单位为**Byte(字节,B)**。
2. 扇区(都会使用扇区了，相信不需要过多解释)

而 `+` 和 `-` 表示的意思如下图:

![Number detail](/r/pic/ITC_FDISK_NUMBER_DETAIL.webp)

全文终，如有任何修改建议或对文章的疑问，欢迎评论。
