---
title: Geforce Now 配置指北
date: 2022-06-30 22:38:12
tags: ['随笔']
categories: 新手指北
id: geforce-now-setup
---

> Geforce Now，一个姣好的云游戏平台解决方案

<!-- more -->

# 1. Geforce Now下载

[Download GeForce NOW | NVIDIA GeForce NOW](https://www.nvidia.com/en-us/geforce-now/download/)点击链接后点击`WINDOWS PC`栏目下的`Download`下载GN Windows版

随后按照指引安装即可

# 2. VPN配置

> VPN这里仅仅是为了登录Geforce Now账号，**登陆完成后即可关闭VPN**

点击[v2ray-Core.zip - 蓝奏云 (lanzouv.com)](https://wwp.lanzouv.com/iAf13076ducj)下载并解压后，点击`v2rayN-Core.exe`运行，在右下角任务栏找到图标双击打开主页面。然后参照[V2rayN[Vmess/Trojan] - 教程WIKI](https://help.loliloli.live/jiao-cheng/windows/v2rayn-v2ray-xie-yi-xin-shou)配置即可。

因为Geforce Now的账号为台湾区的，这里要找到**台湾的节点**，回车选定之后**在任务栏右击图标**开启代理服务（自动配置代理服务的选项）。注意要开启**代理模式为全局模式**。

> 如果不好用的话可能是因为节点不行，可以尝试换个台湾的节点。注意此时加速器关闭，VPN开启。其实可以事先在v2rayN界面右击选定节点进行测速。

# 3. 设置默认浏览器为Chrome并登录GN

在游戏菜单中（网吧环境）找到谷歌浏览器，启动之后按照弹窗提示设置为默认浏览器。如果没有弹窗，可以在右上角菜单里面找到设置 -> 默认浏览器，在系统弹窗内设置默认浏览器为Chrome

或者直接点击Geforce Now中的登录跳转到IE浏览器，复制地址栏URL到Chrome中进行访问，然后登陆。

登录的时候**不要开加速器加速**Geforce Now，开VPN就行。

> 如果还是提示中国不提供服务，那就选择其他地区服务商。找到台湾大哥大点击进入后使用提供的账号密码登陆即可。

# 4. 加速器加速GN

**登陆完成后**，雷神加速器可以加速Geforce Now，搜索之后选择台湾服，任意节点加速即可。

# 5. 启动游戏

此时即可在Geforce Now中启动相应的游戏，此时可以关闭VPN代理，但是加速器不能关。需要注意的是，Geforce Now比较吃网络环境，如果断开连接重连即可。另外，每次登陆游戏的**单次会话时长最长为 6 小时**。超时重新登陆游戏即可。
