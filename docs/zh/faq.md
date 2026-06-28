# 常见问题

**Andrej Karpathy 写了 CLAUDE.md 吗？**

没有——至少不是直接写的。Karpathy 在 2025 年底在 X 上发布了关于 LLM 编程陷阱的观察。开发者 Forrest Chang 在 `andrej-karpathy-skills` 仓库中将其转化为 CLAUDE.md。措辞是 Chang 的；诊断来自 Karpathy。本仓库版本是对 Chang 作品的进一步精简。完整来源链见 [为什么需要这些文件](why-these-exist.md)。

---

**我需要同时使用 CLAUDE.md 和 loop 技能吗？**

不需要。它们工作在不同层次，各自独立。
`CLAUDE.md` 约束每次会话中的单次 Agent 行为。
loop 技能仅在你希望 Agent 按调度自动运行、无需手动 prompt 时才有意义。多数项目会立刻受益于 CLAUDE.md；当手动 prompt 成为瓶颈时，loop 技能才变得相关。

---

**CLAUDE.md 能在 Cursor / Windsurf / 其他编辑器里用吗？**

可以。`CLAUDE.md` 中的行为规则适用于任何接受持久化系统指令的 Agent。

- **Cursor：** 复制到 `.cursor/rules/playbook.mdc`，设置 `alwaysApply: true`
- **Claude Code：** 放在项目根目录的 `CLAUDE.md`
- **Codex：** 复制到 `AGENTS.md` 或项目指令
- **其他工具：** 查阅文档找到等效配置文件

loop-engineering 技能是 `SKILL.md` 格式，可放入 Cursor（`.cursor/skills/`）、Claude Code（`.claude/skills/`）或 Codex 的技能目录。核心逻辑与工具无关；只有路径和调用方式不同。

---

**会让简单任务变慢吗？**

CLAUDE.md 的 §2（先思考再写代码）在实现前增加澄清步骤。对一行修复会有一点摩擦；对多文件架构变更能省数小时。若简单任务上摩擦令人不适，可以说「直接做」——实践中会覆盖该规则。

---

**可以在 CLAUDE.md 里加项目特定规则吗？**

可以。加在最后一节之后。现有规则是通用的；你的补充应是项目特定的（命名约定、禁止模式、测试位置等）。

---

**循环一直在自我点头——评估器总是通过。为什么？**

点头循环（Nodding Loop）是最常见的失败模式。原因：

1. 评估 Agent 没有独立指令——同一 Agent 重读自己的输出。修复：使用 `templates/evaluator-agent/loop-reviewer.md` 作为独立的评估 Agent 配置（Cursor：`.cursor/agents/` 或项目约定；Claude Code：`.claude/agents/`）。

2. 评估器读代码而不执行。修复：评估器模板明确要求运行测试并粘贴实际输出。

3. 停止条件由完成工作的同一模型检查。修复：使用独立评估轮次或 `/goal`（Claude Code）等由新上下文检查条件的机制。

若评估器在真实代码库上超过 50 轮从未产出 REJECT，它就没有正常工作。

---

**循环出问题怎么停？**

- **GitHub Actions：** 在 Actions 页取消 workflow。workflow 有 30 分钟 `timeout-minutes` 作为安全网。
- **Claude Code `/loop`：** 输入 `/stop` 或关闭会话。
- **Cursor Automations / 自定义调度：** 在对应 UI 或 workflow 中取消运行。
- **Worktree：** 每个发现项在隔离 worktree 中运行。即使某个 Agent 出错，也不会影响其他 worktree 或主分支（PR 不会自动合并）。

triage 技能模板中的 `Stop` 部分编码了循环绝不能做的事。部署前请阅读并确保符合项目风险承受度。

---

**应该把 CLAUDE.md 提交到仓库吗？**

应该——若你希望团队里所有使用 AI Agent 的人都遵循相同规则。该文件是项目级配置，不是个人偏好。

若只有你自己适用的规则，放在用户级配置（如 `~/.claude/CLAUDE.md` 或 Cursor 用户规则），而非项目根目录。

---

**模型变强后这些文件会过时吗？**

随着模型更可靠地内化这些行为，`CLAUDE.md` 的需求会逐渐减少。loop-engineering 技能会随各工具原语变化而演进（`/goal`、`/loop`、worktree、Cursor Automations 等）。可关注本仓库更新。

底层原则——生成器与评估器分离、状态持久化在上下文窗口外、不跳过验证——不太可能错。实现细节（具体命令、文件路径）会随工具更新而变化。
