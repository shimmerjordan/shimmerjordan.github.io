---
title: 利用kplayer实现无人值守推流云盘资源
date: 2023-11-28 13:48
tags: 
    - Alist
    - Linux服务
    - NAS
categories: NAS
id: kpalyerWithCloudFile
---

在Linux主机（NAS、软路由等）上部署无人值守的24h推流直播服务，利用Alist，NFS等服务挂载自动挂载云盘资源。

> [Blibili直播地址](http://live.bilibili.com/21256315)

<!-- more -->

# Alist挂载云盘

1. NAS利用`Alist docker`挂载云盘：

```shell
alist:5244
docker run -d --restart=always -v /etc/alist:/opt/alist/data -p 5244:5244 -e PUID=0 -e PGID=0 -e UMASK=022 --name="alist" xhofe/alist:main
```

    QNAP用docker方案挂载Alist会在每次重启NAS的时候丢失挂载的网盘配置，建议直接使用脚本在目录中安装：[One-click Script | AList Docs (nn.ci)](https://alist.nn.ci/guide/install/script.html)

2. 利用HybridMount等应用挂载Alist内云盘到NAS本地
   
   > http://localhost:5244/dav

# KPlayer

> https://docs.kplayer.net/

暂时直接参考这个吧，没时间写
