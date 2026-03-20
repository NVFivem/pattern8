---
name: 代码审查 (Code Review)
description: 约束 Agent 从正确性、安全性、性能和可维护性四个维度进行系统化的结构性代码审查。
assets:
  checklist: assets/checklist.yaml
  template: assets/template.yaml
references:
  guidelines: references/guidelines.yaml
  security: references/security.yaml
---

# 代码审查 (Code Review)

约束 Agent 必须按照标准流程进行结构化的代码审查，审查范围必须强制覆盖四大维度：正确性、安全性、性能和可维护性。

<PIPELINE>

## 第 1 步：反转追问 (对齐前提)

在开始审查之前，你必须逐条检查 `assets/checklist.yaml` 中的项目。

<HARD-GATE>
如果用户没有提供要审查的代码片段或文件路径，你必须停下来提问。绝对不要靠猜。
如果无法确认编程语言和使用的框架，你必须在动手审查前提问。
在确认以上所有清单项之前，绝对不允许开始审查代码。
</HARD-GATE>

## 第 2 步：生成器 (按模板浇筑审查报告)

你的输出必须严格符合 `assets/template.yaml` 中定义的格式框架。
你必须覆盖全四大维度：正确性、安全性、性能和可维护性。
任何一个板块都不允许跳过或遗漏。

## 第 3 步：工具围栏 (安全网)

如果你打算在终端里执行这段代码以验证问题，你必须先查阅 `references/security.yaml`。
任何命中黑名单的命令（比如 `rm -rf`, `sudo`）必须被立即阻断拒绝。

## 第 4 步：审查员 (闭环自验)

在生成审查报告后，拿着 `references/guidelines.yaml` 里的标准对报告做一次自我审核。
如果在自审中发现违反了任何一条标准（例如：没有指出具体的行号，或者没有给出修复代码），立刻退回重写。
最多重试 3 次。如果重试 3 次依然不违规，则如实向用户报告是哪一条审查规则失败了。

</PIPELINE>
