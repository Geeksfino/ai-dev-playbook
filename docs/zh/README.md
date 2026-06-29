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

| 工具 | Harness 层 | Loop 搭建技能（`loop-engineering`） | Loop 运行时（生成产物） |
|------|-----------|-----------------------------------|------------------------|
| **Cursor** | `.cursor/rules/playbook.mdc` | `~/.cursor/skills/`（个人）或 `.cursor/skills/`（项目） | **每个项目**内的 `.cursor/skills/loop-triage/`、`state/` 等 |
| **Claude Code** | 项目根目录 `CLAUDE.md` | `~/.claude/skills/`（个人）或 `.claude/skills/`（项目） | **每个项目**内的 `.claude/skills/loop-triage/`、`state/` 等 |
| **Codex** | `AGENTS.md` 或项目指令 | Codex 技能目录 | 各项目内的 loop 文件 |
| **其他 Agent** | 系统指令文件 | 用户级或项目级技能目录 | 各项目内的 loop 文件 |

下文安装命令中，将 `REPO` 设为 `Geeksfino/ai-dev-playbook`（或你的 fork）。

---

## 个人安装 vs 项目安装

并非所有文件都应放在同一位置。Playbook 将**搭建工具**与**运行中的 loop 文件**分开。

| 文件 | 个人（`~/.cursor/skills/`） | 项目（仓库内 `.cursor/skills/`） |
|------|---------------------------|--------------------------------|
| **`loop-engineering`**（搭建与审计） | **推荐**个人开发者跨多仓库使用 — 装一次，处处可用 | **推荐**团队 — 版本固定，clone 即有 |
| **`loop-triage`**（发现） | 不要 — 规则因仓库而异 | **始终**放在有 loop 的项目并提交 |
| **`state/`、`inbox/`、workflow、evaluator、checklist** | 不要 | **始终**按项目提交 |

**经验法则：** `loop-engineering` 装在*工具*所在处；loop *产出*装在*代码*所在处。

勿使用 `~/.cursor/skills-cursor/` — 该目录为 Cursor 内置技能保留。

---

## 选哪条路径？

```
第一次使用 AI 编程 Agent？
└── 仅从 Harness 层开始——暂时不需要其他文件

每天早上还在手动处理 CI 失败和 open issue？
└── 安装 loop-engineering 技能
   Agent 会引导你完成 7 问访谈，并为项目生成定制化的循环文件

已明确项目的分拣规则和停止条件？
└── 直接复制模板，自行编辑 CUSTOMIZE 标记
   比访谈更快；无需交互式搭建
```

技能路径与模板路径会在项目中生成相同文件、放在相同位置。两者使用**同一套 canonical 模板**——技能路径通过访谈定制后写入；模板路径由你自行编辑 `CUSTOMIZE` 标记。

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

### Loop 层 — 引导式搭建（技能路径）

`loop-engineering` 安装一次即可（个人或项目 — 见上文）。Agent 在相关时自动发现该技能；技能将 loop **运行时**文件写入**当前打开的项目**。

**Cursor** — 设置 `SKILL_HOME` 为个人或项目路径后执行：

```bash
BASE=https://raw.githubusercontent.com/$REPO/main

# 个人（推荐个人开发者）：所有 Cursor 项目可用
SKILL_HOME=~/.cursor/skills/loop-engineering

# 项目（推荐团队）：将 .cursor/skills/loop-engineering/ 提交到仓库
# SKILL_HOME=.cursor/skills/loop-engineering

mkdir -p $SKILL_HOME/references
mkdir -p $SKILL_HOME/templates/{loop-triage,evaluator-agent,state,github-actions,inbox}

curl -o $SKILL_HOME/SKILL.md $BASE/skills/loop-engineering/SKILL.md

for f in five-moves failure-modes toolchain-map; do
  curl -o $SKILL_HOME/references/$f.md \
    $BASE/skills/loop-engineering/references/$f.md
done

curl -o $SKILL_HOME/templates/loop-triage/loop-triage.md \
  $BASE/skills/loop-engineering/templates/loop-triage/loop-triage.md
curl -o $SKILL_HOME/templates/evaluator-agent/loop-reviewer.md \
  $BASE/skills/loop-engineering/templates/evaluator-agent/loop-reviewer.md
curl -o $SKILL_HOME/templates/state/triage.md \
  $BASE/skills/loop-engineering/templates/state/triage.md
curl -o $SKILL_HOME/templates/github-actions/loop-triage.yml \
  $BASE/skills/loop-engineering/templates/github-actions/loop-triage.yml
curl -o $SKILL_HOME/templates/inbox/.gitkeep \
  $BASE/skills/loop-engineering/templates/inbox/.gitkeep
curl -o $SKILL_HOME/templates/loop-checklist.md \
  $BASE/skills/loop-engineering/templates/loop-checklist.md
```

**Claude Code** — 同样使用 `SKILL_HOME`：

