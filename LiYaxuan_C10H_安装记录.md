# LiYaxuan_C10H_安装记录.md

> 时间线：2026-10-09 下午。所有命令、报错、解决尝试都如实记录，没有伪造。
> 机器：Windows 11 原生，PowerShell 5.1，Python 3.13.13（D:\HJY\python.exe），Node v22.23.2。

---

## Step 0：环境确认

```powershell
python --version
# Python 3.13.13
node --version
# v22.23.2
```

## Step 1：下载源码

按任务要求，不用 git clone（本机 git 直连 github.com 超时），走 gh_helper.py 调 GitHub REST API 拿 zipball：

```powershell
python gh_helper.py zipball NousResearch/hermes-agent c10h-work\hermes-agent.zip
# ZIP 91021012 bytes -> c10h-work\hermes-agent.zip
```

解压后顶层目录是 `NousResearch-hermes-agent-93257fd/`。

## Step 2：研读 pyproject.toml 发现关键约束

```
.python-version  → 3.14
requires-python  → ">=3.11,<3.15"
dependencies     → 几乎每条都带 `; python_version >= '3.14'` 条件
```

**这意味着：在 Python 3.13 上 `pip install -e .` 只会装 hermes-agent 包本身，所有运行时依赖（openai/rich/httpx/...）都不会被装。** 这是后面报错的根因。

## Step 3：建 venv 并尝试 pip install -e .

```powershell
cd c10h-work\hermes-src\NousResearch-hermes-agent-93257fd
python -m venv .venv
# venv exit=0
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -e .
```

pip 输出（完整见 `pip_install.log`）：

```
Looking in indexes: https://mirrors.huaweicloud.com/repository/pypi/simple/
Obtaining file:///.../NousResearch-hermes-agent-93257fd
  Installing build dependencies: started
  Installing build dependencies: finished with status 'done'
  Building wheel for hermes-agent ... done
Successfully built hermes-agent
Installing collected packages: hermes-agent
Successfully installed hermes-agent-0.0.0
install exit=0
```

**表面成功，实际只装了一个空壳包**——因为依赖全是 3.14 条件化的。

## Step 4：尝试运行，拿到真实报错

```powershell
.\.venv\Scripts\python.exe cli.py --help
```

报错（原文见 `run_attempt.log`）：

```
Traceback (most recent call last):
  File "...\cli.py", line 34, in <module>
    from hermes_cli.cli_agent_setup_mixin import CLIAgentSetupMixin
  File "...\hermes_cli\cli_agent_setup_mixin.py", line 9, in <module>
    from rich.markup import escape as _escape
ModuleNotFoundError: No module named 'rich'
exit=1
```

再验一下 openai：

```powershell
.\.venv\Scripts\python.exe -c "import openai"
# ModuleNotFoundError: No module named 'openai'
exit=1
```

**结论**：本机 Python 3.13 不满足 Hermes 最新代码的依赖条件（它只在 3.14 上发 wheel）。手动 `pip install -e .` 这条路在 3.13 上走不通。

## Step 5：我做的解决尝试（如实）

1. **尝试 A：把依赖手动装进 venv** —— 还没做，因为 pyproject 里 pinned 版本都是 `==X.Y.Z` 且注释明确说"不要 ranges"，手动装很容易撞版本。而且就算装上，源码里可能还有 3.14-only 的语法。
2. **尝试 B：用系统 Python 3.13 直接跑 cli.py** —— 没必要，同样缺依赖。
3. **意外发现 C**：`shutil.which('hermes')` 返回了 `C:\Users\lenovo\AppData\Local\hermes\bin\hermes.EXE`——说明**官方安装器之前已经在这台机器上装过一份 Hermes**（用它自己捆绑的 Python 3.11.16，不是系统 Python 3.13）。

## Step 6：验证官方预装版能不能跑

```powershell
& "$env:LOCALAPPDATA\hermes\bin\hermes.EXE" --version
```

真实输出：

```
Hermes Agent v0.21.1 (2026.9.7) · upstream fc042f1d · local 16a40853 (+1 carried commit)
Install directory: C:\Users\lenovo\AppData\Local\hermes\hermes-agent
Install method: git
Python: 3.11.16
OpenAI SDK: 2.24.0
exit=0
```

```powershell
hermes doctor
```

真实输出（关键摘要，完整见 `hermes_native_try.log`）：

