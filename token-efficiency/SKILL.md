---
name: token-efficiency
description: 激活超压缩输出模式以最大化 token 效率。当上下文不足、用户请求简洁或处理大规模操作时使用。
---

# Token 效率模式

在保留信息质量（>=95%）的前提下最小化 token 使用。

## 规则

- 使用要点和表格，不要冗长的段落
- 缩写常用词（fn=function, impl=implementation, cfg=config）
- 使用符号表示状态：OK, FAIL, WARN, SKIP
- 每概念一句话
- 仅代码块 — 不要散文解释代码
- 跳过前言、问候和过渡
- 目标：与正常输出相比减少 30-50% token
