---
title: ECS + Cloudflare + NAS/客户端 FRP 内网穿透部署文档
date: 2026-07-07 15:30:00
tags:
    - frp
    - Cloudflare
    - 内网穿透
    - NAS
    - tutorial
categories: Network
id: ecs-cloudflare-nas-frp
---

> 本文档记录基于云 ECS（3M 带宽，未备案）搭建的 frp 内网穿透 + Cloudflare Tunnel 架构。因域名未做 ICP 备案，凡是走 HTTP/HTTPS 域名访问的服务，必须通过 Cloudflare Tunnel 中转，否则会被云厂商的备案检测拦截（直接暴露 vhost 端口会返回 Non-compliance ICP Filing 拦截页）。

<!--more-->

> 📌 文中所有公网 IP、域名、隧道名/ID、密码、认证令牌、密钥均已用 `<占位符>` 替换，部署时请替换为你自己的真实值。

## 🔑 可自定义参数总览（所有配置文件中出现的值，统一在这里改）

| 参数 | 当前值 | 说明 |
| --- | --- | --- |
| ECS 公网 IP | `<ECS_PUBLIC_IP>` | 换 ECS 要改这里 |
| 根域名 | `<YOUR_DOMAIN>` | frps 的 subDomainHost |
| frps 主端口(TCP) | `17000` | frpc 连接用，自定义端口，别用默认 7000 |
| vhost HTTP 端口 | `17080` | 域名 HTTP 访问用 |
| vhost HTTPS 端口 | `17443` | 域名 HTTPS 访问用（当前主要靠 Cloudflare Tunnel 转发到 17080，17443 暂未启用） |
| frps Dashboard 端口 | `17500` | 仅供管理查看，安全组只放行自己 IP |
| Dashboard 账号 | `admin` | |
| Dashboard 密码 | `<DASHBOARD_PASSWORD>` | 建议用随机字符串 |
| frp 认证令牌 auth.token | `<AUTH_TOKEN>` | 建议用 `openssl rand -base64 32` 生成 |
| Cloudflare 隧道名 | `<TUNNEL_NAME>` | |
| Cloudflare 隧道 ID | `<TUNNEL_ID>` | 创建隧道后生成 |

---

## 一、ECS 端：安装官方最新版 frps

> 之前用某面板内置的旧版 frp fork，xtcp 打洞协议不完整，已卸载，改用官方最新二进制，版本对齐、协议最新，原生支持 xtcp。

### 1. 下载安装

```bash
cd /opt
sudo wget https://github.com/fatedier/frp/releases/download/v0.69.1/frp_0.69.1_linux_amd64.tar.gz
sudo tar -zxvf frp_0.69.1_linux_amd64.tar.gz
sudo mv frp_0.69.1_linux_amd64 frps_new
cd frps_new
```

### 2. 写 frps.toml

```bash
sudo tee /opt/frps_new/frps.toml > /dev/null << 'EOF'
bindPort = 17000

auth.method = "token"
auth.token = '<AUTH_TOKEN>'

transport.tcpMux = true
transport.heartbeatTimeout = 300

subDomainHost = "<YOUR_DOMAIN>"
vhostHTTPPort = 17080
vhostHTTPSPort = 17443

webServer.addr = "0.0.0.0"
webServer.port = 17500
webServer.user = "admin"
webServer.password = "<DASHBOARD_PASSWORD>"

log.to = "/opt/frps_new/frps.log"
log.level = "info"
log.maxDays = 7
EOF
```

> ⚠️ 注意：新版官方 frp **没有**类似老版本 `bind_udp_port` 的单独 UDP 打洞端口配置字段。xtcp 打洞在新版 frps 里是默认内置支持的，不需要额外绑定 UDP 端口，直接用上面这份标准配置即可。

先校验配置文件语法（强烈建议每次改完都跑一下，能在启动前就发现字段名拼写错误）：

```bash
/opt/frps_new/frps verify -c /opt/frps_new/frps.toml
```

### 3. systemd 常驻

```bash
sudo tee /etc/systemd/system/frps-new.service > /dev/null << 'EOF'
[Unit]
Description=frp server (official latest)
After=network.target

[Service]
Type=simple
ExecStart=/opt/frps_new/frps -c /opt/frps_new/frps.toml
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable frps-new
sudo systemctl start frps-new
sudo systemctl status frps-new
```