```bash
BASE=https://raw.githubusercontent.com/$REPO/main

# 个人：SKILL_HOME=~/.claude/skills/loop-engineering
SKILL_HOME=~/.claude/skills/loop-engineering

# 项目：SKILL_HOME=.claude/skills/loop-engineering

mkdir -p $SKILL_HOME/references
mkdir -p $SKILL_HOME/templates/{loop-triage,evaluator-agent,state,github-actions,inbox}

curl -o $SKILL_HOME/SKILL.md $BASE/skills/loop-engineering/SKILL.md

for f in five-moves failure-modes toolchain-map; do
  curl -o $SKILL_HOME/references/$f.md \
    $BASE/skills/loop-engineering/references/$f.md
done

curl -o $SKILL_HOME/templates/loop-triage/loop-triage.md \
  $BASE/skills/loop-engineering/templates/loop-triage/loop-triage.md
curl -o $SKILL_HOME/templates/evaluator-agent/loop-reviewer.md \
  $BASE/skills/loop-engineering/templates/evaluator-agent/loop-reviewer.md
curl -o $SKILL_HOME/templates/state/triage.md \
  $BASE/skills/loop-engineering/templates/state/triage.md
curl -o $SKILL_HOME/templates/github-actions/loop-triage.yml \
  $BASE/skills/loop-engineering/templates/github-actions/loop-triage.yml
curl -o $SKILL_HOME/templates/inbox/.gitkeep \
  $BASE/skills/loop-engineering/templates/inbox/.gitkeep
curl -o $SKILL_HOME/templates/loop-checklist.md \
  $BASE/skills/loop-engineering/templates/loop-checklist.md
```

**Codex** — 同样的 `SKILL.md` 与 references；放入 Codex 技能目录，通过 `$loop-engineering` 调用。

然后向 Agent 说：**「帮我为这个项目搭建一个 loop」**

技能会进行 7 问访谈（触发源、停止条件、工具链、调度偏好、预算上限、PR 还是 inbox、现有 harness 规则）。访谈结束后**从内置 `templates/` 复制模板**到项目，根据你的回答定制 `CUSTOMIZE` 标记，并写入正确路径。产出与直接复制模板路径（Path B）一致。

---

### 模板 — 直接复制（模板路径）

若已明确需求，可直接复制模板。每个模板文件顶部有 `# TEMPLATE — copy this file to:` 说明在项目中的目标路径。

**文件对应关系：**

| 模板文件 | 项目中的目标路径 | 用途 |
|---|---|---|
| `templates/loop-triage/loop-triage.md` | `.cursor/skills/loop-triage/SKILL.md` 或 `.claude/skills/loop-triage/SKILL.md` | 发现技能 — 读取 CI/issue，写入状态文件 |
| `templates/evaluator-agent/loop-reviewer.md` | `.claude/agents/loop-reviewer.md`（或你工具的 agent 配置） | 对抗式评审 — 拒绝劣质输出 |
| `templates/state/triage.md` | `state/triage.md` | 循环记忆 — 跨运行持久化发现项 |
| `templates/inbox/.gitkeep` | `inbox/.gitkeep` | 人工审核队列 — Agent 不会单独处理的发现项 |
| `templates/github-actions/loop-triage.yml` | `.github/workflows/loop-triage.yml` | 调度 — 无需本机在线的 cron 运行 |
| `templates/loop-checklist.md` | `loop-checklist.md` | 起飞前检查清单 — 无人值守运行前核对五步法 |

**安装顺序很重要。** 请按此顺序操作，否则循环没有可读的状态：

```bash
BASE=https://raw.githubusercontent.com/$REPO/main
SKILL_DIR=.cursor/skills/loop-triage   # 或 .claude/skills/loop-triage
AGENT_DIR=.claude/agents               # 或 .codex/agents/

# 步骤 1 — 状态文件与 inbox（循环记忆；首次运行前必须存在）
mkdir -p state inbox
curl -o state/triage.md $BASE/templates/state/triage.md
curl -o inbox/.gitkeep $BASE/templates/inbox/.gitkeep

# 步骤 2 — 发现技能（读取 CI/issue；写入 state/triage.md）
mkdir -p $SKILL_DIR
curl -o $SKILL_DIR/SKILL.md $BASE/templates/loop-triage/loop-triage.md

# 步骤 3 — 评估 Agent（每轮 Agent 运行后调用以验证输出）
mkdir -p $AGENT_DIR
curl -o $AGENT_DIR/loop-reviewer.md $BASE/templates/evaluator-agent/loop-reviewer.md

# 步骤 4 — 调度（触发循环；需先完成步骤 1–3）
mkdir -p .github/workflows
curl -o .github/workflows/loop-triage.yml $BASE/templates/github-actions/loop-triage.yml

# 步骤 5 — 检查清单（可选，首次无人值守运行前建议完成）
curl -o loop-checklist.md $BASE/templates/loop-checklist.md
```

**复制后、提交前请编辑：**

- 你的 loop-triage `SKILL.md` — 找到所有 `<!-- CUSTOMIZE -->` 标记，替换为项目的实际分拣规则（标签名、优先级标准、必须进 inbox 的模块）
- `.github/workflows/loop-triage.yml` — 按你的时区设置 cron，调整 `--max-tokens`，在仓库 secrets 中添加 `ANTHROPIC_API_KEY`
- `state/triage.md` — 无需编辑；原样提交，供循环读取

评估 Agent（`loop-reviewer.md`）没有 CUSTOMIZE 标记——其指令是有意为之的硬性规定，不应削弱。

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
