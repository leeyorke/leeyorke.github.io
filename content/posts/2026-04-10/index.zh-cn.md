---
title: "mempalace记忆系统学习 - 01 - 环境搭建"
date: 2026-04-10T16:38:58+08:00
lastmod:
draft: true
tags: ["mempalace", "agent", "AI", "记忆系统"]
categories: ["python", "AI"]
description: ""
---

## 概述

### What（是什么）

MemPalace 是一款高效的、开源的本地 AI 记忆与检索系统，旨在把用户与 AI 的所有关键对话、决策、知识、代码、讨论等原文“逐字存储”下来，完全由用户本地控制，无需任何云端或 API 服务。

### Who（谁发明了它？它又为那些群体服务？）

面向所有需要对长期 AI 交互、工作沟通、工程项目决策内容做“永久记忆”并高效检索的用户，包括开发者、团队、组织或个人。适合 AI 重度用户、项目管理者、团队负责人等。

### When（在什么时机会用到它？）

你频繁与 Claude、ChatGPT、Gemini、Copilot 等 AI 工具对话，并希望保留这些对话和其中的决策过程；
需要对历史项目、代码决策、团队知识进行永久留存和追溯；
有隐私/合规需求，不能让对话内容离开本地；
希望 AI 能随时调用、学习过去所有历史决策/文件/上下文。

### Where（在什么场景/地方用它？）

完全本地部署，运行于个人电脑，无需云端或互联网。核心数据采用 ChromaDB 存储，支持主流操作系统。对代码、文档、会议纪要、AI 聊天等多种本地输入数据源兼容。

### Why（为什么需要它, 它为什么存在？它为了解决什么问题）

传统 AI 工具的记忆断裂：每次对话后历史内容易丢失，关键知识无法长期积累；
现有“记忆插件”多为摘要提取，缺失对原始上下文的还原，信息颗粒度不够；
费用高：基于云服务、API 调用存储记忆的方案成本高昂；
安全合规：部分数据敏感，不能外发；
检索体验与上线文联动不佳。

### How（怎么使用它？流程、方法、步骤？）

“古希腊记忆宫殿”结构（wings/rooms/closets/drawers）：按项目/人/主题/文件组织信息，支持多维度强关联检索、可视化导航；
原文逐字存储：所有输入数据（AI 聊天、代码、文档）原文保留至 ChromaDB，支持“语义+关键词+结构化”检索；
一键集成 Claude/ChatGPT/Gemini 等 AI，通过 MCP 协议让 AI 自动检索并引用历史内容；
“AAAK”缩写实验性功能：大规模实体自动编码压缩，降低 token 量（适合本地 LLM/上下文窗口受限场景）；
支持 CLI、Python API、自动存储钩子（与 Claude Code 等终端 AI 工具集成，自动保存重要分界点）；
全部代码、数据、知识图谱本地存储，零云调用、零订阅费，完全自主可控。
一句话核心需求总结（补充）：
MemPalace 解决了“AI 聊天和团队决策内容丢失、检索困难、安全隐患和高额云服务费用”等核心痛点，是一套免费、本地化、结构化、可拓展的“AI 记忆宫殿”系统，让你和 AI 永远不会忘记任何一次对话、决策和知识点。

## 环境搭建

### 安装运行时依赖

进入项目根目录下
```bash
uv sync
```

### 添加 mcp server 到 claude code

```bash
claude mcp add mempalace -- $(which python) -m mempalace.mcp_server
```

虚拟环境中启动 mcp server
```bash
python -m mempalace.mcp_server
```

### 打开claude code测试连接

输入命令查看是否已连接
```bash
/mcp
```

### 初始化 palace 数据

初始化一个项目目录
```bash
mempalace init ~/projects/你的项目
```

挖掘数据
```bash
mempalace mine ~/projects/你的项目           # 代码/文档
mempalace mine ~/chats/ --mode convos       # 对话记录
```

之后我就能帮你：
  - 语义搜索项目记忆
  - 查询知识图谱中的实体关系
  - 按 wing/room 筛选特定领域的记忆
  - 追踪项目决策时间线