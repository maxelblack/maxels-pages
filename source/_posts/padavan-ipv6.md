---
layout: post
title: "原版 Padavan IPv6 配置"
categories: [ "网络" ]
date: 2021-12-09 17:52:15
---

## 前言

这篇文章适用于较新版本（2019+）的 Padavan 固件。

写这个的缘由是博主在小米的 R3P 上搞了很久的 OpenWrt ，发现在 Google 上能找到的现成的固件基本都不好用（不是内核过于古老就是预设配置太阴间用不惯），官方固件又有问题（不支持镁光闪存），所以最终还是刷了 Padavan ，并在上面成功配置了 IPv6 。

## 正文

博主刷完 Padavan 之后遇到与 OpenWrt 同样的问题。问题特征很明显，作为从路由以动态 IP 方式从主路由获取到了 IPv6 地址，但内网（LAN）上的设备分配不到，同时也没有现成的 IPv6 NAT 支持，也就是说这仍然是个 Bug 。

于是上 Google 搜索了半天，汇总了一下，貌似是因为 DHCPv6 对多层下发的兼容性差，导致内网设备分配不到独立地址。

所以这个配置方式的思路即是把 IPv6 的 DHCP 问题直接扔给主路由，然后从路由本身要做的就只是允许 DHCPv6 服务通过防火墙（这种服务一般不会阻拦）和添加一个对应前缀的路由表了。由于需要上级路由器来提供 DHCPv6 服务，这种方案仅适用于 DHCP 或静态 IP 上网的从路由，不适用拨号。

### 配置 WAN 端的 IPv6 协议设置

推荐设置如下图，其中任何一项设置也不建议更改。

![IPv6 协议](https://upload.cc/i1/2021/12/09/rFaLUz.png)

### 配置 IPv6 防火墙

在启动脚本的最后加入如下指令：

```sh
# IPv6 配置
WAN=eth3 #换成有IPv6地址的WAN接口名（使用 ifconfig 查看）
modprobe ip6table_mangle #确保启用了 IPv6 路由表的模块
ebtables -t broute -A BROUTING -p ! ipv6 -j DROP -i $WAN #添加路由表
brctl addif br0 $WAN
sysctl -w net.ipv6.conf.br0.accept_ra=2
```

其中 `eth3` 可能是其他接口，所以在复制粘贴之前务必 `ifconfig` 确认一下。

重启生效。