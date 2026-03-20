---
name: 代码重构 (Refactor)
description: 约束 Agent 必须遵循安全的重构流程：在保证原有业务逻辑和功能完全等价的前提下，提升代码质量。
assets:
  checklist: assets/checklist.yaml
  template: assets/template.yaml
references:
  guidelines: references/guidelines.yaml
  security: references/security.yaml
---

# 代码重构 (Code Refactor)

约束 Agent 必须执行极端安全的重构：在改善代码结构、抽象度或性能的同时，绝对保证功能的等价性（Functional Equivalence）。

<PIPELINE>

## 第 1 步：反转追问 (对齐前提)

在开始动手重构前，你必须逐条检查 `assets/checklist.yaml` 中的项目。

<HARD-GATE>
如果用户没有说明到底为了什么目的而重构（为了可读性？为了性能？还是为了 DRY 消除重复代码？），你必须先问清楚。
如果你无法 100% 确认哪些核心业务逻辑和对外暴露的行为是“绝对不能变的”，你必须停下来向用户确认。
绝对不允许在不知道边界和底线规则的情况下盲目挪动代码。
</HARD-GATE>

## 第 2 步：生成器 (按模板浇筑重构计划)

你的最终输出必须严格符合 `assets/template.yaml` 中定义的格式框架。
必须实质性涵盖：重构的具体目标、影响面分析说明、带有 Diff 代码的分解行动步骤，以及用来保证不会改坏代码的等效验证打钩清单。

## 第 3 步：工具围栏 (安全网)

如果在重构期间你需要在终端里跑测试跑脚本，必须先查阅 `references/security.yaml` 黑名单。
在重构过程中，绝对绝对不允许随意删掉原有的单元测试代码，或是把原本严谨的错误处理 / catch 逻辑给阉割掉（即使它们看起来很啰嗦）。

## 第 4 步：审查员 (闭环自验)

完成重构方案后，拿着 `references/guidelines.yaml` 里的标准对你的输出做一次自我审核。
核心审查点：代码的运行表现还是和原来完全一样吗？被别人调用的公共 API 签名有没有被你不小心改掉？代码复杂度真的降低了吗？
如果不合规，退回重写，最多重试 3 次。

</PIPELINE>
