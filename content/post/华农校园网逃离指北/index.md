---
title: "华农校园网逃离指北"
description: 
date: 2023-11-01T12:20:18+08:00
image: 
math: 
license: CC-BY-NC-SA
hidden: false
draft: false
tags:
    - 计维躺着滑
categories:
    - 路由器
    - IPV6
author:
---

# 逃离华农校园网指北

## 要干什么？

这篇文章主要是介绍如何通过二手路由器和二手5G手机制作丐版5G CPE并配合物联卡实现宿舍多人低价上网。

## 准备材料

小米路由器3G，中兴远航10，网线，fat32文件系统的U盘

## 文件下载

**建议科学上网并配合IDM下载器**
[WinSCP][1]
[breed-mt7621-xiaomi-r3g.bin][2]
[x-wrt-23.10-b202310211921-ramips-mt7621-xiaomi_mi-router-3g-squashfs-breed-factory.bin][3]
[x-wrt-23.10-b202310211921-ramips-mt7621-xiaomi_mi-router-3g-squashfs-sysupgrade.bin][4]

## 成本计算

小米路由器3G拼多多购买二手55元，中兴远航10于拍机堂软件购买，屏幕有压伤，价格169元。
物联网卡每个月套餐40元1200G流量，宿舍6人平摊每人每月6.7元。

## 网络测速

测速时间：10月29日周日13：28
![网速测试1.png][5]
测速时间：10月26日周四00：08
![网速测速2.jpg][6]

## 中兴领航10开启usb网络共享

在**设置>关于手机**中连续点击5次**版本号**开启开发者模式，在**系统与更新>开发者模式**中使**默认usb配置**改为**usb网络共享**。

## 路由器刷入x-wrt

### 刷入开发版固件

从**miwifi.com**官网下载R3G路由器的开发版固件，进入浏览器输入**192.168.31.1**进入小米路由器后台，在**常用设置>系统状态**中选择**手动升级**，选择你刚下载的固件。![1.png][7]![2.png][8]

### 开启ssh

#### 手机绑定路由器

为了绑定路由器，必须先给路由器连上网，我们先拿网线将电脑的网口连接上路由器的wan口，电脑连上手机开的热点，在**控制面版>网络和Internet>网络连接**中，调整**WLAN属性**，在**共享**中选择**允许其他网络用户通过此计算机的Internet连接来连接**，并选择**以太网**。
再拿另一台手机（不要拿开热点的手机）下载**小米wifi**，连接路由器的wifi并且在软件中绑定路由器。![3.png][9]![4.png][10]

#### 开启ssh

进入miwifi官网，在开放页面中找到开启ssh工具，找到绑定的路由器并下载miwifi_ssh.bin文件，记住root密码。根据官网步骤开启ssh。![5.png][11]![6.png][12]

### 刷入breed

将网线从路由器wan口接到lan口，电脑关闭上一步配置的网络共享，安装WinSCP，新建站点，选择文件协议为SCP，填写主机名为192.168.31.1，用户名为root，密码为上一步得到的root密码
在路由器的/tmp目录下将**breed-mt7621-xiaomi-r3g.bin**文件上传
在终端“输入命令”处输入```mtd -r write /tmp/breed-mt7621-xiaomi-r3g.bin Bootloader```![7.png][13]

### 刷入x-wrt kernel底包

#### 进入breed

路由器断电，按住路由器的复位键再通电，直到路由器状态灯变为蓝色闪烁即可松开，在浏览器输入192.168.1.1即可进入breed控制台。

#### 调整breed设置

在**小米R3G设置**中删除**normal_firmware_md5**这一项，在**环境变量编辑**中添加一个环境变量，名为**xiaomi.r3g.bootfw**，值为2。![8.png][14]![9.png][15]

#### 刷入

在固件更新中，固件选择**x-wrt-23.10-b202310211921-ramips-mt7621-xiaomi_mi-router-3g-squashfs-breed-factory**，**闪存布局**选择**小米路由器固件2**，点击上传即可。![10.png][16]

### 刷入x-wrt固件

等待路由器亮蓝灯后，进入192.168.15.1,在**系统>备份/升级**中，刷写新的固件，选择固件**x-wrt-23.10-b202310211921-ramips-mt7621-xiaomi_mi-router-3g-squashfs-sysupgrade.bin**，并且**去掉勾选**保持设置并保留当前配置。![11.png][17]![12.png][18]

## 路由器联网

将手机插入路由器，再解锁手机即可实现路由器联网。

### 配置Wi-Fi

浏览器进入192.168.15.1，在网络，无线中，选择编辑，在常规设置中，ESSID就是wifi名字。无线安全中的密钥就是wifi密码。![13.png][19]![14.png][20]![15.png][21]

### 配置ipv6（若不需要ipv6可以不用设置）

进入路由器后台，在网络，接口中，分别配置usblan6，usbwan和lan。![16.png][22]

#### 配置usbwan6

