# Andrej Karpathy 编码准则中文版：Claude Code / Codex / Cursor AI 编程规则

面向中文开发者的 **Claude Code、Codex、Cursor、AI Agent 编程行为准则**。本仓库把 Andrej Karpathy 对 LLM 编码问题的观察整理成可直接使用的 `CLAUDE.md`、`AGENTS.md`、Cursor Rules 和 Claude Code Skill，帮助 AI 编程助手减少错误假设、过度工程化、无关改动和不可验证的实现。

如果你正在搜索 **Claude Code 中文规则、Codex AGENTS.md、Cursor AI 编程规范、AI Agent coding guidelines、LLM 代码生成最佳实践、Karpathy 编程准则中文版**，这个仓库就是一套开箱即用的提示词和项目规则模板。

## 适合谁使用

- 使用 Claude Code、Codex、Cursor、Cline、Windsurf、Aider 或其他 AI 编程助手的开发者。
- 希望减少“AI 擅自改代码”“越改越复杂”“没问清楚就实现”的团队。
- 想把项目级 AI 编程规范沉淀为 `CLAUDE.md`、`AGENTS.md`、Cursor Rule 或 Skill 的工程团队。
- 需要一份中文 AI Coding Prompt / Agent Rules / LLM Coding Guidelines 的个人开发者。

## 它解决什么问题

来自 Andrej Karpathy 对 LLM 编码陷阱的观察：AI 编程助手常常会替用户做错误假设，不主动澄清，隐藏困惑，过度设计，顺手改无关代码，甚至删除自己没有充分理解的内容。

本仓库把这些问题收束为四条可执行原则：

| 原则 | 解决的问题 | 对 AI 编程的价值 |
|------|------------|------------------|
| **编码前先思考** | 错误假设、隐藏困惑、缺少取舍说明 | 让 AI 先澄清，再实现 |
| **简洁优先** | 过度复杂、抽象膨胀、提前设计 | 让代码更短、更直接、更易维护 |
| **精准改动** | 无关编辑、顺手重构、误删代码 | 让 diff 更小，PR 更干净 |
| **目标驱动执行** | 目标模糊、缺少验证、实现不可复现 | 让 AI 用测试和检查闭环完成任务 |

## 快速使用

### 1. Claude Code

在 Claude Code 中添加 marketplace：

```text
/plugin marketplace add MackDing/andrej-karpathy-skills-zh
```

安装插件：

```text
/plugin install andrej-karpathy-skills@karpathy-skills
```

也可以直接使用 `CLAUDE.md`：

