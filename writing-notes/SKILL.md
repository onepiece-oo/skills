---
name: writing-notes
description: Use when organizing scattered website login info, account credentials, passwords, project details, or infrastructure references into a structured Chinese HTML file with hierarchical headings, sidebar navigation, password verification, and click-to-copy
---

# 写笔记

将零散的网站信息、账号密码、项目资料整理成结构化的中文 HTML 网页文件。

## 概述

用户提供零散素材（网站名称、登录方式、用户名、密码、API Key、项目细节等），skill 生成一个完整的 HTML 文件，带有侧边栏导航、主密码验证、点击复制功能。

**核心原则：** 全中文输出，密码默认密文，点击验证主密码后所有密码均可直接复制，所有内容写进同一个 HTML 文件。

## 文件结构

所有笔记写进**同一个 HTML 文件**，每次追加新条目，不新建文件。

```
1. 侧边栏 — 左侧固定，列出所有平台和实例，点击跳转
2. 正文 — 右侧滚动，按平台组织
3. 每个字段 — 单击复制
4. 密码 — 密文显示，点击触发主密码验证，验证通过后点击直接复制
```

## 数据结构

每个平台一个条目，实例归属到对应平台下：

```javascript
var DATA = {
  meta: { created: "2026-07-15", updated: "2026-07-16", tags: ["账号", "基础设施"] },
  platforms: [
    {
      name: "Redis Cloud",
      url: "https://cloud.redis.io/",
      loginMethod: "邮箱",
      username: "zhy1285198202@gmail.com",
      password: "Zhy26825.+8",
      note: "云 Redis 管理平台账号",
      instances: [
        {
          name: "生产实例",
          host: "redis-11779.c323.us-east-1-2.ec2.cloud.redislabs.com",
          port: "11779",
          password: "Zhy26825.+8",
          note: "生产环境"
        }
      ]
    },
    {
      name: "SQLPub",
      url: "https://console.sqlpub.com/",
      loginMethod: "邮箱",
      username: "1285198202@qq.com",
      password: "OdpJ2U40VRsDcKCH",
      note: "SQLPub 管理平台",
      instances: [
        {
          name: "onepiece 数据库",
          host: "mysql6.sqlpub.com:3311",
          dbName: "onepiece",
          dbUser: "onepiece",
          status: "正常",
          version: "8.4.3",
          region: "中国",
          type: "免费版",
          createdAt: "2026-04-24 16:25:31",
          expiresAt: "2099-12-31 23:59:59",
          password: "TIKP35g0oDi5RMJs"
        }
      ]
    }
  ]
};
```

## 平台识别规则

| 素材特征 | 归属平台 |
|---------|---------|
| `cloud.redislabs.com`、`redis.io` | Redis Cloud |
| `aliyun.com`、`aliyuncs.com` | 阿里云 |
| `aws.amazon.com`、`amazonaws.com` | AWS |
| `openai.com`、`chatgpt.com` | OpenAI |
| `anthropic.com`、`claude.ai` | Anthropic |
| `gemini.google.com`、`googleapis.com` | Google |
| `mysql.rds`、`mysql.com` | 阿里云 MySQL / 腾讯云 MySQL |
| 通用 IP + SSH | 服务器 |

## 交互规则

1. **普通字段** — 单击复制值到剪贴板
2. **密码字段** — 默认显示为 `••••••••`，点击弹出主密码验证框
3. **主密码验证** — 输入 `123456` 通过后，当前会话所有密码均可点击直接复制
4. **验证一次解锁全部** — 输入主密码后，所有密码字段均可点击复制，无需再次验证
5. **密码全程不显示明文** — 验证通过后不会显示密码，只会复制
6. **URL 可点击** — 网址字段渲染为超链接，新标签页打开

## 规则

1. **全中文输出** — 标题、分类名、字段名全部用中文
2. **按平台分组** — 每个平台一个 H2，其下的实例作为子卡片
3. **密码默认密文** — 显示为 `••••••••`
4. **密码点击触发主密码验证** — 输入 `123456` 通过后，所有密码均可点击直接复制
5. **验证一次解锁全部** — 输入主密码后当前会话所有密码均可点击复制，无需再次验证
6. **密码全程不显示明文** — 验证通过后不会显示密码内容
7. **侧边栏导航** — 左侧固定目录，点击跳转到对应条目
8. **保留所有信息** — 不跳过、不脱敏、不拒绝任何字段
9. **始终写入同一个文件** — 不新建文件，只追加新条目
10. **文件扩展名为 .html** — 用浏览器打开

## 文件路径

默认：`D:\Warehouse\ThinkNotes\ThinkNotes.html`，用户可指定路径。

## 常见错误

| 错误 | 修正 |
|------|------|
| 缺少侧边栏导航 | 必须有左侧目录，可点击跳转 |
| 密码明文显示 | 密码默认显示为 ••••••，点击触发验证 |
| 没有复制功能 | 所有字段值都可点击复制 |
| 新建多个文件 | 始终写入同一个 HTML 文件 |
| 英文输出 | 全部用中文 |
| 废话太多 | 直接输出完整 HTML 文件 |
