---
title: "在 claude code 中为 MiniMax 模型设置 statusline"
date: 2026-04-11T22:19:20+08:00
lastmod:
draft: false
tags: ["claude code", "agent", "AI", "statusline"]
categories: ["claude", "AI"]
description: ""
---

## 安装Jq

```bash
which jq || sudo apt install jq
```

## 编写脚本

路径 `~/.claude/statusline-command.sh`

```bash
#!/usr/bin/env bash
# Claude Code Status Line - Minimax
# exec 2>/dev/null

MINIMAX_API_KEY="${MINIMAX_API_KEY:-}"

input=$(cat)

model=$(echo "$input" | jq -r '.model.display_name // .model.id // ""' 2>/dev/null)
model_lower=$(echo "$model" | tr '[:upper:]' '[:lower:]')
is_minimax=false
[[ "$model_lower" == *"minimax"* || "$model_lower" == *"m2"* || "$model_lower" == *"m*"* ]] && is_minimax=true

five_pct=""
seven_pct=""

if [ "$is_minimax" = true ] && [ -n "$MINIMAX_API_KEY" ]; then
    response=$(curl -s -m 10 \
        -H "Authorization: Bearer ${MINIMAX_API_KEY}" \
        -H 'Content-Type: application/json' \
        'https://www.minimaxi.com/v1/api/openplatform/coding_plan/remains')

    if echo "$response" | jq -e '.model_remains' >/dev/null 2>&1; then
        data=$(echo "$response" | jq -r '
            [.model_remains[] | select(.current_interval_total_count > 0)]
            | (map(select(.model_name | contains("MiniMax-M") or contains("M2"))) | first) // first
            | {total: (.current_interval_total_count // 0),
               remain: (.current_interval_usage_count // 0),
               w_total: (.current_weekly_total_count // 0),
               w_remain: (.current_weekly_usage_count // 0)}
        ')

        total=$(echo "$data" | jq -r '.total')
        remain=$(echo "$data" | jq -r '.remain')
        w_total=$(echo "$data" | jq -r '.w_total')
        w_remain=$(echo "$data" | jq -r '.w_remain')

        if [ "$total" -gt 0 ]; then
            used=$((total - remain))
            five_pct=$(awk -v u="$used" -v t="$total" 'BEGIN { printf "%.0f", (u / t) * 100 }')
        fi
        if [ "$w_total" -gt 0 ]; then
            w_used=$((w_total - w_remain))
            seven_pct=$(awk -v u="$w_used" -v t="$w_total" 'BEGIN { printf "%.0f", (u / t) * 100 }')
        fi
    fi
fi

# ==================== 显示（强制显示计算结果，即使是0也显示） ====================
RESET="\033[0m"
DIM="\033[2m"
GREEN="\033[32m"
BLUE="\033[94m"

short=$(echo "$model" | sed 's/claude-//; s/MiniMax-//')
model_part="${DIM}model:${RESET} ${GREEN}minimax${RESET} ${DIM}(${short})${RESET}"

ctx_pct=$(echo "$input" | jq -r '
  (.context_window.current_usage.input_tokens // 0) as $u |
  (.context_window.context_window_size // 1) as $t |
  if $t > 0 then (($u / $t) * 100 | floor) else 0 end
')

context_part="${DIM}ctx:${RESET} ${GREEN}${ctx_pct}%${RESET}"

# 强制显示（即使是0也显示，不显示 --）
if [ -n "$five_pct" ]; then
    five_part="${DIM}5h:${RESET} ${BLUE}${five_pct}%${RESET}"
else
    five_part="${DIM}5h: --${RESET}"
fi

if [ -n "$seven_pct" ]; then
    seven_part="${DIM}7d:${RESET} ${BLUE}${seven_pct}%${RESET}"
else
    seven_part="${DIM}7d: --${RESET}"
fi

printf "%b | %b | %b | %b\n" "$model_part" "$context_part" "$five_part" "$seven_part"
```

## 配置MINIMAX_API_KEY

```bash
vi ./~bashrc

# 写入MINIMAX_API_KEY
# MINIMAX_API_KEY=sk-api-key

source ./~bashrc
```

## 配置 `~/.claude/settings.json`

加入以下配置
```json
{
    "alwaysThinkingEnabled": true,
    "statusLine": {
        "type": "command",
        "command": "~/.claude/statusline-command.sh"
    }
}
```

## 测试

```bash
$ echo '{"model": {"display_name": "MiniMax-M2.7"}, "context_window": {"current_usage": {"input_tokens": 5000}, "context_window_size": 200000}}' | ~/.claude/statusline-command.sh

model: minimax (M2.7) | ctx: 2% | 5h: 1% | 7d: 9%
```