# claude code配置

需要先安装 `git` 和 `node`。

## Calude Code下载
```bash
npm install -g @anthropic-ai/claude-code
```
[claude-code github仓库文档](https://github.com/anthropics/claude-code)

## Calude Code Router下载

用于国产模型代理。

```bash
npm install -g @musistudio/claude-code-router
```

## Calude Code Router配置

见模板：[claude-code-router配置模板](https://github.com/musistudio/claude-code-router/blob/main/packages/ui/config.example.json)

**方法一**：打开以下文件进行配置
```bash
%USERPROFILE%\.claude-code-router\config.json
```

**方法二**：打开网页配置
```bash
ccr ui
```

配置模型时候尽量选用非thinking模型，thinking模型输出太慢。

## Calude Code Statusline 配置
运行以下命令配置
```bash
npx ccstatusline@latest
```
文档链接：[ccstatusline GitHub 源码](https://github.com/sirmalloc/ccstatusline)

预览:
```powershell
Model: aliyuncs,qwen3-coder-plus | Ctx: 0 | ⎇ feature/ai-dev | (+791,-150)
In: 0 | Out: 0 | Cached: 0 | Session: <1m
```
## .claude 配置
calude code的配置文件
```json
{
  "model": "aliyuncs,qwen3-coder-plus",
  "statusLine": {
    "type": "command",
    "command": "npx ccstatusline@latest"
  }
}

```
## 使用Claude Code写代码

1. 命令行进入项目根目录
2. 执行以下命令，启动路由服务。
```bash
ccr restart
```

初始化一个Claude项目
```bash
# 进入cc
ccr code

# 初始化
/init
```
这将会新建一个Claude.md文件在根目录。用来帮助claude熟悉整个项目。
为了让Claude Code可以用中文输出，需要给文件添加这么一行。
即：`如若没有必要，不要添油加醋。使用中文回复，尽量言简意赅`。
```markdown
## Global Instructions

Unless absolutely necessary, do not add unnecessary details. Please reply in Chinese, concisely and to the point.
```

## 一些常用快捷键

- **shift + tab**：用来替换运行模式。有`default模式` / `edit模式` / `plan模式`。
- **Esc**：返回上一步。

其他命令可以用 `/` 调用。
