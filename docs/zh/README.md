> [English](../../README.md) · **中文**

# AI Dev Playbook

面向 AI 辅助软件开发的两个互补层次：**Harness（约束层）** 控制单次 Agent 运行的行为，**Loop（循环层）** 技能在你需要时搭建自动化 Agent 系统。

两者相互独立——可以只用其一、两者都用、或都不用——但设计上可以干净地叠加。

---

## 仓库内容

| 文件 | 层次 | 何时生效 | 解决什么问题 |
|------|------|---------|-------------|
| `CLAUDE.md` | Harness | 每次会话，始终加载 | 静默假设、过度设计、无关改动 |
| `skills/loop-engineering/` | Loop | 按需调用 | 缺乏搭建自动化 Agent 循环的结构 |
| `templates/` | Loop（快捷方式） | 复制粘贴 | 跳过访谈，直接获得预填好的循环文件 |

---

## 两个层次

**`CLAUDE.md` 工作在 Harness 层。** 它约束单次 Agent 运行的行为——不要过度设计、做外科手术式修改、先思考再写代码、声明你的假设。它始终在上下文中，每次会话约消耗 ~670 tokens，适用于 Agent 执行的每一项任务。

**loop-engineering 技能工作在 Loop 层。** 仅当开发者明确要求搭建或审计自动化循环时才会被调用。平时处于休眠状态，触发后该次会话约消耗 ~2,800 tokens。

两者不冲突。`CLAUDE.md` 的规则实际上会**加强**技能产出的循环：

- **§4 外科手术式修改** 直接防止评估 Agent 改动其分配工作区之外的代码
- **§6 目标驱动执行** 与技能的停止条件访谈完全一致——在开始前把「修 bug」变成可验证的条件
- **§2 先思考再写代码** 对应技能的 7 问访谈关卡——都要求「在生成任何东西之前声明假设」

---

## 三层叠加结构

同时部署两者时，层次结构如下：

```
CLAUDE.md（或等效的 Harness 规则）  ← 始终生效，单次 Agent 行为
  └── loop-engineering/SKILL.md       ← 按需激活，搭建循环
        └── loop-triage/SKILL.md      ← 生成的产物，在循环内运行
```

loop-engineering 生成的 **Artifact 1（triage `SKILL.md`）** 本身也是一个技能——专属于你项目的发现（Discovery）技能。每一层都独立，但设计上可以叠在下一层之上。

---

## 适用哪些工具？

本 Playbook **不**绑定 Claude Code 或 Codex。行为规则和循环模式适用于任何支持持久化指令和技能的 Agent。

| 工具 | Harness 层 | Loop 技能 |
|------|-----------|----------|
| **Cursor** | `.cursor/rules/playbook.mdc`（`alwaysApply: true`） | `.cursor/skills/loop-engineering/` |
| **Claude Code** | 项目根目录的 `CLAUDE.md` | `.claude/skills/loop-engineering/` |
| **Codex** | `AGENTS.md` 或项目指令 | `.codex/skills/` 或通过 `$skill-name` 调用 |
| **其他 Agent** | 各工具的系统指令文件 | 将 `SKILL.md` 复制到对应技能目录 |

下文安装命令中，将 `REPO` 设为 `Geeksfino/ai-dev-playbook`（或你的 fork）。

---

## 安装

### Harness 层 — 始终生效的 Agent 行为

**Cursor** — 创建项目规则：

```bash
mkdir -p .cursor/rules
curl -o .cursor/rules/playbook.mdc \
  https://raw.githubusercontent.com/$REPO/main/CLAUDE.md
```

若 `playbook.mdc` 顶部尚无 frontmatter，请添加：

```yaml
---
description: AI dev playbook — harness 层行为规则
alwaysApply: true
---
```

**Claude Code** — 放入项目根目录（会话开始时自动读取）：

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/$REPO/main/CLAUDE.md
```

若已有 `CLAUDE.md`，改为追加：

```bash
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/$REPO/main/CLAUDE.md >> CLAUDE.md
```

**Codex / 其他工具** — 将 `CLAUDE.md` 内容复制到对应的项目指令文件（`AGENTS.md`、工作区规则等）。

---

### Loop 层 — 按需搭建循环

**Cursor：**

```bash
mkdir -p .cursor/skills/loop-engineering/references

curl -o .cursor/skills/loop-engineering/SKILL.md \
  https://raw.githubusercontent.com/$REPO/main/skills/loop-engineering/SKILL.md

