---
date: "2026-01-23T20:42:53+08:00"
draft: false
title: "Linux琐碎记录"
tags:
  - linux
---

## 临时添加ip

1. ipv4

```bash
ip addr add 192.168.1.11/24 dev eth0
ip route add 192.168.1.1 dev eth0
ip route add default via 192.168.1.1 dev eth0
```

2. ipv4

```bash
ip -6 addr add fe80::7270:fcff:fe03:2c08/64 dev eth0
ip -6 route add fe80::10a9:f6ff::1 dev eth0
ip -6 route add default via fe80::10a9:f6ff::1 dev eth0
```

## Nixos 自编译 kenel

在 linuxManualConfig 中提到的 Image 对应的 install 和 vmlinuz.efi 对应的 zinstall ，它们其实是同一个命令（好坑爹）。而这个 Makefile 最终会去寻找的内核文件名是通过 DEFAULT_KBUILD_IMAGE 去指定的，它的取值跟内核配置 .config 里的 CONFIG_COMPRESSED_INSTALL 有关，只有当这个选项没有被打开时它才会去找 Image 文件。因此,配置文件中需要关闭该选项。

```
# CONFIG_COMPRESSED_INSTALL is not set
```