新项目：

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/MackDing/andrej-karpathy-skills-zh/main/CLAUDE.md
```

已有项目追加：

```bash
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/MackDing/andrej-karpathy-skills-zh/main/CLAUDE.md >> CLAUDE.md
```

### 2. Codex

Codex 和通用 AI Coding Agents 推荐使用 `AGENTS.md`：

```bash
curl -o AGENTS.md https://raw.githubusercontent.com/MackDing/andrej-karpathy-skills-zh/main/AGENTS.md
```

已有项目追加：

```bash
echo "" >> AGENTS.md
curl https://raw.githubusercontent.com/MackDing/andrej-karpathy-skills-zh/main/AGENTS.md >> AGENTS.md
```

`AGENTS.md` 更偏执行规则，适合让 Codex 在编码、审查、重构和验证时持续遵守。

### 3. Cursor

本仓库已经包含 Cursor 项目规则：

```text
.cursor/rules/karpathy-guidelines.mdc
```

把这个文件复制到你的项目 `.cursor/rules/` 目录中，即可让 Cursor 在该项目中自动应用同一套 AI 编程准则。

## 支持的 AI 编程工具

| 工具 | 推荐文件 | 用途 |
|------|----------|------|
| Claude Code | [`CLAUDE.md`](./CLAUDE.md) / Claude Code Skill | 项目级规则和插件安装 |
| Codex | [`AGENTS.md`](./AGENTS.md) | Codex 与通用 AI Agent 项目规则 |
| Cursor | [`.cursor/rules/karpathy-guidelines.mdc`](./.cursor/rules/karpathy-guidelines.mdc) | Cursor 自动应用规则 |
| 其他 AI Coding Agents | [`AGENTS.md`](./AGENTS.md) 或 [`CLAUDE.md`](./CLAUDE.md) | 复制为项目级提示词或规则文件 |

## CLAUDE.md、AGENTS.md 和 Cursor Rules 有什么区别

| 文件 | 面向工具 | 内容定位 |
|------|----------|----------|
| `CLAUDE.md` | Claude Code | Claude Code 项目级说明 |
| `AGENTS.md` | Codex / 通用 AI Agent | 更直接的执行规则和禁止事项 |
| `.cursor/rules/karpathy-guidelines.mdc` | Cursor | Cursor 项目规则，支持自动应用 |

## 四项核心准则

### 1. 编码前先思考

**不要擅自假设。不要掩盖困惑。把取舍讲清楚。**

LLM 经常会悄悄选择一种理解，然后直接开始写代码。正确做法是先暴露假设：

- 明确说明当前假设，不确定时先问。
- 如果存在多种理解，列出选项而不是默默选择。
- 如果有更简单、更稳妥的方案，主动指出。
- 如果上下文不足，先停下来说明哪里不清楚。

### 2. 简洁优先

**用能解决问题的最少代码。不要做推测性设计。**

AI 编程助手很容易把小需求写成大框架。这个准则要求：

- 不实现需求之外的功能。
- 不为只用一次的代码抽象一套体系。
- 不添加用户没有要求的配置项、扩展点和“灵活性”。
- 如果 200 行可以缩成 50 行，就重写成 50 行。

### 3. 精准改动

**只改必须改的地方。只清理你自己造成的问题。**

在真实项目中，AI 最大的风险之一是顺手改坏无关代码。这个准则要求：

- 不顺手改相邻代码、注释或格式。
- 不重构没有坏掉的模块。
- 匹配项目既有风格，而不是强行套自己的偏好。
- 每一行 diff 都能追溯到用户请求。

### 4. 目标驱动执行

**定义成功标准。循环推进直到完成验证。**

不要只说“修复 bug”或“添加功能”，而是把任务变成可验证目标：

| 模糊指令 | 更好的 AI 编程目标 |
|----------|-------------------|
| 添加校验 | 先为非法输入写测试，再让测试通过 |
| 修复 bug | 先写复现 bug 的测试，再实现修复 |
| 重构模块 | 确保重构前后测试都通过 |
| 优化性能 | 给出基线指标，再验证优化后的指标 |

## 仓库内容

| 文件 | 用途 |
|------|------|
| [`CLAUDE.md`](./CLAUDE.md) | Claude Code 项目级中文规则 |
| [`AGENTS.md`](./AGENTS.md) | Codex 和通用 AI Agent 项目级中文规则 |
| [`CURSOR.md`](./CURSOR.md) | Cursor 使用说明 |
| [`.cursor/rules/karpathy-guidelines.mdc`](./.cursor/rules/karpathy-guidelines.mdc) | Cursor 自动应用规则 |
| [`skills/karpathy-guidelines/SKILL.md`](./skills/karpathy-guidelines/SKILL.md) | Claude Code Skill |
| [`EXAMPLES.md`](./EXAMPLES.md) | 错误示例与正确示例对比 |

## 推荐关键词

Claude Code 中文规则、Codex AGENTS.md、Cursor Rules 中文、AI 编程规范、AI Agent 编程准则、LLM Coding Guidelines、Karpathy Guidelines 中文版、Claude Code Skill、Codex AI Rules、Cursor AI Rules、AI Coding Prompt、AI 代码生成最佳实践、提示词工程、Agentic Coding、AI 辅助编程、代码审查规则、项目级 CLAUDE.md、项目级 AGENTS.md。

## FAQ

### 这个仓库和普通提示词有什么不同？

它不是一句泛泛的“请写好代码”，而是一组面向真实工程场景的行为约束：先澄清、少抽象、少改动、可验证。它适合放进项目根目录，让 AI 编程助手在每次任务中持续遵守。

### 可以用于 Claude Code 吗？

可以。你可以直接复制 `CLAUDE.md`，也可以通过 Claude Code 插件方式安装。

### 可以用于 Codex 吗？

可以。将 `AGENTS.md` 放在项目根目录，作为 Codex 的项目级协作规则。它比 README 更适合给 Agent 执行，因为内容包含角色定位、执行原则、禁止事项和输出要求。

### 可以用于 Cursor 吗？

可以。本仓库包含 `.cursor/rules/karpathy-guidelines.mdc`，适合直接复制到 Cursor 项目的 `.cursor/rules/` 目录。

### 适合团队使用吗？

适合。团队可以在此基础上继续添加语言、框架、测试、代码审查、分支管理等项目专属规则。

### 为什么强调“少改动”？

AI 编程最大的成本往往不是写不出代码，而是改了不该改的地方。精准改动能降低 review 成本，也能减少隐藏回归。

## 取舍说明

这套准则偏向 **谨慎胜过速度**。对于拼写修复、明显的一行改动等简单任务，可以轻量执行；对于涉及生产代码、共享模块、重构、bug 修复和用户可见行为的任务，建议严格遵守。

## 许可证

MIT
