---
title: "现有系统安装ArchLinux（ARM篇）"
date: 2021-08-06T18:29:32+08:00
draft: false
tags:
 - "ArchLinux"
 - "vps"
toc: true
---

## 前言

之前写过`现有系统安装ArchLinux`，针对的是x86架构，后来甲骨文推出了arm架构的vps,虽然也重新安装了`ArchLinuxARM`，但未做记录。昨天估计首尔区放水，成功开通。趁这次重装，记录安装过程。因大体与x86安装过程类似，仅简要记录关键步骤。

## 前期准备

与前文一致，不作赘述。

## 创建chroot环境

- 安装`debootstrap`，创建`bootstrap`。

```
apt update
apt install debootstrap
mkdir /tmp/stable-chroot
debootstrap stable /tmp/stable-chroot http://deb.debian.org/debian/
```

- chroot
  
```
cd /tmp/stable-chroot
cp /etc/resolv.conf etc
mount -t proc /proc proc
mount --make-rslave --rbind /sys sys
mount --make-rslave --rbind /dev dev
mount --make-rslave --rbind /run run
chroot ./ /bin/bash
```

## 使用chroot环境安装系统

挂载设备、删除旧系统部分，与前文一致，不作赘述。

- 安装基础系统

因`debootstrap`没有预装wget,需先安装，然后下载`ArchLinuxARM-aarch64-latest.tar.gz`,并解压。

```
apt update
apt install wget
wget http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz
tar -xpf ArchLinuxARM-aarch64-latest.tar.gz -C /mnt
```

- chroot，进入目标系统
  
```
cd /mnt
mount -t proc /proc proc
mount --make-rslave --rbind /sys sys
mount --make-rslave --rbind /dev dev
mount --make-rslave --rbind /run run
chroot /mnt /bin/bash
```

- 初始化 pacman keyring

```
pacman-key --init
pacman-key --populate archlinuxarm
```

## 完善系统

以下部分与前文一致，不作赘述。

- 时区，此处选择东八区
- 本地化
- SSH
- 设置root密码

以下部分与前文有所改动。

- 网络设置

无需创建systemd-networkd配置文件，其他不变。

- 设置boot

/boot/loader/entries/arch.conf内容如下，其他不变。

```
title   Arch Linux
linux   /Image
initrd  /initramfs-linux.img
options root="LABEL=cloudimg-rootfs"  rw
```

## 进入新系统

连续输入两次 exit 退出到原系统， 然后 reboot 重启，等待几分钟后，重新连接ssh,进入则表明重装成功。

![img](2021-08-06_19-08-01.png)