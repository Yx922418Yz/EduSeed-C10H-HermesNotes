# LiYaxuan_C10H_拿来说明.md

> C10H 本身就是最大的拿来主义——我整个站在 Nous Research 的 Hermes Agent 肩膀上。下面写清楚我拿了什么、自己做了什么。

## 一、我从 Hermes Agent 整个拿了什么

| 模块 | 我拿了什么 | 为什么 |
|---|---|---|
| 整个 Agent 主循环 | run_agent.py + agent/ 目录那套"LLM 决定调哪个 tool → 执行 → 结果塞回上下文"的循环 | 我自己 C10 写的 kstar_loop.py 是极简版，Hermes 是工业版，照着它的结构以后我可以把我的 loop 升级 |
| 工具注册机制 | `tools/registry.register(name=, toolset=, handler=, check_fn=)` 的设计 | 我打算以后在自己技能 frontmatter 里加 `requires:` 字段，就是抄它的 check_fn |
| 记忆系统 | SQLite + FTS5 全文搜索 + MEMORY.md/USER.md 两层 | 我 C10 现在是纯 Markdown，以后量大了可以抄它上 SQLite |
| SOUL.md | 人格定义文件这个位置和概念 | 我自己写了一份个性化的，替换默认的 |
| 技能系统 | agentskills.io 开放标准（frontmatter + markdown body） | 我的"书法练习点评"技能就是按这个标准写的 |
| Cron 定时任务 | 自然语言写 cron、跨平台投递 | 我以后想让 Hermes 每天早上提醒我背政治知识点 |

## 二、我自己做的（不是拿的）

1. **LiYaxuan_C10H_SOUL.md**：官方默认 SOUL.md 只有一句"Be direct"，我换成了带"墨墨"人格、知道我膝盖疼、禁止 AI 味套话的定制版。
2. **custom_skills/calligraphy-feedback/**：按 agentskills.io 标准手写的"书法练习点评"技能，解决我自己书法 MaaS 项目的真实需求——这套三层点评 SOP（笔法/结构/章法）是我自己从练字经验里总结的。
3. **LiYaxuan_C10H_安装记录.md**：把"手动 pip install -e . 在 Python 3.13 上失败"这个坑原样记录下来，下一个零基础同学不会再踩。
4. **LiYaxuan_C10H_工具探索报告.md**：50+ 个工具按类整理，是我自己读源码 + doctor 输出交叉出来的，不是抄 README。

## 三、我从 C10 反向又"拿来"回 Hermes 的

有趣的是双向的：

- 我在 C10 里写的 KSTAR 闭环思路（R̂ vs R → ΔR → 写回），其实和 Hermes 的"closed learning loop / skills self-improve during use"是同一个哲学。
- 我读 Hermes 源码时发现它也是"每次复杂任务后自动创建技能"——这和我 C10 里 `skill_manager.py iterate` 是一个意思。
- 两边互相印证：我自己设计的方向是对的，只是工业界把它做得更完整。

## 四、没拿什么 / 没抄什么

- 没抄它的 200+ 个 tools/*.py——我用不上，也维护不动。
- 没抄它的 gateway（Telegram/Discord 那套）——我没 token，接进来也白接。
- 没改它的源码——我是使用者，不是贡献者（至少这轮不是）。
