---
title: "魔百盒CM201-2刷机"
date: 2020-04-11T19:37:29+08:00
draft: false
tags: ["机顶盒","魔百盒","刷机"]
toc: true
---


## 设备简介

| key  | value   |
| ---- | ------- |
| 名称 | 魔百盒  |
| 型号 | CM201-2 |
| 芯片 | MV300   |
| 代工 | 朝歌    |
| 闪存 | nand    |


## 进入Recovery

机顶盒完全断电，通电后立即不停按遥控器菜单键+音量减键，即可进入Recovery模式。

{{% admonition type="note" title="补充说明" details="true" %}}
在自带系统中，插入装有update.zip的U盘操作成功，刷机后尝试这种方式2次均失败。
{{% /admonition %}}

## 连接设备

有以下2种方式：

### ADB方式

机顶盒连接网络，同一局域网中，使用`adb`命令连接设备。

> adb connect <设备ip>

{{% admonition type="note" title="" details="false" %}}
该方法未在原版系统尝试
{{% /admonition %}}

### Serial方式

使用TTL转USB连接电脑和机顶盒，**TTL转USB上的TXD、RXD、GND分别对应机顶盒RX、TX、GND**（主板右下方4个孔中右侧3个），速率设置为115200。在Windows系统可使用`Putty`工具连接。

## 刷机过程

1. 准备刷机文件，例如[魔百盒CM201-2朝歌代工MV300芯片-蓝牙语音版](http://v.own-cloud.cn/s/6bgjg422),密码：0000
2. 刷入底包（不是所有固件都需要底包），将底包放入U盘中，路径为/upgrade/update.zip，机顶盒开机会自动识别并刷入底包。我实际操作并未识别，使用遥控器进入Recovery模式手动刷入的。
3. 使用ADB方式连接机顶盒
4. 刷入固件。将固件（emmc、recovery.img、update.zip）放入U盘根目录中，使用以下命令刷入固件：
   
进入shell

> adb shell 

识别闪存类型，输出有`ubifs`字样说明是nand闪存，输出有`ext4`字样的是emmc闪存

> mount | grep system

nand闪存机顶盒刷机

> cd mnt/sda/sda1  
> dd if=recovery.img of=/dev/block/mtdblock2  
> dd if=recovery.img of=/dev/block/mtdblock3  
> dd if=emmc of=/dev/block/mtdblock4  
> reboot  

重启后进入Recovery模式，选择**apply update from external storage**选项，选择update.zip进行更新，运行完毕机顶盒会自动重启。

## 结束语

魔百盒作为移动自家的机顶盒，限制颇多，使用过程中总有被束缚的感觉，刷机后去除限制，能访问Samba，直接观看局域网内的影视资源还是很不错的。