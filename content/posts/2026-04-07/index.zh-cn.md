---
title: "用5W3H解释clarify技能有什么用"
date: 2026-04-07T10:20:47+08:00
lastmod:
draft: false
tags: ["polanyi", "维特根斯坦", "苏格拉底", "思考模式", "skill", "AI"]
categories:
description: ""
---

## what 是什么？

[clarify-skill](https://github.com/riiiku/clarify-skill) 是一个命题分解与决策澄清系统。工作原理如下：
外层 · 维特根斯坦（扫描）
  → 发现哪里模糊，把复合表述拆成原子单元

中层 · 苏格拉底（挖掘）
  → 用追问把已有但未表达的想法挖出来

内层 · 波兰尼（感应）
  → 处理追问后仍说不清的默会知识
  → 示范法 / 反面法 / 行为提取法

我对其做的定制化修改：[leeyorke/clarify](https://github.com/leeyorke/leeyorke-skills/blob/main/skills/clarify/SKILL.md)
## why 为什么要它？

维特根斯坦说：语言是思想的边界，波兰尼说：你所知道的远比你能做的要多。故
人有时候很难准确表达自己的想法/感觉给他人或者AI，所以需要进一步澄清，以便让表述清晰，准确，易理解。

## where 干什么事的时候用？

把你的一个想法转述为文章、与AI交流、你要实现一个想法但你的表述不够明了。
写测试用例？写prompt?

## when 何时用？

在你有一个想法但是自己说不清的时候，在你的想法难以被他人准确理解的时候。

## who 什么人可以用？

任何人。

## how 如何使用？

- `/clarify`, 自动判断进入哪个模式
- `/clarify prompt`, `/clarify -p`「帮我理清一下」「我想做个事但说不清楚」 → 强制进入 - Prompt 模式
- `/clarify decision`, `/clarify -d` 或 「帮我澄清」「我有点纠结」 → 强制进入 - Decision 模式
- `/clarify memo`, `/clarify -m` 或 「帮我记一下」 → 强制进入 memo 模式

### Prompt 模式
**把模糊想法拆解成精准指令**。适用于你想让 AI 帮你做一件事，但说不清楚你的意图。

流程：命题分类（F/D/Q） → 模糊词拆解 → 苏格拉底三问 → 波兰尼兜底 → 组装结构化指令

### Decision 模式
**把说不清楚的困扰、纠结、冲动问清楚。** Claude 不替你决定，只帮你挖出你自己的答案。

流程：收集场景 → 语言澄清 → 递进追问 → 波兰尼兜底 → 反馈结论 → 行动锚点

## how much 成本如何？

零成本。

## how feel 效果如何？
迭代优化中