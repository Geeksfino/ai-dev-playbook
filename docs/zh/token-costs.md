# Token 成本

各文件的真实成本与加载时机。用于在上下文预算紧张时做取舍。

---

## Agent 如何加载这些文件

**Harness 层（`CLAUDE.md` 或等效规则）** 在会话开始时读取一次，纳入系统提示，贯穿整个会话。不会每条消息重复发送——是每会话的固定开销，分摊到所有轮次。

- **Cursor：** `.cursor/rules/*.mdc` 中 `alwaysApply: true` 的规则
- **Claude Code：** 项目根目录 `CLAUDE.md`
- **Codex / 其他：** 各工具的项目指令机制

**技能（`SKILL.md`）** 在 Agent 判断与当前任务相关时加载。技能内的 reference 文件仅在技能正文明确要求读取时加载。

**模板** 不会被模型加载——仅供人复制使用。

---

## 文件成本

| 文件 | Tokens | 加载时机 | 说明 |
|------|-------:|---------|------|
| `CLAUDE.md` | ~670 | 每次会话 | 每会话固定开销 |
| `skills/loop-engineering/SKILL.md` | ~2,800 | 请求搭建循环时 | 普通编码会话不加载 |
| `references/five-moves.md` | ~1,400 | 搭建循环（BUILD 模式） | |
| `references/failure-modes.md` | ~1,300 | 审计循环（AUDIT 模式） | |
| `references/toolchain-map.md` | ~950 | 配置调度时 | |
| 模板 | 0 | 从不 | 仅复制粘贴 |

**最坏情况**（所有 loop reference 同时加载）：约 ~6,500 tokens。仅当开发者在单次会话中做完整 loop 搭建或审计时发生。

**典型会话（仅 CLAUDE.md）：** 约 ~670 tokens 开销。

**典型 loop 搭建会话：** 约 ~4,200 tokens（SKILL.md + 一个 reference 文件）。

---

## 若要降低成本

### CLAUDE.md 已精简

本仓库版本为 90 行 / ~670 tokens，源自 184 行 / ~1,800 tokens 的版本。再删减可能损失行为信号。若必须更短，§7（调试）和 §8（依赖）的单位 token 影响最低，可优先删除。

### loop-engineering 技能已是渐进加载

SKILL.md 正文指示 Agent 仅在需要时加载 reference 文件。无需额外操作——渐进加载已内置。

### 若上下文压力极大

可将 loop-engineering 技能放在与大型代码库不共享上下文的项目中。它最适合规划阶段，而非活跃编码期间。

---

## 实际意味着什么

以当前 Claude Sonnet 定价计，6,500 tokens 输入约 $0.02。对多数项目，这些文件的 token 成本不是实质性支出。

更相关的成本是**上下文窗口稀释**：在已加载大量文件的超长会话中，追加 ~670 tokens（CLAUDE.md）或 ~2,800 tokens（loop 技能）会与代码上下文竞争空间。若发现 Agent 在长会话中丢失较早上下文，可考虑：

1. 为 loop 搭建任务开新会话
2. 移除已执行完毕的 reference 文件
3. 使用各工具提供的压缩/摘要功能（如 Claude Code 的 `/compact`）再加载大文件
