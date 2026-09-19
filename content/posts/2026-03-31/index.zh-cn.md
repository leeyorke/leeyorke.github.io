---
title: "github ssh clone 慢的解决方案"
date: 2026-03-31T22:32:11+08:00
lastmod:
draft: false
tags: ["github", "ssh"]
categories:
description: ""
---

以下记录在 windows 上配置过程。

## 01 配置 ~/.ssh/config文件

```bash
# ========================
# 2. GitHub（通过本地 SOCKS5 代理）
# ========================
Host github.com
  HostName github.com
  User git
  Port 22
  IdentityFile ~/.ssh/id_ed25519
  ProxyCommand connect -H 127.0.0.1:7897 %h %p
```

## 开启终端代理
> 不确定要不要配置，目前是开着，但是是 HTTP/HTTPS 的代理。

```powershell
$env:HTTP_PROXY="http://127.0.0.1:7897"
$env:HTTPS_PROXY="http://127.0.0.1:7897"
```

## clash加规则

> 右键订阅 > 编辑规则 > 添加前置规则 > 保存

```text
"AND,(DOMAIN-SUFFIX,github.com),(DST-PORT,22),节点选择"
```

----

# 2026-09-03

上面的方法好像有点问题，改为以下方式：

### 方法一：使用 `connect.exe`（最常用）

1. 下载 `connect.exe`（推荐从这里获取：
   https://github.com/gotoh/connect 或搜索 “connect.exe windows”）

2. 把 `connect.exe` 放到系统 PATH 里（比如放到 `C:\Windows` 或 Git 的 `bin` 目录下）。

3. 修改 `C:\Users\你的用户名\.ssh\config` 为：

```sshconfig
Host github.com
  HostName ssh.github.com
  User git
  Port 443
  IdentityFile ~/.ssh/id_ed25519
  ServerAliveInterval 10
  ServerAliveCountMax 10
  ProxyCommand connect -S 127.0.0.1:7897 %h %p
```

**小提示：**
- Windows 的 SSH 配置文件路径是：`C:\Users\你的用户名\.ssh\config`
- 如果用的是 **Git for Windows**，也可以尝试把 `connect.exe` 放到 `C:\Program Files\Git\mingw64\bin\` 目录下。
- 确保你的代理软件（Clash / V2RayN 等）已经开启，并且 **允许局域网连接**（Allow LAN）。

### 方法二：使用 Nmap 的 `ncat`（更推荐，功能更完整）

> 本次使用该方式

1. 安装 [Nmap](https://nmap.org/download.html)（安装时勾选 Ncat）。
2. 配置写成：

```sshconfig
Host github.com
  HostName ssh.github.com
  User git
  Port 443
  IdentityFile ~/.ssh/id_ed25519
  ServerAliveInterval 10
  ServerAliveCountMax 10
  ProxyCommand "C:\Program Files (x86)\Nmap\ncat.exe" --proxy-type socks5 --proxy 127.0.0.1:7897 %h %p
```

**路径根据实际安装位置调整**

### 测试

打开 PowerShell 或 CMD，执行：

```bash
ssh -T git@github.com
```

如果看到 `Hi xxx! You've successfully authenticated...` 就说明代理生效了。


### Nmap 的安装界面组件说明

| 组件 | 含义 | 是否需要勾选 |
|------|------|--------------|
| **Nmap Core Files** | Nmap 的核心程序（扫描工具本身） | **必须勾选** |
| **Register Nmap Path** | 把 Nmap 相关命令加入系统 PATH，方便在命令行直接使用 | **强烈建议勾选** |
| **Npcap 1.88** | Windows 上抓包驱动，Nmap 扫描时需要用到 | 建议勾选（装了也没事） |
| **Check online for newer versions** | 安装时检查是否有新版本 | 可不勾选 |
| **Network Performance** | 网络性能相关组件 | 可勾选 |
| **Ncat (Modern Netcat)** | **现代版 netcat**，这就是我们要用的 `ncat` 命令 | **必须勾选**（最重要） |
| **Nping (Packet generator)** | 发包测试工具 | 可选 |
| **Zenmap (GUI Frontend)** | Nmap 的图形界面 | 可选（不需要可以取消） |

---

### 针对SSH 走代理

只需要用到 **Ncat**，所以最少只需要勾选这几个：

- ✅ **Nmap Core Files**
- ✅ **Register Nmap Path**（很重要，勾了之后才能直接在命令行用 `ncat`）
- ✅ **Ncat (Modern Netcat)** ← 这个是核心

其他的（Npcap、Nping、Zenmap）都可以不勾，能省一点空间和时间。