- ✓ Python 3.11.16 / SQLite 3.53.1 / venv 激活
- ✓ 必需包齐全：OpenAI SDK、Rich、python-dotenv、PyYAML、HTTPX
- ✓ 目录结构齐全：`%LOCALAPPDATA%\hermes\` 下有 cron/sessions/logs/skills/memories/SOUL.md
- ✓ 可用工具：browser-use / clarify / code_execution / computer_use / cronjob / delegation / desktop_ui / file / memory / project / session_search / skills / terminal / todo / web search(exa) / web extract(exa)
- ⚠ OpenRouter API (not configured) —— 没 key
- ⚠ discord/telegram/image_gen/vision/tts 等需要对应 token 或额外系统依赖
- ⚠ MEMORY.md / USER.md / state.db 还没建（第一次对话才会建）
- 只 1 个 issue：config 版本 v0→v42 待迁移（`hermes doctor --fix` 可自动修）

## Step 7：hermes tools 命令的限制

```powershell
hermes tools
```

真实输出：

```
Error: 'hermes tools' requires an interactive terminal.
It cannot be run through a pipe or non-interactive subprocess.
Run it directly in your terminal instead.
exit=1
```

**诚实记录**：我没法在这个自动化环境里跑 `hermes tools` 的交互版。但 `hermes doctor` 的 "Tool Availability" 一节已经把可用/不可用工具全列了，加上我直接读 `tools/registry.py` 源码，足以写工具探索报告。

## Step 8：hermes skills list（非交互可用）

```powershell
hermes skills list
```

真实输出：**51 个 builtin skills 全部 enabled**，分类如下（完整表格见 `hermes_tools.log`）：
- autonomous-ai-agents: claude-code / codex / computer-use / hermes-agent / opencode
- creative: architecture-diagram / ascii-video / baoyu-infographic / claude-design / design-md / humanizer / manim-video / p5js / popular-web-designs / songwriting-and-ai-music
- email: email-inbox-triage / himalaya
- media: gif-search / songsee / youtube-content
- note-taking: obsidian
- productivity: airtable / box / docx / google-workspace / maps / meeting-action-items / notion / pdf / powerpoint / product-price-monitor / teams-meeting-pipeline / weekly-review-planning / xlsx
- research: arxiv / competitor-news-monitoring / grounded-citations / llm-wiki
- software-development: codebase-inspection / dogfood / github / hermes-agent-skill-authoring / inspecting-hermes-desktop / node-inspect-debugger / requesting-code-review / simplify-code / spike / systematic-debugging / test-driven-development
- web: blocked-page-recovery

## Step 9：WSL 检查（只读，不安装）

```powershell
wsl --status
# 用于 Linux 的 Windows 子系统未安装（exit=50）
wsl -l -v
# 同上（exit=1）
```

按任务纪律：**没有执行 `wsl --install`**（那需要管理员 + 重启）。而且既然原生 Windows 版 Hermes 已经能跑，WSL 这条路本来也不是必须的。

## Step 10：卡点与剩余步骤（别人可照着做）

**我卡在**：
1. 没有 OpenRouter API key，没法真正发起一次对话（`hermes` 交互式 CLI 需要 LLM 后端）。
2. `hermes tools` 必须在真终端里跑，管道里跑不了。
3. 没有 Telegram/Discord bot token，Level 2 多平台接不进来。

**别人接手时照着做就能往下走**：

```powershell
# 1. 修一下旧 config
hermes doctor --fix

# 2. 去 https://openrouter.ai 注册 → Keys → Create Key
#    把 key 写进 %LOCALAPPDATA%\hermes\.env：
#    OPENROUTER_API_KEY=sk-or-xxxxxxxx

# 3. 在真终端（不是管道里）跑：
hermes setup
hermes model          # 选 OpenRouter + 一个便宜模型（如 deepseek/deepseek-chat）
hermes                # 开始第一次对话

# 4. 把我写的个性化 SOUL.md 覆盖到：
#    %LOCALAPPDATA%\hermes\SOUL.md

# 5. 把我写的 calligraphy-feedback 技能放到：
#    %LOCALAPPDATA%\hermes\skills\calligraphy-feedback\SKILL.md
#    然后 hermes skills list 应该能看到它。

# 6. 如果想要 browser 工具：
cd C:\Users\lenovo\AppData\Local\hermes\hermes-agent
npx playwright install chromium
```
