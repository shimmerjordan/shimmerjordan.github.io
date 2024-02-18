---
title: Chrome浏览器规避cookie覆盖以及前端开发跨域问题解决
date: 2020-06-27 15:55:53
tags:
	- reference
	- 随笔
	- 工具
categories: 新手指北
id: multi-chrome
---

Chrome浏览器在通常情况下只能在同一网站登录同时登陆一个账号，无论是新标签页还是新窗口下登录其他账号均将覆盖原有cookie，强制下线原有账号。考虑到工作、学习以及生活娱乐的不同需求，需要在同一网站多开账号，以及拥有不同的Chrome账户的同步需求。这里为Chrome多开帐号以及修改默认安装位置（C盘）做了简单笔记，以供后续参考。

<!--more-->

# Chrome默认安装位置的修改

众所周知，默认状态下的直接运行`chrome_84.0.4147.68x64.exe`Google Chrome只能安装在系统分区（C盘）：

> Vista：C:\Users\用户名\AppData\Local\Google\Chrome
>
> XP：C:\Documentsand Settings\用户名\Local Settings\Application Data\Google\Chrome

实际上可以通过自定义的方式解决安装未知的问题，默认下载到的`Chrome_installer.exe`其实是一个自解压压缩文件包，使用常规的解压缩软件（比如Bandzip）等就可以将其解压。譬如最新（2020年6月27日 16:08:19）的84.0.4147.68版本，以此为例：

1. 下载Chrome 84.0.4147.68版本的`chrome_84.0.4147.68x64.exe`安装文件；
2. 右键，单击`“用Bandzip打开”`将其解压缩；
3. 得到一个`Chrome.7z`的压缩包，继续将其解压缩；
4. 所得的`Chrome-bin`文件夹中便可以找到`Chrome.exe`文件了；
5. 将文件夹` 84.0.4147.68`中的所有文件包括文件夹都剪切出来，使其与`Chrome.exe`在同一个目录中。现在双击这个exe文件，便可以启动Chrome。

# 使用启动参数实现多用户数据共存

接着以上的安装步骤，右击`chrome.exe`选择发送桌面快捷方式，然后继续右击桌面的`chrome.exe快捷方式`进入属性窗口。在属性的`目标`输入框的文本后添加：

```
 --user-data-dir=D:\Chrome\1
```

注意这里的`--user`前有一个空格，`D:\Chrome\1`可自定义，具体参数为`chrome.exe路径\分用户数据存储路径`，这里我如法炮制设置了四个用户（修改1、2、3、4用户数据文件夹即可）：

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://s1.ax1x.com/2020/06/27/N68ao6.jpg" width="50%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">chrome 属性.</div>
</center>

打开网页测试可用，回查资源管理器中Chrome安装目录已经独立存放了四个分用户的数据（文件夹分别为1、2、3、4），至此便解决了Chrome浏览器自定义安装以及多开的问题。

# 跨域问题解决

在开发前后端分离项目的时候，发现前端出现跨域问题，但服务器并不是本地服务器而是第三方接口（百度OCR识别）。使用HBuilder内置浏览器无报错，但使用Chrome浏览器会报如下的错误：

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://s1.ax1x.com/2020/07/14/UthshD.jpg" width="70%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">跨域报错</div>
</center>

因此判定问题出现在浏览器，这里同样以上面的方式打开Chrome快捷方式的属性，在页面的目标输入框内的最后加上` --disable-web-security`即可解决问题。

可能会出现以下的提示，也表示设置成功，部署前端工程重新尝试发现无报错。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://s1.ax1x.com/2020/07/14/UthzNT.jpg" width="70%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">--disable-web-security</div>
</center>

## 另有Tips：

1. `Locales`目录中是各国的语言dll文件，只保留`en-US.dl`l和`zh-CN.dll`就可以了，建议还是保留。
2. 进一步定制，可以用的参数：
   1. ` --single-process` 单进程运行Google Chrome
   2. ` --start-maximized` 启动Google Chrome就最大化
   3. ` --disable-java` 禁止Java