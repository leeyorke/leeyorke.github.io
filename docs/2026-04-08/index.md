# claude相关配置问题


## claude 命令无法使用

```powershell
➜ claude -c
claude: The term 'claude' is not recognized as a name of a cmdlet, function, script file, or executable program.
Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
```
`npm list -g` 也不见 @anthropic-ai/claude-code了。

问题原因：在wsl上装用原生命令 `curl -fsSL https://claude.ai/install.sh | bash` 装claude code, 会把本地通过npm等其他方式安装的给删掉。
解决方式：
- 先删除原生安装：`claude uninstall`, `rm -f $(which claude)`
- 确保删除干净：`claude --version`
- 禁止 将 Windows 的环境变量 PATH 自动添加到 Linux 的 PATH 中。`vi /etc/wsl.conf`。这样wsl不会扫描到window上的nvm和npm。
  ```bash
  [interop]
  appendWindowsPath=false
  ```
- 在wsl上安装nvm，依次下面的命令。
  ```bash
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

  export NVM_DIR="$HOME/.nvm"
  [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
  [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"

  nvm -v

  nvm install --lts
  node -v
  npm -v
  ```
- 安装claude code，不要从官方脚本/apt安装，感觉有很多坑。
  ```bash
  # 安装最新版
  npm install -g @anthropic-ai/claude-code

  # 安装指定版本
  npm install -g @anthropic-ai/claude-code@2.1.91
  ```
- 配置 `~/.claude/backups/.claude.json.backup.1775639744028`，解决登录的问题，不然一直让你登录，无法使用第三方订阅。
  ```bash
  # 在顶层配置加入
  "hasCompletedOnboarding": true,
  ```

- 最后重新在powershell安装本地的claude code
  ```powershell
  npm install -g @anthropic-ai/claude-code
  ```

这样就可以同时在Linux、Windows上使用claude code了。

## claude code权限问题

没有在wsl上安装claude code前，我是进到wsl路径下，再使用本地claude code写代码的，但是这有个问题
操作 \wsl.localhost\Ubuntu-20.04\home这样的目录，每一次编辑都要权限，烦得要死。

解决方式，只能再wsl安装新的claude code。

## 如何在Ubuntu-20.04实现一键切换供应商

cc-swtich 是GUI软件，最低支持Ubuntu-22.04，我是Ubuntu-20.04，用不了这个软件，要用只能升级为Ubuntu-22.04.
很麻烦，所以直接写个脚本通过修改软链接的方式来切换 `~/.claude/settings.json` 里的环境变量就行。

第一步创建脚本， `vi ~/cc-swtich.sh`
```bash
#!/bin/bash
DIR="$HOME/.claude"

case $1 in
    m|minimax)
        ln -sf "$DIR/settings.minimax.json" "$DIR/settings.json"
        echo "🚀 已切换到 MiniMax 模式"
        ;;
    v|volces)
        ln -sf "$DIR/settings.volces.json" "$DIR/settings.json"
        echo "🎨 已切换到 volces 模式"
        ;;
    *)
        echo "用法: cc m (切换到 MiniMax) 或 cc v (切换到volces)"
        ;;
esac
```

第二步，增加权限

```bash
# 增加权限
chmod +x ~/cc-swtich.sh`
```

第三步，增加快捷命令

`vi .bashrc` 写入以下：
```text
alias cc='~/cc-switch.sh'
```
然后 刷新配置 `source ~/.bashrc`。


## Claude Code 退出后屏幕有“残留”现象

这个在windows上没发现，再Linux上好像有。

解决方式：在 `settingsd.json`中加入：`"CLAUDE_CODE_NO_FLICKER": "1"`
