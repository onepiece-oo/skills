---
name: writing-notes
description: 整理零散的网站登录信息、账号密码、项目详情或基础设施参考为结构化的中文 HTML 网页文件，带有层级标题、侧边栏导航、主密码验证、点击复制、暗色海贼主题、侧栏折叠与平台分组动画
---

# 写笔记

将零散的网站信息、账号密码、项目资料整理成结构化的中文 HTML 网页文件。

## 概述

用户提供零散素材（网站名称、登录方式、用户名、密码、API Key、项目细节等），skill 生成一个完整的 HTML 文件，带有暗色海贼王主题、侧边栏导航、主密码验证、点击复制功能、侧栏折叠、平台分组收起、入场动画。

**核心原则：** 全中文输出，密码默认密文，点击验证主密码后所有密码均可直接复制，所有内容写进同一个 HTML 文件。

## 文件结构

所有笔记写进**同一个 HTML 文件**，每次追加新条目，不新建文件。

```
1. 侧边栏 — 左侧固定，列出所有平台和实例，可整体收起 + 各平台独立折叠
2. 正文 — 右侧滚动，按平台组织
3. 每个字段 — 单击复制值到剪贴板
4. 密码 — 密文显示，点击触发主密码验证，验证通过后点击直接复制
5. 背景 — 航海地图网格纹理 + 径向渐变光晕 + 缓慢流动动画
```

## 数据结构

每个平台一个条目，实例归属到对应平台下：

```javascript
var DATA = {
  meta: { created: "2026-07-15", updated: "2026-07-24", tags: ["账号", "基础设施"] },
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
    }
    // ...更多平台
  ]
};
```

## 视觉设计

### 暗色海贼主题
- **背景色**：深蓝黑 `#080c16` → `#0b1020`
- **金箔色** `#c9a66b` 作为强调色（标题、卡片线、按钮）
- **草帽红** `#c0392b` 用于密码密文、确认按钮
- **娜美橙** `#e8a84c` 用于链接悬停
- **航海地图网格**：极淡金色线条 + 径向渐变光晕 + 8s 缓慢流动动画

### Emoji 使用（克制）
| 位置 | emoji | 说明 |
|------|-------|------|
| 侧栏标题 | 📍 | 地图标 |
| 侧栏副标 | ⚓ | 锚点分隔 |
| 主标题 | 👒 | 草帽 |
| 主副标 | ☠ | 悬赏令 |
| 平台卡片 | 📍 | 据点标识 |
| 实例舰队 | ⛵ | 帆船 |
| 折叠箭头 | ▼/▶ | CSS |
| 侧栏实例 | ⚓ | 绳索锚 |

CSS 中通过 `content: '\UE4D3'` 等 Unicode 编码写入，避免转义问题。

### 动画（统一 `cubic-bezier(0.4, 0, 0.2, 1)`）
- **入场**：卡片/标题交错渐入（`cardStaggerIn`），草帽 bounce-in
- **侧栏标题**：呼吸发光（`sidebarGlow` 3s），地图标脉冲（`mapPulse` 4s）
- **密码密文**：hover 红色光晕闪烁（`pwGlow` 1.2s）
- **折叠按钮**：hover 缩放 + active 压缩反馈
- **背景网格**：8s 无限循环漂移
- 入场动画只执行一次（CSS `animation-fill-mode: both`），无持续性能损耗

## 交互规则

1. **普通字段** — 单击复制值到剪贴板，绿色闪烁反馈 + Toast 通知
2. **密码字段** — 默认 `••••••••`，点击弹出主密码验证框
3. **主密码验证** — 输入 `123456` 通过后，会话内所有密码均可点击直接复制
4. **验证一次解锁全部** — 输入主密码后无需再次验证
5. **密码全程不显示明文** — 验证通过后只复制，不暴露内容
6. **URL 可点击** — 网址渲染为超链接，新标签页打开
7. **侧边栏收起** — 右上角折叠按钮，带动画过渡
8. **平台折叠** — 每个平台列表项可点击折叠/展开实例子项
9. **主密码锁状态提示** — 密码验证通过后 Toast 提示"验证通过，之后密码点击即可复制"

## 侧边栏设计

- **布局**：280px 固定宽度，overflow-y 自适应，thin scrollbar
- **header**：居中标题 "海贼笔记" + 副标 "ONE PIECE LOG BOOK"（锚 icon + 金线分割）
- **平台列表**：h3 元素，圆点指示器 + 右侧三角箭头折叠标识
- **实例列表**：h4 元素，锚 icon + 左竖线 hover 高亮
- **收起态**：translateX(-280px) + opacity 0 + pointer-events none

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

## 卡片设计

- **标题**：emoji 📍 + 平台名 — "平台据点"（平台级），实例名（实例级）
- **金线底框**：标题下方 1px border-bottom 分隔线
- **顶部金线**：hover 时从中心向两侧生长（`width: 0→100%`, `opacity: 0→1`）
- **响应**：上浮 2px + 阴影增强（金色光晕）

