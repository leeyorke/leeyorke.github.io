---
title: "nanobot配置obsidian mcp server"
date: 2026-06-23T20:58:17+08:00
lastmod:
draft: false
tags: ["nanobot", "mcp", "obsidian", "AI", "wsl"]
categories: ["AI"]
description: "nanobot配置obsidian mcp server时遇到错误排查解决方案"
---

## 背景

推上刷到 [How to Build an AI Second Brain With Claude and Obsidian That Gets Smarter Every Day (Full Guide)](https://x.com/undefinedKi/status/2068306794116501544)，直接进行实践。不幸遇到了错误。

### 环境

- nanobot 运行在 **wsl2** 里，并配置代理为 `172.17.121.1:7897`。
- obsidian 运行在宿主机上。
- 宿主机通过 **Clash Party** （使能局域网）提供 `127.0.0.1:7897` 网络代理，给 wsl2。


### 步骤

1. **安装 [Local REST API with MCP](https://github.com/coddingtonbear/obsidian-local-rest-api)**：同时开启 `https://127.0.0.1:27123`(要安装证书) 和 `http://127.0.0.1:7897:27123`服务(不需要证书但可能不安全)。由于https较繁琐，后面只配置http。
2. **[配置nanobot mcp server](https://nanobot.wiki/cn/docs/0.1.5/use-nanobot/tools-and-mcp#mcp-servers)** 如下：
    ```json
    "mcpServers": {
      "obsidian": {
        "toolTimeout": 120,
        "url": "https://172.17.121.1:27123/mcp/",
        "headers": {
        "Authorization": "Bearer $TOKEN"
        }
      }
    }
    ```
3. 然后启动 `nanobot gateway --verbose`, 报错如下：
    ```
    2026-06-23 17:51:37 | WARNING | - | MCP server 'obsidian': blocked unsafe URL https://172.17.121.1:27123/mcp/ (Blocked: 172.17.121.1 resolves to private/internal address 172.17.121.1)
    2026-06-23 17:51:37 | WARNING | - | No MCP servers connected successfully (will retry next message)
    ```

## 问题原因

排查了一下，如果要实现wsl访问宿主机的 Obsidian MCP Server，需要做这些修改：

### 改源码

Nanobot 代码层不允许私有、内网地址连接mcp。所以需要对源码进行更改。

```python
# nanobot/security/network.py:line 100

if _is_private(addr):
    # ip 加入白名单
    +++ if str(addr) in ("172.17.121.1",):
        +++ continue
    return False, f"Blocked: {hostname} resolves to private/internal address {addr}"

# nanobot/agent/tools/mcp.py:line 96

except (OSError, asyncio.TimeoutError):
        # 捕获异常，便于调试
        +++ import traceback
        +++ traceback.print_exc()
        return False
```

### 宿主机监听172.17.121.1

Obsidian Local REST API 服务默认只绑 127.0.0.1（Obsidian MCP 默认配置）。所以要用 netsh portproxy 转发到 `172.17.121.1`

配置步骤：
```powershell
➜ netsh interface portproxy add v4tov4 listenport=27123 listenaddress=172.17.121.1 connectport=27123 connectaddress=127.0.0.1

➜ netstat -ano | findstr 27123
  TCP    127.0.0.1:27123        0.0.0.0:0              LISTENING       17388
  TCP    172.17.121.1:27123     0.0.0.0:0              LISTENING       2704
```

>在 WSL（Windows Subsystem for Linux）中，宿主机（Windows）的 127.0.0.1 和 WSL 内部的 127.0.0.1 是完全隔离的。这意味着你在 WSL 里直接访问 127.0.0.1:27123 是找不到宿主机服务的。要成功访问，需要根据你的 WSL 版本（WSL 2 或 WSL 1）来采取不同的方法。宿主机上运行在 27123 端口的服务必须允许外部连接。
>如果该服务只监听了 Windows 的 127.0.0.1，WSL 2 是无法跨虚拟网络访问它的。
>解决办法：将 Windows 服务的监听地址改为 0.0.0.0（允许所有网络接口连接）。
>WSL 2 运行在独立的虚拟机中，Windows 宿主机对于 WSL 2 来说就像是局域网中的另一台电脑。WSL 2 会将宿主机的 IP 自动写入 /etc/resolv.conf 中。
>通过以下命令动态获取宿主机的 IP 并进行访问：`curl $(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):27123`, 得到地址为172.17.121.1

### 宿主机防火墙放行27123端口

Windows 防火墙丢包（入站规则缺失）→ 补了 New-NetFirewallRule 放行 27123。

```powershell
➜ New-NetFirewallRule -DisplayName "WSL2 to Host MCP Port 27123" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 27123

Name                          : {57428229-e13a-4ebc-8815-228eeb203ccd}
DisplayName                   : WSL2 to Host MCP Port 27123
Description                   :
DisplayGroup                  :
Group                         :
Enabled                       : True
Profile                       : Any
Platform                      : {}
Direction                     : Inbound
Action                        : Allow
EdgeTraversalPolicy           : Block
LooseSourceMapping            : False
LocalOnlyMapping              : False
Owner                         :
PrimaryStatus                 : OK
Status                        : 已从存储区成功分析规则。 (65536)
EnforcementStatus             : NotApplicable
PolicyStoreSource             : PersistentStore
PolicyStoreSourceType         : Local
RemoteDynamicKeywordAddresses :
```

## 总结

nanobot要在 wsl 内访问宿主机的 Obsidian MCP Server，需要进行三步：
1. 修改源码将 `ip` 加入白名单。
2. 宿主机需监听 `172.17.121.1`。
3. 宿主机防火墙需放行 `27123` 端口。

最后，运行 `nanobot gateway --verbose`，大功告成！然后在手机上就可以通过 `MCP` 操作 `Obsidian` 第二大脑了。

```bash
(nanobot) ~/workspace/nanobot$ nanobot gateway --verbose
🐈 Starting nanobot gateway version 0.1.4.post6 on port 18790...
2026-06-23 21:51:12 | INFO  | - | Runtime model switched for next turn: gemma-4-26b-a4b-it -> gemma-4-26b-a4b-it
2026-06-23 21:51:12 | INFO  | - | Registered 19 tools: ['apply_patch', 'run_cli_app', 'complete_goal', 'cron', 'edit_file', 'exec', 'find_files', 'grep', 'list_dir', 'list_exec_sessions', 'long_task', 'message', 'read_file', 'spawn', 'web_fetch', 'web_search', 'write_file', 'write_stdin', 'my']
2026-06-23 21:51:13 | INFO  | - | Discord channel enabled
2026-06-23 21:51:13 | INFO  | - | WebSocket channel enabled
2026-06-23 21:51:13 | INFO  | - | WeChat channel enabled
✓ Channels enabled: discord, websocket, weixin
✓ Cron: 2 scheduled jobs
✓ Heartbeat: every 1800s
2026-06-23 21:51:13 | INFO  | - | Cron: registered system job 'dream' (dream)
✓ Dream: every 2h
2026-06-23 21:51:13 | INFO  | - | Cron: registered system job 'heartbeat' (heartbeat)
2026-06-23 21:51:13 | INFO  | - | Cron service started with 2 jobs
2026-06-23 21:51:13 | INFO  | - | Starting discord channel...
2026-06-23 21:51:13 | INFO  | - | Starting websocket channel...
2026-06-23 21:51:13 | INFO  | - | Starting weixin channel...
2026-06-23 21:51:13 | INFO  | - | Outbound dispatcher started
2026-06-23 21:51:13 | INFO  | discord | Starting client via discord.py...
2026-06-23 21:51:13 | INFO  | websocket | WebSocket server listening on ws://127.0.0.1:8765/
2026-06-23 21:51:13 | INFO  | weixin | channel starting with long-poll...
✓ Health endpoint: http://0.0.0.0:18790/health
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_vault_list' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_vault_read' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_vault_write' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_vault_append' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_vault_patch' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_vault_delete' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_vault_move' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_vault_get_document_map' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_active_file_get_path' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_periodic_note_get_path' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_search_query' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_search_simple' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_tag_list' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_command_list' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_command_execute' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered tool 'mcp_obsidian_open_file' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP: registered resource 'mcp_obsidian_resource_openapi-spec' from server 'obsidian'
2026-06-23 21:51:13 | DEBUG | - | MCP server 'obsidian': prompts not supported or failed: Method not found
2026-06-23 21:51:13 | INFO  | - | MCP server 'obsidian': connected, 17 capabilities registered
2026-06-23 21:51:13 | INFO  | - | MCP connected servers: ['obsidian']
2026-06-23 21:51:13 | INFO  | - | Agent loop started
```