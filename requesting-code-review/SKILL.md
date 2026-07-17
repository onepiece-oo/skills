---
name: requesting-code-review
description: 完成任务、实现主要功能或合并前验证工作时使用，用于在合并前验证工作是否满足需求
---

# 请求代码审查

派发一个代码审查者子代理来在问题级联之前发现问题。审查者获得精确编写的上下文进行评估 — 永远不是你的会话历史。这让审查者专注于工作成果，而不是你的思维过程，并为你的持续工作保留你自己的上下文。

**核心原则：** 早审查，常审查。

## 何时请求审查

**必须：**
- subagent-driven-development 中每个任务之后
- 完成主要功能之后
- 合并到 main 之前

**可选但有价值：**
- 卡住时（新的视角）
- 重构之前（基线检查）
- 修复复杂 bug 之后

## 如何请求

**1. 获取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派发代码审查者子代理：**

派发一个 `general-purpose` 子代理，填写 [code-reviewer.md](code-reviewer.md) 中的模板

**占位符：**
- `{DESCRIPTION}` — 你构建了什么的简要摘要
- `{PLAN_OR_REQUIREMENTS}` — 它应该做什么
- `{BASE_SHA}` — 起始提交
- `{HEAD_SHA}` — 结束提交

**3. 根据反馈行动：**
- 立即修复 Critical 问题
- 在继续之前修复 Important 问题
- 记录 Minor 问题供稍后处理
- 如果审查者错了用理由反驳

## 示例

```
[刚刚完成 Task 2: 添加验证函数]

你：让我在继续之前请求代码审查。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[派发代码审查者子代理]
  DESCRIPTION：添加了 verifyIndex() 和 repairIndex() 以及 4 种问题类型
  PLAN_OR_REQUIREMENTS：docs/superpowers/plans/deployment-plan.md 中的 Task 2
  BASE_SHA：a7981ec
  HEAD_SHA：3df7661

[子代理返回]：
  优点：干净的架构，真实测试
  问题：
    Important：缺少进度指示器
    Minor：报告间隔的魔法数字（100）
  评估：可以继续

你：[修复进度指示器]
[继续到 Task 3]
```

## 与工作流的集成

**Subagent-Driven Development：**
- 每个任务后审查
- 在问题累积之前发现问题
- 修复后再移动到下一个任务

**执行计划：**
- 每个任务或自然检查点后审查
- 获取反馈，应用，继续

**临时开发：**
- 合并前审查
- 卡住时审查

## 红旗

**绝不做：**
- 因为"很简单"就跳过审查
- 忽略 Critical 问题
- 在未修复 Important 问题时继续
- 与有效的技术反馈争论

**如果审查者错了：**
- 用技术理由反驳
- 展示证明它工作的代码/测试
- 请求澄清

模板位置：[code-reviewer.md](code-reviewer.md)
