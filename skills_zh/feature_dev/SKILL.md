---
name: 特性开发 (Feature Dev)
description: 约束 Agent 必须遵循结构化的功能特性开发流程：理解需求 → 技术设想 → 编写代码 → 自验自测。
assets:
  checklist: assets/checklist.yaml
  template: assets/template.yaml
references:
  guidelines: references/guidelines.yaml
  security: references/security.yaml
---

# 特性开发 (Feature Dev)

约束 Agent 必须老老实实地走“理解需求 → 技术设计与设想 → 编写实现代码 → 编写并运行自测”这套严谨的研发闭环规范。

<PIPELINE>

## 第 1 步：反转追问 (对齐前提)

在开始写任何一行新功能的代码前，你必须逐条检查 `assets/checklist.yaml` 中的项目。

<HARD-GATE>
如果用户没有提供“验收标准 (Acceptance Criteria)”，你必须停下来提问。
如果开发目前功能所要求的技术栈、框架不明确，你必须在下手前提问。
在没拿到验收标准和技术栈要求之前，绝对不允许凭空开始敲代码。
</HARD-GATE>

## 第 2 步：生成器 (按模板浇筑研发报告)

你的输出必须严格符合 `assets/template.yaml` 中定义的格式框架。
必须实质性涵盖：对需求的重述理解、技术设计方案、具体的实现代码、能跑起来的测试用例代码，以及如何集成改动的注意事项。

## 第 3 步：工具围栏 (安全网)

你在跑一些终端命令（例如安装包、执行脚本）时，必须先查阅 `references/security.yaml` 黑名单。
绝对不要为了图省事引入有已知高危漏洞的第三方库，也绝对不要在写新功能时去绕过项目的身份认证机制。

## 第 4 步：审查员 (闭环自验)

写完代码后，拿着 `references/guidelines.yaml` 里的标准对你的输出做一次自我审核。
重点死抠的点：你写的代码和用户的一开始提的“验收标准”是不是100%匹配的？你写测试了吗？这次改动兼不兼容历史老代码？
如果不合规，退回重写，最多重试 3 次。

</PIPELINE>
