<div align="center">

# Pattern 8 (P8)

**Zero-Trust Governance Framework to stop AI Agents from hallucinating, breaking things, and bypassing your rules.**
<br/>

> *"Your prompt is merely a suggestion. P8 is the law."*

[![PyPI version](https://img.shields.io/pypi/v/pattern8?color=blue&style=for-the-badge)](https://pypi.org/project/pattern8/)
[![Python](https://img.shields.io/pypi/pyversions/pattern8?style=for-the-badge)](https://pypi.org/project/pattern8/)
[![CI](https://github.com/Aquifer-sea/pattern8/actions/workflows/ci.yml/badge.svg?style=for-the-badge)](https://github.com/Aquifer-sea/pattern8/actions)
[![Coverage](https://img.shields.io/badge/Coverage-100%25-brightgreen.svg?style=for-the-badge)](https://github.com/Aquifer-sea/pattern8/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

**[ 🇨🇳 简体中文](README_zh-CN.md) · [Architecture (源码导读)](ARCHITECTURE_zh-CN.md)**
</div>

---

## Table of Contents

- [The Problem](#the-chaos-vs-the-law)
- [Quick Start](#-zero-to-hero-in-30-seconds)
- [Architecture](#-architecture-overview)
- [Project Structure](#-project-structure)
- [How It Works: Law vs. Police](#-how-it-works-law-vs-police)
- [The 5 Patterns](#-the-5-enforcement-patterns)
- [Anatomy of a SKILL](#-anatomy-of-a-skill)
- [The 5 Built-in SKILLs](#-the-5-built-in-skills)
- [MCP Enforcement Engine](#-mcp-enforcement-engine-deep-dive)
- [Data Flow](#-end-to-end-data-flow)
- [CLI Reference](#%EF%B8%8F-cli-reference)
- [IDE Integration](#-ide-integration)
- [Pre-commit Hooks](#-pre-commit-hooks)
- [Creating Custom SKILLs](#-creating-custom-skills)
- [Contributing](#-contributing)
- [License](#-license)

---

## The Chaos vs. The Law

Are you tired of AI coding agents (Claude, Cursor, Devin) ignoring your instructions, deleting the wrong files, or pushing code without tests? 

Prompts are not enough. **Prompt injection defence is impossible.** To truly control an agent, constraints must be enforced at the OS and code level.

### ❌ Without P8 (The Chaos)
- **Agent decides** to skip writing tests because it's "too trivial".
- **Agent runs** `rm -rf` by mistake during a multi-step refactor.
- **Agent outputs** a feature without ever writing a design doc.
- **Agent ignores** your 5,000-word system prompt because its context window is full.

### 🛡️ With P8 (The Law)
- **MCP SecurityGuard** intercepts and blocks dangerous commands at the OS level.
- **MCP Reviewer** forces the agent into a strict retry-loop if output doesn't match the `template.yaml`.
- **Pre-commit Hooks** ensure the agent hasn't tampered with the rules themselves.
- **Inversion Pattern** forces the agent to stop and ask you clarifying questions instead of hallucinating.

---

## ⚡ Zero to Hero in 30 Seconds

Take absolute control of your codebase with 3 commands:

```bash
# 1. Install the enforcer (Python 3.8+)
pip install pattern8

# 2. Add handcuffs to your current project
p8 init

# 3. Done. Your Agents are now under control.
p8 list
```

> 💡 For Chinese-language teams: `p8 init --lang zh` generates all SKILL files with Chinese annotations.

---

## 🏛️ Architecture Overview

P8 is **NOT** an AI Agent framework. It does not call LLMs or drive pipelines.  
P8 is a **governance layer** — a set of enforceable rule files + a runtime enforcement engine that constrains *how* any AI Agent works on your project.

```text
┌─────────────────────────────────────────────────────────────────────────┐
│                            YOUR PROJECT                                │
│                                                                        │
│  ┌──────────────────────────┐    ┌──────────────────────────────────┐  │
│  │    📜 LAW (Editable)      │    │    🚔 POLICE (Read-Only Engine)  │  │
│  │                          │    │                                  │  │
│  │  skills/                 │    │  src/p8/enforcement/             │  │
│  │  ├── prd/                │──→ │  ├── mcp_server.py    (Gateway) │  │
│  │  ├── bug_fix/            │    │  ├── security_guard.py (Block)  │  │
│  │  ├── code_review/        │    │  └── reviewer.py      (Audit)  │  │
│  │  ├── refactor/           │    │                                  │  │
│  │  └── feature_dev/        │    │  Runs as MCP stdio server       │  │
│  │                          │    │  Agent ↔ MCP ↔ Police           │  │
│  │  AGENTS.md               │    └──────────────────────────────────┘  │
│  │  .cursor/rules/*.mdc     │                                          │
│  └──────────────────────────┘    ┌──────────────────────────────────┐  │
│                                  │    🔗 HOOKS (Git-level)          │  │
│                                  │    hooks/pre-commit              │  │
│                                  └──────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

**Key Insight**: The Agent can read `SKILL.md`, `checklist.yaml`, and `template.yaml` (the "law"). But it **cannot** read `guidelines.yaml` or `security.yaml` (the "audit criteria"). This prevents the Agent from gaming the audit.

---

## 📂 Project Structure

```text
pattern8/
├── src/p8/                          # Python package (pip install pattern8)
│   ├── __init__.py                  #   Package metadata (version)
│   ├── cli.py                       #   CLI entry point (click-based)
│   └── enforcement/                 #   🚔 Enforcement engine
│       ├── __init__.py
│       ├── mcp_server.py            #     MCP protocol gateway (3 Resources + 2 Tools)
│       ├── security_guard.py        #     OS-level command blocker (regex blacklist)
│       └── reviewer.py              #     Static rule audit engine (format + rules)
│
├── skills/                          # 📜 Built-in SKILL rules (English)
│   ├── prd/                         #   Product Requirements Document
│   │   ├── SKILL.md                 #     Pipeline definition (frontmatter + steps)
│   │   ├── assets/
│   │   │   ├── checklist.yaml       #     Inversion: pre-flight questions
│   │   │   └── template.yaml        #     Generator: output format
│   │   └── references/
│   │       ├── guidelines.yaml      #     🔒 Reviewer audit rules (hidden from Agent)
│   │       └── security.yaml        #     🔒 SecurityGuard blacklist (hidden from Agent)
│   ├── bug_fix/                     #   Bug Fix (same structure)
│   ├── code_review/                 #   Code Review (same structure)
│   ├── feature_dev/                 #   Feature Development (same structure)
│   └── refactor/                    #   Refactoring (same structure)
│
├── skills_zh/                       # 📜 Built-in SKILL rules (Chinese)
│   └── (same structure as skills/)
│
├── hooks/
│   └── pre-commit                   # 🔗 Git hook: SKILL integrity + secret scan
│
├── AGENTS.md                        # Global agent behavior instructions
├── .cursor/rules/
│   └── p8-enforcement.mdc          # Cursor IDE injection rules
│
├── tests/
│   ├── test_p8.py                   # CLI + SKILL management tests
│   └── test_enforcement.py          # SecurityGuard + Reviewer + MCP tests
│
├── .github/workflows/
│   └── ci.yml                       # CI: pytest on Python 3.11-3.13 × Ubuntu/macOS
│
├── pyproject.toml                   # Build config (hatchling)
├── CONTRIBUTING.md                  # Contributor guide
├── CHANGELOG.md                     # Version history
└── LICENSE                          # MIT
```

---

## 🚔 How It Works: Law vs. Police

P8 separates **Law** (editable rules) from **Police** (read-only execution engine):

```text
Developer-editable (Law)              Read-only Engine (Police)
┌──────────────────────┐          ┌──────────────────────────┐
│ SKILL.md             │          │ SecurityGuard            │
│ checklist.yaml       │  read →  │  ↳ regex blacklist       │
│ template.yaml        │          │  ↳ OS command hooks      │
│ guidelines.yaml  🔒  │          │ Reviewer                 │
│ security.yaml    🔒  │          │  ↳ static rule engine    │
│                      │          │  ↳ P8AuditError rollback │
│ "The Constitution"   │          │ "The Police"             │
└──────────────────────┘          └──────────────────────────┘
                ↕ Agent calls via MCP ↕
```

You write the **Law** in simple Markdown and YAML. The **Police** engine enforces them automatically via MCP (Model Context Protocol). Files marked with 🔒 are deliberately hidden from the Agent so it cannot read the security parameters used to audit it.

---

## 🔐 The 5 Enforcement Patterns

Every SKILL enforces **5 patterns** in sequence. These are the backbone of P8's governance philosophy:

| # | Pattern | What It Does | Controlled By |
|---|---------|-------------|---------------|
| 1 | **Pipeline** | Tasks execute in a strict ordered sequence. No step may be skipped. | `SKILL.md` |
| 2 | **Inversion** | Before starting, the Agent must verify all preconditions. If info is missing, it **stops and asks** — no guessing. | `assets/checklist.yaml` |
| 3 | **Generator** | Output must follow a strict template. Every section must be filled. No freestyle. | `assets/template.yaml` |
| 4 | **Tool Wrapper** | Before executing OS commands, the Agent must pass through a security checkpoint. Blacklisted operations are rejected. | `references/security.yaml` 🔒 |
| 5 | **Reviewer** | After completing output, a static audit engine scores the result. Non-compliant output triggers rollback + retry (up to 3×). | `references/guidelines.yaml` 🔒 |

> **Why are patterns 4 and 5 hidden?** If the Agent can read the exact audit criteria, it can game the system by producing output that technically passes but is semantically garbage. By hiding them, the audit stays honest.

---

## 🧬 Anatomy of a SKILL

Each SKILL is a self-contained directory with **4 YAML config files** and **1 Markdown pipeline definition**:

```text
skills/<skill_name>/
├── SKILL.md                    # Pipeline definition (YAML frontmatter + Markdown steps)
├── assets/
│   ├── checklist.yaml          # Pattern 2 — Inversion: pre-flight checklist
│   └── template.yaml           # Pattern 3 — Generator: output format template
└── references/
    ├── guidelines.yaml         # Pattern 5 — 🔒 Reviewer: audit rules
    └── security.yaml           # Pattern 4 — 🔒 SecurityGuard: command blacklist
```

### `SKILL.md` — The Pipeline Script

Uses YAML frontmatter to declare metadata + file references, then Markdown to define each pipeline step:

```yaml
---
name: feature_dev
description: Constrains the Agent to follow structured feature development.
assets:
  checklist: assets/checklist.yaml
  template: assets/template.yaml
references:
  guidelines: references/guidelines.yaml
  security: references/security.yaml
---
```

Steps are wrapped in `<PIPELINE>` tags, with `<HARD-GATE>` for blocking conditions:

```markdown
<PIPELINE>

## Step 1: Inversion (Requirements Alignment)
<HARD-GATE>
If the user has not provided acceptance criteria, you must block and ask.
</HARD-GATE>

## Step 2: Generator (Generate from Template)
Your output must strictly follow `assets/template.yaml`.

## Step 3: Tool Wrapper (Security Fence)
Commands must be checked against `references/security.yaml`.

## Step 4: Reviewer (Self-Audit Loop)
Audit against `references/guidelines.yaml`. Retry up to 3x if non-compliant.

</PIPELINE>
```

### `checklist.yaml` — What the Agent Must Verify

```yaml
checklist:
  - "Feature requirements are clearly described with user stories"
  - "Technology stack and framework are specified"
  - "Acceptance criteria are defined"
  - "Edge cases and error scenarios are considered"
```

### `template.yaml` — What the Output Must Look Like

```yaml
template: |
  # Feature Development Report
  ## 1. Requirement Understanding
  ## 2. Technical Design
  ## 3. Implementation
  ## 4. Tests
  ## 5. Integration Notes
```

### `security.yaml` — What's Forbidden (🔒 Hidden from Agent)

```yaml
tool_security:
  blacklist:
    - "rm -rf *"
    - "sudo *"
    - "curl * | sh"
  denied_paths:
    - "/etc"
    - "~/.ssh"
    - "~/.aws"
  max_shell_timeout: 30
```

### `guidelines.yaml` — How Output Is Scored (🔒 Hidden from Agent)

Supports 4 rule types for static analysis:

| Rule Type | Description | Example |
|-----------|-------------|---------|
| `regex_match` | Must match a regex pattern | Code blocks must exist (`\`\`\``) |
| `regex_exclude` | Must NOT match a regex | No `[TODO]` or `[placeholder]` |
| `length_limit` | Min/max character/line count | At least 150 chars, 8 lines |
| `format_verify` | Required Markdown headings | Must have "Implementation", "Tests" |

---

## 💎 The 5 Built-in SKILLs

P8 comes with 5 industrial-grade developer SKILLs out of the box. *Don't just code. Engineer.*

### 📝 PRD (`skills/prd/`)
*Don't just build. Think first.*  
Forces the agent to gather requirements and generate a structured Product Requirements Document before writing a single line of logic.

### 🐛 Bug Fix (`skills/bug_fix/`)
*Find the root cause, or don't fix it at all.*  
Forces the agent through a strict 4-step golden path: Reproduce → Root Cause Analysis → Fix → Regression Test. 

### 🔒 Code Review (`skills/code_review/`)
*Never merge unreviewed AI slop.*  
The agent must submit its changes to the `Reviewer` engine. If the code fails security, performance, or correctness guidelines, the engine throws a `P8_AUDIT_FAILED` error, forcing the agent to retry and fix its own mess (up to 3 times) before presenting it to you.

### 🏗️ Refactor (`skills/refactor/`)
*Change structure, not behavior.*  
Forces the agent to guarantee functional equivalence test-passes after moving code around.

### 🚀 Feature Dev (`skills/feature_dev/`)
*End-to-end delivery.*
Requirements → Technical Design → Implementation → Unit Tests.

---

## 🔌 MCP Enforcement Engine Deep Dive

The enforcement engine (`src/p8/enforcement/`) runs as a **MCP stdio server**. When connected to an IDE (Cursor, Windsurf, Claude Desktop), it exposes:

### Resources (Cognitive Base — Agent reads on startup)

| URI | Description | Source File |
|-----|-------------|-------------|
| `skill://index` | Lists all available SKILLs with names and descriptions | Scans `skills/*/SKILL.md` |
| `skill://{name}/skill_md` | Full SKILL.md pipeline definition | `skills/{name}/SKILL.md` |
| `skill://{name}/checklist` | Inversion checklist items | `skills/{name}/assets/checklist.yaml` |
| `skill://{name}/template` | Output template | `skills/{name}/assets/template.yaml` |

### Tools (Enforcement Checkpoints — Agent must call these)

#### `execute_tool` — OS Command Sandbox

```json
{
  "command": "npm install lodash",
  "path": "./src/",
  "operation": "write",
  "skill": "feature_dev"
}
```

**Internal chain** (invisible to Agent):
1. Loads `references/security.yaml` for the specified SKILL
2. `SecurityGuard.check_command(command)` — matches against regex blacklist
3. `SecurityGuard.check_path(path, operation)` — validates path is allowed
4. Returns `{"allowed": true}` or `{"allowed": false, "action": "BLOCKED"}`

#### `submit_review` — Output Audit Gate

```json
{
  "content": "# Feature Development Report\n## 1. Requirement Understanding\n...",
  "skill": "feature_dev"
}
```

**Internal chain** (invisible to Agent):
1. Loads `references/guidelines.yaml` (hidden audit rules)
2. Loads `assets/template.yaml` (format reference)
3. `Reviewer.audit(content)` runs 4 check types:
   - **Format verify**: Are all required Markdown headings present?
   - **Regex match/exclude**: Do patterns match/not match?
   - **Length limit**: Is the content long enough?
4. All pass → `{"passed": true, "score": 100, "status": "APPROVED"}`
5. Any fail → Throws `P8AuditError` → returns `P8_AUDIT_FAILED` + violation list

### The Self-Correction Loop

```text
Agent completes work
        │
        ▼
  submit_review()
        │
    ┌───┴───┐
    │ PASS? │
    └───┬───┘
   Yes  │  No
    │   │   │
    ▼   │   ▼
 APPROVED  P8_AUDIT_FAILED
           + violation list
                │
                ▼
        Agent reads violations,
        fixes its output,
        resubmits (up to 3×)
                │
           ┌────┴────┐
           │ 3 fails │
           └────┬────┘
                ▼
        Agent reports failure
        to user with details
```

---

## 🔄 End-to-End Data Flow

Here's what happens when an Agent works on a task in a P8-governed project:

```text
Step 1: Agent reads skill://index          → Discovers available SKILLs
Step 2: Agent reads skill://X/checklist    → Gets pre-flight checklist
Step 3: Agent verifies checklist items     → ASKS user if anything missing (Inversion)
Step 4: Agent reads skill://X/template     → Learns required output format
Step 5: Agent starts working...
        └─ Before any OS command:
           execute_tool(command, skill)    → SecurityGuard checks regex blacklist
           ├─ allowed: true  → proceed
           └─ allowed: false → BLOCKED, Agent must stop
Step 6: Agent finishes output
        └─ submit_review(content, skill)   → Reviewer audits against hidden rules
           ├─ passed: true  → APPROVED, deliver to user
           └─ P8_AUDIT_FAILED → Agent self-corrects and resubmits (up to 3×)
```

---

## 🛠️ CLI Reference

| Command | Description |
|---------|-------------|
| `p8 init [target]` | Initialize P8 in a project and generate the 5 default SKILLs |
| `p8 init --lang zh` | Initialize with Chinese-language SKILL files |
| `p8 list` | List all available SKILLs in the current project |
| `p8 validate <skill_path>` | Validate SKILL file integrity (run after editing YAML) |
| `p8 new <skill_name>` | Create a scaffold for a new custom SKILL |
| `p8 serve` | Start the MCP enforcement server (stdio mode) |
| `p8 mcp-config --client cursor` | Generate MCP config JSON for Cursor IDE |
| `p8 --version` | Show installed version |

---

## 🔌 IDE Integration

### Cursor / Windsurf / Claude Desktop

To turn on the active "Police" enforcement engine, install the MCP extension:

```bash
# Full install with MCP server support
pip install 'pattern8[enforcement]'

# Generate MCP config for Cursor
p8 mcp-config --client cursor
```

Paste the output into `.cursor/mcp.json`. Now, every time the Cursor Agent tries to run a command or finish a task, it **must** pass through the P8 `execute_tool` and `submit_review` checkpoints.

### How Cursor Rules Work

P8 also installs a `.cursor/rules/p8-enforcement.mdc` file that is **automatically injected into every Agent conversation**. This file instructs the Agent to:

1. Read `skill://index` on startup to discover SKILLs
2. Call `execute_tool()` before running any OS command
3. Call `submit_review()` after completing a SKILL pipeline
4. Never read `guidelines.yaml` or `security.yaml` directly
5. Never modify `skills/` or `AGENTS.md` without explicit user permission

### AGENTS.md

The `AGENTS.md` file at the project root is a global instruction file read by Agents that support project-level config (Cursor, Windsurf, etc.). It tells the Agent to:
- Check `skills/` for matching SKILLs before starting any task
- Follow the 5-pattern pipeline strictly
- Respect security red lines as absolute hard constraints

---

## 🔗 Pre-commit Hooks

P8 installs a Git pre-commit hook that runs automatically on every `git commit`:

```bash
# What the hook does:
1. Validates all SKILL file integrity (p8 validate)
2. Scans staged files for hardcoded secrets (API keys, passwords, tokens)
3. Blocks the commit if any issues are found
```

Install manually:
```bash
cp hooks/pre-commit .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

Or automatically via `p8 init`.

---

## 🏗️ Creating Custom SKILLs

```bash
# Scaffold a new SKILL
p8 new my_custom_skill

# This creates:
# skills/my_custom_skill/
# ├── SKILL.md              ← Edit pipeline steps
# ├── assets/
# │   ├── checklist.yaml    ← Edit pre-flight questions
# │   └── template.yaml     ← Edit required output format
# └── references/
#     ├── guidelines.yaml   ← Edit audit rules
#     └── security.yaml     ← Edit command blacklist
```

Then edit each file to match your team's governance needs. Run `p8 validate skills/my_custom_skill` to check integrity.

---

## 🧪 Testing

```bash
# Install dev dependencies
pip install -e ".[dev]"

# Run all tests (59 tests, 100% coverage)
pytest tests/ -v

# Tests cover:
# - CLI commands (init, list, validate, new)
# - SecurityGuard (blacklist matching, path blocking)
# - Reviewer (format checking, rule auditing, P8AuditError)
# - MCP server (resource reading, tool routing)
```

CI runs automatically on every push/PR against `main`, testing on:
- **Python**: 3.11, 3.12, 3.13
- **OS**: Ubuntu, macOS

---

## 🤝 Contributing

We welcome completely new SKILLs! See [CONTRIBUTING.md](CONTRIBUTING.md) for architectural details and how to open a PR.

For Chinese developers looking to deep-dive into the source code, see [ARCHITECTURE_zh-CN.md](ARCHITECTURE_zh-CN.md).

## 📄 License

MIT
