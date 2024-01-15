---
title: "Ubuntu 绑定设备串口的两种方式"
description: "在 Ubuntu中，有时使用多个USB设备时，出现USB端口号混乱，可以通过编写 rules 文件解决这个问题"
keywords: "ubuntu;串口;rules;tty"

date: 2024-01-15T23:13:59+08:00
lastmod: 2024-01-15T23:13:59+08:00

categories:
  - "Linux"
tags:
  - "Ubuntu"
  -

# 原文作者
# Post's origin author name
#author:
# 原文链接
# Post's origin link URL
#link:
# 图片链接，用在open graph和twitter卡片上
# Image source link that will use in open graph and twitter card
#imgs:
# 在首页展开内容
# Expand content on the home page
#expand: true
# 外部链接地址，访问时直接跳转
# It's means that will redirecting to external links
#extlink:
# 在当前页面关闭评论功能
# Disabled comment plugins in this post
#comment:
#  enable: false
# 关闭文章目录功能
# Disable table of content
#toc: false
# 绝对访问路径
# Absolute link for visit
url: "ubuntu 绑定设备串口的两种方式.html"
# 开启文章置顶，数字越小越靠前
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
# 开启数学公式渲染，可选值： mathjax, katex
# Support Math Formulas render, options: mathjax, katex
#math: mathjax
# 开启各种图渲染，如流程图、时序图、类图等
# Enable chart render, such as: flow, sequence, classes etc
mermaid: true
---

<!--more-->

##  前言

>在Ubuntu中，有时使用多个USB设备时，出现USB端口号混乱；  
>比如：A设备本来对应 /dev/ttyUSB0，B设备对应 /dev/ttyUSB1；  
>发现重启系统后，A设备本来对应 /dev/ttyUSB1，B设备对应 /dev/ttyUSB0，两个设备的分配的端口号不固定的情况

>Linux是按照插入顺序对设备进行编号的；这种不稳定因素

参考了网上帖子和资料后，测试了两种绑定设备的方式：  
二选一即可，根据绑定设备的情况选择

## 一、通过设备 ID 信息绑定

终端输入：
```shell
lsusb
```

输出信息：
```shell
...skip...
Bus 003 Device 005: ID 1a86:55d4 QinHeng Electronics CP2102 USB to UART Bridge Controller
...skip...
```


>绑定串口，需要在 `/etc/udev/rules.d/` 路径下，创建一个 `*.rules`文件

首先，在 `/etc/udev/rules.d/`目录下，创建或打开文件

> 数字-文件名.rules ，数字会影响加载文件的优先级，数字越大，越优先加载
> 例如：`99-myusb.rules`

root 权限打开 `99-myusb.rules` 文件
```shell
sudo vim /etc/udev/rules.d/99-myusb.rules
or
sudo vi /etc/udev/rules.d/99-myusb.rules
```

将输出信息中的 `ID 1a86:55d4` 按如下格式写入 `/etc/udev/rules.d/99-myusb.rules` 文件
在最后一行添加

```bash
SUBSYSTEM=="tty" ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="55d4", GROUP="dialout", MODE="0777", SYMLINK+="lidar"
```

敲击键盘的 `ESC`后输入`:wq` 保存退出，重新激活绑定规则

```shell
sudo udevadm control --reload && sudo udevadm trigger
或
sudo udevadm trigger
```

## 二、通过设备的内核编号绑定

终端输入：
```shell
sudo udevadm info /dev/ttyCH343USB0
```

输出信息：
```shell
P: /devices/pci0000:00/0000:00:14.0/usb3/3-3/3-3.1/3-3.1:1.0/tty/ttyCH343USB0
...skip...
```

root 权限打开 `99-myusb.rules` 文件
```shell
sudo vim /etc/udev/rules.d/99-myusb.rules
or
sudo vi /etc/udev/rules.d/99-myusb.rules
```

将输出信息中的 `P: .../3-3.1:1.0/...` 按如下格式写入 `/etc/udev/rules.d/99-myusb.rules` 文件
在最后一行添加

```bash
KERNELS=="3-3.1:1.0", MODE:="0777", GROUP:="dialout", SYMLINK+="lidar"
```

敲击键盘的 `ESC`后输入`:wq` 保存退出，重新激活绑定规则

```bash
sudo udevadm control --reload && sudo udevadm trigger
或
sudo udevadm trigger
```

## 三、查看是否绑定成功

**上面两种绑定的方式，根据自己设备的情况，选择一种即可**

```shell
ll /dev
>>
...skip...
lrwxrwxrwx   1 root  root            12 1月  15 17:10 lidar -> ttyCH343USB0
crw-rw-rw-   1 root  root      1,     5 1月  15 17:10 zero
crw-------   1 root  root     10,   249 1月  15 17:10 zfs
...skip...
```
## 参考资料

1. https://blog.csdn.net/qq_41204464/article/details/115694264
