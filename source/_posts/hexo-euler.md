---
title: Hexo的云服务器部署
date: 2021-02-01 15:38:12
tags: 
    - Cloud Serve
    - Hexo
    - tutorial
categories: 新手指北
id: hexo_on_cloud_server
---

部署hexo博客到云服务器上的教程

# 环境部署

环境部署环节应该不需要多说，遇到问题了google之即可。

## 相关环境表

| 本地       | 远端欧拉   | 备注                         |
| -------- | ------ | -------------------------- |
| nodejs   | nodejs | 用`node -v`以及`npm -v`检查安装情况 |
| git      | git    |                            |
| hexo-cli |        | `npm i hexo-cli -g`安装      |

# 部署hexo

在本地安装完`hexo-cli`之后使用`hexo`检查安装成功与否。如果失败，先重新起一个命令行再尝试。

1. 在自定义位置初始化blog文件夹：

```shell
mkdir hexo-blog
cd hexo-blog && npm init -y
```

> 公司的网执行`npm i`可能会失败，因为其本质是从github拉取仓库，可以直接从github下载压缩包后解压到该目录：[GitHub - hexojs/hexo-starter: Hexo Starter site)](https://github.com/hexojs/hexo-starter)

2. 下载主题
   
   同以上一样，从[Themes | Hexo](https://hexo.io/themes/)选择合适主题下载，大部分主题都有教程如何安装。同理，如果直接使用npm无法拉取的话可以直接去github下载后放在`theme`文件夹下。
   
   在本地配置文件`_config.yml`文件中修改：
   
   ```yaml
   # 这里修改为自定义主题
   theme: redefine
   ```
   
   注意有的主体需要新建形如`_config.redefine.yml`的文件进行主题的配置

3. 本地执行`hexo`项目，添加`start`脚本
   
   先在`package.json`文件中修改：
   
   ```json
   "scripts": {
       "build": "hexo generate",
       "clean": "hexo clean",
       "deploy": "hexo clean && hexo g -d",
       "server": "hexo server",
       "start": "hexo clean && hexo g && hexo s"
   },
   ```
   
   再使用`npm start`启动项目，可以在 http://localhost:4000 验证效果了

# 部署远端git

> git的安装与配置不再详述，包括ssh认证的问题请自行解决。主要记录远端git私库的配置。

1. ssh登录到远端服务器后，创建用户并配置其仓库：
   
   ```shell
   useradd git
   # 设置密码
   passwd git 
   # 这步很重要，不切换用户后面会很麻烦
   su git 
   cd /home/git/
   # 项目存在的真实目录
   mkdir -p projects/hexo-blog
   mkdir repos && cd repos
   # 创建一个裸露的仓库
   git init --bare blog.git
   ```

2. 创建裸露仓库并配置hook
   
   ```shell
   cd blog.git/hooks
   # 创建 hook 钩子函数，输入了内容如下
   vi post-receive
   ```
   
   ```shell
   #!/bin/bash
   git --work-tree=/home/git/projects/hexo-blog --git-dir=/home/git/repos/blog.git checkout -f
   ```

3. 修改权限
   
   ```shell
   chmod +x post-receive
   #  退出到 root 登录
   exit
   # 添加权限
   chown -R git:git /home/git/repos/blog.git
   ```

4. 在本地测试git仓是否可用，在空白文件夹下发送以下命令，如果有空仓库下拉，说明git私库搭建成功：
   
   ```shell
   git clone git@server_ip:/home/git/repos/blog.git
   ```

5. 建立ssh信任关系，用于免密登录，在本地：
   
   ```shell
   ssh-copy-id -i C:/Users/yourname/.ssh/id_rsa.pub git@server_ip
   # 测试能否登录
   ssh git@server_ip
   ```
   
   如果windows无法使用`ssh-copy-id`，参考[windows无法使用ssh-copy-id解决方案](http://3ms.huawei.com/km/blogs/details/14354899)

6. （可选）安全起见禁用git用户的shell登录权限，只能通过`git clone`，`git push`等登录：
   
   ```shell
   # 查看 git-shell 是否在登录方式里面
   cat /etc/shells
   # 查看是否安装
   which git-shell
   vi /etc/shells
   ```
   
   添加上两步显示出来的路径，通常在 `/usr/bin/git-shell`，随后修改`/etc/passwd`中的权限：
   
   ```shell
   # 将原来的
   git:x:1000:1000::/home/git:/bin/bash
   # 修改为
   git:x:1000:1000:,,,:/home/git:/usr/bin/git-shell
   ```

#### 搭建`nginx`服务器

> nginx的安装与配置不再赘述

1. 配置`nginx`文件
   
   ```shell
   # 先启动检查是否安装成功，浏览器查看 server_ip，默认是 80 端口
   nginx
   # 先停止nginx
   nginx -s stop 
   # 可以使用nginx -t查看配置文件位置
   cd /etc/nginx/
   vim nginx.conf
   ```
   
   修改 `root`解析路径，同时将 `user`改为 `root` ，不然`nginx`无法访问`/home/git/projects/hexo-blog`。此外配置location为`/`的部分，自定义监听端口号。修改部分参考如下：
   
   ```roboconf
   user root;
   worker_processes auto;
   error_log /var/log/nginx/error.log;
   pid /run/nginx.pid;
   
   include /usr/share/nginx/modules/*.conf;
   
   events {
       worker_connections 1024;
   }
   
   http {
       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';
   
       access_log  /var/log/nginx/access.log  main;
   
       sendfile            on;
       tcp_nopush          on;
       tcp_nodelay         on;
       keepalive_timeout   65;
       types_hash_max_size 2048;
       client_max_body_size 100m;
   
       include             /etc/nginx/mime.types;
       default_type        application/octet-stream;
   
       include /etc/nginx/conf.d/*.conf;
   
       server {
           listen       800 default_server;
           listen       [::]:800 default_server;
           server_name  localhost;
           root         /usr/share/nginx/html;
           # Load configuration files for the default server block.
           include /etc/nginx/default.d/*.conf;
   
           location / {
               root /home/git/projects/hexo-blog;
               index index.html index.html;
           }
       }
   }
   ```
   
   然后使用`nginx -s reload`重加载`nginx`配置

# 发布

至此我们就把本地和服务器的环境全部搭建完成，现在利用 hexo 配置文件进行链接

1. 配置`_config.yml`文件中的`deploy`：

```yml
deploy:
  type: git
  repo: git@server_ip:/home/git/repos/blog.git
  branch: master
```

2. 在 `package.json` 中添加 npm 脚本
   
   ```json
   "scripts": {
     "deploy": "hexo clean && hexo g -d",
     "start": "hexo clean && hexo g && hexo s"
   },
   ```

3. 这下在本地调试就用`npm start`，调试好了就使用`npm run deploy`部署到服务器。即可通过`server_ip:port/location`进行访问了
   
   如果遇到上传不上的情况，在远端先`rm`删除分支，再创建`_config.yml`中配置的分支即可。