### 4. 安全组（云 ECS 控制台 → 安全组 → 入方向规则）

| 端口 | 协议 | 授权对象 | 用途 |
| --- | --- | --- | --- |
| 17000 | TCP | 0.0.0.0/0 | frpc 连接 |
| 17080 | TCP | 0.0.0.0/0 | vhost HTTP |
| 17443 | TCP | 0.0.0.0/0 | vhost HTTPS（预留） |
| 17500 | TCP | 仅自己公网 IP | Dashboard，不对外网开放 |

> xtcp 打洞不需要额外开 UDP 端口（新版本已确认不需要单独绑定）。

### 5. 验证

```bash
sudo ss -tlnp | grep -E '17000|17080|17443|17500'
```

浏览器打开 `http://<ECS_PUBLIC_IP>:17500`，账号密码登录，能看到 Dashboard 即成功。

---

## 二、ECS 端：Cloudflare Tunnel（解决未备案域名被拦截问题）

### 背景

国内云服务商会在网络层检测 HTTP 请求的 Host 头（或 HTTPS 的 SNI），域名未备案直接返回拦截页，**且这个检测不区分端口**，任何直接暴露给公网的 vhost 端口都会被拦。Cloudflare Tunnel 让 ECS **主动连出去**，不再有公网直接入站的 HTTP 请求，从而绕开这层检测。

> 注意：这解决的是云厂商自动检测层面的技术限制，不代表法规意义上不需要备案，自行评估使用范围和风险。

### 1. 安装 cloudflared（Ubuntu 24.04）

```bash
cd /opt
sudo wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo apt install ./cloudflared-linux-amd64.deb
cloudflared --version
```

### 2. 登录授权

```bash
cloudflared tunnel login
```

复制输出的链接，在**自己电脑**的浏览器打开，登录 Cloudflare 账号，选择 `<YOUR_DOMAIN>` 完成授权。

### 3. 创建隧道

```bash
cloudflared tunnel create <TUNNEL_NAME>
```

记下输出的隧道 ID。如果忘记记录，后续可用以下命令查询：

```bash
cloudflared tunnel list
# 或
cloudflared tunnel info <TUNNEL_NAME>
```

### 4. 写隧道配置文件

```bash
sudo mkdir -p /etc/cloudflared

sudo tee /etc/cloudflared/config.yml > /dev/null << 'EOF'
tunnel: <TUNNEL_ID>
credentials-file: /root/.cloudflared/<TUNNEL_ID>.json

ingress:
  - hostname: app.<YOUR_DOMAIN>
    service: http://localhost:17080
  - hostname: nas-fs.<YOUR_DOMAIN>
    service: http://localhost:17080
  - service: http_status:404
EOF
```

> 💡 **每新增一个走域名访问的服务，就在 `ingress` 列表里加一条 `hostname` 规则**（放在最后的 `http_status:404` 兜底规则之上）。多个域名可以共用同一个 `service: http://localhost:17080`，因为 frps 是靠请求的 Host 头做子域名路由分发的，不需要给每个域名单独开端口。
>
> ⚠️ 不要在 ingress 规则里加 `originRequest.httpHostHeader` 硬编码某个固定域名 —— 一旦硬编码，所有请求的 Host 头都会被强制改写成那一个域名，导致多域名互相冲突。默认不写这项，cloudflared 会原样传递用户实际访问的域名，vhost 路由才能正常按子域名分发。

### 5. 添加 DNS 路由（每个域名都要单独加一次）

```bash
cloudflared tunnel route dns <TUNNEL_NAME> app.<YOUR_DOMAIN>
cloudflared tunnel route dns <TUNNEL_NAME> nas-fs.<YOUR_DOMAIN>
```

> 如果某个域名之前手动加过 Cloudflare 的 A 记录，执行这条命令前先去 Cloudflare 后台把那条 A 记录删掉，避免和自动生成的 CNAME 记录冲突。

### 6. 注册为系统服务（必须做，否则隧道只在手动运行时才有效）

```bash
sudo cloudflared service install
sudo systemctl daemon-reload
sudo systemctl start cloudflared
sudo systemctl enable cloudflared
sudo systemctl status cloudflared
```

> ⚠️ **踩坑记录**：曾经只手动执行过 `cloudflared tunnel run`，忘记执行 `service install`，导致关闭终端后隧道断开，访问报 **Error 1033**（Cloudflare 边缘找不到任何活跃连接）。务必确认 `systemctl status cloudflared` 显示 `active (running)`。

