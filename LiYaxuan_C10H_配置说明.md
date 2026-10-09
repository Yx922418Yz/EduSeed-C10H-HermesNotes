# LiYaxuan_C10H_配置说明.md

> **状态更新（2026-10-09）：已用 DeepSeek（OpenAI 兼容）完成 Hermes 首次真实对话，并成功加载、调用自定义技能「书法练习点评」。** 证据见 `LiYaxuan_C10H_首次对话证据.png` 与 `证据日志/hermes_first_chat_intro.log`、`证据日志/hermes_calligraphy_skill.log`。

## 一、为什么最终选 DeepSeek 作为 LLM provider

我最初按"学生零预算、想多试模型"的思路把 OpenRouter 写为首选；但在拿到真实 key 并实测后，改用 **DeepSeek**。对比与理由如下：

| Provider | 实测结果 | 优点 | 我为什么（不）选 |
|---|---|---|---|
| **DeepSeek** ✅ | `GET /user/balance` 返回 `is_available=true`、余额 10.39 CNY；one-shot 真实返回 | 与 OpenAI **完全兼容**、国内直连无需海外卡、价格低、中文好、注册即送可用余额 | **最终选用**：实测可用 + 兼容标准协议 + 成本低，最适合我这种零基础学生 |
| OpenRouter | 用手上这把 key 调用返回 **HTTP 401**（它不是 OpenRouter key，我也没有 OpenRouter key） | 一个 key 调 200+ 模型 | 聚合器思路好，但我没有可用的 OpenRouter key，实测不通，放弃 |
| Nous Portal | 未登录 | Hermes 原生优化 | 需另注册，模型选择少 |
| OpenAI | 未配置 | 模型质量高 | 贵、要绑海外卡，我没卡 |
| Kimi / MiniMax | 未配置 | 中文好、便宜 | 本次以打通标准 OpenAI 兼容链路为目标，DeepSeek 已满足 |

**结论**：DeepSeek 提供标准 OpenAI 协议（`/chat/completions`、`Authorization: Bearer`），Hermes 原生支持，且实测真的能通、成本低——这是"零基础 + 低预算 + 要真实跑通"约束下的最合理选择。

> 模型 ID 注意：DeepSeek 当前线上模型为 **`deepseek-flash`** 与 **`deepseek-v4-pro`**（`GET https://api.deepseek.com/models` 可查）。早期的 `deepseek-chat`/`deepseek-reasoner` 已退役（作为旧别名仍可能响应，但在 Hermes 里应使用新 ID）。本次默认模型固定为 `deepseek-flash`。

## 二、DeepSeek 接入步骤（已实际执行；以下为可复现 SOP，key 已打码）

1. 到 https://platform.deepseek.com 注册，在"API keys"页面创建一把 `sk-` 开头的 key。
2. 把 key 写入 **本机** Hermes 的 `.env`（该文件已在 `.gitignore`，不会上传）：

```powershell
notepad $env:LOCALAPPDATA\hermes\.env
# 加一行（真实 key 只存在这里）：
DEEPSEEK_API_KEY=sk-你的DeepSeek密钥
```

3. 固定 provider、模型与 base_url：

```powershell
hermes config set model.provider deepseek
hermes config set model.default deepseek-flash
hermes config set model.base_url https://api.deepseek.com/v1
```

> **踩坑记录（关键）**：我的 `config.yaml` 里原本残留一条 `model.base_url: https://openrouter.ai/api/v1`。结果 Hermes 虽然正确读到了 DeepSeek key，却把请求发到 OpenRouter，返回 `HTTP 401: Missing Authentication header`。把该 base_url 改回 `https://api.deepseek.com/v1` 后立即恢复正常。排查时我用 Hermes 自带 Python 调用 `resolve_runtime_provider()` 才看清"key 对了、base_url 错了"。

4. 验证首次对话（one-shot，只打印最终回复，适合脚本/留证）：

```powershell
hermes -z "Hello! Please introduce yourself and state your model/provider."
# 真实回复（2026-10-09）：
# Hello! I'm Hermes Agent, an assistant built by Nous Research ...
# I'm currently running on the model deepseek-flash, provided by deepseek.
```

## 三、自定义技能的安装、加载与调用（已演示）

我为 C10H 编写的技能 `calligraphy-feedback`（书法练习点评）已真实安装并被调用：

```powershell
# 1) 把技能目录放进用户技能目录（结构：skills/<分类>/<技能名>/SKILL.md）
#    实际位置：%LOCALAPPDATA%\hermes\skills\personal\calligraphy-feedback\SKILL.md
hermes skills list            # 能看到：calligraphy-feedback | personal | local | enabled

# 2) 预加载技能并发起对话
hermes -s calligraphy-feedback -z "我今天练了永字，横画总写得斜，起笔不知道怎么下笔。"
```

模型严格按技能规定的格式输出【笔法】【结构】【章法】【今天最该改的一件事】【下次练习建议】，并以"明天把你的永字拍给我看改进版"收尾；在只有文字、没有照片时还会先声明假设（符合技能末节要求）。完整输出见 `证据日志/hermes_calligraphy_skill.log`。

## 四、多平台接入（Level 2，本次未做）

| 平台 | 难度 | 步骤摘要 |
|---|---|---|
| Telegram | ★☆☆ | @BotFather 创建 bot → 拿 token → `.env` 加 `TELEGRAM_BOT_TOKEN=...` → `hermes gateway` |
| Discord | ★★☆ | discord.com/developers 创建 Application/Bot → 拿 token → 加 `DISCORD_BOT_TOKEN=...` |
| Slack | ★★☆ | Create New App → Bot Token Scopes → Install |

与上一版不同：**LLM 链路已经打通**，平台接入不再是"接了也聊不了天"，现在只需按上表补齐对应 token 即可，后续步骤我会按需再做，本次不冒充已完成。

## 五、SOUL.md 个性化

完整个性化 SOUL.md 见 `LiYaxuan_C10H_SOUL.md`：给 Agent 起名"墨墨"（来自书法 MaaS 项目），规定禁止 AI 味套话、报错先安抚、可撒娇但不说教，并写入我的真实约束。部署方式：将其内容覆盖到 `%LOCALAPPDATA%\hermes\SOUL.md`。

## 六、安全与占位（红线）

- 真实 key **只**存在于本机 `%LOCALAPPDATA\hermes\.env`，仓库与桌面交付里**没有**、也不会有。
- 仓库/桌面只提供模板 `.env.example`（占位 `sk-你的DeepSeek密钥` + 获取 SOP）；`.gitignore` 已包含 `.env`。
- 所有证据日志、截图、commit message 中均不含真实 key。
