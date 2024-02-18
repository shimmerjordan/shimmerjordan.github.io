---
title: 威联通QNAP切换MariaDB
date: 2024-01-20 11:48
tags: 
    - 开发环境
    - QNAP
    - MariaDB
categories: 开发环境
id: qnap_change_datadir
---

威联通QNAP装了MariaDB，部署了chevereto项目打算做私人图床，但MariaDB默认安装位置和数据存档位置都是系统盘，我给系统盘分配的空间空间不大，因此需要将数据存档位置迁移到其它数据卷。但苦于QNAP系统和传统ubuntu不一样，因此网上大部分教程都不适用。自己摸索出来可行的方案做个记录以免遗忘。

<!-- more -->

> 为方便操作，这里全都用WinSCP做浏览和编辑工具

# 新增QNAP数据卷信息

修改`/etc/config/def_share.info`文件（实际软链接到`/mnt/HDA_ROOT/.config/def_share.info`），在最下面新增：

```shell
defDataVol = /share/CACHEDEV2_DATA
```

这里新增的是需要设置的默认数据卷位置，也即存储池位置

# 修改MariaDB数据目录配置

修改MariaDB启动脚本`/share/CACHEDEV1_DATA/.qpkg/MariaDB5/etc/init.d/mariadb5.sh`中`DEF_VOL`以及`MARIADB_DIR_NAME`字段：

```shell
# 原始
DEF_VOL=`/sbin/getcfg SHARE_DEF defVolMP -f /etc/config/def_share.info`
MARIADB_DIR_NAME=.@qmariadb
# 修改
DEF_VOL=`/sbin/getcfg SHARE_DEF defDataVol -f /etc/config/def_share.info`
MARIADB_DIR_NAME=Study/data/Maria5_data
```

`DEF_VOL`修改为上一步配置的数据卷，`MARIADB_DIR_NAME`对应MariaDB数据的存储位置。

# 修改MySQL数据存储

> 可以从上面的配置文件中看出，`Maria5_data`的子目录包含`data`以及`tmp`。

修改MySQL的启动脚本配置文件：`/share/CACHEDEV1_DATA/.qpkg/MariaDB5/scripts/mysql_util.sh`中`DEF_VOL`以及`datadir`字段

```shell
# DEF_VOL=`/sbin/getcfg SHARE_DEF defVolMP -f /etc/config/def_share.info`
# datadir="${DEF_VOL}/.@qmariadb/data" #sql server saved location

DEF_VOL=`/sbin/getcfg SHARE_DEF defDataVol -f /etc/config/def_share.info`
datadir="${DEF_VOL}/Study/data/Maria5_data/data" #sql server saved location
```

这里的可以看出MySQL的数据存储是在MariaDB的数据目录的`data`子目录中

# 应用修改

在QNAP的Web中打开MariaDB应用关闭再打开即可生效。