点击usbwan6的编辑选项，在*DHCP服务器>常规设置*中，勾选忽略此接口。在IPV6设置中，勾选指定的主接口，RA服务，DHCPv6服务，NDP代理均选择中继模式，其他默认。![17.png][23]![18.png][24]

#### 配置usbwan

点击usbwan的编辑选项，在DHCP服务器>常规设置中，勾选忽略此接口。在IPV6设置中，RA服务，DHCPv6服务，NDP代理均选择中继模式，其他默认。

#### 配置lan

点击lan的编辑选项，在DHCP服务器>常规设置中，不要勾选忽略此接口。在IPV6设置中，RA服务，DHCPv6服务，NDP代理均选择已禁用，其他默认。

## 手机放置位置

可以通过手机设置中的灵犀智能助理中的5g智慧通讯中查看当前位置的信号强度，数字越大信号越好（注意，数字为负数），一般放置在窗台信号较好。

## 物联网卡注意事项

可以在淘宝中搜索纯流量卡，找到有评论存在时间久的商家，价格不要便宜的离谱，一般19.9元150G，40块钱1200G，这种商家跑路概率低，最好不要购买需要预存的卡。注意，不管要不要预存，商家随时可能跑路。（我已经使用了3个月，商家目前还没跑路）
物联网卡插入手机后，不要更换卡槽，若要更换，请咨询商家。


[1]: https://pan.200502.xyz/d/%E6%9C%AC%E5%9C%B0/%E9%80%83%E7%A6%BB%E5%8D%8E%E5%86%9C%E6%A0%A1%E5%9B%AD%E7%BD%91%E6%8C%87%E5%8C%97/WinSCP-6.1.2-Setup.exe?sign=wWn80X5VUygl4k75GEO0W54mm3aSnVVmdjHxHEQJV8M=:0
[2]: https://pan.200502.xyz/d/%E6%9C%AC%E5%9C%B0/%E9%80%83%E7%A6%BB%E5%8D%8E%E5%86%9C%E6%A0%A1%E5%9B%AD%E7%BD%91%E6%8C%87%E5%8C%97/breed-mt7621-xiaomi-r3g.bin?sign=z27oHJN4_SggRxtBo211MQ6FgCR2d_P9BdS9cBv8Otk=:0
[3]: https://pan.200502.xyz/d/%E6%9C%AC%E5%9C%B0/%E9%80%83%E7%A6%BB%E5%8D%8E%E5%86%9C%E6%A0%A1%E5%9B%AD%E7%BD%91%E6%8C%87%E5%8C%97/x-wrt-23.10-b202310211921-ramips-mt7621-xiaomi_mi-router-3g-squashfs-breed-factory.bin?sign=X6pdTqMRT3KVOki8iwfKuILXIpwaRua-w6fgshCic_A=:0
[4]: https://pan.200502.xyz/d/%E6%9C%AC%E5%9C%B0/%E9%80%83%E7%A6%BB%E5%8D%8E%E5%86%9C%E6%A0%A1%E5%9B%AD%E7%BD%91%E6%8C%87%E5%8C%97/x-wrt-23.10-b202310211921-ramips-mt7621-xiaomi_mi-router-3g-squashfs-sysupgrade.bin?sign=ta-EObOOyy0P4N8Hobaz9LtToiBMMDjOwE_zyQV28s8=:0
[5]: https://200502.xyz/usr/uploads/2023/10/1994915458.png
[6]: https://200502.xyz/usr/uploads/2023/10/1986880329.jpg
[7]: https://200502.xyz/usr/uploads/2023/10/2365582423.png
[8]: https://200502.xyz/usr/uploads/2023/10/4055184457.png
[9]: https://200502.xyz/usr/uploads/2023/10/3451820574.png
[10]: https://200502.xyz/usr/uploads/2023/10/3888824326.png
[11]: https://200502.xyz/usr/uploads/2023/10/196583575.png
[12]: https://200502.xyz/usr/uploads/2023/10/1086359669.png
[13]: https://200502.xyz/usr/uploads/2023/10/441784800.png
[14]: https://200502.xyz/usr/uploads/2023/10/1618741694.png
[15]: https://200502.xyz/usr/uploads/2023/10/1955564416.png
[16]: https://200502.xyz/usr/uploads/2023/10/2700655735.png
[17]: https://200502.xyz/usr/uploads/2023/10/4139157707.png
[18]: https://200502.xyz/usr/uploads/2023/10/331530815.png
[19]: https://200502.xyz/usr/uploads/2023/10/3484232852.png
[20]: https://200502.xyz/usr/uploads/2023/10/3604632226.png
[21]: https://200502.xyz/usr/uploads/2023/10/1229593936.png
[22]: https://200502.xyz/usr/uploads/2023/10/1793225585.png
[23]: https://200502.xyz/usr/uploads/2023/10/207740750.png
[24]: https://200502.xyz/usr/uploads/2023/10/894154629.png