### 7. 验证

```bash
sudo journalctl -u cloudflared -f
```

浏览器访问对应域名，能打通说明隧道生效。

---

## 三、NAS（QNAP，x86_64，有 Container Station）端配置

### 场景一：域名访问，不需要客户端（如 File Station）

#### 1. 建目录、写配置文件

```bash
mkdir -p /share/Web/frp
cat > /share/Web/frp/frpc.toml << 'EOF'
serverAddr = "<ECS_PUBLIC_IP>"
serverPort = 17000

auth.method = "token"
auth.token = '<AUTH_TOKEN>'

transport.tcpMux = true

[[proxies]]
name = "nas-filestation-http"
type = "http"
localIP = "127.0.0.1"
localPort = 5000
subdomain = "nas-fs"
EOF
```

> `localIP = "127.0.0.1"` 的前提是容器用 **host 网络模式**，容器内的 127.0.0.1 就是 NAS 本机。

#### 2. Docker 运行（host 网络，常驻）

```bash
docker run -d \
  --name frpc-nas \
  --network host \
  --restart always \
  -v /share/Web/frp/frpc.toml:/etc/frp/frpc.toml \
  fatedier/frpc:v0.69.1 \
  -c /etc/frp/frpc.toml
```

#### 3. 验证

```bash
docker ps | grep frpc-nas
docker logs -f frpc-nas
```

看到 `login to server success` 和 `[nas-filestation-http] start proxy success` 即成功。然后别忘了回到 ECS，在 Cloudflare Tunnel 的 `config.yml` 里加上这个新域名的 `ingress` 规则 + 执行 `route dns`（见第二章第 4、5 步）。

---

### 场景二：同一个端口，同时支持 STCP 和 XTCP（自动优先直连，失败兜底中转）

适用于：追求低延迟直连（xtcp），但网络环境不确定能否打洞成功，需要自动兜底（stcp）的服务。

#### 原理

frp 里 xtcp 和 stcp 是两个完全独立的 proxy 类型，**没有自动退化机制** —— 除非显式使用新版本的 `fallbackTo` 字段。做法是：同一个 `localPort`，在 frpc 配置里注册**两条独立的 proxy**（一条 xtcp、一条 stcp），共用同一个 `secretKey`；访问端(visitor)也配两条 visitor，并在 xtcp 的 visitor 里通过 `fallbackTo` 指向 stcp 的 visitor，设置超时时间，打洞失败自动切换。

#### NAS 端（frpc.toml，以某个自建 API 服务为例，端口自定义为 `<NAS_API_PORT>`）

```toml
serverAddr = "<ECS_PUBLIC_IP>"
serverPort = 17000

auth.method = "token"
auth.token = '<AUTH_TOKEN>'

transport.tcpMux = true

[[proxies]]
name = "nas-api-xtcp"
type = "xtcp"
localIP = "127.0.0.1"
localPort = <NAS_API_PORT>
secretKey = "<SECRET_KEY，建议用 openssl rand -hex 16 生成>"

[[proxies]]
name = "nas-api-stcp"
type = "stcp"
localIP = "127.0.0.1"
localPort = <NAS_API_PORT>
secretKey = "<与上面 xtcp 那条完全一致的密钥>"
```

> 两条 proxy 的 `secretKey` **必须一致**，这样客户端只需要记一个密钥即可访问两种模式。`name` 必须不同（一个 `-xtcp` 结尾、一个 `-stcp` 结尾，便于区分）。

#### 客户端（Windows/其他访问方，frpc.toml）

```toml
serverAddr = "<ECS_PUBLIC_IP>"
serverPort = 17000

auth.method = "token"
auth.token = '<AUTH_TOKEN>'

transport.tcpMux = true

[[visitors]]
name = "nas-api-stcp-visitor"
type = "stcp"
serverName = "nas-api-stcp"
secretKey = "<与NAS端一致的密钥>"
bindAddr = "127.0.0.1"
bindPort = <本地占位端口，如 6071>

[[visitors]]
name = "nas-api-xtcp-visitor"
type = "xtcp"
serverName = "nas-api-xtcp"
secretKey = "<与NAS端一致的密钥>"
bindAddr = "127.0.0.1"
bindPort = <本地实际访问端口，如 6070>
fallbackTo = "nas-api-stcp-visitor"
fallbackTimeoutMs = 3000
```

