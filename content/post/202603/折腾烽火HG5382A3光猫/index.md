---
date: "2026-03-29T22:04:23+08:00"
draft: false
title: "折腾烽火HG5382A3光猫"
tags:
  - 光猫
---

## 前言

家中是千兆移动宽带，运营商送的光猫是4个千兆口，且体积略大，为了缩小体积，同时验证本地宽带是否支持xgpon,入手了烽火HG5382A3（16元，不带电源）。

## 获取超级密码

移动光猫默认超级账号密码如下：

```text
账号：CMCCAdmin
密码：aDm8H%MdA
```

到货后，尝试使用默认超级账号密码无法登录,通过`telnet`获取超级密码。

1. 开启telnet。访问`http://192.168.1.1/cgi-bin/telnetenable.cgi?telnetenable=1&key={MAC地址}`即可开启，其中{MAC地址}替换为光猫的实际MAC地址，字母需大写，且后期如果更改MAC地址，输入也要同步更改。
2. 连接telnet。telnet账号密码如下：

```text
普通账户：
账号：admin
密码：Fh@7926B0
root账户
账号：root
密码：hg2x0
```

先从web端登录光猫，如果用user账号登录，连接telnet时，只能连接admin，如果用CMCCAdmin账号登录，连接telnet时，可以连接root。此处是为了获取超级密码，肯定是连接admin。

3. 获取账号密码。输入以下命令，即可找到超级账号密码。

```bash
cd /fhconf # 切换目录
cat usrconfig_conf # 可选，查看所有参数
cfg_cmd get InternetGatewayDevice.DeviceInfo.X_CMCC_TeleComAccount.Username # 获取默认超级账号用户名 类似：get success!value=CMCCAdmin
cfg_cmd get InternetGatewayDevice.DeviceInfo.X_CMCC_TeleComAccount.Password # 获取默认超级账号密码 类似：get success!value==CMCCAdminBpDe!5c
```

## 更改信息

实际操作发现，安徽移动仅验证逻辑ID,并不验证SN码和MAC地址（我先改过去，发现不验证后，又改回来...）。超级密码登录后，访问`http://192.168.1.1/cgi-bin/FactoryInfoCheck.cgi`进入抽检模式。
再使用root账户连接telnet，更改信息命令如下：

```bash
load_cli factory
set factorymode enable        # 启用工厂模式
load preconfig Anhui          # 切换省份
set onu_mac EC:CF:70:58:A2:60 # 更改MAC
exit                          # 退出工厂模式

cfg_cmd set InternetGatewayDevice.ManagementServer.PeriodicInformEnable 0 # 关闭RMS的上报
cfg_cmd set InternetGatewayDevice.ManagementServer.URL http://127.0.0.1/ # 修改RMS认证地址
```

修改GPON_SN、DEVICE_SN，它们不能通过命令更改，只能直接更改文件，假设设备当前SN是SKYW319C79CA，需要更改为FHTTC0F5A806，
其DEVICE_SN是GPON_SN将字母转换成十六进制ASCII码获得。

```bash
sed -i 's/SKYW319C79CA/FHTTC0F5A806/g' /fhdata/factory_conf
sed -i 's/534B5957319C79CA/46485454C0F5A806/g' /fhdata/factory_conf
```

## 认证

前文已经说明，安徽移动仅验证逻辑ID,但我操作时认证失败，所以才更改的SN、MAC等信息，更改后显示注册正常，但访问任何网站都会跳转到注册页面。
进入光猫web页面，将应用/服务设置/ 宽带账号注册状态修改为【1】即可。

```
状态/设备信息	光路（OLT）认证：注册正常
```

## 参考链接

1. [[烽火] HG5382A3 光猫密码以及开telnet](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=8405332&highlight=hg5382a3)

2. [[烽火] 烽火HG5382A3 到底是怎么修改SN的，有大佬支招吗](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=8449055&highlight=hg5382a3)

3. [“我可以不用，但不能没有啊”：30块更换光猫让网速提升了30%+](https://post.smzdm.com/p/a5p758mx/)
