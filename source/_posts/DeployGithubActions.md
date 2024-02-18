---
title: Deploy Github Actions
date: 2021-02-01 15:38:12
tags: 
	- Cloud Server
	- git
categories: 新手指北
id: DeployGithubActions
---

Sometimes automated deployment projects or timed tasks need to be deployed online, but these are for temporary use or these lightweight applications do not warrant the additional purchase of a cloud server for deployment. This is when you might consider using Github Actions for automated deployments.

[GitHub Actions[1]](#1) is GitHub's [continuous integration service[2]](#2), launched in [October 2018[3]](#3). It is very powerful in that each action is used to perform an action, such as grabbing code, running tests, logging into a remote server, publishing to a third-party service, and so on. Combining these actions is a process of continuous integration. Of course, these actions are shared in the GitHub code repository, so we can refer to them directly.

Github Actions provides a complete server environment with the following server specifications

- 2-core CPU
- 7 GB RAM memory
- 84 GB of SSD hard disk space

Detailed system environment information is shown in the following figure:

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2021/02/02/ym5WQJ.png" width="76%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">System environment information</div>
</center>

Of course, you can use Windows Server 2019 and macOS X Catalina 10.15 in addition to Ubuntu.

<!-- more -->

This looks great, but GitHub Actions itself doesn't allow direct connections for interactive operations, which means you can't connect to the server via SSH. If there was a way to connect directly to the server interactively, wouldn't that be like getting one or more VPS with an E5 2vCPU/7G RAM/90G SSD configuration for free?

This article will show you how you can get around the limitations of GitHub Actions itself and connect directly to the server by doing a few tricks!

> Note: Please do not use it for malicious purposes. All consequences such as banning, deterioration of Sino-American relations, atomic bombing, World War III, etc. are not the author's responsibility.

# Option One

- [mxschmitt/action-tmate[4]](#4)

This is the first action that implements [tmate[5]](#5) to connect to the Actions server, but this solution cannot proceed to the next step after exiting the connection, so it has little value in practice and can only be used for SSH connections. However, due to its groundbreaking role, I decided to put it first.

Example workflow file:

```yml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Setup tmate session
      uses: mxschmitt/action-tmate@v2
```

# Option Two

- [csexton/debugger-action[6]](#6)

This action is inspired by [mxschmitt/action-tmate[7]](#7), which also connects via tmate and allows you to continue to the next step after exiting the connection, making it better suited for use in real-world projects. The authors have added an automatic disconnect for 15 minutes by default, probably in the interest of saving resources for GitHub, but this can be removed by running `touch /tmp/keepalive`.

Example workflow file:

```yml
name: debugger-action
on: 
  watch:
    types: started
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
     - uses: actions/checkout@v2
 
     - name: Setup Debug Session
       uses: csexton/debugger-action@master
```

Action Log output:

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2021/02/01/yZi4BV.png" width="100%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Action Log</div>
</center>

Here you can use the connection given for SSH connections, but sometimes I have some problems running OpenSSH under windows. So I choose to use a browser to open `https://tmate.io/t/authToken` to connect to the remote host. The `authToken ` here is shown in the token string after the ssh command. In the example given above, the link is `https://tmate.io/t/vX3de9KCggEWpQTcc8B9xYuaP`

# Option Three

Instead of using action to achieve this, the solution goes the other way and uses ngrok to penetrate the intranet directly, with the following script.

```sh
#!/bin/bash
 
 
if [[ -z "$NGROK_TOKEN" ]]; then
  echo "Please set 'NGROK_TOKEN'"
  exit 2
fi
 
if [[ -z "$USER_PASS" ]]; then
  echo "Please set 'USER_PASS' for user: $USER"
  exit 3
fi
 
echo "### Install ngrok ###"
 
wget -q https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-386.zip
unzip ngrok-stable-linux-386.zip
chmod +x ./ngrok
 
echo "### Update user: $USER password ###"
echo -e "$USER_PASS\n$USER_PASS" | sudo passwd "$USER"
 
echo "### Start ngrok proxy for 22 port ###"
 
 
rm -f .ngrok.log
./ngrok authtoken "$NGROK_TOKEN"
./ngrok tcp 22 --log ".ngrok.log" &
 
sleep 10
HAS_ERRORS=$(grep "command failed" < .ngrok.log)
 
if [[ -z "$HAS_ERRORS" ]]; then
  echo ""
  echo "=========================================="
  echo "To connect: $(grep -o -E "tcp://(.+)" < .ngrok.log | sed "s/tcp:\/\//ssh $USER@/" | sed "s/:/ -p /")"
  echo "=========================================="
else
  echo "$HAS_ERRORS"
  exit 4
fi
```

This script is used to create a TCP tunnel for the SSH service and print out the commands to connect to the remote server over the public network.

First you need to register an account on [ngrok's official website[8]](#8) and generate a Tunnel Authtoken: https://dashboard.ngrok.com/auth. Then create the following workflow.

```yml
name: Debugging with SSH
on: push
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
     - uses: actions/checkout@v1
 
     - name: Try Build
       run: ./not-exist-file.sh it bloke build
 
     - name: Start SSH via Ngrok
       if: ${{ failure() }}
       run: curl -sL https://gist.githubusercontent.com/retyui/7115bb6acf151351a143ec8f96a7c561/raw/7099b9db76729dc5761da72aa8525f632d8875c9/debug-github-actions.sh | bash
       env:
        # After sign up on the https://ngrok.com/
        # You can find this token here: https://dashboard.ngrok.com/get-started/setup
        NGROK_TOKEN: ${{ secrets.NGROK_TOKEN }}
 
        # This password you will use when authorizing via SSH 
        USER_PASS: ${{ secrets.USER_PASS }}
 
     - name: Don't kill instace
       if: ${{ failure() }}
       run: sleep 1h # Prevent to killing instance after failure
```

The default server duration is 1 hour, but you can adjust this. The TOKEN and SSH passwords in this case are best done the way recommended in workflow, by creating the Secret in GitHub and then referencing the Secret in workflow. See the official documentation [9] for details.

# Additional notes

## Switching to a root user on linux

**(1)`sudo` command** 

`example@ubuntu:~$ sudo`

This will give you superuser privileges by entering the current admin user password. However, by default root privileges are disabled after 5 minutes.

**(2) `sudo -i`**

`example@ubuntu:~$ sudo -i`

In this way you can get to the root user by entering the password of the current admin user.

**(3) If you want to use root privileges all the time, you have to switch to the root user via `su`.**

Then we first have to reset the password for the root user:

`example@ubuntu:~$ sudo -i`

`example@ubuntu:~$ sudo passwd root`

This will set the password for the root user.

**(4) After that you can freely switch to the root user**

`example@ubuntu:~$ su`

Enter the password for the root user.

`su "king" `or `exit ` to return to user rights

## Example

Here are some automated Github Actions I have deployed:

- [支持Github Action的NEU健康打卡自动化python脚本](https://github.com/shimmerjordan/health-check-automation)
- [Mirror github repo to other platforms such as gitee via github actions](https://github.com/shimmerjordan/hub-mirror)
- [github account statistics based on 'github action', using py script to generate svg statistics graphs](https://github.com/shimmerjordan/github-statis)
- [Automatic check-in to GLaDOS via timed tasks based on Github Actions](https://github.com/shimmerjordan/GLaDOS-autoCheckin)
- [This is a repository using github actions to run super-linter automatically](https://github.com/shimmerjordan/sz-education)

# Reference

<div id="1" ><a href="https://github.com/features/actions" target="_blank">[1]. GitHub Actions</a></div>

<div id="2" ><a href="http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html" target="_blank">[2]. 持续集成服务</a></div>

<div id="3" ><a href="https://github.blog/changelog/2018-10-16-github-actions-limited-beta/" target="_blank">[3]. 推出</a></div>

<div id="4" ><a href="https://github.com/mxschmitt/action-tmate" target="_blank">[4]. mxschmitt/action-tmate</a></div>

<div id="5" ><a href="https://github.com/tmate-io/tmate" target="_blank">[5]. tmate</a></div>

<div id="6" ><a href="https://github.com/csexton/debugger-action" target="_blank">[6]. csexton/debugger-action</a></div>

<div id="7" ><a href="https://github.com/mxschmitt/action-tmate" target="_blank">[7]. mxschmitt/action-tmate</a></div>

<div id="8" ><a href="https://ngrok.com/" target="_blank">[8]. ngrok 的官网</a></div>

<div id="9" ><a href="https://docs.github.com/cn/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets" target="_blank">[9]. 官方文档</a></div>

<div id="9" ><a href="https://p3terx.com/archives/ssh-to-the-github-actions-virtual-server-environment.html" target="_blank">[10]. SSH 连接到 GitHub Actions 虚拟服务器环境</a></div>