---
title: "deepseek harness使用记录"
date: 2026-08-22T10:50:20+08:00
lastmod:
draft: false
tags: ["deepseek", "AI"]
categories: ["AI"]
description: ""
---

本次记录两个对 deepseek harness 的修改：
- 把deepseek harness 3080 端口暴漏到信任网段使之可以多端访问。
- 支持在手机、iPad上也可以选择 workspace。

## CHANGE 1 | 多端访问

### 步骤

```powershell
➜ netsh interface portproxy add v4tov4 listenaddress=192.168.30.71 listenport=3080 connectaddress=127.0.0.1 connectport=3080

➜ netsh advfirewall firewall add rule name="dsh-web-3080-lan" dir=in action=allow protocol=TCP localport=3080 remoteip=192.168.31.0/24
确定。

➜ netsh interface portproxy show v4tov4

侦听 ipv4:                 连接到 ipv4:

地址            端口        地址            端口
--------------- ----------  --------------- ----------
127.0.0.1       8765        172.17.237.150  8765
192.168.30.71   3080        127.0.0.1       3080

# 重启
➜ pnpm dsh web --no-open --trusted-host 192.168.31.70
```

如果要删除规则，则：

```powershell
➜ netsh interface portproxy delete v4tov4 listenaddress=192.168.30.71 listenport=3080

➜ netsh advfirewall firewall delete rule name="dsh-web-3080-lan"
```

### 根因

`crypto.randomUUID()` 是**仅在安全上下文**(HTTPS 或 localhost)下存在的浏览器 API。你从 `http://192.168.30.71:3080` 打开时它不存在，于是：

- 浏览器端**每一个 RPC 请求**在铸造 rpcId 时就抛错(`AbstractApiClient.mintRpcId`)→ 拿不到任何数据，侧边栏空白——这才是之前“空”的真正原因，围栏和转发从头到尾都是好的；
- “添加工作区”只是你第一个撞上它的操作；
- 附件草稿 id 也有一处同样的问题。

### 改动(本地补丁，2 个文件)

- `packages/host/apiproxy/src/fetch/client.ts` — `mintRpcId` 增加 `getRandomValues` 兜底(有原生 `randomUUID` 时仍优先用)，与 connection 包既有的兜底实现同构；
- `packages/client/ui-conversation/src/client/service.ts` — 草稿附件 id 同样处理。

已执行 `pnpm run build:lib` 重建，并验证了**运行中的服务实际下发的 bundle 字节**里已经带上守卫+兜底代码(connection 和 ui-conversation 两条都确认)。服务端每次请求都从磁盘读 bundle 并按内容哈希做缓存击穿，所以**不用重启 dsh,刷新页面即可生效**。

### 你现在要做的

iPad / 手机 / 电脑浏览器上刷新 `http://192.168.30.71:3080`(保险起见可以强制刷新)。这次工作区列表应该能直接列出来，点选 `deepseek-harness-master` 就能看到正在跑的任务；添加新工作区也不会再报红。

### 两点提醒

1. 这是打在你本地 checkout 上的补丁，以后 `git pull` / 重置可能被冲掉。这是个真实的上游产品 bug(HTTP 局域网部署全坏)，值得给 deepseek-harness 提个 PR;
2. 普通 HTTP 下还可能有别的依赖安全上下文的 API(比如剪贴板写入)在某些操作上报错——遇到了把报错发我，同样的套路修。

## CHANGE 2 | 支持在手机、iPad上也可以选择 workspace

> 但我当前使用 `0.1.0-rc.8`， 这种还不够，因为通过 `http://192.168.30.71:3080` 访问，workspace要重新选，添加workspace直接报红：crypto.randomUUID is not a function，无法实现同步。所以就叫 deepseek harness 修复了这个问题。当前默认只能在电脑上选择 workspace 的目录，然后弹出文件资源管理器。

### 步骤

auto 包的注释说了："pinning an interaction remains composing that pair directly instead of this row"——即：**禁用 auto 行，直接插入一对新行**(host 后端 + client 界面)。把 `~\.dsh\profiles\web\cordis.patch.yml` 整个改成：

```yaml
- id: directory-picker
  disabled: true
- insert:
    - id: directory-picker-browse-backend
      name: '@deepseek-ai/dsh-host-directory-picker-browse'
    - id: directory-picker-browse-surface
      name: '@deepseek-ai/dsh-client-ui-directory-picker-browse'
```

用于钉死 browse 目录选择器。

然后重启：

```powershell
pnpm dsh web --no-open --trusted-host 192.168.30.71
```

现在在手机 / iPad 上刷新页面，点“添加工作区”——应该弹出 **网页版目录浏览器** ：默认是从 `~` 路径。可以随意选择。