# 在 Cursor 中使用本仓库

本项目包含一条 **Cursor 项目规则**，用于在你打开本仓库时自动应用这套受 Karpathy 启发的行为准则。

## 在本仓库中

1. 在 Cursor 中打开该文件夹。
2. 规则 [`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc) 已随仓库提交，并设置了 `alwaysApply: true`，因此不需要额外安装步骤。
3. 你可以在 Cursor 的 **Settings → Rules**（或项目规则界面）中确认，应该能看到 `karpathy-guidelines`。

## 在其他项目中复用同一套准则

**Cursor（推荐）：** 将 `.cursor/rules/karpathy-guidelines.mdc` 复制到目标项目的 `.cursor/rules/` 目录中（如不存在则创建）。你可以按需调整，或与现有规则合并。

**其他工具：** 如果某个工具只支持根目录说明文件，请改为复制 [`CLAUDE.md`](CLAUDE.md) 到该项目，或把内容合并进已有说明。

## 可选：个人 Agent Skills

如果你希望把同样内容作为 `~/.cursor/skills` 下的可复用 skill 使用，请使用 [`skills/karpathy-guidelines/SKILL.md`](skills/karpathy-guidelines/SKILL.md)。你可以把它复制或软链接到个人 skills 目录，布局按你现有习惯即可。

## Claude Code 与 Cursor

- **Claude Code：** 按插件市场和 [`README.md`](README.md) 中的说明安装；插件会暴露本仓库中的 skill。按项目使用时，也可以直接依赖 `CLAUDE.md`。
- **Cursor：** 使用上面描述的已提交 `.cursor/rules/` 文件。Cursor 默认不会读取 `.claude-plugin/` 或 `CLAUDE.md`。

## 给贡献者

修改四项原则时，请保持 **[`CLAUDE.md`](CLAUDE.md)** 和 **[`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc)** 同步。如果已发布的 skill 或插件文本也应保持一致，请同时更新 **[`skills/karpathy-guidelines/SKILL.md`](skills/karpathy-guidelines/SKILL.md)**。
