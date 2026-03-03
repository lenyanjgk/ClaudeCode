# 第三章：告别重复提示词，自定义Commands让AI秒懂你

## 目录

- [1. 前言](#1-前言)
- [2. Commands 是什么](#2-commands-是什么)
  - [2.1 什么是 Slash 命令](#21-什么是-slash-命令)
  - [2.2 命令的三大类型](#22-命令的三大类型)
  - [2.3 为什么要学 Commands](#23-为什么要学-commands)
- [3. 创建第一个自定义命令](#3-创建第一个自定义命令)
- [4. 自定义命令开发进阶](#4-自定义命令开发进阶)
  - [4.1 命令文件结构](#41-命令文件结构)
  - [4.2 作用域与优先级](#42-作用域与优先级)
  - [4.3 frontmatter 配置详解](#43-frontmatter-配置详解)
  - [4.4 $ARGUMENTS 参数处理](#44-arguments-参数处理)
  - [4.5 可调用的工具](#45-可调用的工具)
  - [4.6 条件逻辑设计](#46-条件逻辑设计)
  - [4.7 实战：完整写作命令](#47-实战完整写作命令)
- [5. 命令高级用法](#5-命令高级用法)
  - [5.1 命令组合与链式调用](#51-命令组合与链式调用)
  - [5.2 模块化设计](#52-模块化设计)
  - [5.3 社区命令资源](#53-社区命令资源)
  - [5.4 故障排查](#54-故障排查)
- [6. 内置命令速查](#6-内置命令速查)
- [7. 总结](#7-总结)
- [8. 参考资料](#8-参考资料)

---

## 1. 前言

学完前两章，你已经能熟练启动 Claude Code、使用各种 Slash 命令和快捷键。

但你可能发现一个痛点：**每次让 Claude 做同类型的事，都要重复一大段要求**。

Commands（自定义命令）就是解决这个问题的——**把重复的提示词压缩成一个词，一次配置，永久生效。**

---

## 2. Commands 是什么

### 2.1 什么是 Slash 命令

Slash 命令是 Claude Code 的"快捷方式"：输入 `/命令名` 触发预设操作。

**工作原理**：输入 `/write AI教程` → 找到 `.claude/commands/write.md` → 读取内容作为提示词 → 把 `AI教程` 赋值给 `$ARGUMENTS` → 执行。

核心等式：**命令名 = 文件名（不含 `.md`）**，**参数 = `$ARGUMENTS`**。

### 2.2 命令的三大类型

| 类型 | 存放位置 | 生效范围 |
|------|---------|---------|
| 内置命令 | 程序内部（不可改） | 所有项目 |
| 项目级自定义 | `.claude/commands/` | 仅当前项目，可提交 Git 团队共享 |
| 用户级自定义 | `~/.claude/commands/` | 所有项目，个人通用工具 |

选择原则：会话管理/诊断用内置；项目专属工作流用项目级；跨项目通用工具用用户级。

### 2.3 为什么要学 Commands

| 对比维度 | 手动输入 | 使用 Commands |
|---------|---------|--------------|
| 效率 | 每次重复输入 | 一次配置，永久使用 |
| 一致性 | 容易遗漏要求 | 标准化执行 |
| 可复用 | 困在聊天记录 | 团队共享、版本控制 |

---

## 3. 创建第一个自定义命令

**Step 1：建目录**

```Bash
# macOS / Linux
mkdir -p .claude/commands

# Windows PowerShell
New-Item -ItemType Directory -Path ".claude\commands" -Force
```

**Step 2：创建命令文件** `.claude/commands/hello.md`

```markdown
# 问候命令

用户想要问候的对象是：$ARGUMENTS

如果没有提供名字，请使用"朋友"作为默认称呼。
请用热情友好的方式问候，并询问今天可以帮助什么。
```

> 推荐新手直接在 Claude Code 对话框说："帮我创建文件 `.claude/commands/hello.md`，内容是……"，让 AI 帮你写文件。

**Step 3：测试**

```
claude
You: /hello 张三   → 你好，张三！……
You: /hello        → 你好，朋友！……
```

> **小技巧**：输入 `/` 后按 `Tab` 键可查看所有可用命令。

---

## 4. 自定义命令开发进阶

### 4.1 命令文件结构

命令文件由两部分组成：**YAML frontmatter 配置区**（机器读）+ **Markdown 正文**（AI 读）。

```markdown
---
description: 公众号文章创作命令
argument-hint: <主题关键词>
allowed-tools:
  - Read
  - Write
  - WebSearch
model: claude-sonnet-4-5-20250929
---

# 公众号文章创作

你是一位资深的公众号写作专家。
主题：$ARGUMENTS

写作要求：接地气 · 1500-2000字 · 金句开头 → 核心内容 → 行动号召
```

### 4.2 作用域与优先级

**优先级**：项目级 > 用户级 > 内置命令（非核心）

> `/clear`、`/help`、`/compact` 等核心内置命令受保护，自定义命令无法覆盖。

支持子目录，用 `:` 作命名空间：

```
.claude/commands/
├── write.md              # /write
├── dev/
│   └── code-review.md    # /dev:code-review
└── test/
    └── generate.md       # /test:generate
```

### 4.3 frontmatter 配置详解

| 配置项 | 作用 |
|--------|------|
| `description` | 命令描述，显示在 `/help` 和 Tab 补全中 |
| `argument-hint` | 输入命令后显示的参数占位符提示 |
| `allowed-tools` | 限制可调用的工具（安全边界） |
| `model` | 强制指定模型，覆盖当前会话模型 |
| `disable-model-invocation` | 设为 `true` 时只做文本替换，不调用 AI |

**`disable-model-invocation` 示例**（节省 Token 的纯模板命令）：

```markdown
---
description: 快速插入版权声明
disable-model-invocation: true
---

© 2025 $ARGUMENTS. All rights reserved.
```

执行 `/copyright 凯神` → 直接输出 `© 2025 凯神. All rights reserved.`，不经过 AI 处理。

### 4.4 $ARGUMENTS 参数处理

`$ARGUMENTS` 接收命令后的全部输入（整段字符串）：

```
/write AI工具            → $ARGUMENTS = "AI工具"
/write AI工具 技术 3000  → $ARGUMENTS = "AI工具 技术 3000"
/write                   → $ARGUMENTS = ""（空）
```

多参数解析和空值校验直接用自然语言在提示词中描述即可：

```markdown
$ARGUMENTS 格式：<主题> [风格] [字数]
- 第一个词：主题（必需，若为空请提示用户补充）
- 第二个词：风格（可选，默认"接地气"）
- 第三个词：字数（可选，默认 1500）
```

### 4.5 可调用的工具

| 工具名 | 功能 | 常用场景 |
|--------|------|---------|
| `Read` | 读取文件 | 分析代码、读取配置 |
| `Write` | 写入新文件 | 创建文件、保存结果 |
| `Edit` | 编辑已有文件 | 修改代码 |
| `Bash` | 执行命令 | 运行测试、Git 操作 |
| `WebSearch` | 网络搜索 | 获取最新信息 |
| `WebFetch` | 抓取网页内容 | 下载指定页面分析 |
| `Glob` | 按文件名匹配查找 | 批量找 `*.md`、`*.ts` |
| `Grep` | 按内容搜索文件 | 找含 TODO 的代码 |
| `Task` | 启动子代理 | 并行执行复杂任务 |
| `TodoWrite` | 任务管理 | 创建和更新待办清单 |

> **最小权限原则**：审查类只需 `Read, Grep`；写代码加 `Write, Edit`；跑命令才开 `Bash`。
>
> MCP 工具命名格式：`mcp__服务器名__工具名`（如 `mcp__github__create_issue`），详见第04章。

### 4.6 条件逻辑设计

Markdown 不支持代码逻辑，但 Claude 能理解自然语言描述的条件分支：

```markdown
根据 $ARGUMENTS 判断：
- 包含"深度"或"详细" → 深度分析，输出 3000 字以上完整报告
- 包含"快速"或"简要" → 快速分析，输出 500 字以内摘要
- 其他情况（默认）    → 标准分析，输出 1500 字标准报告
```

```markdown
检查 $ARGUMENTS 的第一个关键词：
- "测评" → 测评模板，重点写优缺点对比
- "教程" → 教程模板，重点写步骤和代码
- "对比" → 对比模板，重点写表格和结论
- 其他   → 通用模板
```

### 4.7 实战：完整写作命令

文件：`.claude/commands/write.md`

```markdown
---
description: 公众号文章全自动创作，从信息收集到成稿保存
argument-hint: <主题关键词>
allowed-tools:
  - Read
  - Write
  - WebSearch
  - Grep
---

# 公众号文章创作系统

你是资深公众号写作专家，擅长创作接地气、有深度的技术科普文章。

**主题**：$ARGUMENTS

## 执行步骤

1. **信息收集**：WebSearch 搜索"$ARGUMENTS 最新资讯 2025"，收集核心概念、最新动态、用户痛点
2. **构思大纲**：金句开头 → 问题引入(2-3段) → 核心内容(5-8段) → 总结号召
3. **撰写文章**：说人话、用类比、多短句，字数 1500-2000，每段≤150字
4. **保存文章**：Write 保存到 `articles/drafts/[日期]_[主题].md`
5. **生成标题**：5个备选标题（含数字、引发好奇、≤30字）

## 输出格式

# [选定标题]

[文章正文]

---
## 备选标题
1. [标题1] ... 5. [标题5]
```

使用：`/write Claude Code入门` → 自动完成搜索、构思、撰写、保存全流程。

---

## 5. 命令高级用法

> 命名空间（子目录组织）见 [4.2 作用域与优先级](#42-作用域与优先级)。

### 5.1 命令组合与链式调用

单个命令可以描述多步骤工作流，Claude 会按顺序执行：

```markdown
# 完整发布流程

1. WebSearch 搜索 "$ARGUMENTS 最新动态"
2. 根据搜索结果撰写文章并 Write 保存
3. Read 读取文章，进行自我审查并输出修改意见
4. 输出最终版本与 5 个备选标题
```

多命令串联：先 `/research 主题` 生成素材文件，再 `/write 主题` 读取该文件写作，每个命令职责单一、可单独复用。

### 5.2 模块化设计

把多个命令共用的角色设定、写作风格提取为**共享片段**，存入 `.claude/modules/`：

```
.claude/
├── commands/
│   ├── write.md        # 引用 modules/writer-role.md
│   └── review.md       # 引用 modules/writer-role.md
└── modules/
    └── writer-role.md  # 共享角色设定，修改一次全部生效
```

在命令文件中加载模块（需在 `allowed-tools` 开启 `Read`）：

```markdown
请先读取 `.claude/modules/writer-role.md` 作为角色设定，再执行以下任务……
```

### 5.3 社区命令资源

| 资源 | 搜索关键词 | 内容 |
|------|-----------|------|
| Claude Command Suite | GitHub 搜索 | 审查、测试、文档类命令集合 |
| Awesome Claude Code | GitHub 搜索 | 社区精选命令、模板、工作流 |
| 官方文档示例 | docs.anthropic.com | 官方推荐命令写法 |

> 使用社区命令前，先审查 `allowed-tools` 列表，避免权限过宽。

### 5.4 故障排查

| 现象 | 原因 | 解决方法 |
|------|------|---------|
| 输入命令无响应 | 文件路径错误 | 确认路径：`.claude/commands/命令名.md` |
| 命名空间命令找不到 | 目录层级错误 | `/dev:review` 对应 `commands/dev/review.md` |
| frontmatter 配置未生效 | YAML 格式有误 | 检查缩进用空格、冒号后有空格 |
| 工具调用被拒绝 | 工具未声明 | 将所需工具加入 `allowed-tools` 列表 |
| 参数被截断 | 特殊字符问题 | 用引号包裹：`/write "AI 教程 2025"` |

---

## 6. 内置命令速查

> 详细用法见[第二章 6. Slash 命令大全](第02章：30+命令与快捷键，编程效率直接翻倍.md#6-slash-命令大全)，这里提供速查表。

| 分类 | 命令 | 功能 | 重要度 |
|------|------|------|--------|
| 会话管理 | `/clear` `/compact` `/resume` | 清空/压缩/恢复会话 | ⭐⭐⭐ |
| | `/export` `/rename` | 导出/重命名会话 | ⭐⭐ |
| 上下文控制 | `/context` `/model` | 查看Token / 切换模型 | ⭐⭐⭐ |
| | `/cost` `/usage` | 查看费用/用量 | ⭐⭐ |
| 项目配置 | `/init` `/add-dir` | 初始化CLAUDE.md / 添加目录 | ⭐⭐⭐ |
| | `/memory` `/permissions` | 编辑记忆 / 管理权限 | ⭐⭐ |
| 开发辅助 | `/rewind` `/review` `/todos` | 回退/审查/待办 | ⭐⭐⭐ |
| | `/agents` | 管理子代理 | ⭐⭐ |
| 诊断工具 | `/doctor` `/status` | 健康检查/完整状态 | ⭐⭐ |
| MCP 相关 | `/mcp` `/hooks` | 管理MCP/Hooks | ⭐⭐⭐ |
| 其他 | `/help` `/bug` `/release-notes` | 帮助/报告Bug/更新日志 | ⭐⭐ |

---

## 7. 总结

本章你已掌握：

1. **Commands 本质**：`.claude/commands/` 下的 Markdown 文件，文件名即命令名，`$ARGUMENTS` 接收参数
2. **三种类型**：内置 / 项目级 / 用户级，按需选择
3. **frontmatter**：5 个配置项控制描述、参数提示、工具权限、模型、是否调用 AI
4. **最小权限原则**：按角色只开放必要工具
5. **条件逻辑**：自然语言描述分支，Claude 正确执行
6. **高级用法**：链式调用多步骤工作流、模块化共享片段、社区命令资源借力

---

## 8. 参考资料

**官方文档与教程**：
- [知乎讲解：Claude Commands 自定义命令完整指南](https://zhuanlan.zhihu.com/p/1974062756123658107)
- [Claude 中文文档：自定义 Commands 开发指南](https://claudecn.com/docs/claude-code/advanced/custom-commands/)
- [知乎：Commands 实战案例与最佳实践](https://zhuanlan.zhihu.com/p/1960414407042512481)
