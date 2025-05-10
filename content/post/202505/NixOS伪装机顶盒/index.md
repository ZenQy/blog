---
title: "NixOS伪装机顶盒"
date: 2025-05-09T21:15:46+08:00
draft: false
tags:
  - nixos
  - 机顶盒
---

## 前言

伪装的好处是家中所有设备均可播放直播，突破机顶盒的物理限制。网上也有很多教程，但主要是用于`OpenWRT`或`爱快`,我这里是用的`Nixos`,实际上任何适用于任何使用`systemd-networkd`配置网络的发行版。

## 抓包

需要获得以下3个数据，网上教程很多，我这里就不放过程了（主要是我这是后补的记录，很多过程资料都没留存😅）

|    名称    |  类型  |       示例        |
| :--------: | :----: | :---------------: |
| MACAddress | 字符串 | 1a:2b:3c:d4:e5:f6 |
|  Hostname  | 字符串 |   android-xxxx    |
| SendOption | 二进制 |

## 配置

其在`systemd-networkd`的配置大概是下面这样的，此处以`nix`形式展示，需要注意的是，虽然`systemd-networkd`已经支持`SendVendorOption`字段，但该字段似乎只能是字符串，对于机顶盒的二进制值无能为力，只好使用`SendOption`字段替代，且二进制值全部转码。我也是一点点尝试出来的，后面又没在使用，所以忘记了`SendVendorOption`字段这样使用是否可以。

```nix
{
  systemd.network.networks = {
    eth1 = {
      matchConfig = {
        Name = "eth1";
      };
      networkConfig = {
        DHCP = "ipv4";
        IPv6AcceptRA = "no";
        LinkLocalAddressing = "no";
      };
      linkConfig = {
        MACAddress = "1a:2b:3c:d4:e5:f6";
      };
      dhcpV4Config = {
        RouteMetric = 20;
        Hostname = "android-xxxx";
        SendOption = "60:string:\\x00\\x00\\x1f\\x1f\\x1f\\x1f\\x1f\\x1f\\xe0\\xe0";
      };
    };
  };
}
```

## 使用

在`OpenWRT`中，可以使用`udproxy`方便的转发，但我在`NixOS`没有找到类似的软件，最后通过`sing-box`透明代理实现转发。大概配置如下所示。当使用`udp`协议访问**238.1.78.0/23**(安徽地区iptv源地址)时，使用eth1接口出栈。

```nix
{
  route = {
    rule_set = [
      {
        tag = "rule_set_iptv";
        type = "inline";
        rules = [
          {
            network = [
              "udp"
            ];
            ip_cidr = [
              "238.1.78.0/23"
            ];
          }
        ];
      }
    ];
    rules = [
      {
        rule_set = "rule_set_iptv";
        outbound = "iptv";
      }
    ];
  };
  outbounds = [
    {
      tag = "iptv";
      type = "direct";
      bind_interface = "eth1";
    }
  ];
}
```
