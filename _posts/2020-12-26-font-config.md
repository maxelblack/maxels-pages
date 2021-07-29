---
layout: post
author: maxelbk
title: "FontConfig优先级问题解决方案"
categories: [ "技术", "Linux" ]
---

大多数Linux用户可能经常遇到 FontConfig 的优先级问题, 简单来说就是某些应用的字体显示不正常, 让人看起来很奇怪.<!--MORE--> 例如用 Qt 写的 Telegram Desktop 客户端, 经常出现一些字符宽度偏窄(如"关","复"字), 甚至更有一些字符直接显示出不像是汉字的轮廓(如"直"字).

这两个问题都是由于系统在查找字体时, 因日文字体默认优先级高于中文而返回了对应的日文字形. 在日语中存在的部分汉字字形与现代汉语不同, 我们看到的实际是日文字形.

解决这个问题最简单的方法就是修改 FontConfig 的字体优先级, 但那个配置文件的文档相当难懂(至少我是没能看懂), 于是 Google 了半天, 最后找到一个有效的解决方案.

后来 [JackieCat](https://www.jackiecat.xyz){:target="_blank"} 在 Arch Wiki 翻到了一个更完善的解决方案并分享出了地址, 可以参考一下:
- [本地化/简体中文#中文字体配置 - ArchWiki](https://wiki.archlinux.org/index.php/Localization_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29/Simplified_Chinese_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29#%E4%B8%AD%E6%96%87%E5%AD%97%E4%BD%93%E9%85%8D%E7%BD%AE){:target="blank"}

### 找到个人配置文件位置

`fonts.conf` 是 FontConfig 对于单独用户字体的配置文件名称, 一般在每个用户的主目录中.

不同的发行版这个文件的位置甚至可能不一样, 我的 ArchLinux 是在 `~/.config/fontconfig` 目录. (Arch Wiki 中指出正常情况下是在 `$XDG_CONFIG_HOME/fontconfig` 目录, 各大比较常规的发行版都可以试一下这个目录, CentOS 之类超级旧版就不要尝试了, 乖乖 Google 反倒效率更高)

P.S.: 最好不要修改 `/etc` 中的全局字体配置文件, 在刷新字体缓存时这个文件会被更新, 所有修改都会失效.

### 修改配置文件

用任意编辑器打开 `fonts.conf`, 在 `<fontconfig>` 标记中添加 `<alias>` 标记, 再在其中添加 `<family>` 和 `<prefer>` 标记

`<family>` 标记中指定要设置优先级的字体族, 如:

```xml
<family>sans-serif</family>
```

`<prefer>` 标记使用多个 `<family>` 子标记来指定字体优先级顺序, 如:

```xml
<prefer>
    <family>Noto Sans CJK SC</family>
    <family>Noto Sans CJK JP</family>
</prefer>
```

修改保存之后更新字体缓存使配置生效：

```shell
sudo fc-cache -fv
```

示例配置:

```xml
<?xml version='1.0'?>
<!DOCTYPE fontconfig SYSTEM 'urn:fontconfig:fonts.dtd'>
<fontconfig>
    <!--字体优先级配置-->
    <alias>
        <family>sans-serif</family>
        <prefer>
            <family>Noto Sans Display</family>
            <family>Noto Sans CJK SC</family>
            <family>Noto Sans CJK HK</family>
            <family>Noto Sans CJK JP</family>
            <family>Noto Sans CJK KR</family>
            <family>Noto Sans CJK TC</family>
        </prefer>
    </alias>
   <!--配置结束-->
   <!--==文件中原有的其他内容请勿删除==-->
</fontconfig>
```

P.S.: `Noto Sans Display` 是 `Noto Sans` 家族的一款适合显示器的字体, 比默认看起来更舒适一些.

### 检查是否生效

终端执行以下代码(其中 `sans-serif` 可以换成任意上方设置了优先级的字体族名称):
```sh
fc-match -s 'sans-serif'
```

若程序输出的前几行出现中文字体的名称(如 `NotoSansCJK-Regular.ttc: "Noto Sans CJK SC" "Regular"`), 则说明配置生效, 至此这个问题就解决了.
