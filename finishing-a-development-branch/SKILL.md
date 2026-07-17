---
name: finishing-a-development-branch
description: 实现已完成，所有测试通过，需要决定如何整合工作时使用 — 引导完成开发工作，呈现结构化合并、PR 或清理选项
---

# 完成开发分支

## 概述

通过呈现清晰的选项并处理所选工作流来引导开发工作完成。

**核心原则：** 验证测试 → 检测环境 → 呈现选项 → 执行选择 → 清理。

**开头声明：** "我正在使用 finishing-a-development-branch 技能来完成此工作。"

## 流程

### 第一步：验证测试

**在呈现选项之前，验证测试通过：**

```bash
# 运行项目测试套件
npm test / cargo test / pytest / go test ./...
```

**如果测试失败：**
```
测试失败（<N> 个失败）。完成前必须修复：

[显示失败]

测试不通过之前无法继续合并/PR。
```

停止。不要继续到第二步。

**如果测试通过：** 继续到第二步。

### 第二步：检测环境

**在呈现选项之前确定工作区状态：**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

这决定了显示哪个菜单以及如何清理：

| 状态 | 菜单 | 清理 |
|------|------|------|
| `GIT_DIR == GIT_COMMON`（正常仓库） | 标准 4 个选项 | 无需清理工作树 |
| `GIT_DIR != GIT_COMMON`，命名分支 | 标准 4 个选项 | 基于来源（见第六步） |
| `GIT_DIR != GIT_COMMON`，分离 HEAD | 缩减 3 个选项（无合并） | 无需清理（外部管理） |

### 第三步：确定基础分支

```bash
# 尝试常见的基础分支
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

或询问："这个分支是从 main 分出来的 — 对吗？"

### 第四步：呈现选项

**正常仓库和命名分支工作树 — 恰好呈现这 4 个选项：**

```
实现已完成。你想怎么做？

1. 合并回 <基础分支>（本地）
2. 推送并创建拉取请求
3. 保持分支不变（稍后处理）
4. 丢弃此工作

请选择？
```

**分离 HEAD — 恰好呈现这 3 个选项：**

```
实现已完成。你处于分离 HEAD 状态（外部管理工作区）。

1. 作为新分支推送并创建拉取请求
2. 保持现状（稍后处理）
3. 丢弃此工作

请选择？
```

**不要添加解释** — 保持选项简洁。

### 第五步：执行选择

#### 选项 1：本地合并

```bash
# 获取主仓库根目录以确保 CWD 安全
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"

# 先合并 — 在删除任何东西之前验证成功
git checkout <基础分支>
git pull
git merge <功能分支>

# 验证合并后的测试结果
<测试命令>

# 合并成功后才清理工作树（第六步），然后删除分支
```

然后：清理工作树（第六步），然后删除分支：

```bash
git branch -d <功能分支>
```

#### 选项 2：推送并创建 PR

```bash
# 推送分支
git push -u origin <功能分支>
```

**不要清理工作树** — 用户需要它来处理 PR 反馈。

#### 选项 3：保持现状

报告："保持分支 <名称>。工作树保留在 <路径>。"

**不要清理工作树。**

#### 选项 4：丢弃

**先确认：**
```
这将永久删除：
- 分支 <名称>
- 所有提交：<提交列表>
- 工作树位于 <路径>

输入'discard'确认。
```

等待精确确认。

如果确认：
```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
```

然后：清理工作树（第六步），然后强制删除分支：
```bash
git branch -D <功能分支>
```

### 第六步：清理工作区

**仅对选项 1 和 4 运行。** 选项 2 和 3 始终保留工作树。

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

**如果 `GIT_DIR == GIT_COMMON`：** 正常仓库，无需清理工作树。完成。

**如果工作树路径在 `.worktrees/` 或 `worktrees/` 下：** Superpowers 创建了此工作树 — 我们负责清理。

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
git worktree remove "$WORKTREE_PATH"
git worktree prune  # 自愈：清理任何过期注册
```

**否则：** 宿主环境（harness）拥有此工作区。不要删除它。如果你的平台提供了 workspace-exit 工具，使用它。否则，将工作区留在原处。

## 速查表

| 选项 | 合并 | 推送 | 保留工作树 | 清理分支 |
|------|------|------|-----------|---------|
| 1. 本地合并 | 是 | - | - | 是 |
| 2. 创建 PR | - | 是 | 是 | - |
| 3. 保持现状 | - | - | 是 | - |
| 4. 丢弃 | - | - | - | 是（强制） |

## 常见错误

**跳过测试验证**
- **问题：** 合并损坏的代码，创建失败的 PR
- **修复：** 提供选项前始终验证测试

**开放式问题**
- **问题：** "接下来做什么？" 有歧义
- **修复：** 恰好呈现 4 个结构化选项（或分离 HEAD 的 3 个）

**为选项 2 清理工作树**
- **问题：** 删除用户需要迭代 PR 的工作树
- **修复：** 仅对选项 1 和 4 清理

**在删除工作树之前删除分支**
- **问题：** `git branch -d` 失败因为工作树仍引用该分支
- **修复：** 先合并，移除工作树，然后删除分支

**从工作树内部运行 git worktree remove**
- **问题：** 当 CWD 在被删除的工作树内时命令静默失败
- **修复：** 始终在 `git worktree remove` 之前 `cd` 到主仓库根目录

**清理由 harness 拥有的工作树**
- **问题：** 删除 harness 创建的工作树会导致幻影状态
- **修复：** 仅清理 `.worktrees/` 或 `worktrees/` 下的工作树

**丢弃时没有确认**
- **问题：** 意外删除工作
- **修复：** 要求输入 "discard" 确认

## 红旗

**绝不做：**
- 在测试失败时继续
- 未在结果上验证测试就合并
- 未确认就删除工作
- 未经明确请求就强制推送
- 在确认合并成功之前移除工作树
- 清理不是你创建的工作树（来源检查）
- 从工作树内部运行 `git worktree remove`

**始终做：**
- 提供选项前验证测试
- 呈现菜单前检测环境
- 恰好呈现 4 个选项（或分离 HEAD 的 3 个）
- 对选项 4 获取输入确认
- 仅对选项 1 和 4 清理工作树
- 在工作树移除之前 `cd` 到主仓库根目录
- 移除后运行 `git worktree prune`
