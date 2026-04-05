---
title: "WSL2 代理配置及 GitHub SSH 加速终极指南"
date: 2026-04-05T16:27:24+08:00
lastmod:
draft: flase
tags: ["wsl", "github", "git", "ssh"]
categories:
description: ""
---

> Win10 NAT 模式版

---

## 1. 背景说明

| 项目 | 详情 |
|------|------|
| 环境 | Windows 10 (Version 19044) + WSL2 |
| 网络模式 | NAT（不支持镜像网络模式，Win10 版本限制） |
| 宿主机虚拟网卡 IP | `172.22.176.1`（动态，每次启动可能变化） |
| 代理端口 | `7897`（HTTP 代理，非 SOCKS5） |

**核心限制：**
- WSL2 在 NAT 模式下无法使用 `127.0.0.1` 访问宿主机代理，必须通过宿主机虚拟网卡 IP（`172.x.x.x`）。
- SSH 直连 `github.com:22` 在国内网络下极不稳定，需通过 `ssh.github.com:443` 绕过封锁。
- 宿主机防火墙默认阻断来自 WSL 的入站连接，必须手动放行。


**核心需求**：
- 加速 github clone 速度。
- 加速 apt 安装速度。

---

## 2. 宿主机（Windows）准备工作

### 2.1 代理软件开启局域网访问

在代理软件（Clash for Windows / Clash Verge 等）中开启 **Allow LAN（允许局域网连接）**。

> 此选项使代理从只监听 `127.0.0.1:7897` 变为监听 `0.0.0.0:7897`，WSL 才能通过宿主机 IP 访问。

### 2.2 Windows 防火墙放行代理端口

以**管理员身份**运行 PowerShell，执行：

```powershell
New-NetFirewallRule -DisplayName "Allow WSL Proxy" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 7897
```

成功后输出 `PrimaryStatus: OK`。

### 2.3 验证代理是否可从 WSL 访问

在 WSL 中执行（应返回 `200 Connection established`，而不是挂起）：

```bash
curl -v --proxy http://172.22.176.1:7897 https://www.google.com
```

---

## 3. 安装 connect 工具

`connect` 是实现 SSH-over-HTTP-proxy 的关键工具：

```bash
sudo apt install connect-proxy -y

# 验证安装
which connect   # 应输出 /usr/bin/connect
```

---

## 4. GitHub SSH 专项配置（解决 git clone 失败）

### 4.1 创建代理脚本 `~/.ssh/proxy.sh`

```bash
cat > ~/.ssh/proxy.sh << 'EOF'
#!/bin/bash
# 自动获取宿主机虚拟网卡 IP（WSL NAT 模式下每次启动可能变化）
PROXY_IP=$(grep nameserver /etc/resolv.conf | awk '{print $2}')
exec connect -H "${PROXY_IP}:7897" "$@"
EOF

chmod +x ~/.ssh/proxy.sh
```

### 4.2 配置 `~/.ssh/config`

```bash
cat > ~/.ssh/config << 'EOF'
Host github.com
    HostName ssh.github.com
    User git
    Port 443
    ProxyCommand ~/.ssh/proxy.sh %h %p
    ServerAliveInterval 30
    ServerAliveCountMax 10
EOF
```

> ⚠️ **常见错误**：`HostName` 的值必须是纯主机名 `ssh.github.com`，不能写成 `://ssh.github.com` 或 `https://ssh.github.com`。

> **为什么用 `ssh.github.com:443` 而不是 `github.com:22`？**
> GitHub 在 `ssh.github.com:443` 上提供了与 `github.com:22` 等效的 SSH 服务。HTTP 代理通常只允许隧穿 443 端口的 CONNECT 请求，22 端口会被拒绝，所以必须走 443。

### 4.3 验证 SSH 连接

```bash
ssh -T git@github.com
```

首次连接会提示确认主机指纹，输入 `yes`。

成功输出：
```
Hi leeyorke! You've successfully authenticated, but GitHub does not provide shell access.
```

---

## 5. WSL 终端全局代理（写入 `~/.bashrc`）

将以下内容添加到 `~/.bashrc` 末尾：

```bash
# --- WSL Proxy Settings ---
export HOST_IP=$(grep nameserver /etc/resolv.conf | awk '{print $2}')
export PROXY_PORT=7897
export PROXY_HTTP="http://${HOST_IP}:${PROXY_PORT}"

function proxy_on() {
    export http_proxy="${PROXY_HTTP}"
    export https_proxy="${PROXY_HTTP}"
    export all_proxy="${PROXY_HTTP}"
    alias sudo='sudo -E'
    echo -e "\033[32m[Proxy ON]\033[0m Host: ${HOST_IP}:${PROXY_PORT}"
}

function proxy_off() {
    unset http_proxy https_proxy all_proxy
    unalias sudo 2>/dev/null
    echo -e "\033[31m[Proxy OFF]\033[0m"
}

# 默认开启代理
proxy_on

# 路径增强
export PATH="$PATH:$HOME/.local/bin"
```

> **注意**：`all_proxy` 使用 HTTP 协议而非 SOCKS5。本次排查发现 SOCKS5 在该环境下存在 TLS 握手失败问题（`SSL_ERROR_SYSCALL`），HTTP 代理工作正常。

生效：

```bash
source ~/.bashrc
```

---

## 6. 完整验证清单

```bash
# 1. 确认代理 IP 获取正常
grep nameserver /etc/resolv.conf | awk '{print $2}'

# 2. 测试 HTTP 代理连通性
curl -I --proxy http://172.22.176.1:7897 https://www.google.com
# 期望：HTTP/1.1 200 OK

# 3. 测试 SSH 连接 GitHub
ssh -T git@github.com
# 期望：Hi leeyorke! You've successfully authenticated...

# 4. 测试 git clone
git clone git@github.com:leeyorke/some-repo.git

# 5. 查看当前出口 IP
curl -L ip.gs

# 6. 测试 Sudo 代理，观察是否能正常连接外部镜像源。
sudo apt update
```

---

## 7. 踩坑记录（本次排查过程）

| # | 错误现象 | 根本原因 | 解决方法 |
|---|----------|----------|----------|
| 1 | `ssh -T` 卡在 `Local version string` 后无响应 | 代理端口 7897 只绑定了 `127.0.0.1`，WSL 从 `172.x.x.x` 来的连接被丢弃 | 代理软件开启 Allow LAN |
| 2 | Allow LAN 开启后仍然挂起 | Windows 防火墙阻断了来自 WSL 网段的入站 TCP 7897 | `New-NetFirewallRule` 放行端口 |
| 3 | 改用 SOCKS5 后出现 `SSL_ERROR_SYSCALL` | 该代理环境下 SOCKS5 的 TLS 握手异常 | 改用 HTTP 代理（`-H` 参数） |
| 4 | SSH config 里 `HostName` 写了 `://` 前缀 | SSH config 的 `HostName` 是纯主机名字段，不接受 URL 格式 | 改为 `ssh.github.com` |
| 5 | `ProxyCommand` 中 `$(...)` 子命令不展开 | SSH config 不执行 shell 变量替换 | 抽离为独立的 `proxy.sh` 脚本 |