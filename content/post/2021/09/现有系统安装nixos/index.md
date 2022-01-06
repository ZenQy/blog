---
title: "现有系统安装nixos"
date: 2021-09-16T18:02:28+08:00
draft: false
tags:
 - "nixos"
 - "vps"
toc: true
---

## 前言

最近改用`nixos`，当然希望vps也能安装相同的系统。`virmach`家可通过`netboot,xyz`安装，比较方便。但手上有一台vps,商家只支持`debian`和`centos`，只能自己想办法安装。

## 内存安装最小化系统

此处使用的是[menhera.sh](https://github.com/Jamesits/menhera.sh)方案。

```bash
wget https://raw.githubusercontent.com/Jamesits/menhera.sh/master/menhera.sh
chmod +x menhera.sh
./menhera.sh
```

根据提示即可安装成功，然后重启进入新系统。

## 准备环境

先卸载磁盘，然后使用`cfdisk`重新分配磁盘，格式化后挂载。为了防止启动时出现问题，可以修改格式化后的根分区UUID,与原UUID一致。

```bash
umount /mnt/oldroot/
cfdisk
mkfs.ext4 /dev/vda1
tune2fs /dev/vda1 -U 6ac329fb-3f1f-4168-a3c6-7eeac76b1d4b  # 可选
mkswap /dev/vda2
mount /dev/vda1 /mnt
swapon /dev/vda2
```

安装必要的工具。

```bash
apt update
apt install -y wget curl xz-utils sudo git nano
```

创建相关组和用户

```bash
groupadd -g 30000 nixbld
useradd -m -u 30000 -g nixbld -G nixbld -s /bin/bash nixbld
```

`nixbld`用户添加`sudo`权限，并添加`nixUnstable`配置

```bash
echo "nixbld  ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
mkdir -p /etc/nix
echo "experimental-features = nix-command flakes" > /etc/nix/nix.conf
```

切换至`nixbld`用户

```bash
su nixbld
cd ~
```

## 安装`nix`工具和新系统

~~此处使用[nix-unstable-installer](https://github.com/numtide/nix-unstable-installer)方案。~~官方稳定版已经支持`nix flake`。

```bash
curl -L https://nixos.org/nix/install | sh
. $HOME/.nix-profile/etc/profile.d/nix.sh
```

安装`nixos-install-tools`

```bash
nix-env -f '<nixpkgs>' -iA nixos-install-tools
```

创建配置文件，配置文件默认安装在`/mnt/etc/nixos`（可选,注意：此处UUID错误，需要修改）

```bash
sudo `which nixos-generate-config` --root /mnt
```

因为我已经提前准备好配置，所以不需要上一步，或者仅需要上一步产生的`hardware-configuration.nix`。下载配置文件，添加缺失的`profile.nix`，修改后进行安装。

```bash
git clone https://github.com/ZenQy/nixos.git
cd nixos
cp /mnt/etc/nixos/hardware-configuration.nix machines/nbhost/hardware.nix
sudo PATH="$PATH" `which nixos-install` --root /mnt --flake .#nbhost
```

重启即可进入新系统，重启前可以考虑把配置文件转移到新系统中。

```bash
cd ../
sudo mv nixos /mnt/root/
```