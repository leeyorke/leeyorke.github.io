# github ssh clone 慢的解决方案


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
