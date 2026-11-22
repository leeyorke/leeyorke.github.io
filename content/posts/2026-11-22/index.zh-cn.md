---
title: "初次搭建梯子记录"
date: 2026-11-22T11:42:53+08:00
lastmod:
draft: false
tags: ["raksmart", "xray", "梯子"]
categories: ["技术教程"]
description: ""
---

## 01 购买 VPS

购买 [raksmart](https://cn.raksmart.com/) 的最便宜的一档2美元一月，支持支付宝付款。
选择公钥登录。然后配置安全组，记得放行443端口。
重置一下密码。


## 02 安装 xray-core

之前设置后会给你下载一个公钥文件 `id_rsa.pem`，保存到 `~/.ssh/` 下就行，然后配置 ssh 方便连到 VPS 上。
`~/.ssh/config` 配置：

```yaml
# ========================
# raksmart
# ========================
Host rks
    HostName 173.148.27.45
    Port 60321
    User root
    IdentityFile ~/.ssh/id_rsa.pem
```

接着打开 ternimal 输入 `ssh rks` 就能直接登上去。

进入 VPS 的终端，安装 `xray`, 一键安装脚本：
```bash
curl -fsSL https://raw.githubusercontent.com/leeyorke/xray-onexray-reality/refs/heads/main/install.sh | bash
```
安装完成后命令行会打印出一个二维码和 `vless://` 链接。直接用 [v2rayNG](https://github.com/2dust/v2rayNG) 扫描就能用。
如果要在 flclash、clash party 里用, 就需要转换一下订阅，实测直接导入是不能用的，原因是 xray 最新版本升级了 reality 身份验证机制。flclash、clash party 这些客户端可能没有及时支持。

## 03 测试连通性

浏览器打开 [tcp.ping.pe](https://tcp.ping.pe/), 输入 ip 及端口号，如 `173.148.27.45:8443`, 如全部为绿，则证明你的 VPS 无问题。

## FAQ

**connect error: REALITY authentication failed 如何解决?**

可能是 Public Key（公钥）、Short ID、SNI（Server Name）、UUID 以及 Flow（如 xtls-rprx-vision）与服务端不完全一致。
也可能是客户端与服务端版本不兼容导致认证失败：服务端升级 Xray-core 后（例如从 v26.6.27 升级到部分新版本），可能与客户端或 Mihomo 内核出现兼容问题，所以需要 Xray-core 降级。

命令：

```bash
bash -c "$(curl -L https://raw.githubusercontent.com/XTLS/Xray-install/main/install-release.sh)" @ install --version 26.6.27
```

重启 `systemctl restart xray`, 查看版本 `xray version`。

**REALITY localAddr: 173.148.27.45:8443 is using X25519MLKEM768 for TLS' communication: false**

is using X25519MLKEM768 for TLS' communication: false 意味着 Xray 服务端在 REALITY 配置中强制开启了后量子密钥交换（X25519MLKEM768），或者你的节点本身已成功连通到了 Xray 服务端（端口与防火墙没问题），但因为客户端与服务端在 后量子加密（ML-KEM-768） 协商上不一致导致握手被拒。通过 vless:// 链接导入时，FlClash 和 Clash Party 默认往往不会自动添加 support-x25519mlkem768: true，导致 Mihomo 默认以传统 X25519 发起握手，所以 clash 配置改为以下:

```yaml
proxies:
  - name: "你的节点"
    type: vless
    server: 173.148.27.45
    port: 8443
    uuid: 你的UUID
    network: tcp
    tls: true
    udp: true
    flow: xtls-rprx-vision
    servername: 你的SNI域名
    client-fingerprint: chrome
    reality-opts:
      public-key: 你的公钥
      short-id: "你的ShortID"
      support-x25519mlkem768: true
```
完整最简配置模板：[clash-template-from-vless.yaml](https://raw.githubusercontent.com/leeyorke/rules/refs/heads/main/template/clash-template-from-vless.yaml)