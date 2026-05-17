# gao-prompts

> 把"你使用 AI 的过程"沉淀为"懂你的 AI"——一组面向真实工作流的中文提示词集合。

这个仓库收集我在实际项目里反复打磨、有效的提示词。每一条都不是"通用模板",而是**针对一个具体问题、有明确输入输出、可立刻拿去用**的工程化产物。

## 它是什么 / 它不是什么

| | |
|---|---|
| ✅ 它是 | 一份**可复用提示词集** + **使用方法说明**,直接复制到 Claude / ChatGPT / Cursor 等对话框即可执行 |
| ✅ 它是 | **流程化、证据驱动**的提示词——强调多轮提炼、关键节点追问、最终归纳产出 |
| ❌ 它不是 | 一个 npm / pip 包,**无需安装**;也不是一键工具,需要你按工作流执行 |
| ❌ 它不是 | 通用模板库——每条提示词都有明确的输入前提,不满足前提请勿强用 |

## 提示词索引

| 名称 | 用途 | 输入 | 输出 | 状态 |
|---|---|---|---|---|
| [claude-session-to-subagent](prompts/claude-session-to-subagent.md) | 从你的 Claude Code 历史会话中反推"方法论 DNA",生成一个"懂你"的定制化 subagent | 至少 10 个清洗过的高价值需求/设计讨论 session | 一份 300-500 行的 subagent 系统提示文件 | 稳定 |

> 更多提示词陆续加入。欢迎通过 [Issue](../../issues) 提出你想看到的主题。

---

## 当前主打:`claude-session-to-subagent`

### 它解决什么问题

你和 AI 协作久了,会发现:**好的对话过程才是你真正的"方法论"——什么时候追问、什么时候给标准、什么时候打断、什么时候放手——这些细节藏在历史 session 里,但没人会主动总结。**

这条提示词的目标是用**6 轮独立提炼 + 综合归纳**,把你过往的有效 session 反向解构为可执行的系统提示,让一个新的 AI 实例(subagent)从第一次对话起就"像懂你一样"工作。

### 6 轮提炼框架(核心)

| 轮次 | 提炼主题 | 关键产出 |
|---|---|---|
| 第 1 轮 | 开场模式 | 你提需求的句式套路、信息密度、典型/异常模板 |
| 第 2 轮 | 追问触发器 | 你什么时候追问、追问什么、疲态信号识别 |
| 第 3 轮 | 收敛信号 | 何时讨论该结束、何时进入实施 |
| **第 4 轮** | **补丁式指令**(最高优先级) | 你反复手动说的 5 类指令——这是 subagent 的"必须主动做" |
| **第 5 轮** | **走偏与翻车**(次关键) | 5 种典型翻车模式 + 规避策略——这是 subagent 的"必须避免" |
| 第 6 轮 | 隐式标准 | 8 个维度的深层偏好(复杂度、改动范围、抽象程度等) |
| 综合归纳 | 汇总 + 归纳 | 一份可直接放进 `.claude/agents/` 或类似目录的 subagent 文件 |

第 1、2 轮各有一个**关键追问**(异常模板、半成品 vs 从零、@file 语义、疲态识别等),建议执行。

---

## 使用前提

> ⚠️ **本仓库不提供 session 提取脚本**——因为 Claude Code 等工具的 session 存储格式因版本而异,且涉及你的私人数据。请自备脚本,把历史 session 按下列规范清洗到 `./extracted/`。

### `./extracted/` 目录格式规范

```
extracted/
├── session-01.md
├── session-02.md
├── session-03.md
└── ...
```

每个 markdown 文件代表一次完整的需求/设计讨论。文件内消息按时间顺序排列,**每条消息标号化**,格式示例:

```markdown
【1】USER
我想在登录页面加一个"忘记密码"按钮,跟 @login.tsx 风格保持一致...

【2】ASSISTANT
好的,我先看看 login.tsx 的现有结构...

【3】USER
不,先别动 LoginForm,我们先确认按钮放哪里...
```

### 输入数量与质量要求

- **数量**:至少 10 个 session,建议 15-30 个
- **类型平衡**:必须包含**成功**(产出令你满意)和**失败**(走偏、需返工)两类
- **筛选**:只保留需求讨论 / 设计 / 架构 session,**剔除**纯代码生成、IDE 噪声、工具调用日志

---

## 使用步骤

```
第 1 步:准备数据
       └─ 用你自备的脚本筛 + 清洗 session,放入 ./extracted/
              ↓
第 2 步:逐轮提炼(每轮新开会话,避免上下文污染)
       ├─ 第 1 轮 → round1.md → (可选追问)
       ├─ 第 2 轮 → round2.md → (可选追问)
       ├─ 第 3 轮 → round3.md
       ├─ 第 4 轮 → round4.md   (最关键)
       ├─ 第 5 轮 → round5.md   (次关键)
       └─ 第 6 轮 → round6.md
              ↓
第 3 步:汇总
       └─ 合并所有轮次产出为 all-rounds.md
              ↓
第 4 步:综合归纳(再开新会话,旧上下文已爆)
       └─ 执行"综合归纳"prompt → 产出 subagent.md
              ↓
第 5 步:实战验证 + 迭代
       └─ 放进项目 subagent 目录 → 跑 3-5 次真实需求 → 复盘哪条失效/缺失 → 回到 subagent 文件迭代
```

完整提示词与综合归纳逻辑见:[prompts/claude-session-to-subagent.md](prompts/claude-session-to-subagent.md)

---

## License

[MIT](LICENSE) © 2026 Gao

---

## English Summary

**gao-prompts** is a curated collection of Chinese-language prompts engineered for real-world AI workflows.

The flagship prompt — [`claude-session-to-subagent`](prompts/claude-session-to-subagent.md) — turns your **historical Claude Code conversations into a personalized subagent** that "already knows how you work." It distills your methodology DNA via a 6-round evidence-driven analysis (opening patterns, follow-up triggers, convergence signals, patch-style directives, failure modes, and implicit standards), then synthesizes a 300–500-line system prompt ready to drop into any subagent runtime.

**Requirements**: at least 10 cleaned session transcripts in `./extracted/` with numbered messages. Session-extraction scripts are intentionally **not included** — formats vary across tool versions and the data is privacy-sensitive.