for f in five-moves failure-modes toolchain-map; do
  curl -o .cursor/skills/loop-engineering/references/$f.md \
    https://raw.githubusercontent.com/$REPO/main/skills/loop-engineering/references/$f.md
done
```

然后向 Agent 说：**「帮我为这个项目搭建一个 loop」**，技能会接管后续流程。

**Claude Code：**

```bash
mkdir -p .claude/skills/loop-engineering/references

curl -o .claude/skills/loop-engineering/SKILL.md \
  https://raw.githubusercontent.com/$REPO/main/skills/loop-engineering/SKILL.md

for f in five-moves failure-modes toolchain-map; do
  curl -o .claude/skills/loop-engineering/references/$f.md \
    https://raw.githubusercontent.com/$REPO/main/skills/loop-engineering/references/$f.md
done
```

**Codex** — 同样的 `SKILL.md` 与 references；放入 Codex 技能目录，通过 `$loop-engineering` 调用。

---

### 模板 — 跳过访谈

若已明确需求、不需要交互式搭建：

```bash
# 发现技能（Artifact 1 — 路径按你的工具调整）
mkdir -p .cursor/skills/loop-triage   # 或 .claude/skills/loop-triage
curl -o .cursor/skills/loop-triage/SKILL.md \
  https://raw.githubusercontent.com/$REPO/main/templates/loop-triage/SKILL.md

# 评估 Agent
mkdir -p .claude/agents               # 或 .codex/agents/
curl -o .claude/agents/loop-reviewer.md \
  https://raw.githubusercontent.com/$REPO/main/templates/evaluator-agent/loop-reviewer.md

# 状态文件与收件箱
mkdir -p state inbox
curl -o state/triage.md \
  https://raw.githubusercontent.com/$REPO/main/templates/state/triage.md
touch inbox/.gitkeep

# GitHub Actions 调度（与具体 Agent 工具无关）
mkdir -p .github/workflows
curl -o .github/workflows/loop-triage.yml \
  https://raw.githubusercontent.com/$REPO/main/templates/github-actions/loop-triage.yml
```

提交前请按项目修改各文件。

---

## 我需要哪些文件？

```
第一次使用 AI 编程 Agent？
└── 是 → 仅从 Harness 层开始（CLAUDE.md 或 .cursor/rules/）

每天早上还在手动处理 CI / Issue / PR？
└── 是 → 添加 loop-engineering 技能

希望 Agent 在你睡觉时自动运行、无需手动触发？
└── 是 → 复制模板并按项目修改
```

---

## 这些文件不会做什么

- 不会给项目添加依赖。
- 安装时不会执行任何代码。
- 不会回传数据或要求注册账号。
- 不会替换你已有的 Harness 规则——可以干净地追加。
- 不会提供 API Key——需自行配置。

---

## Token 成本

| 文件 | 加载时机 | 大约成本 |
|------|---------|---------|
| `CLAUDE.md` | 每次会话 | ~670 tokens（固定开销） |
| `loop-engineering/SKILL.md` | 请求搭建循环时 | ~2,800 tokens |
| `references/*.md` | 技能需要时 | 各 ~950–1,400 tokens |
| 模板 | 模型不会加载 | 0 tokens |

详见 [Token 成本](token-costs.md)。

---

## 文档

| English | 中文 |
|---------|------|
| [README](../../README.md) | 本文 |
| [Why these exist](../why-these-exist.md) | [为什么需要这些文件](why-these-exist.md) |
| [FAQ](../faq.md) | [常见问题](faq.md) |
| [Token costs](../token-costs.md) | [Token 成本](token-costs.md) |

---

## 来源

`CLAUDE.md` 源自 Andrej Karpathy 2026 年 1 月关于 LLM 编程陷阱的观察，由 Forrest Chang 提炼为 `andrej-karpathy-skills` 仓库；本仓库版本从 184 行精简至 90 行（约减少 63% token），同时保留全部行为信号。

loop-engineering 技能源自 IEEE 工作笔记 *Loop Engineering: The Anthropic Playbook*（HuaShu，2026 年 6 月），将其五步法框架与六部分解剖结构落地为可部署的项目产物。

详见 [为什么需要这些文件](why-these-exist.md)。

---

## 许可证

MIT。复制、Fork、修改、使用均可。注明出处更好，非强制。