**使用方式**：访问方始终用 `http://127.0.0.1:<xtcp的bindPort>`（如 `6070`）。如果打洞在 `fallbackTimeoutMs` 时间内成功，走真正直连，不占用 ECS 带宽；如果失败，自动切到 stcp 中转，同一个地址无缝兜底，使用体感上无区别。`<stcp的bindPort>`（如 `6071`）是内部占位，不需要手动访问。

> ⚠️ `fallbackTo` 填的是**同配置文件里另一个 visitor 的 name**，不是 NAS 端 proxy 的 name。

---

## 四、客户端（Windows）配置与常驻

### 1. frpc.toml 示例（以某主机的 8787 服务 stcp 访问为例）

```toml
serverAddr = "<ECS_PUBLIC_IP>"
serverPort = 17000

auth.method = "token"
auth.token = '<AUTH_TOKEN>'

transport.tcpMux = true

[[visitors]]
name = "host-8787-visitor"
type = "stcp"
serverName = "host-8787-stcp"
secretKey = "<SECRET_KEY>"
bindAddr = "127.0.0.1"
bindPort = 6060
```

访问方式：`http://127.0.0.1:6060`

### 2. 简易启动方式：批处理脚本（适合个人电脑，登录后手动/自动跑一次）

新建 `start-frpc.bat`（路径按实际安装位置调整）：

```shell
@echo off
cd /d D:\dev-tools\frp_0.69.1_windows_amd64
start /min frpc.exe -c frpc.toml
```

开机自动运行（需要用户登录桌面后才会触发）：

1. `Win + R` 输入 `shell:startup`（或手动进入 `%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup`）
2. 把 `start-frpc.bat` 的**快捷方式**拖进这个文件夹

配套的停止脚本 `stop-frpc.bat`：

```shell
@echo off
taskkill /IM frpc.exe /F
```

### 3. 更可靠的方式：注册为 Windows 系统服务（NSSM，不依赖用户登录，系统启动即运行）

```powershell
# 下载 nssm(如果没有)
Invoke-WebRequest -Uri "https://nssm.cc/release/nssm-2.24.zip" -OutFile "nssm.zip"
Expand-Archive -Path "nssm.zip" -DestinationPath "."
copy .\nssm-2.24\win64\nssm.exe .\nssm.exe

# 注册服务
.\nssm.exe install FrpcClient
# 弹出界面填:
# Path: <frpc.exe完整路径>
# Startup directory: <frpc所在目录>
# Arguments: -c <frpc.toml完整路径>
# Details标签: Startup type 选 Automatic

.\nssm.exe start FrpcClient
```

验证开机自启：重启电脑，不手动操作，直接访问对应的 `127.0.0.1:端口`，能打开即生效。

---

## 五、已知限制与注意事项

1. **带宽瓶颈**：ECS 是 3M 带宽，凡是走 http/vhost、stcp 这类需要中转的模式，流量都要经过 ECS，受这 3M 限制（理论峰值约 375KB/s）。纯文字类 API 请求完全不受影响，但大文件传输（如 NAS File Station 下载大文件）会明显受限。真正能绕开带宽限制的只有 **xtcp 打洞成功后的直连**。
2. **xtcp 不保证成功**：如果任一端处于对称型 NAT（常见于部分公司网络、移动网络），打洞会失败，此时必须靠 `fallbackTo` 兜底 stcp，或者本身就该用 stcp。
3. **认证令牌含特殊字符**：`auth.token` 若含反引号、竖线等符号，写入配置文件时，务必用 `sudo tee ... << 'EOF' ... EOF` 这种整体写入的方式，避免用 `nano` 手动输入或简单 echo 导致换行错乱（曾经因此触发 `toml: literal strings cannot have new lines` 报错）。
4. **未备案域名的 HTTP 访问，必须走 Cloudflare Tunnel**，不能直接把 vhost 端口暴露在安全组里给公网直连，否则会被云厂商拦截返回 ICP 备案提示页。
5. **frps 配置字段名以官方最新 TOML 文档为准**，不要套用老版本 ini 格式的字段名（如 `bind_udp_port` 在新版本已不存在，新版本的 xtcp 打洞不需要单独绑定 UDP 端口）。改配置后建议先跑 `frps verify -c frps.toml` 校验语法再重启服务。
