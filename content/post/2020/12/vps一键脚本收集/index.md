---
title: "Vps一键脚本收集"
date: 2020-12-05T11:31:05+08:00
draft: false
tags:
 - "vps"
toc: true
---

## 前言

一般只适用于 Debian、Ubuntu、Contos、Redhat等主流服务器发行版

## BBR

- 手动开启，无效脚本

```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
lsmod | grep bbr
```

- N合一脚本

```
wget -N --no-check-certificate https://raw.githubusercontent.com/ylx2016/Linux-NetSpeed/master/tcp.sh
chmod +x tcp.sh
./tcp.sh
```

## v2ray

```
wget -N --no-check-certificate -q -O v2ray.sh https://raw.githubusercontent.com/wulabing/V2Ray_ws-tls_bash_onekey/master/install.sh
chmod +x v2ray.sh
./v2ray.sh
```

## 探针

- ServerStatus-Hotaru

```
wget https://raw.githubusercontent.com/CokeMine/ServerStatus-Hotaru/master/status.sh
chmod +x status.sh
./status.sh s   # 服务端
./status.sh c   # 客户端
```

- 哪吒面板

```
curl -L https://raw.githubusercontent.com/naiba/nezha/master/script/install.sh -o nezha.sh
chmod +x nezha.sh
./nezha.sh
```

## bench

```
wget http://vpstest.cn/it && bash it
```