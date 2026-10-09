# LiYaxuan_C10H_AI日志.md

> 整个 C10H 过程中 AI（豆包）怎么帮我的。

## 轮次 1：对齐任务
- **我问 AI**："C10H 要求装 NousResearch/hermes-agent，但我这台是 Windows 原生，git push 还超时，第一步该干嘛？"
- **AI 建议**：不要 git clone，用 GitHub REST API 下 zipball；先读 pyproject.toml 看 Python 版本约束，再决定怎么装。
- **我采纳**：zipball 91MB 一次拉下来，省了 git 超时的坑。

## 轮次 2：发现 Python 版本坑
- **我让 AI 帮我读 pyproject.toml**。AI 一眼看出"dependencies 全是 `; python_version >= '3.14'` 条件化的"。
- **预测**：你这台是 3.13，`pip install -e .` 会成功但什么依赖都不装，跑起来必然 ModuleNotFoundError。
- **验证**：完全应验——`rich` / `openai` 都没装上，报错原文我留在 run_attempt.log 里。

## 轮次 3：意外发现官方预装版
- **我跑 `shutil.which('hermes')`** 本来只是想确认 hermes 命令在不在，结果发现 `%LOCALAPPDATA%\hermes\bin\hermes.EXE` 已经存在。
- **我问 AI**："这是之前谁装的？我要不要假装是我自己装的？"
- **AI 立场**：不能假装。要如实写"机器上已经有一份官方安装器装的 Hermes v0.21.1，用的是它自带的 Python 3.11.16，不是我手动装的"。我照做了。

## 轮次 4：hermes doctor 输出解读
- doctor 输出很长（100+ 行），我贴给 AI 让它分类：哪些✓能用、哪些⚠是缺 token、哪些是真问题。
- AI 帮我归成：17 个 toolset 可用、17 个 toolset 缺依赖、1 个 config 待迁移。

## 轮次 5：写自定义技能
- **我问 AI**："agentskills.io 的 SKILL.md 到底要什么格式？"
- AI 让我去看官方 skills/note-taking/obsidian/SKILL.md 当模板，然后照 frontmatter（name/description/version/author/license/platforms/metadata.hermes.tags）写我自己的"书法练习点评"。
- 我写的技能解决的是我自己书法 MaaS 项目的真实需求，不是凑数。

## 轮次 6：WSL 决策
- **我问 AI**："wsl --status 说没装，我要不要 wsl --install？"
- **AI 提醒纪律**：任务书明确说"需要管理员/重启就停"，而且原生 Windows 版 hermes 已经能跑，根本不需要 WSL。所以不装。

## 轮次 7：用 DeepSeek 真正跑通首次对话（关键）
- **背景**：我拿到一把 DeepSeek key（OpenAI 兼容），但第一次 one-shot 直接报 `HTTP 401: Missing Authentication header`。
- **我让 AI 帮我定位**：AI 建议别只看报错文字，用 Hermes 自带 Python 直接调它的 `resolve_runtime_provider()` 打印"provider/base_url/api_key/source"。结果一眼看清——key 读对了（source=env:DEEPSEEK_API_KEY），但 **base_url 是残留的 `https://openrouter.ai/api/v1`**，DeepSeek key 被发到 OpenRouter 才 401。
- **我修复**：`hermes config set model.base_url https://api.deepseek.com/v1`，并把模型从已退役的 deepseek-chat 换成当前的 deepseek-flash，重跑立即返回真实回复；随后安装并用 `-s calligraphy-feedback` 真实调用了我写的书法点评技能。
- **收获**：这次是我自己理解了"key、base_url、模型 ID"三件套必须同时对上，比单纯让 AI 给答案扎实得多。

## AI 使用质量自评
- 每一轮都是"我跑命令→贴真实输出→AI 解读→我再跑下一步"，不是一句话指令。
- 关键判断（Python 3.13 装不上、官方预装版不能冒领、WSL 不装、401 是 base_url 错位）都是 AI 提醒我，我再验证。
- 没有伪造任何"对话截图"：没 key 时如实写卡点；拿到 key 后真实跑通，才补上首次对话证据（`LiYaxuan_C10H_首次对话证据.png` + 两份 `.log`）。
- 真实 key 只写在本机 `.env`，没有进入任何对话记录、截图或仓库。
