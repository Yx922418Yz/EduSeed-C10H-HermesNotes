# LiYaxuan_C10H_工具探索报告.md

> 基于两路证据：(1) 本机 `hermes doctor` 真实输出的 "Tool Availability" 一节；(2) 直接读 `tools/registry.py` 和各 `tools/*_tool.py` 源码里的 `registry.register(name=..., toolset=...)`。
> 因为 `hermes tools` 是交互命令、管道里跑不了（见安装记录 Step 7），这份报告是"源码 + doctor 输出"双路交叉的结果。

---

## 一、文件操作类（file toolset）

| 工具名 | 用途 | 证据 |
|---|---|---|
| `read_file` | 读文件，自动分页、标行号，支持 PDF/Office 转 markdown | 源码 file_tools.py:1396 |
| `write_file` | 写文件，有 write guard 防误写 | 源码 file_tools.py:1397 |
| `patch` | 对文件做精确字符串替换（比 write_file 省 token） | 源码 file_tools.py:1420 |
| `search_files` | 按 glob/内容搜文件 | 源码 file_tools.py:1421 |

## 二、终端 / 代码执行类

| 工具名 | 用途 | 证据 |
|---|---|---|
| `terminal` | 在本机 shell 里跑命令（7 种 backend：local/docker/ssh/singularity/modal/daytona/vercel-sandbox） | doctor ✓ |
| `close_terminal` | 关掉一个后台终端会话 | 源码 close_terminal_tool.py |
| `read_terminal` | 读后台终端的输出（不阻塞） | 源码 read_terminal_tool.py |
| `execute_code` | 沙箱里跑 Python 代码片段 | 源码 code_execution_tool.py:965 |
| `code_kernel` | 持久化的 Python REPL kernel（比每次冷启动快） | 源码 code_kernel.py |
| `process_manage` | 列进程、杀进程、管理后台任务 | 源码 process_registry.py:2783 |

## 三、浏览器类（doctor 里标⚠，因为缺 Playwright Chromium）

| 工具名 | 用途 |
|---|---|
| `browser_cdp` | 通过 Chrome DevTools Protocol 控浏览器 |
| `browser_dialog` | 处理 alert/confirm 弹窗 |
| `browser_exec` | 在浏览器里跑 JS |
| `browser_vault_list` / `unlock` / `save_login` / `enter_code` / `fill` | 浏览器密码库：列/解锁/保存登录/填验证码/填表单 |
| `computer_use` | 控制整个桌面（鼠标键盘截图） |

## 四、Web 信息类

| 工具名 | 用途 | 证据 |
|---|---|---|
| `web_search` | Exa API 网页搜索 | 源码 web_tools.py:560，doctor ✓ |
| `web_extract` | 抓 URL 正文并清洗成 markdown | 源码 web_tools.py:566，doctor ✓ |
| `x_search` | 搜 X (Twitter)，需要 XAI_API_KEY | 源码 x_search_tool.py |

## 五、记忆 / 会话类

| 工具名 | 用途 | 证据 |
|---|---|---|
| `memory` | 显式写/读 MEMORY.md（长期记忆） | 源码 memory_tool.py:449，doctor ✓ |
| `session_search` | FTS5 全文搜索过去所有会话 | 源码 session_search_tool.py:787，doctor ✓ |
| `todo_list` | 维护待办清单 | 源码 todo_tool.py:333，doctor ✓ |
| `cronjob_manage` | 增删改查定时任务（自然语言写 cron） | 源码 cronjob_tools.py:1193，doctor ✓ |

## 六、技能系统类

| 工具名 | 用途 | 证据 |
|---|---|---|
| `skills_list` | 列所有技能 | 源码 skills_tool.py:692，doctor ✓ |
| `skill_view` | 看某个技能的详情 | 源码 skills_tool.py:725 |
| `skill_manage` | 创建/编辑/删除技能（agent 自动沉淀） | 源码 skill_manager_tool.py:923 |

## 七、委派 / 并行类

| 工具名 | 用途 | 证据 |
|---|---|---|
| `delegate_task` | 派生子 agent 并行干子任务 | 源码 delegate_tool.py:790，doctor ✓ |

## 八、桌面 UI / TUI 类

| 工具名 | 用途 |
|---|---|
| `desktop_ui` / `react_to_message` | 在桌面客户端里给消息加 emoji reaction |
| `desktop_preview` | 预览一个 UI 组件 |
| `drive_preview` / `open_preview` / `close_preview` / `annotate_preview` / `apply_layout` | 预览/标注/布局调整一系列 UI 工具 |
| `focus_pane` / `read_window_below` | TUI 多窗格管理 |
| `show_tip` / `gui_tour` | 给用户弹提示 / 走引导 tour |

## 九、消息平台 / 连接器类（大多需要 token）

| 工具名 | 用途 | 状态 |
|---|---|---|
| `discord` / `discord_admin` | Discord bot 收发消息 | ⚠ 缺 DISCORD_BOT_TOKEN |
| `feishu_doc_read` / `feishu_drive` | 飞书文档/云盘读写 | ⚠ 系统依赖未满足 |
| `manage_connections` / `manage_catalog` | MCP 连接器目录 | doctor ⚠ |
| `send_message` / `react_to_message` | 跨平台发消息 | desktop_ui ✓ |

## 十、媒体生成类（doctor 里标⚠）

| 工具名 | 用途 |
|---|---|
| `image_generate` | 文生图（Fal 等后端） |
| `video_generate` / `xai_video` | 文生视频 |
| `tts` | 文字转语音 |
| `transcription` | 语音转文字（whisper.cpp / cloud） |
| `vision` | 看图（多模态） |

## 十一、其它

| 工具名 | 用途 |
|---|---|
| `clarify` | 信息不够时 agent 主动反问用户 |
| `kanban` | 看板（仅 dispatcher worker 加载） |
| `project` / `desktop_project` | 项目级工作目录管理 |
| `setup_choose` | 初始设置向导里选东西 |
| `mcp_tool` | 接外部 MCP server（扩展工具生态） |

---

## 汇总

- **doctor 报告里明确 ✓ 可用的**：17 个 toolset（browser-use / clarify / code_execution / computer_use / cronjob / delegation / desktop_ui / file / memory / project / session_search / skills / terminal / todo / kanban / web_search / web_extract）。
- **源码里 `registry.register` 真实出现的工具名**：≥ 40 个（上面表里列了 50+ 行）。
- **doctor 里⚠ 的**：需要额外 token（discord/xai）或额外系统依赖（playwright/ffmpeg/docker）的，共 17 个 toolset。

## 我对工具系统的理解

Hermes 的工具不是"写死在一个 dict 里"，而是**每个 tools/*.py 在 import 时自己调 `registry.register(name=..., toolset=..., handler=..., check_fn=...)`**。`check_fn` 就是 doctor 里那个✓/⚠ 的来源——工具注册时顺便声明"我需要什么环境"，运行时和 doctor 时分别检查一次。这个设计我以后在自己的 C10 第二大脑里也可以学：每个技能 frontmatter 里加一个 `requires:` 字段，loop 启动时先检查。
