# 代码风格

## 命名约定

| 实体 | 约定 | 示例 |
|------|------|------|
| 变量 / 函数 | camelCase | `getUserProfile`, `itemCount` |
| 类型 / 接口 / 类 | PascalCase | `UserProfile`, `DatabaseClient` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRIES`, `API_BASE_URL` |
| 文件 / 目录 | kebab-case | `user-service.ts`, `auth-middleware/` |
| 私有成员 | 下划线前缀 | `_internalState` |

## 文件组织

- **每文件单一职责。** 文件超过 200 行时拆分。
- **就近放置胜过抽象。** 相关代码放一起，不要按类型分组（把所有 service 放一个文件夹）。
- **index 文件只暴露公共 API。** 绝不泄露内部实现。
- **文件名反映内容而非模式。** `validate-email.ts` 而非 `email-validator.ts`。

## 函数设计

- **函数最大 30 行**（不含空行和注释）。
- **单一出口。** 在末尾返回，不要分散 return。
- **0-3 个参数。** 超过 3 个 → 改用 options 对象。
- **命名导出优于默认导出。** 始终使用命名导出，便于重构和 tree-shaking。

```typescript
// 正确
export function createUser(data: CreateUserInput): User { ... }

// 错误
export default class UserCreator { ... }
```

## 导入顺序

分组并按字母排序：

```typescript
// 1. 标准库
import { join, resolve } from 'path';

// 2. 外部包
import { z } from 'zod';
import { Pool } from 'pg';

// 3. 内部模块
import { DatabaseClient } from '@/database/client';
import { validateEmail } from '@/utils/validate-email';
```

## 注释规范

- **注释解释为什么（WHY），而非做什么（WHAT）。** 代码本身已经展示了做什么。
- **不要注释噪音。** 跳过显而易见的注释：`// 计数器加 1`。
- **公共 API 使用 JSDoc。** 包含参数类型、返回类型和一行描述。
- **TODO 附带上下文。** `// TODO(@作者): 替换为缓存 — 当前导致 N+1 查询`

```typescript
/**
 * 获取用户资料并返回缓存头像。
 * @param id - 用户 UUID
 * @returns 包含解析后头像 URL 的用户资料
 */
export async function getUserProfile(id: string): Promise<UserProfile> { ... }
```

## 错误处理

- **快速失败。** 在边界处校验（API 入口、文件 I/O）。
- **类型化错误。** 自定义错误类带错误码，不用裸字符串。
- **不吞掉错误。** 始终记录或重新抛出。绝不使用 `catch {}`。

```typescript
class ValidationError extends Error {
  constructor(message: string, public code: 'INVALID_EMAIL' | 'TOO_SHORT') {
    super(message);
    this.name = 'ValidationError';
  }
}
```

## 反模式

| 反模式 | 问题 | 修复 |
|--------|------|------|
| 上帝对象（>500 行） | 职责过多 | 提取内聚的代码组 |
| 嵌套三元表达式 | 难以阅读 | 改用早期返回或策略对象 |
| 函数布尔标志 | 组合爆炸 | 使用 options 对象或拆分为多个函数 |
| 深层嵌套（>3 层） | 难以跟踪 | 提取辅助函数 |
| 魔法字符串 / 数字 | 脆弱、难改 | 命名常量 |
