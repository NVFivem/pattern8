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

**[ 🇺🇸 English](README.md)**
</div>

---

## 混沌 vs. 法律

你是否厌倦了 AI 编程 Agent（Claude、Cursor、Devin）无视你的指令、删错文件，或者在没有写测试的情况下就提交代码？

光靠提示词（Prompt）是不够的。**防御提示词注入在理论上是不可能的。** 要真正控制一个 Agent，必须在操作系统和代码层面对其进行强制约束。

### ❌ 没有 P8 的世界 (混沌)
- **Agent 决定** 跳过编写测试，因为它觉得这“太简单了”。
- **Agent 执行** 了 `rm -rf`，导致多文件重构时误删核心代码。
- **Agent 输出** 了一个新功能，但全程没有写过一行设计文档。
- **Agent 无视** 了你写了哪怕 5000 字的 System Prompt，因为它的上下文窗口已经爆满了。

### 🛡️ 用 P8 统治代码 (法律)
- **MCP SecurityGuard（保安）** 在底层拦截并阻断危险的主机命令。
- **MCP Reviewer（审核员）** 强制 Agent 进入严格的重试循环，如果生成的代码不符合 `template.yaml` 的格式要求会被立刻打回重做。
- **Pre-commit Hooks（提交钩子）** 确保 Agent 没有暗中篡改 P8 属于“法律”本身的文件。
- **Inversion Pattern（反转模式）** 强制 Agent 在产生幻觉前停下来向你提问澄清需求。

---

## ⚡ 30 秒上手指南

只需要 3 条命令，即可对你的代码库拥有绝对的控制权：

```bash
# 1. 安装执法引擎 (需要 Python 3.8+)
pip install pattern8

# 2. 给当前项目带上「手铐」
p8 init --lang zh

# 3. 搞定。你的 AI Agents 现在必须遵守规则了
p8 list
```
*(注：添加 `--lang zh` 参数将自动生成全中文注释的协议文件)*

---

## 🚔 工作原理：法律 vs. 警察

P8 架构的核心思想是将 **法律** (开发者可编辑的规则) 与 **警察** (只读的强制执行引擎) 彻底分离：

```text
开发者可编辑 (法律)                    只读引擎 (警察)
┌──────────────────────┐          ┌──────────────────────────┐
│ SKILL.md             │          │ SecurityGuard            │
│ checklist.yaml       │  读取 →   │  ↳ 正则黑名单拦截拦截          │
│ template.yaml        │          │  ↳ OS 级别命令拦截           │
│ guidelines.yaml  🔒  │          │ Reviewer                 │
│ security.yaml    🔒  │          │  ↳ 静态规则审计引擎           │
│                      │          │  ↳ P8AuditError 重试回退   │
│ "项目基本法"           │          │ "执法警察"                  │
└──────────────────────┘          └──────────────────────────┘
                ↕ Agent 通过 MCP 协议调用 ↕
```

你只需要用极其简单的 Markdown 和 YAML 编写 **法律**。引擎 **警察** 会通过 MCP (模型上下文协议) 自动执行它们。标记为 🔒 的文件对 Agent 故意隐藏，防止其窥探用于审计它自身的安全参数。

想要深入了解 Python 源码逻辑的中国开发者，请查阅 [《体系架构与源码导读》](ARCHITECTURE_zh-CN.md)。

---

## 💎 5 大工业级 SKILL（护法）

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

## 🛠️ 命令行参考

```bash
p8 init [target] --lang zh      # 在目录初始化 P8 并生成全中文配置规则
p8 list                         # 列出当前项目中所有可用的 SKILL
p8 validate <skill_path>        # 在手动修改 YAML 后，验证配置文件是否写错
p8 new <skill_name> --lang zh   # 为你的业务创建一个全新的防患规则骨架
```

---

## 🔌 接入 Cursor / Windsurf / Claude Desktop

想要开启活跃的 "Police（警察）" 拦截引擎，你需要安装 P8 的 MCP 扩展：

```bash
# 安装包含 MCP server 支持的完整版
pip install 'pattern8[enforcement]'

# 自动生成适配 Cursor 的 MCP 配置文件
p8 mcp-config --client cursor
```

将上面命令输出的内容粘贴到你项目根目录的 `.cursor/mcp.json` 里。现在，Cursor Agent 每次尝试运行命令或终结任务时，**绝对必须** 通过 P8 的 `execute_tool` 与 `submit_review` 拦截安检口。

---

## 🤝 参与贡献

我们非常欢迎开发者贡献全新的 SKILL 模板！想了解内部工作原理请参阅 [ARCHITECTURE_zh-CN.md](ARCHITECTURE_zh-CN.md) 以及英文版的 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 📄 开源许可证

MIT
