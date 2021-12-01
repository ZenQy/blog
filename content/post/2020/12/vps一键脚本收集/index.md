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

- 手动开启，无需脚本

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

## 测试脚本

- Bench.sh

```
wget -qO- bench.sh | bash
```

- wget -qO- bench.sh | bash

```
wget -qO- --no-check-certificate https://raw.githubusercontent.com/oooldking/script/master/superbench.sh | bash
```

- Yet-Another-Bench-Script

```
curl -sL yabs.sh | bash
```

- 合集

```
wget -N --no-check-certificate https://raw.githubusercontent.com/veip007/hj/master/hj.sh
chmod +x hj.sh
./hj.sh
```
## DD脚本

```
bash <(wget --no-check-certificate -qO- 'https://raw.githubusercontent.com/MoeClub/Note/master/InstallNET.sh') -d 11 -v 64 -p "自定义root密码" -port "自定义ssh端口"
```
参数|例子|说明
:--:|:--:|:--:
-d|9,10,11|Debian
-u|18.04,20.04|Ubuntu
-c|6.9|CentOS
-v|32,64|arch
-p|QWERqwer|password,默认MoeClub.org
-port|1234|ssh port,默认22
--mirror|http://mirrors.ustc.edu.cn/debian|mirror
-dd|http://d.nat.ee/win/lite/wes7-x86-cn-lite/wes7-x86-cn-lite.vhd.gz|win uri