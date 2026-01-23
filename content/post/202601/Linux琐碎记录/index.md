---
date: "2026-01-23T20:42:53+08:00"
draft: true
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
