---
name: 产品需求文档 (PRD)
description: 约束 Agent 遵循标准流程生成结构化、高质量且要素齐全的产品需求文档。
assets:
  checklist: assets/checklist.yaml
  template: assets/template.yaml
references:
  guidelines: references/guidelines.yaml
  security: references/security.yaml
---

# 产品需求文档 (PRD)

约束 Agent 必须按照标准流程来输出 PRD，确保其内容完整、格式合规并符合安全要求。

<PIPELINE>

## 第 1 步：反转追问 (对齐需求)

在开始生成 PRD 之前，你必须逐条检查 `assets/checklist.yaml` 中的项目。

<HARD-GATE>
如果用户没有说明目标受众和核心功能，你必须停下来向用户提问。
如果技术限制和非功能性需求不明确，你必须在写正文前向用户提问。
如果成功衡量指标和产品平台（Web/APP等）未指定，你必须停下来提问。
绝对不允许在不知道目标用户、核心功能、技术限制和平台背景的情况下，靠自己的幻觉强行生成一份 PRD。
</HARD-GATE>

## 第 2 步：生成器 (按模板浇筑 PRD)

你的输出必须严格符合 `assets/template.yaml` 中定义的 PRD 格式框架。
所有的章节（目标、用户画像、功能说明、技术限制等）都必须被实质性地填充。
不允许遗漏任何章节，不允许用 [TODO] 或无意义的占位符糊弄。

## 第 3 步：工具围栏 (安全网)

PRD 绝对不能包含任何 `references/security.yaml` 中被禁止的内容。
比如：不允许在文档示例里写死真实的客户隐私数据（姓名、邮箱、手机号），不允许在文档中建议采用明文存储密码等极其危险的方案。

## 第 4 步：审查员 (闭环自验)

在完成 PRD 草稿后，拿着 `references/guidelines.yaml` 里的标准对你的输出做一次自我审核。
如果在自审中发现违反了任何一条标准，重置状态并重新生成草稿，该自审-重试机制最多允许自动执行 3 次。

</PIPELINE>
