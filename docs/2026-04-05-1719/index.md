# nanobot学习 - 01 - 开发环境安装


🐈 **nanobot** 是一款受 [OpenClaw](https://github.com/openclaw/openclaw) 启发的**超轻量级**个人AI助手。

---
**个人PC环境** : win10、wsl2

wsl2 的环境配置见 [WSL2 代理配置及 GitHub SSH 加速终极指南](../2026-04-05/)

## **01 克隆仓库**

- fork 仓库：[HKUDS/nanobot](https://github.com/HKUDS/nanobot)
- 克隆仓库到本地：[leeyorke/nanobot](https://github.com/leeyorke/nanobot)

## **02 安装UV**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh

# uv 0.11.3 (x86_64-unknown-linux-gnu)
```

## **03 设置 python**

> 要求 >= 3.11

```bash
# 安装
uv python install 3.11

# 固定 python版本为3.11
uv python pin 3.11

# 创建虚拟环境
uv venv
# Using CPython 3.11.15

# 激活虚拟环境
source .venv/bin/activate

# 安装依赖
uv sync
```

## **04 运行 nanobot**

```bash
nanobot onboard
```
输出
```text

Next steps:
  1. Add your API key to /home/leeyorke/.nanobot/config.json
     Get one at: https://openrouter.ai/keys
  2. Chat: nanobot agent -m "Hello!" --config /home/leeyorke/.nanobot/config.json

Want Telegram/WhatsApp? See: https://github.com/HKUDS/nanobot#-chat-apps
```

配置 .nanobot/config.json
```text
"defaults": {
      "workspace": "~/.nanobot/workspace",
      "model": "MiniMax-M2.7",
      "provider": "minimax",
}
```

提供商配置如下，尽量使用openAI兼容端点
```text
"minimax": {
      "apiKey": "sk-cp-apikey",
      "apiBase": "https://api.minimaxi.com/v1",
      "extraHeaders": null
    },
```

然后再次运行, 成功响应
```bash
(nanobot) ~/workspace/nanobot$ nanobot agent -m "Hello!" --config /home/leeyorke/.nanobot/config.json
Using config: /home/leeyorke/.nanobot/config.json

🐈 nanobot
Hello! 👋 How can I help you today?
```

## **05 配置 channel**

这里设置微信，将config.json weixin的enabled改为 true,

然后开始运行：`nanobot gateway`

打印如下

```bash
(nanobot) leeyorke@leeyorke:~/workspace/nanobot$ nanobot channels login weixin
2026-04-05 18:16:26.749 | DEBUG    | nanobot.channels.registry:discover_all:64 - Skipping built-in channel 'matrix': Matrix dependencies not installed. Run: pip install nanobot-ai[matrix]
🐈 WeChat Login


Login URL: https://liteapp.weixin.qq.com/q/7GiQu1?qrcode=1b77caaa0ccecef0aa76c8fdee5d5a19&bot_type=3

2026-04-05 18:16:57.633 | INFO     | nanobot.channels.weixin:_qr_login:370 - WeChat login successful! bot_id=ac3@im.bot user_id=o9cq804n-VAucgObc@im.wechat
```
点击链接进入网页, 用微信扫码。会添加一个机器人到微信。
然后设置 allowFrom: ["o9cq804n-VAuc@im.wechat"] 或 ["*"]，重新运行：`nanobot gateway` 即可正常对话。


## FAQ
如果打印权限不足：
```text
Access denied for sender o9cq804n-VAuc@im.wechat on channel weixin. Add them to allowFrom list in config to grant access.
```
CTRL + C退出，然后设置 allowFrom: ["o9cq804n-VAuc@im.wechat"] 或 ["*"]。
然后重新运行：`nanobot gateway` 即可正常。
