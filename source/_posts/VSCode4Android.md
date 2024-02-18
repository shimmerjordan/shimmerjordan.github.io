---
title: VSCode For Android 安卓使用vscode
date: 2024-01-17 19:48
tags: 
    - 开发环境
    - Android
categories: 开发环境
id: vscode4android
---

无意发现鸿蒙4可以在连接显示器的时候启用电脑模式，如果可以运行生产力软件——Visual Studio Code那就完美了。拾人牙慧实施了一些方案，发现截至20240117还是这种本地Termex+proot+vscode_server(pacman)的后端结合via浏览器全屏操作的前端方案较为舒适且坑少。

<!-- more -->

# 安装Termux

进入[IXCM工作室下载站 (icdown.club)](https://d.icdown.club/repository/main/ZeroTermux/)下载比较新的版本，Zero-Termux比老旧的Termux更好用。如果下载站挂掉可以尝试：

1. [F-Droid - Free and Open Source Android App Repository](https://f-droid.org/?utm_source=ld246.com)下载，不要用Play商店（老旧不再维护）

2. 查询资源备用站：[网站备忘/备用 | 丛烨-shimmerjordan](https://blog.shimmerjordan.eu.org/2024/01/17/web4memo/)

# 安装Linux系统

使用 [tmoe](https://shenzilong.cn/record/%E6%AF%8F%E6%97%A5%E6%80%BB%E7%BB%93/2022/2%E6%9C%88.html?utm_source=ld246.com#20230922210648-3rxty2b) 安装 proot ( 没有 root 的情况下 ) 的 linux 系统 （你会其他的也行）

文档： [android - 天萌参考手册 (tmoe.me)](https://doc.tmoe.me/zh/android.html?utm_source=ld246.com)

## 大致步骤

```shell
curl -LO https://l.tmoe.me/2.awk;  
awk -f 2.awk 
# 选 proot 容器 -> arm64发行版列表 -> Arch
```

安装完毕后在 termux 中输入 debian 即可进入刚安装的系统，还可以输入 tmoe 有很多有用的功能

# 安装VSCode

> 为什么不用 code-server 呢，因为版本落后

## 安装

在 arch 系统终端中输入（maybe需要魔法）

```shell
bash
pacman -S code
```

## 启动

输入

```shell
code --no-sandbox --user-data-dir="/root/app/code" serve-web --host 127.0.0.1
```

即可得到vscode的运行地址，复制到浏览器中等待安装完毕（需要魔法Maybe）——via浏览器可以启用全屏模式更舒适
