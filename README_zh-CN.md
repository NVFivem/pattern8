<div align="center">

# Pattern 8 (P8)

**零信任 AI Agent 治理框架：阻止 AI 产生幻觉、删库跑路以及绕过你的规则。**
<br/>

> *"你的提示词只是一种建议，而 P8 是法律。"*

[![PyPI version](https://img.shields.io/pypi/v/pattern8?color=blue&style=for-the-badge)](https://pypi.org/project/pattern8/)
[![Python](https://img.shields.io/pypi/pyversions/pattern8?style=for-the-badge)](https://pypi.org/project/pattern8/)
[![CI](https://github.com/Aquifer-sea/pattern8/actions/workflows/ci.yml/badge.svg?style=for-the-badge)](https://github.com/Aquifer-sea/pattern8/actions)
[![Coverage](https://img.shields.io/badge/Coverage-100%25-brightgreen.svg?style=for-the-badge)](https://github.com/Aquifer-sea/pattern8/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

**[ 🇺🇸 English](README.md) · [架构与源码导读](ARCHITECTURE_zh-CN.md)**
</div>

---

## 目录

- [问题背景](#混沌-vs-法律)
- [30 秒上手](#-30-秒上手指南)
- [架构总览](#-架构总览)
- [项目目录结构](#-项目目录结构)
- [核心工作原理：法律 vs. 警察](#-核心工作原理法律-vs-警察)
- [5 大强制执行模式](#-5-大强制执行模式)
- [SKILL 解剖学](#-skill-解剖学)
- [5 大内置 SKILL](#-5-大内置-skill)
- [MCP 执法引擎详解](#-mcp-执法引擎详解)
- [端到端数据流](#-端到端数据流)
- [命令行参考](#%EF%B8%8F-命令行参考)
- [IDE 接入](#-ide-接入)
- [Pre-commit 钩子](#-pre-commit-钩子)
- [自定义 SKILL](#%EF%B8%8F-自定义-skill)
- [参与贡献](#-参与贡献)
- [许可证](#-开源许可证)

---

## 混沌 vs. 法律

你是否厌倦了 AI 编程 Agent（Claude、Cursor、Devin）无视你的指令、删错文件，或者在没有写测试的情况下就提交代码？

光靠提示词（Prompt）是不够的。**防御提示词注入在理论上是不可能的。** 要真正控制一个 Agent，必须在操作系统和代码层面对其进行强制约束。

### ❌ 没有 P8 的世界 (混沌)
- **Agent 决定** 跳过编写测试，因为它觉得这"太简单了"。
- **Agent 执行** 了 `rm -rf`，导致多文件重构时误删核心代码。
- **Agent 输出** 了一个新功能，但全程没有写过一行设计文档。
- **Agent 无视** 了你写了哪怕 5000 字的 System Prompt，因为它的上下文窗口已经爆满了。

### 🛡️ 用 P8 统治代码 (法律)
- **MCP SecurityGuard（保安）** 在底层拦截并阻断危险的主机命令。
- **MCP Reviewer（审核员）** 强制 Agent 进入严格的重试循环，如果生成的代码不符合 `template.yaml` 的格式要求会被立刻打回重做。
- **Pre-commit Hooks（提交钩子）** 确保 Agent 没有暗中篡改 P8 属于"法律"本身的文件。
- **Inversion Pattern（反转模式）** 强制 Agent 在产生幻觉前停下来向你提问澄清需求。

---

## ⚡ 30 秒上手指南

只需要 3 条命令，即可对你的代码库拥有绝对的控制权：

```bash
# 1. 安装执法引擎 (需要 Python 3.8+)
pip install pattern8

# 2. 给当前项目带上「手铐」(--lang zh 生成全中文协议文件)
p8 init --lang zh

# 3. 搞定。你的 AI Agents 现在必须遵守规则了
p8 list
```

---

## 🏛️ 架构总览

P8 **不是**一个 AI Agent 框架。它不调用大模型，也不驱动流水线。  
P8 是一个**治理层** —— 一套可执行的规则文件 + 一个运行时执法引擎，约束*任何* AI Agent 在你项目中的行为方式。

```text
┌─────────────────────────────────────────────────────────────────────────┐
│                            你 的 项 目                                  │
│                                                                        │
│  ┌──────────────────────────┐    ┌──────────────────────────────────┐  │
│  │    📜 法律 (可编辑)        │    │    🚔 警察 (只读引擎)             │  │
│  │                          │    │                                  │  │
│  │  skills/                 │    │  src/p8/enforcement/             │  │
│  │  ├── prd/                │──→ │  ├── mcp_server.py    (协议网关)  │  │
│  │  ├── bug_fix/            │    │  ├── security_guard.py (命令拦截) │  │
│  │  ├── code_review/        │    │  └── reviewer.py      (审计引擎) │  │
│  │  ├── refactor/           │    │                                  │  │
│  │  └── feature_dev/        │    │  作为 MCP stdio 服务器运行         │  │
│  │                          │    │  Agent ↔ MCP ↔ 警察              │  │
│  │  AGENTS.md               │    └──────────────────────────────────┘  │
│  │  .cursor/rules/*.mdc     │                                          │
│  └──────────────────────────┘    ┌──────────────────────────────────┐  │
│                                  │    🔗 钩子 (Git 层)               │  │
│                                  │    hooks/pre-commit              │  │
│                                  └──────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

**核心设计要点**：Agent 可以读取 `SKILL.md`、`checklist.yaml` 和 `template.yaml`（"法律"）。但它**无法**读取 `guidelines.yaml` 和 `security.yaml`（"审计标准"）。这防止了 Agent 利用规则做手脚。

---

## 📂 项目目录结构

```text
pattern8/
├── src/p8/                          # Python 包 (pip install pattern8)
│   ├── __init__.py                  #   包元数据（版本号）
│   ├── cli.py                       #   CLI 入口（基于 click）
│   └── enforcement/                 #   🚔 执法引擎
│       ├── __init__.py
│       ├── mcp_server.py            #     MCP 协议网关（3 Resources + 2 Tools）
│       ├── security_guard.py        #     底层命令拦截器（正则黑名单）
│       └── reviewer.py              #     静态规则审计引擎（格式 + 规则检查）
│
├── skills/                          # 📜 内置 SKILL 规则（英文版）
│   ├── prd/                         #   产品需求文档
│   │   ├── SKILL.md                 #     流水线定义（frontmatter + 步骤）
│   │   ├── assets/
│   │   │   ├── checklist.yaml       #     反转模式：预检问题
│   │   │   └── template.yaml        #     生成器：输出格式
│   │   └── references/
│   │       ├── guidelines.yaml      #     🔒 审计规则（对 Agent 隐藏）
│   │       └── security.yaml        #     🔒 安全黑名单（对 Agent 隐藏）
│   ├── bug_fix/                     #   缺陷修复（同上结构）
│   ├── code_review/                 #   代码审查（同上结构）
│   ├── feature_dev/                 #   功能开发（同上结构）
│   └── refactor/                    #   代码重构（同上结构）
│
├── skills_zh/                       # 📜 内置 SKILL 规则（中文版）
│   └── (与 skills/ 相同结构)
│
├── hooks/
│   └── pre-commit                   # 🔗 Git 钩子：SKILL 完整性 + 密钥泄漏扫描
│
├── AGENTS.md                        # 全局 Agent 行为指令
├── .cursor/rules/
│   └── p8-enforcement.mdc          # Cursor IDE 注入规则
│
├── tests/
│   ├── test_p8.py                   # CLI + SKILL 管理测试
│   └── test_enforcement.py          # SecurityGuard + Reviewer + MCP 测试
│
├── .github/workflows/
│   └── ci.yml                       # CI：Python 3.11-3.13 × Ubuntu/macOS
│
├── pyproject.toml                   # 构建配置（hatchling）
├── CONTRIBUTING.md                  # 贡献者指南
├── CHANGELOG.md                     # 版本历史
└── LICENSE                          # MIT
```

---

## 🚔 核心工作原理：法律 vs. 警察

P8 架构的核心思想是将 **法律** (开发者可编辑的规则) 与 **警察** (只读的强制执行引擎) 彻底分离：

```text
开发者可编辑 (法律)                    只读引擎 (警察)
┌──────────────────────┐          ┌──────────────────────────┐
│ SKILL.md             │          │ SecurityGuard            │
│ checklist.yaml       │  读取 →   │  ↳ 正则黑名单拦截          │
│ template.yaml        │          │  ↳ OS 级别命令拦截         │
│ guidelines.yaml  🔒  │          │ Reviewer                 │
│ security.yaml    🔒  │          │  ↳ 静态规则审计引擎         │
│                      │          │  ↳ P8AuditError 重试回退   │
│ "项目基本法"           │          │ "执法警察"                │
└──────────────────────┘          └──────────────────────────┘
                ↕ Agent 通过 MCP 协议调用 ↕
```

标记为 🔒 的文件对 Agent 故意隐藏，防止其窥探用于审计它自身的安全参数。

---

## 🔐 5 大强制执行模式

每个 SKILL 按順序执行 **5 大模式**。这是 P8 治理哲学的脊梁：

| # | 模式 | 作用 | 配置文件 |
|---|------|------|----------|
| 1 | **Pipeline（流水线）** | 任务必须按严格顺序执行，不可跳步 | `SKILL.md` |
| 2 | **Inversion（反转）** | 开工前 Agent 必须核验所有前置条件，缺信息就**停下来问**，不准猜 | `assets/checklist.yaml` |
| 3 | **Generator（生成器）** | 输出必须严格遵循模板，每个字段必须填满，不准自由发挥 | `assets/template.yaml` |
| 4 | **Tool Wrapper（工具哨卡）** | 执行系统命令前必须过安全检查站，黑名单命令直接拒绝 | `references/security.yaml` 🔒 |
| 5 | **Reviewer（审核员）** | 完成输出后静态审计引擎打分，不合规则触发回滚+重试（最多 3 次） | `references/guidelines.yaml` 🔒 |

> **为什么模式 4 和 5 是隐藏的？** 如果 Agent 能读到审计标准，它就能"应试"—— 输出形式上通过但语义上是垃圾。隐藏审计标准保证了审查的公正性。

---

## 🧬 SKILL 解剖学

每个 SKILL 是一个自包含的目录，包含 **4 个 YAML 配置文件** 和 **1 个 Markdown 流水线定义**：

```text
skills/<skill_name>/
├── SKILL.md                    # 流水线定义（YAML frontmatter + Markdown 步骤）
├── assets/
│   ├── checklist.yaml          # 模式 2 — 反转：预检清单
│   └── template.yaml           # 模式 3 — 生成器：输出格式模板
└── references/
    ├── guidelines.yaml         # 模式 5 — 🔒 审核员：审计规则
    └── security.yaml           # 模式 4 — 🔒 保安：命令黑名单
```

### `SKILL.md` — 流水线脚本

使用 YAML frontmatter 声明元数据和文件引用，然后用 Markdown 定义每个流水线步骤：

```yaml
---
name: feature_dev
description: 约束 Agent 遵循结构化的功能开发流程
assets:
  checklist: assets/checklist.yaml
  template: assets/template.yaml
references:
  guidelines: references/guidelines.yaml
  security: references/security.yaml
---
```

步骤用 `<PIPELINE>` 标签包裹，阻断条件用 `<HARD-GATE>` 标记：

```markdown
<PIPELINE>

## 步骤 1：反转（需求对齐）
<HARD-GATE>
如果用户没有提供验收标准，你必须阻断并询问。
</HARD-GATE>

## 步骤 2：生成器（从模板生成）
输出必须严格遵循 assets/template.yaml。

## 步骤 3：工具哨卡（安全围栏）
命令必须通过 references/security.yaml 检查。

## 步骤 4：审核员（自审循环）
对照 references/guidelines.yaml 审计。不合规最多重试 3 次。

</PIPELINE>
```

### `guidelines.yaml` — 审计评分规则 (🔒 对 Agent 隐藏)

支持 4 种规则类型用于静态分析：

| 规则类型 | 说明 | 示例 |
|---------|------|------|
| `regex_match` | 必须匹配某个正则 | 代码块必须存在（`\`\`\``） |
| `regex_exclude` | 绝对不能匹配某个正则 | 不准有 `[TODO]` 或 `[placeholder]` |
| `length_limit` | 最小/最大 字符/行数限制 | 至少 150 字符，8 行 |
| `format_verify` | 必须包含指定的 Markdown 标题 | 必须有 "Implementation"、"Tests" |

---

## 💎 5 大内置 SKILL

P8 开箱即附带 5 个达到工业级标准的开发者 SKILL。*别只会写代码，要有工程思维。*

### 📝 需求设计 (PRD) (`skills/prd/`)
*不要上来就写代码，先动脑子。*  
强制 Agent 收集所有核心需求，并在写任何逻辑之前输出结构化的产品需求文档 (PRD)。

### 🐛 缺陷修复 (Bug Fix) (`skills/bug_fix/`)
*找不到根本原因，就别乱改。*  
强制 Agent 走严格的 4 步黄金流程：复现问题 → 根本原因分析 → 实施修复 → 回归测试验证。

### 🔒 代码审查 (Code Review) (`skills/code_review/`)
*永远不要合并未经审查的 AI 垃圾代码。*  
Agent 必须将其修改提交给内核的 `Reviewer` 引擎。如果不符合安全、性能或正确性规范，引擎抛出 `P8_AUDIT_FAILED` 错误，强制打回要求 Agent 自己修复自己的烂摊子（最多重试 3 次）。

### 🏗️ 代码重构 (Refactor) (`skills/refactor/`)
*只改变结构，不改变行为。*  
强制保证 Agent 无论怎么挪动代码，最后所有的测试都必须 100% 绿灯通过。

### 🚀 特性开发 (Feature Dev) (`skills/feature_dev/`)
*端到端的闭环交付。*
彻底覆盖：需求同步 → 技术设计方案 → 代码实现 → 单元测试。

---

## 🔌 MCP 执法引擎详解

执法引擎（`src/p8/enforcement/`）作为 **MCP stdio 服务器** 运行。连接 IDE 后对外暴露：

### Resources（认知基础 — Agent 启动时读取）

| URI | 说明 | 数据来源 |
|-----|------|----------|
| `skill://index` | 列出所有可用 SKILL 及描述 | 扫描 `skills/*/SKILL.md` |
| `skill://{name}/skill_md` | 完整的 SKILL.md 流水线定义 | `skills/{name}/SKILL.md` |
| `skill://{name}/checklist` | 反转模式预检清单 | `skills/{name}/assets/checklist.yaml` |
| `skill://{name}/template` | 输出模板 | `skills/{name}/assets/template.yaml` |

### Tools（执法检查站 — Agent 必须调用）

#### `execute_tool` — 操作系统命令沙箱

```json
{
  "command": "npm install lodash",
  "path": "./src/",
  "operation": "write",
  "skill": "feature_dev"
}
```

**内部处理链**（对 Agent 不可见）：
1. 加载指定 SKILL 的 `references/security.yaml`
2. `SecurityGuard.check_command(command)` — 逐条匹配正则黑名单
3. `SecurityGuard.check_path(path, operation)` — 校验路径是否允许
4. 返回 `{"allowed": true}` 或 `{"allowed": false, "action": "BLOCKED"}`

#### `submit_review` — 输出审计关卡

```json
{
  "content": "# Feature Development Report\n## 1. Requirement Understanding\n...",
  "skill": "feature_dev"
}
```

**内部处理链**（对 Agent 不可见）：
1. 加载 `references/guidelines.yaml`（隐藏的审计规则）
2. 加载 `assets/template.yaml`（格式参照）
3. `Reviewer.audit(content)` 执行 4 类检查：
   - **格式验证**：所有必要的 Markdown 标题是否存在？
   - **正则匹配/排除**：模式是否匹配/不匹配？
   - **长度限制**：内容是否足够充实？
4. 全部通过 → `{"passed": true, "score": 100, "status": "APPROVED"}`
5. 任一失败 → 抛出 `P8AuditError` → 返回 `P8_AUDIT_FAILED` + 违规列表

### 自纠错循环

```text
Agent 完成工作
        │
        ▼
  submit_review()
        │
    ┌───┴───┐
    │ 通过？ │
    └───┬───┘
   是   │  否
    │   │   │
    ▼   │   ▼
 APPROVED  P8_AUDIT_FAILED
           + 违规列表
                │
                ▼
        Agent 阅读违规列表，
        修正输出，
        重新提交（最多 3 次）
                │
           ┌────┴────┐
           │ 3次失败  │
           └────┬────┘
                ▼
        Agent 向用户报告
        失败详情
```

---

## 🔄 端到端数据流

以下是 Agent 在 P8 治理项目中工作的完整流程：

```text
步骤 1：Agent 读取 skill://index          → 发现可用 SKILL
步骤 2：Agent 读取 skill://X/checklist    → 获取预检清单
步骤 3：Agent 核验清单项                    → 缺信息就向用户提问（反转模式）
步骤 4：Agent 读取 skill://X/template     → 学习输出格式要求
步骤 5：Agent 开始工作...
        └─ 执行任何 OS 命令前：
           execute_tool(command, skill)    → SecurityGuard 检查正则黑名单
           ├─ allowed: true  → 放行
           └─ allowed: false → 阻断，Agent 必须停止
步骤 6：Agent 完成输出
        └─ submit_review(content, skill)   → Reviewer 对照隐藏规则审计
           ├─ passed: true  → 通过，交付给用户
           └─ P8_AUDIT_FAILED → Agent 自纠正并重新提交（最多 3 次）
```

---

## 🛠️ 命令行参考

| 命令 | 说明 |
|------|------|
| `p8 init [target]` | 在项目中初始化 P8 并生成 5 个默认 SKILL |
| `p8 init --lang zh` | 初始化时生成全中文配置规则 |
| `p8 list` | 列出当前项目中所有可用的 SKILL |
| `p8 validate <skill_path>` | 验证 SKILL 文件完整性（编辑 YAML 后运行） |
| `p8 new <skill_name>` | 为新业务创建 SKILL 骨架 |
| `p8 serve` | 启动 MCP 执法服务器（stdio 模式） |
| `p8 mcp-config --client cursor` | 生成适配 Cursor IDE 的 MCP 配置 JSON |
| `p8 --version` | 显示已安装版本 |

---

## 🔌 IDE 接入

### Cursor / Windsurf / Claude Desktop

想要开启活跃的"警察"拦截引擎：

```bash
# 安装包含 MCP server 支持的完整版
pip install 'pattern8[enforcement]'

# 自动生成适配 Cursor 的 MCP 配置文件
p8 mcp-config --client cursor
```

将命令输出的内容粘贴到 `.cursor/mcp.json`。现在 Cursor Agent 每次执行命令或完成任务时，**必须**通过 P8 的 `execute_tool` 与 `submit_review` 拦截安检口。

### Cursor Rules 工作机制

P8 还会安装 `.cursor/rules/p8-enforcement.mdc` 文件，该文件**自动注入到每一次 Agent 对话中**，指令包括：

1. 启动时读取 `skill://index` 发现 SKILL
2. 执行 OS 命令前必须调用 `execute_tool()`
3. 完成 SKILL 流水线后必须调用 `submit_review()`
4. 禁止直接读取 `guidelines.yaml` 或 `security.yaml`
5. 未经用户明确许可，不得修改 `skills/` 或 `AGENTS.md`

### AGENTS.md

项目根目录的 `AGENTS.md` 是全局指令文件，被支持项目级配置的 Agent（Cursor、Windsurf 等）自动读取，告知 Agent：
- 开始任何任务前先检查 `skills/` 中是否有匹配的 SKILL
- 严格遵循 5 大模式流水线
- 将安全红线视为绝对不可违反的硬约束

---

## 🔗 Pre-commit 钩子

P8 安装了 Git pre-commit 钩子，在每次 `git commit` 时自动运行：

```bash
# 钩子执行内容：
1. 验证所有 SKILL 文件完整性 (p8 validate)
2. 扫描暂存文件中是否有硬编码的密钥（API key、密码、token）
3. 发现问题则阻断提交
```

手动安装：
```bash
cp hooks/pre-commit .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

或通过 `p8 init` 自动安装。

---

## 🏗️ 自定义 SKILL

```bash
# 创建新 SKILL 骨架
p8 new my_custom_skill --lang zh

# 将创建：
# skills/my_custom_skill/
# ├── SKILL.md              ← 编辑流水线步骤
# ├── assets/
# │   ├── checklist.yaml    ← 编辑预检问题
# │   └── template.yaml     ← 编辑输出格式要求
# └── references/
#     ├── guidelines.yaml   ← 编辑审计规则
#     └── security.yaml     ← 编辑命令黑名单
```

然后根据团队治理需求编辑每个文件。运行 `p8 validate skills/my_custom_skill` 检查完整性。

---

## 🧪 测试

```bash
# 安装开发依赖
pip install -e ".[dev]"

# 运行全部测试（59 个测试，100% 覆盖率）
pytest tests/ -v

# 测试覆盖：
# - CLI 命令（init, list, validate, new）
# - SecurityGuard（黑名单匹配、路径拦截）
# - Reviewer（格式检查、规则审计、P8AuditError）
# - MCP 服务器（资源读取、工具路由）
```

CI 在每次 push/PR 到 `main` 时自动运行：
- **Python**: 3.11, 3.12, 3.13
- **OS**: Ubuntu, macOS

---

## 🤝 参与贡献

我们非常欢迎开发者贡献全新的 SKILL 模板！详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

想深入理解 Python 源码逻辑的中国开发者，请查阅 [《体系架构与源码导读》](ARCHITECTURE_zh-CN.md)。

## 📄 开源许可证

MIT
