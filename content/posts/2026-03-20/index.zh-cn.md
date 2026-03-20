---
title: "vscode设置ssh连接服务器配置"
date: 2026-03-20T20:06:35+08:00
lastmod:
draft: false
tags: ["vscode", "ssh"]
categories:
description: ""
---

vscode 连接服务器时，每次都要输密码，很麻烦，设置一下公钥私钥，每次就不用输密码直接连接了。

## 步骤

### 配置本机

新建 `~/.ssh/config` 文件，内容如下
```text
# ========================
# 你的 Linux 服务器
# ========================
Host 46.153.204.xxx
  HostName 46.153.204.xxx
  User root
  IdentityFile ~/.ssh/id_rsa
  Port 22
```

**IdentityFile** 是你的私钥，如果没有，键入以下生成命令, 将生成两个文件：`id_rsa.pub`、`id_rsa.pub` 在 `~/.ssh/` 目录下。
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
```

### 配置服务器
将公钥也就是 `id_rsa.pub` 重的内容写入服务器的 `~/.ssh/authorized_keys` 文件中。
> 注意： 检查下，行首行末不能包含空格

添加相关权限。
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### 测试连接

在本地命令行输入测试，连接成功的话，在 vscode 也会连接成功。
```bash
ssh -T root@43.153.204.151
```