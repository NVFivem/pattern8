---
name: 缺陷修复 (Bug Fix)
description: 约束 Agent 必须遵循结构化的 Bug 修复流程：定位 → 根因分析 → 实施修复 → 回归测试验证。
assets:
  checklist: assets/checklist.yaml
  template: assets/template.yaml
references:
  guidelines: references/guidelines.yaml
  security: references/security.yaml
---

# 缺陷修复 (Bug Fix)

约束 Agent 必须遵循严谨的 "定位 → 根因分析 → 实施修复 → 回归测试验证" 这套结构化工业级流程来修 Bug。

<PIPELINE>

## 第 1 步：反转追问 (对齐前提)

在开始修 Bug 之前，你必须逐条检查 `assets/checklist.yaml` 中的项目。

<HARD-GATE>
如果用户没有提供准确的报错信息或者复现步骤，你必须停下来提问。
绝对不允许在 Bug 描述不清、无法复现的情况下，盲目地去代码里乱改一通。
</HARD-GATE>

## 第 2 步：生成器 (按模板浇筑修复报告)

你的输出必须严格符合 `assets/template.yaml` 中定义的格式框架。
必须实质性包含：根因分析、修复方案、修复代码 (Diff 格式)、影响面评估、回归测试清单。
不允许给出类似于“你可以尝试改一下这个变量”这样模棱两可的模糊建议。

## 第 3 步：工具围栏 (安全网)

如果你打算在系统中执行命令来复现 Bug 或运行测试验证，必须先查阅 `references/security.yaml`。
绝对不允许执行任何可能破坏现有代码文件或搞崩系统环境的高危指令。

## 第 4 步：审查员 (闭环自验)

完成修复方案后，拿着 `references/guidelines.yaml` 里的标准对你的方案做一次自我审核。
核心审查点：修复策略是否真的解决了“根本原因”？是否引入了新的 Bug？代码变动 (Diff) 是否做到了最小化侵入？
如果不合规，退回重写，最多重试 3 次。

</PIPELINE>