## CSS 类名约定

| 类名 | 用途 |
|------|------|
| `.sidebar-title` | 侧栏标题 |
| `.sidebar-eng` | 侧栏英文副标 |
| `.resource-card` | 平台/实例卡片 |
| `.card-title` | 卡片标题行 |
| `.field-label` | 字段标签 |
| `.field-value` | 字段值 |
| `.field-value.copyable` | 可复制字段 |
| `.field-value.copied-flash` | 复制成功闪烁 |
| `.password-masked` | 密文密码 |
| `.sensitive-row` | 密码行包裹容器 |
| `.section-label` | 实例舰队标题 |
| `.meta-tag` | 元信息标签胶囊 |
| `.collapse-btn` | 侧栏收起按钮 |
| `.pw-dialog` | 密码验证弹窗 |
| `.pw-dialog-inner` | 弹窗内容区 |
| `.toast` | 通知消息 |

## 关键 JS 变量

| 变量 | 类型 | 说明 |
|------|------|------|
| `DATA` | Object | 所有账号数据 |
| `MASTER_PW` | String | 主密码硬编码（默认 '123456'） |
| `GATEWAY_UNLOCKED` | Boolean | 验证通过标志，验证后密码直接复制无需弹窗 |
| `SIDEBAR_COLLAPSED` | Boolean | 侧栏收起状态 |
| `FOLDED_GROUPS` | Object | 平台折叠状态 map |

## 工具函数

| 函数 | 作用 |
|------|------|
| `renderSidebar()` | 渲染侧边栏导航，含折叠按钮绑定 |
| `renderContent()` | 渲染主内容区，含交错动画延迟 |
| `renderPlatformCard(platform, cardIndex)` | 渲染平台卡片 |
| `renderInstanceCard(instance, platformName, cardIndex)` | 渲染实例卡片 |
| `buildInstanceFields(instance)` | 动态构建实例字段映射（只返回有值的字段） |
| `renderFields(fieldsMap, pwPrefix)` | 渲染字段行，**密码字段自动拼接 `-密码` 后缀到 data-pw-key** |
| `escapeHtml(s)` | XSS 转义 |
| `escapeAttr(s)` | HTML 属性转义 |
| `showToast(msg)` | 弹出 Toast 通知 |
| `copyValue(span)` | 复制到剪贴板 + 绿色闪烁反馈 |
| `handlePasswordClick(span)` | 密码点击处理：已解锁直接复制，未解锁弹验证框 |
| `copyPasswordByKey(pwKey)` | 通过 key 查找并复制密码 |
| `findPasswordByKey(pwKey)` | 遍历 DATA 匹配 pwKey，找到即返回 |
| `toggleGroup(groupKey)` | 平台折叠/展开（切换 folded class + 子元素 display） |
| `toggleSidebar()` | 侧栏收起/展开（同时更新 sidebar + btn + content margin） |
| `createCollapseBtn()` | 动态创建折叠按钮 |

## 文件路径

默认：`D:\Warehouse\ThinkNotes\ThinkNotes.html`，用户可指定路径。

## 常见错误

| 错误 | 修正 |
|------|------|
| 缺少侧边栏导航 | 必须有左侧目录，可点击跳转 |
| 密码明文显示 | 密码默认显示为 ••••••，点击触发验证 |
| 没有复制功能 | 所有字段值都可点击复制 |
| 新建多个文件 | 始终写入同一个 HTML 文件 |
| 英文输出 | 全部用中文（除品牌名、URL 外） |
| 废话太多 | 直接输出完整 HTML 文件 |
| 密码字段渲染空值 | 空密码跳过该字段渲染 |
| XSS 未转义 | 使用 escapeHtml/escapeAttr 转义 |
| sidebar h3 CSS 伪元素冲突 | `::before` 和 `::after` 不能共存需分离（如 `::before` 是圆点、`::after` 是箭头） |
| 侧栏折叠后 content margin 不同步 | toggleSidebar 中同时更新 sidebar class + content marginLeft |
| 入场动画重复闪烁 | 使用 `animation-fill-mode: both` 确保只播放一次 |
| **密码解锁后无法复制** | `renderFields` 拼接 `data-pw-key` 时必须追加 `-密码` 后缀（即 `pwPrefix + '-密码'`），与 `findPasswordByKey` 构造的 key 格式保持一致 |
| 空密码跳过渲染 | `renderFields` 中 `if (!val) continue` 防止空密码字段无意义渲染 |
| `GATEWAY_UNLOCKED` 控制分支 | `handlePasswordClick` 开头必须检查此变量，已解锁直接调用 `copyPasswordByKey` 跳过弹窗 |
| clipboard API 在 `file://` 下可能静默失败 | 某些浏览器 HTTPS/localhost 限制剪贴板，需加 fallback 或 Toast 确认提示 |
