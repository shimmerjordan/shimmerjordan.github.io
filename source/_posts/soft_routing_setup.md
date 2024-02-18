---
title: 软路由折腾笔记
date: 2023-01-07 21:28:12
tags: 
    - 软路由
categories: Network
id: soft-routing-setup
---

> 软路由的入门折腾笔记，给自己以后换房子重新部署做个参考

目前计划实现的功能：

- ipv6公网的DDNS，绑定域名之后实现公网访问
- 广告过滤，（翻阅长城的功能暂定不加）
- 利用`x-ui`搭建一个小型流量中转站
- 搭建青龙面板跑一些基础的脚本
- 离线下载
- 私有云

<!-- more -->

# 软路由刷机

> 第一次选用了NanoPi R2S作为入门机器，这个刷机流程记录了的是R2S的。后面更换机型需要重新找固件，比较靠谱的有[恩山](https://www.right.com.cn/)、知乎等等
> 
> R2S性能比较差，部署不了几个功能。后面打算直接上x86的软路由一步到位

## 准备工作

1. 刷机软件：[ethcer](https://www.balena.io/etcher/) or [rufus](https://rufus.ie/zh/)；

2. 软路由系统固件：（这个R2S刷的openwrt是在恩山上找的。[klever](https://github.com/klever1988/nanopi-openwrt/releases/)版本的话，选择版本时请注意带slim的为轻量级固件，因为R2S实在性能堪忧）[恩山无线论坛-openwrt-r2s](https://www.right.com.cn/forum/thread-4387071-1-1.html)；初始管理地址：`192.168.8.1`，初始管理密码：空；

3. 用来刷系统的存储设备（R2S这里用的TF卡，要搭读卡器。后面x86软路由换用U盘）；

4. 电脑＋网线

## 简要流程

1. 系统烧录：这个没什么好说的，用刷机软件把下载好的固件镜像刷进存储介质就行，需要注意的点只是：**刷入成功后会提醒你格式化U盘，千万不要！！！**

2. R2S简单初始化：完成上述步骤后，将R2S电源接通，用网线把PC与R2S的`LAN`口相连。然后在浏览器根据固件设置的默认初始管理地址和密码进入后台。然后进行一些基本的设置：
   
   - 设置`LAN`口：网络 - 接口 - `LAN`口 - 高级设置 - 打开`动态DHCP`
   
   - 设置`WAN`口：
     
     - 如果家里是**光猫拨号**上网，那就设置模式为`DHCP`客户端。  
     
     - 如果家里是**路由器拨号**上网，或者是使用**R2S做主路由**，设置为`PPPOE`拨号上网，输入宽带账号和密码，保存并应用。
   
   - 设置好之后将光猫与R2S的`WAN`口连接起来，这时候应该就可以上网了。拨号成功后会显示IP地址。

> 关于这个拨号账号密码以及光猫桥接模式的设置，不同型号的光猫配置方式略有不同；关于这次配置的是移动宽带的GS3101，可以参考{% post_link GS3101_bridge 移动光猫吉比特GS3101超级账号获取更改桥接 %}

3. 如果要把R2S设置为主路由，将无线路由器的LAN口与PC相连，进入管理后台（这次使用的HIKVISION的地址是`http://192.168.0.1/`）。开启DHCP后将光猫`LAN`口连接R2S的`WAN`口，R2S的`LAN`口连接路由器的`WAN`口，即可完成组网。

# 软路由扩容

> 一开始用的`squashfs` 文件系统的固件，虽然支持还原出厂设置，但是扩容起来很麻烦，后来改用`ext4`文件系统的固件了。

1. SSH登录OpenWrt后，安装磁盘分区工具，先后顺序不能错：
   
   ```shell
   opkg update
   opkg install block-mount e2fsprogs
   opkg update
   opkg install fdisk blkid vim
   opkg install cfdisk
   ```
   
   命令行中 `cfdisk` 、`fdisk` 均为磁盘分区工具，`e2fsprogs` 包含了 `mkfs` 命令，用于格式化分区。

2. 使用`fdisk -l`查看分区列表，找到自己想要扩展的磁盘分区，我这里是`/dev/mmcblk0`

3. 使用`cfdisk /dev/mmcblk0` 命令进入磁盘分区界面（`/dev/mmcblk0`根据实际情况替换），通过键盘上下键切换到 **Free space** （剩余空间），左右键切换至 **NEW** （新增分区），然后按下回车键。分区类型设置为 **Primary** （主分区），按下回车键。左右键切换 **Write** 按下回车键，写入新分区。输入 **yes** 按下回车键，确认写入新分区。

4. 切换 **Quit** ，按下回车键退出。运行 `fdisk -l` 命令，查看是否成功创建新分区。

5. 格式化新分区：运行命令`mkfs.ext4 /dev/mmcblk0p3`，将新分区的文件系统格式化为 `ext4`

6. 挂载新分区：
   
   1. 进入 Open­Wrt 管理后台，依次点击 **系统** - **挂载点** 找到并点击全局设置中的 **生成配置** 。如果没有自动生成，在挂载点手动添加。
   
   2. 在 **挂载点** 找到创建的新分区，点击 **修改** 重新调整挂载项目的设置。
   
   3. 勾选 **启用此挂载点**，**挂载点** 选择为 **作为根文件系统使用** ，完整复制根目录准备中的所有命令行后，点击 **保存并应用**。
      
      > ⚠️ **上面复制得到的命令行不要直接运行！！！不要直接运行！！！否则报错。**
   
   4. 需要手动修改命令行中 `mount /dev/sda1 /tmp/extroot` 为 `cfdisk` 创建的新分区盘符，例如这里应修改为 `mount /dev/mmcblk0p3 /tmp/extroot`，然后进入 SSH 终端，运行修改后的完整命令行，如下：
      
      ```shell
      mkdir -p /tmp/introot
      mkdir -p /tmp/extroot
      mount --bind / /tmp/introot
      mount /dev/mmcblk0p3 /tmp/extroot
      tar -C /tmp/introot -cvf - . | tar -C /tmp/extroot -xf -
      umount /tmp/introot
      umount /tmp/extroot
      ```
      
      回车键到底，直到跑完所有命令行。然后运行 `reboot` 重启 Open­Wrt后重新进入 SSH 终端运行 `df -h` 即可查看扩容效果。

# 家宽ipv6 DDNS公网访问

> 现在家宽基本只能给到动态的公网ipv6了，有很多DDNS服务的提供商。阿里云也提供付费的DDNS服务，这里以免费的[dynv6](https://dynv6.com)为例进行部署。

在dynv6申请了子域名之后进入`Documentations -> APIs`，在管理界面复制`token`中的内容。对于路由器而言，我们的需求是路由器应当能够定期向dynv6网站汇报自己的IP地址，在Openwrt里我们采用DDNS软件包完成这件事。

一般来说固件编译的时候就包含了这个软件包，在`服务 -> 动态DNS`中进入DDNS界面。添加一条DDNS规则时注意要将ipv4以及ipv6分别添加，我这里只有ipv6的公网，那就只添加ipv6的规则。其中部分字段可以参考以下表格：

| 字段        | 值                           | 备注                     |
|:---------:| --------------------------- |:---------------------- |
| 启用        | 勾选                          |                        |
| 查询主机名     | ipv6.dynv6.com              | 这是采用dynv6进行ddns的固定填写方式 |
| IP地址版本    | ipv6地址                      |                        |
| DDNS服务提供商 | dynv6.com                   |                        |
| 域名        | xxxx.dynv6.com              | 填写申请的子域名               |
| 密码        | **************              | 填写之前复制的token           |
| IP地址来源    | URL                         | 以下为高级设置                |
| 用于检测的URL  | http://checkipv6.dyndns.com |                        |
| 事件网络      | wan                         |                        |

其他没有涉及的字段可以考虑保持默认设置，再自行设置定时器（更新频率）。地址来源选择URL即可以使用运营商分配的/64短码IPv6地址，看起来美观一些，当然，实际使用上并没有太大区别

随后保存并应用即可，可以试试用路由器的诊断工具测试IPv6的连接情况，访问[这里](http://checkipv6.dyndns.com/)或[这里](https://testipv6.cn)得到你的IPv6地址（电脑访问得到电脑的，路由器访问得到路由器的，由于在这里没有NAT的概念，我们说的都是全局的、唯一的IPv6地址）

可以看到现在DDNS检查进程已经启动了，如果没有显示进程的PID，那么手动点击“启动”即可。

当然，我们可以发现API界面也推荐使用一个小脚本进行更新。可以在路由器上部署此脚本的定时任务完成这一需求。

如果配置完成后，地址更新成功且https://testipv6.cn测试通过，但仍旧不能从外网访问路由器，可能有以下几种原因：

- 运营商分配了IPv6，但禁止了传入连接（罕见，多见于教育网、机关单位等统一配置的防火墙，个人无法解决）

- 光猫拨号，向下级设备分配了IPv6地址，但光猫的防火墙禁止了传入连接（较常见，可通过修改光猫配置关闭防火墙或改为桥接模式、路由器拨号）

- Openwrt自带防火墙默认拒绝传入连接（常见，修改设置即可，参考：https://blog.csdn.net/weixin_43593122/article/details/95766357）

- 运营商禁用了常用端口80、443、8080等（常见，修改http服务器的端口再次测试即可，参考：https://blog.csdn.net/weixin_43593122/article/details/95766357）

# Docker安装X-UI

Openwrt已经编译包含了Docker，在自建服务器中可以使用`curl -fsSL https://get.docker.com | sh`完成Docker的安装，安装完之后会自动启动。

推荐新建一个`x-ui`的目录进行部署：

```shell
mkdir x-ui && cd x-ui
docker run -itd --network=host -v $PWD/db/:/etc/x-ui/ -v $PWD/cert/:/root/cert/ --name x-ui --restart=unless-stopped enwaiax/x-ui:latest
```

访问`http://服务器IP:54321`使用账号admin密码admin登录.注意需开放相关端口防火墙,并及时修改账号密码。

如果没有自动启动Docker那就试试看重启路由器，我就重启法修复了，不明所以。

可以先检索所有运行中的Docker然后根据ID进入Docker环境：

```shell
docker ps
docker exec -it XXXX /bin/bash # 容器ID
```

# Docker安装青龙

> 青龙面板仓库地址：[Docker Hub](https://hub.docker.com/r/whyour/qinglong "Docker Hub")

## 拉取镜像

使用以下命令拉取青龙面板的镜像，等待下载完成（失败或缓慢可以挂梯子）

```shell
docker pull whyour/qinglong:latest
```

## 创建容器

命令行输入一下命令创建容器，如果报错那就重启路由器再试试：

```shell
docker run -dit \
-v $pwd/ql/config:/ql/config \
-v $pwd/ql/log:/ql/log \
-v $pwd/ql/db:/ql/db \
-v $pwd/ql/scripts:/ql/scripts \
-v $pwd/ql/jbot:/ql/jbot \
-p 5700:5700 \
-e ENABLE_HANGUP=true \
-e ENABLE_WEB_PANEL=true \
--name qinglong \
--hostname qinglong \
--restart always \
whyour/qinglong:latest
```

## 登录面板

默认地址为openwrt后台管理ip+端口5700，例如`192.168.8.1:5700`或者之前DDNS的域名＋端口，在浏览器打开即可完成配置。
