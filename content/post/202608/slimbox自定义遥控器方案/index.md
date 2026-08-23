---
date: "2026-08-23T22:35:17+08:00"
draft: false
title: "Slimbox自定义遥控器方案"
tags:
  - 遥控器
  - slimbox
---

## 整体思路

使用[MIK_v4.0.zip](./MIK_v4.0.zip)工具解包，修改`dtbs`文件，重新打包即可。

## 重新挂载分区

`/vendor`分区默认是只读分区，需重新挂载为读写分区，方可进行一些修改。
读写挂载需执行以下命令：

```bash
adb shell
su
mount -o remount,rw /vendor
```

## 遥控器修改为debug模式

将`/vendor/etc/remote.cfg`文件中的`debug_enable = 0`改为`debug_enable = 1`即可。

```bash
vi /vendor/etc/remote.cfg
```

## 监听遥控器动作

使用以下命令即可进行监听，然后逐一按下遥控器按键，并记录按键代码。`framecode`是遥控器的红外代码，`scancode`是遥控器的按键代码。

```bash
dmesg -w | grep -E "scancode=|framecode="
```

## 修改遥控器映射

`/vendor/usr/keylayout/Vendor_0001_Product_0001.kl`文件定义功能键值，文件内容大概如下所示，第二列键值，第三列是实现的功能。

```text
key 1     BACK
key 2     1
key 3     2
key 15    BACK
key 158   BACK
key 58    SOFT_RIGHT
key 107   ENDCALL
key 62    GUIDE
key 139    MENU
key 125    MENU
```

`/vendor/etc/remote.tab{1,2,3}`文件记录遥控器按键代码与功能键值映射，文件内容大概如下所示。
`custom_code`应填写遥控器的红外代码，即`framecode`，这样才能成功匹配。`release_delay`是按键延迟，保持默认即可。`fn_key_scancode`、`cursor_left_scancode`等是一些功能键的定义。
`0x01 2 #No.1`代表的意思是`scancode`0x01映射为键值2,根据Vendor_0001_Product_0001.kl可知，键值2执行的功能是数字1。
假如遥控器数字2的红外代码是0x02,我们改为`0x02 2`，那么当按下数字2按键后，实际输出的是数字1。
修改映射，就可以自定义遥控器的响应。

```text
custom_name = amlogic-remote-1
custom_code = 0x4040
release_delay = 80

fn_key_scancode = 0x47
cursor_left_scancode = 0x10
cursor_right_scancode = 0x11
cursor_up_scancode = 0x0b
cursor_down_scancode = 0x0e
cursor_ok_scancode = 0x0d

key_begin
    0x43 113 #MUTE
    0x47 71
    0x01 2 #No.1
    0x45 125 #MENU
    0x42 1   #BACK/EXIT
key_end
```

## 自定义方案生效

之下以下命令就可以重新加载配置文件，使修改后的配置文件生效。但不幸的是，这只是临时方案，重启后便会失效。因为固件将遥控器方案写死在`dtbs`文件中了，只有修改`dtbs`文件才能真正实现自定义方案。

```bash
remotecfg -c /vendor/etc/remote.cfg -d
```

## 解包固件

使用工具解包，`dtbs`文件在_aml_dtb文件中，对其进行修改。
需修改部分大概如下文所示：

```text
    	map_0 {
    		mapname = "amlogic-remote-1";
    		customcode = <0xfb04>;
    		release_delay = <0x50>;
    		size = <0x32>;
    		keymap = <0x47000b 0x130002 0x100003 0x110004 0xf0005 0xc0006 0xd0007 0xb0008 0x80009 0x9000a 0x5c0061 0x51003d 0x50003e 0x40003f 0x4d0040 0x430041 0x170042 0x43 0x10044 0x160057 0x49000e 0x60082 0x140083 0x440067 0x1d006c 0x1c0069 0x48006a 0x53007d 0x450068 0x19006d 0x520077 0x5007a 0x59007b 0x1b0078 0x40079 0x1a0074 0xa000f 0xe0071 0x1f0066 0x1e0084 0x70085 0x120086 0x540087 0x20088 0x4f001e 0x420030 0x5d002e 0x4c0020 0x580089 0x55008c>;
    		phandle = <0x2d>;
    	};
```

需要修改`customcode`、`size`、`keymap`等信息，这些信息均可通过解构`remote.tab{1,2,3}`文件获得。解构命令如下：

```bash
remotecfg -t /vendor/etc/remote.tab1 -d
```

返回内容大概如下所示：

```text
tabdir = /vendor/etc/remote.tab1
custom_name = amlogic-remote-1
fn_key_scancode = 0x47
cursor_left_scancode = 0x10
cursor_right_scancode = 0x11
cursor_up_scancode = 0xb
cursor_down_scancode = 0xe
cursor_ok_scancode = 0xd
custom_code = 0x4040
release_delay = 80
map_size = 36
key[0] = 0x430071
key[1] = 0x470047
key[2] = 0x10002
key[3] = 0x20003
key[4] = 0x30004
key[5] = 0x40005
key[6] = 0x50006
key[7] = 0x60007
```

其中`custom_code`、`map_size`、`key[0-n] =`即为所需修改信息。
实际上，除了修改也可以新增方案，比如原方案是`map_0 { ... phandle = <0x2d>; }`，完全可以在其后添加节点`map_1 { ... phandle = <0x2e>; }`。
修改好后，重新打包、重新刷机即可。
