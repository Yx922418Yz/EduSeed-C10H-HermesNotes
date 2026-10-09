# LiYaxuan_C10H_配置说明.md

## 一、为什么选 OpenRouter 作为 LLM provider

我对比过 Hermes 支持的几个 provider，最终选 OpenRouter，理由按权重排：

| Provider | 优点 | 缺点 | 我为什么不选 |
|---|---|---|---|
| **OpenRouter** ✅ | 一个 key 调 200+ 模型；有免费额度；不用分别注册 | 延迟稍高 | **首选**：我是学生没预算，免费额度 + 可随时切模型试错 |
| Nous Portal | Hermes 原生优化 | 模型选择少，要另注册 | 我只是体验，不想被锁死 |
| OpenAI | GPT-4o 质量高 | 贵，要绑海外卡 | 我没卡 |
| Kimi/Moonshot | 中文好 | 国际访问有时不稳 | 我做的是英文源码研读为主 |
| MiniMax | 便宜 | 英文弱 | 同上 |

**结论**：OpenRouter 是"零基础 + 没预算 + 想多试几个模型"这三个约束下的唯一合理解。而且 `hermes model` 是运行时切换，不用改代码，以后想换随时换。

## 二、OpenRouter 接入步骤（SOP）

> **诚实声明：我现在没有 OpenRouter key，下面是我写好的 SOP，等我自己去注册后照着执行。**

1. 打开 https://openrouter.ai ，用 GitHub 账号登录。
2. 右上角头像 → Keys → **Create Key**。
3. 起个名字（比如 `hermes-liyaxuan`），权限默认就行，点 Create。
4. **立刻复制那串 `sk-or-v1-xxxxxxxx...`**，关掉弹窗就再也看不到了。
5. 写到 Hermes 的 env 文件里：

```powershell
# 用记事本打开：
notepad $env:LOCALAPPDATA\hermes\.env
# 加一行：
OPENROUTER_API_KEY=sk-or-v1-把你的key粘这里
# 保存关闭
```

6. 选模型（在真终端里）：

```powershell
hermes model
# 选 OpenRouter
# 模型推荐（学生穷鬼版）：
#   - deepseek/deepseek-chat-v3   便宜，中文好，我日常用
#   - meta-llama/llama-3.3-70b-instruct:free   免费额度里试
#   - qwen/qwen-2.5-coder-32b-instruct   写代码时切这个
```

7. 验证：

```powershell
hermes
# 进去后输入：/model
# 确认当前模型是你刚选的
# 随便说一句"你好"，能回就通了
```

## 三、平台接入步骤（多平台，Level 2，我还没做）

| 平台 | 难度 | 步骤摘要 |
|---|---|---|
| Telegram | ★☆☆ | @BotFather 创建 bot → 拿 token → `.env` 里加 `TELEGRAM_BOT_TOKEN=...` → `hermes gateway` |
| Discord | ★★☆ | 去 discord.com/developers 创建 Application → Bot → 拿 token → 加 `DISCORD_BOT_TOKEN=...` |
| Slack | ★★☆ | Create New App → Bot Token Scopes → Install to Workspace |

我这次一个都没配，因为：(1) 没有这些平台的开发者账号；(2) 核心卡点是 LLM key，没有 key 平台接进来也聊不了天。

## 四、SOUL.md 个性化

我写了一份完整的个性化 SOUL.md，见 `LiYaxuan_C10H_SOUL.md`。核心是：

- 给 Agent 起名叫"墨墨"（来自我的书法 MaaS 项目）。
- 说话方式硬性规定：禁止 AI 味套话、报错先安抚、可以撒娇但不许说教。
- 把我的真实约束写进去：膝盖疼、四级备考、零基础但当队长、对 AI 味零容忍。

**部署方式**：把 `LiYaxuan_C10H_SOUL.md` 的内容覆盖到 `%LOCALAPPDATA%\hermes\SOUL.md`。（官方预装版已经有一个默认 SOUL.md，我这份是替换它。）

## 五、API key 占位说明

- 我**没有** OpenRouter key，这一点不掩饰。
- 上面 `.env` 示例里的 `sk-or-v1-把你的key粘这里` 就是占位符。
- 我没有把任何真 key 写进仓库——这是纪律，不是谦虚。
- 等我自己注册完，按上面 SOP 第 5 步贴进去即可，不需要改任何代码。
