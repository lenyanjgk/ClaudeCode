# 第六章：Plugins全攻略——一键安装海量扩展，还能自己造轮子

## 目录

- [1. 前言](#1-前言)
- [2. Plugin 核心概念](#2-plugin-核心概念)
  - [2.1 Plugin 是什么](#21-plugin-是什么)
  - [2.2 Plugin vs Commands / Skills / MCP](#22-plugin-vs-commands--skills--mcp)
  - [2.3 生态现状](#23-生态现状)
- [3. 5 分钟安装第一个 Plugin](#3-5-分钟安装第一个-plugin)
  - [3.1 前置检查](#31-前置检查)
  - [3.2 浏览与安装 Marketplace](#32-浏览与安装-marketplace)
- [4. Plugin 管理全流程](#4-plugin-管理全流程)
  - [4.1 安装 Plugin（Discover 标签页）](#41-安装-plugindiscover-标签页)
  - [4.2 管理已安装的 Plugin（Installed 标签页）](#42-管理已安装的-plugininstalled-标签页)
  - [4.3 管理 Marketplace 源（Marketplaces 标签页）](#43-管理-marketplace-源marketplaces-标签页)
- [5. 创建自定义 Plugin](#5-创建自定义-plugin)
  - [5.1 Plugin 结构规范](#51-plugin-结构规范)
  - [5.2 plugin.json 详解](#52-pluginjson-详解)
  - [5.3 实战：Hello World Plugin](#53-实战hello-world-plugin)
  - [5.4 进阶：带 Skill 的完整 Plugin](#54-进阶带-skill-的完整-plugin)
  - [5.5 开发最佳实践](#55-开发最佳实践)
- [6. 发布与分享](#6-发布与分享)
  - [6.1 发布前检查清单](#61-发布前检查清单)
  - [6.2 发布到 GitHub](#62-发布到-github)
  - [6.3 提交到官方 Marketplace](#63-提交到官方-marketplace)
- [7. 故障排查](#7-故障排查)
- [8. 常见问题 FAQ](#8-常见问题-faq)
- [9. 总结](#9-总结)
- [10. 参考资料](#10-参考资料)
- [下一步学习](#下一步学习)

---

## 1. 前言

学完 Skills，你已经能给 Claude 装上各种专属能力包了。

但你有没有想过一个问题——**辛辛苦苦写好的 Commands + Skills + Hooks 配置，换个项目又得重来一遍？想分享给团队成员？手动复制粘贴？**

**Plugin 就是解决这个问题的终极方案：**

| 痛点 | Plugin 怎么解决 | 类比 |
|------|---------------|------|
| 配置不可移植 | 一个 Plugin 打包所有配置，一键安装 | 手机备份恢复 |
| 分享靠手动复制 | Marketplace 搜一下装一下 | App Store |
| 更新全靠自己盯 | 自动更新，作者推了新版你自动获得 | APP 自动更新 |
| 找好用的工具费时间 | 社区 200+ Plugin 直接选 | 排行榜推荐 |

说白了：**Plugin = 可分享、可安装、可自动更新的 Commands + Skills + Hooks + MCP 打包体。**

---

## 2. Plugin 核心概念

### 2.1 Plugin 是什么

Plugin 是 Claude Code 的**扩展包**——把 Commands、Skills、Hooks、MCP 配置打包成一个可安装、可分享、可自动更新的整体。

**类比理解：**

| 手机 | Claude Code |
|------|------------|
| 操作系统（iOS/Android） | Claude Code 核心 |
| App Store | Plugin Marketplace |
| 安装的 APP | 已安装的 Plugins |
| APP 自动更新 | Plugin 自动更新 |

**核心价值：**

| 价值 | 说明 |
|------|------|
| 可复用 | 一次开发，多个项目使用 |
| 可分享 | 通过 Marketplace 一键安装，不用手动复制 |
| 模块化 | 每个 Plugin 专注一个领域，互不干扰 |
| 社区驱动 | 200+ 社区 Plugin 开箱即用 |

### 2.2 Plugin vs Commands / Skills / MCP

很多人问："我已经有 Commands 和 Skills 了，为什么还要 Plugin？"

一张表说清楚：

| 维度 | Commands | Skills | MCP | **Plugins** |
|------|---------|--------|-----|------------|
| 定义 | Markdown 提示词 | 专业 Agent 能力 | 外部服务集成 | **打包的扩展** |
| 存放位置 | `.claude/commands/` | `.claude/skills/` | `.mcp.json` | `.claude/plugins/` |
| 可分享性 | ❌ 手动复制 | ❌ 手动复制 | ⚠️ 需配置 | ✅ 一键安装 |
| 自动更新 | ❌ 手动更新 | ❌ 手动更新 | ⚠️ 部分支持 | ✅ 自动更新 |
| 包含内容 | 单个提示词 | 多个文件+配置 | 服务器配置 | **全部都能包含** |
| 适用场景 | 简单重复任务 | 复杂专业任务 | 外部 API 调用 | **所有场景** |

> **关键区别**：Plugin 是一个**"超集"**概念——`Plugin = Commands + Skills + Hooks + MCP 配置 + 文档`，打包成一个可以一键安装和分享的整体。

**何时用什么？决策指南：**

- **Commands**：项目内简单重复任务（如 `/format-code`）——够用就行，别上 Plugin
- **Skills**：项目内复杂专业任务（如代码注释生成）——能力需要积累和复用
- **MCP**：需要调用外部服务（如 GitHub API、数据库）——解决连接问题
- **Plugins**：✅ **想分享给团队或社区的任何功能**——打包分发的最佳选择

### 2.3 生态现状

**三大 Marketplace：**

| 平台 | 地址 | 特点 |
|------|------|------|
| Anthropic 官方 Marketplace | code.claude.com/plugins | 审核严格，质量保证 |
| Jeremy Longshore 社区合集 | github.com/jeremylongshore/claude-code-plugins-plus | 200+ Plugin，持续更新 |
| Composio Integration | composio.dev | 集成 2000+ 外部工具 |

**热门 Plugin 分类速查：**

| 分类 | 典型 Plugin | 用途 | 来源 | 热度 |
|------|-----------|------|------|------|
| 文档处理 | document-skills | PDF/PPTX/XLSX 全套文档处理 | Anthropic 官方 | ⭐⭐⭐⭐⭐ |
| 示例学习 | example-skills | 官方 Skill 开发示例 | Anthropic 官方 | ⭐⭐⭐ |
| 代码质量 | code-review-expert | 自动代码审查 | 社区 | ⭐⭐⭐⭐ |
| 项目管理 | task-master-ai | 任务拆解和跟踪 | 社区 | ⭐⭐⭐⭐ |
| API 集成 | connect-apps | Gmail/Slack/GitHub 联动 | 社区 | ⭐⭐⭐⭐⭐ |
| 数据分析 | data-viz-pro | 数据可视化 | 社区 | ⭐⭐⭐ |

> **说明**：`document-skills` 是 Anthropic 官方出品的文档处理套件（来自 `anthropics/skills` 源），安装后包含 `document-skills:pdf`、`document-skills:pptx`、`document-skills:xlsx` 等多个 Skill，一次安装全部可用。

---

## 3. 5 分钟安装第一个 Plugin

### 3.1 前置检查

```bash
# 1. 确认 Claude Code 版本（需 v2.1+）
claude --version

# 2. 确认在项目目录中
cd /path/to/your/project
```

> Plugin 功能于 2025 年 10 月 9 日随 Claude Code v2.1 发布。如果版本过低，先升级。

### 3.2 浏览 Marketplace

**方式一：对话内 `/plugin` 命令（最方便）**

在 Claude Code 对话中直接输入：

```
/plugin
```

会进入 Plugin 管理界面，切换到 `Marketplace` 标签页即可浏览所有可用 Plugin。

**方式二：CLI 命令**

```bash
# 添加社区 Marketplace 源（第一次用需要先添加）
claude plugin marketplace add anthropics/skills

# 查看已添加的 Marketplace 源
claude plugin marketplace list
```

添加成功后，就能从该源浏览和安装 Plugin 了。

![image-20260306152553451](D:\Desktop\cc\assets\image-20260306152553451.png)

![image-20260306152751059](D:\Desktop\cc\assets\image-20260306152751059.png)

**实战：添加 Marketplace 源并安装 Plugin**

以添加 VoltAgent Subagent 代理库为例，完整流程：

```bash
# 1. 添加 Marketplace 源
claude plugin marketplace add VoltAgent/awesome-claude-code-subagents
```

```bash
# 2. 从该源安装你需要的 Plugin
claude plugin install <plugin-name>
```

或者在对话中输入 `/plugin`，切换到 `Installed` 界面自行浏览安装。

![image-20260306143259905](D:\Desktop\cc\assets\image-20260306143259905.png)

![image-20260306143333962](D:\Desktop\cc\assets\image-20260306143333962.png)

![image-20260306144520059](D:\Desktop\cc\assets\image-20260306144520059.png)

> **网络问题？** 如果安装过程中遇到超时或下载失败，需要检查代理软件配置：
>
> ```powershell
> # Windows PowerShell（根据自己的代理端口修改）
> $env:HTTP_PROXY = "http://127.0.0.1:7897"
> $env:HTTPS_PROXY = "http://127.0.0.1:7897"
> ```
>
> ```bash
> # macOS / Linux
> export HTTP_PROXY="http://127.0.0.1:7897"
> export HTTPS_PROXY="http://127.0.0.1:7897"
> ```

**方式三：Web 浏览器**

直接访问官方 Marketplace 网页：`https://code.claude.com/plugins`

---

## 4. Plugin 管理全流程

在 Claude Code 对话中输入 `/plugin`，进入图形化管理界面。界面顶部有四个标签页，覆盖所有管理场景：

| 标签页 | 功能 |
|--------|------|
| **Discover** | 浏览并安装 Plugin（默认打开） |
| **Installed** | 查看和管理已安装的 Plugin |
| **Marketplaces** | 管理 Plugin 源 |
| **Errors** | 排查加载错误 |

> **界面导航**：`↑↓` 上下移动，`空格` 快速安装/切换，`回车` 进入详情，`ESC` 返回上级

### 4.1 安装 Plugin（Discover 标签页）

打开 `/plugin`，默认进入 **Discover** 标签页，列出所有已添加 Marketplace 源中的可用 Plugin：

![image-20260306165915173](D:\Desktop\cc\assets\image-20260306165915173.png)

**安装步骤：**

1. 用 `↑↓` 移动光标，选中想安装的 Plugin
2. 按**回车**查看详情（或直接按**空格**快速安装）
3. 在详情页选择**安装范围**，确认安装

![image-20260306165927770](D:\Desktop\cc\assets\image-20260306165927770.png)

**安装范围说明：**

| 范围 | 说明 | 适用场景 |
|------|------|---------|
| Install for you (user scope) | 个人级：你的所有项目都可用 | 日常通用工具（推荐） |
| Install for all collaborators on this repository (project scope) | 项目级：整个仓库的协作者共享 | 团队协作项目 |
| Install for you, in this repo only (local scope) | 本地仓库级：仅自己在此项目可用 | 本地测试、不想影响他人 |

**三种安装来源：**

| 来源 | 操作方式 |
|------|---------|
| Marketplace Plugin（推荐） | 在 Discover 标签页直接选中安装 |
| GitHub URL | 在 Discover 页顶部输入 GitHub 仓库地址 |
| 本地目录 | 在 Discover 页顶部输入本地路径（开发测试用） |

> **找不到想装的 Plugin？** 先去 **Marketplaces** 标签页添加对应的源，再回 Discover 刷新列表。

### 4.2 管理已安装的 Plugin（Installed 标签页）

切换到 **Installed** 标签页，查看所有已启用的 Plugin：

![image-20260306165957152](D:\Desktop\cc\assets\image-20260306165957152.png)

列表中同时显示 Plugin 系统安装的扩展和 MCP 连接，每条显示名称、版本和状态。

**更新 / 禁用 / 卸载操作：**

选中某个 Plugin，按**回车**进入详情，可执行以下操作：

| 操作 | 效果 |
|------|------|
| 更新 | 升级到最新版本（有新版本时显示） |
| 禁用 | 临时停用，保留文件，下次可重新启用 |
| 卸载 | 彻底删除 Plugin 及其所有文件 |

> **自动更新**：Claude Code 启动时会自动检查所有 Plugin 更新，无需手动触发。

![image-20260306170011720](D:\Desktop\cc\assets\image-20260306170011720.png)

### 4.3 管理 Marketplace 源（Marketplaces 标签页）

切换到 **Marketplaces** 标签页，管理 Plugin 的来源：

![image-20260306170026630](D:\Desktop\cc\assets\image-20260306170026630.png)

界面列出所有已添加的 Marketplace 源，每个源显示可用 / 已安装的 Plugin 数量，以及最后更新时间。

**添加新 Marketplace 源：**

1. 在 Marketplaces 标签页选择"添加源"
2. 输入源地址（格式：`用户名/仓库名`）
3. 确认添加后，Discover 标签页自动同步新来源的 Plugin

![image-20260306170039452](D:\Desktop\cc\assets\image-20260306170039452.png)

**推荐 Marketplace 源：**

| 源地址 | 内容 | 推荐指数 |
|--------|------|---------|
| `anthropics/skills` | 官方 Skills 套件（含 document-skills 全家桶） | ⭐⭐⭐⭐⭐ |
| `anthropics/claude-plugins-official` | Anthropic 官方 Plugin | ⭐⭐⭐⭐ |
| `VoltAgent/awesome-claude-code-subagents` | 100+ 专家 Subagent（详见第04章） | ⭐⭐⭐⭐ |
| `jeremylongshore/claude-code-plugins-plus` | 社区 200+ Plugin，持续更新 | ⭐⭐⭐ |

**Marketplace 源详情示例**（`anthropics/skills`）：

```
anthropic-agent-skills (anthropics/skills)
3 available · 1 installed · Updated 2026/3/6

  ✅ document-skills (installed)
     Collection of document processing suite including Excel...
  ○  example-skills
     Collection of example skills demonstrating various capabi...
```

> **推荐先装**：`document-skills`（官方文档处理套件），安装后自动获得 pdf、pptx、xlsx 等全套 Skill，一次搞定所有文档格式。

---

## 5. 创建自定义 Plugin

### 5.1 Plugin 结构规范

Plugin 本质上是一个目录，里面有个 `.claude-plugin/plugin.json` 清单文件，加上你要打包的各种能力。

**最小结构（一个 Skill）：**

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json      # 必需：Plugin 清单
└── skills/
    └── my-skill/
        └── SKILL.md
```

**完整结构（所有能力）：**

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json      # 必需：Plugin 清单（只有这个在 .claude-plugin/ 内）
├── README.md            # 推荐：使用文档
├── LICENSE              # 推荐：开源协议
├── skills/              # 可选：Agent Skills
│   └── my-skill/
│       └── SKILL.md
├── commands/            # 可选：Slash Commands
│   └── my-command.md
├── agents/              # 可选：自定义 Agent 定义
├── hooks/               # 可选：事件钩子
│   └── hooks.json
├── .mcp.json            # 可选：MCP 服务配置
└── settings.json        # 可选：Plugin 启用时的默认设置
```

> ⚠️ **常见踩坑**：`commands/`、`skills/`、`agents/`、`hooks/` 都放在**插件根目录**，不要放进 `.claude-plugin/` 里——那里只放 `plugin.json`。

各目录职责速查：

| 目录/文件 | 职责 |
|----------|------|
| `.claude-plugin/plugin.json` | Plugin 清单，定义名称/版本/作者 |
| `skills/` | Agent Skills（含 SKILL.md 的子目录） |
| `commands/` | Slash 命令（Markdown 文件） |
| `agents/` | 自定义 Agent 定义 |
| `hooks/` | 事件处理器（hooks.json） |
| `.mcp.json` | MCP 服务配置 |
| `settings.json` | Plugin 启用时应用的默认设置 |

### 5.2 plugin.json 详解

`plugin.json` 放在 `.claude-plugin/` 目录下，是 Plugin 的「身份证」。

**完整示例：**

```json
{
  "name": "my-awesome-plugin",
  "description": "一句话说明这个 Plugin 做什么",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  },
  "license": "MIT",
  "homepage": "https://github.com/yourname/my-plugin",
  "repository": "https://github.com/yourname/my-plugin"
}
```

**字段速查：**

| 字段 | 是否必填 | 说明 |
|------|---------|------|
| `name` | ✅ | Plugin 唯一标识，同时是 Skill 的命名空间前缀 |
| `description` | ✅ | 功能描述，Marketplace 中用于搜索和展示 |
| `version` | ✅ | 语义化版本号（`主.次.补丁`，如 `1.0.0`） |
| `author` | 推荐 | 作者信息（`name` / `email` / `url`） |
| `license` | 推荐 | 开源协议（推荐 `MIT` 或 `Apache-2.0`） |
| `homepage` | 可选 | 项目主页或文档地址 |
| `repository` | 可选 | 代码仓库地址 |

> **关键点**：`name` 字段决定了 Skill 的调用前缀。Plugin 名叫 `my-plugin`，里面的 `hello` Skill 就要用 `/my-plugin:hello` 来触发——命名空间设计防止多个 Plugin 的 Skill 名称冲突。

### 5.3 实战：Hello World Plugin

5 分钟从零创建并测试你的第一个 Plugin：

**Step 1：创建目录结构**

```bash
mkdir -p hello-plugin/.claude-plugin
mkdir -p hello-plugin/skills/hello
```

**Step 2：创建清单 `.claude-plugin/plugin.json`**

```json
{
  "name": "hello-plugin",
  "description": "一个简单的问候示例 Plugin",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  },
  "license": "MIT"
}
```

**Step 3：创建 Skill `skills/hello/SKILL.md`**

```markdown
---
description: Greet the user with a friendly message
---

用友好的方式向用户打招呼。

步骤：
1. 获取当前系统时间
2. 根据时间段（上午/下午/晚上）调整问候语
3. 用轻松愉快的语气回复

示例输出：
"早上好！现在是 10:23，新的一天，有什么我能帮你的？"
```

**Step 4：本地测试**

不需要安装，直接用 `--plugin-dir` 标志加载：

```bash
# 加载插件并启动 Claude Code
claude --plugin-dir ./hello-plugin

# 启动后在对话里输入（注意命名空间格式）
> /hello-plugin:hello
```

看到问候输出，第一个 Plugin 就跑通了！

> `--plugin-dir` 是开发专用标志，每次修改 Skill 后重启 Claude Code 即可生效，无需安装。要同时测试多个 Plugin，可以多次指定：`claude --plugin-dir ./plugin-a --plugin-dir ./plugin-b`

**Step 5：带参数的 Skill**

`$ARGUMENTS` 占位符可以捕获用户输入的文本，让 Skill 动态响应：

```markdown
---
description: Greet the user with a personalized message
---

用友好的方式向名叫 "$ARGUMENTS" 的用户打招呼，让问候更有温度。
```

重启后测试：

```bash
> /hello-plugin:hello 凯神
# Claude 会用你传入的名字问候
```

### 5.4 进阶：带 Skill 的完整 Plugin

创建一个代码质量检查 Plugin，演示 Skill + Commands 的组合威力：

**项目结构：**

```
code-quality-checker/
├── .claude-plugin/
│   └── plugin.json
├── README.md
├── skills/
│   └── code-review/
│       └── SKILL.md
└── commands/
    └── check.md
```

**`.claude-plugin/plugin.json`：**

```json
{
  "name": "code-quality-checker",
  "description": "自动检查代码质量，按严重程度给出改进建议",
  "version": "1.0.0",
  "author": { "name": "Your Name" },
  "license": "MIT"
}
```

**`skills/code-review/SKILL.md`：**

```markdown
---
description: Reviews code for best practices, security issues, and quality problems. Use when reviewing code, checking PRs, or analyzing code quality.
---

# 代码质量检查专家

## 角色定义
你是一位资深代码审查专家，擅长发现代码质量问题并给出具体改进建议。

## 检查维度
1. **命名规范**：变量名/函数名是否语义清晰
2. **函数复杂度**：单个函数是否过长或嵌套过深
3. **重复代码**：是否存在可抽取的重复逻辑
4. **错误处理**：异常边界是否覆盖
5. **安全隐患**：是否存在注入/XSS 等风险

## 输出格式
按严重程度排序：
- 🔴 严重：必须修复
- 🟡 警告：建议修复
- 🟢 建议：可以改进

每条包含：问题描述 + 代码位置 + 修复建议
```

**`commands/check.md`：**

```markdown
对指定代码进行质量检查。

参数：$ARGUMENTS（可选，指定检查的文件路径）

步骤：
1. 如果指定了文件路径，只检查该文件
2. 如果未指定，检查最近修改的文件（通过 git diff --name-only 获取）
3. 按 5 个维度逐项分析，输出分级报告
```

**测试：**

```bash
claude --plugin-dir ./code-quality-checker

# 触发 Skill（自动识别）
> 帮我检查这段代码的质量

# 显式调用命令（注意命名空间）
> /code-quality-checker:check src/app.ts
```

> Plugin 的「超集」威力在这里体现得很清楚：**Commands 提供触发入口，Skills 提供专业能力**，打包在一起就是可分享的完整工具。

### 5.5 开发最佳实践

**1. 先用独立配置，再转 Plugin**

| 阶段 | 方式 | 原因 |
|------|------|------|
| 实验期 | 直接放 `.claude/skills/` | Skill 名短（`/hello`），迭代快 |
| 准备共享 | 打包成 Plugin | Skill 带命名空间（`/my-plugin:hello`），便于分发 |

**2. 语义化版本号**

```
版本号格式：主.次.补丁（如 1.2.3）
  - 主版本（1.x.x）：破坏性变更，不向后兼容
  - 次版本（x.2.x）：新增功能，向后兼容
  - 补丁版（x.x.3）：Bug 修复
```

**3. README 必须包含四个部分**

| 部分 | 内容 |
|------|------|
| 功能说明 | 这个 Plugin 做什么，解决什么问题 |
| 安装方法 | 从 Marketplace 或 GitHub 安装的命令 |
| 使用示例 | 至少一个真实使用场景（含 Skill 调用格式） |
| 配置说明 | 如果有可配置项的话 |

**4. Skill description 是关键**

`SKILL.md` frontmatter 中的 `description` 决定 Claude 能否自动识别场景并激活 Skill：

```markdown
# ❌ 太模糊，自动激活不准
description: Code review tool

# ✅ 明确触发场景，自动识别准确
description: Reviews code for best practices and potential issues. Use when reviewing code, checking PRs, or analyzing code quality.
```

---

## 6. 发布与分享

### 6.1 发布前检查清单

**必须完成：**

- ✅ `.claude-plugin/plugin.json` 格式正确，`name`/`description`/`version` 填写完整
- ✅ `README.md` 包含安装和使用说明（含 Skill 调用格式 `/插件名:skill名`）
- ✅ `LICENSE` 文件存在（推荐 MIT）
- ✅ 用 `claude --plugin-dir .` 本地测试通过，所有 Skill 和命令正常工作

**推荐完成：**

- ⭐ `CHANGELOG.md` 记录每个版本的变更内容
- ⭐ GitHub 仓库添加 Topics 标签（如 `claude-code-plugin`），便于被搜索
- ⭐ README 中注明最低 Claude Code 版本要求

### 6.2 发布到 GitHub

```bash
# 1. 初始化仓库
cd my-plugin
git init
git add .
git commit -m "feat: initial release v1.0.0"

# 2. 推送到 GitHub
git remote add origin https://github.com/yourname/my-plugin.git
git branch -M main
git push -u origin main
```

**创建 Release（让别人能按版本安装）：**

1. 进入 GitHub 仓库页面，点击 **Releases → Draft a new release**
2. Tag version 填 `v1.0.0`（必须以 `v` 开头）
3. 填写本次发布说明
4. 点击 **Publish release**

发布后，其他人就能通过 GitHub URL 直接安装：

```bash
# 在 /plugin 界面的 Discover 页顶部输入 GitHub 地址安装
# 或用 CLI
claude plugin install https://github.com/yourname/my-plugin
```

> 在 README 中加个版本徽章，一眼看出当前版本：
> ```markdown
> ![Version](https://img.shields.io/github/v/release/yourname/my-plugin)
> ```

### 6.3 提交到官方 Marketplace

想让全球 Claude Code 用户都能搜到你的 Plugin？直接用**应用内提交表单**，不需要 Fork 仓库、提 PR，一个表单搞定：

| 平台 | 提交入口 |
|------|---------|
| Claude.ai | `claude.ai/settings/plugins/submit` |
| Anthropic Console | `platform.claude.com/plugins/submit` |

**提交前确认：**

| 审核项 | 要求 |
|--------|------|
| `.claude-plugin/plugin.json` | 格式正确，必填字段完整 |
| README | 安装和使用说明完整，含 Skill 调用格式 |
| 代码安全 | 无恶意代码，依赖来源可信 |
| 功能完整 | `--plugin-dir` 测试全部通过 |

提交后等待 Anthropic 审核（通常 1-3 个工作日）。审核通过后，你的 Plugin 就会出现在官方 Marketplace，全球 Claude Code 用户都能一键安装。

---

## 7. 故障排查

### 7.1 安装问题

| 现象 | 原因 | 解决方法 |
|------|------|---------|
| `Plugin not found` | 名称拼写错误或 Marketplace 源未添加 | 先在 Marketplaces 标签页添加源，再到 Discover 搜索 |
| 下载超时 | 网络问题 | 设置代理或切换 npm 镜像源 |
| `/plugin` 命令不存在 | Claude Code 版本过低 | 升级到 v1.0.33+（运行 `claude --version` 确认） |

```bash
# 网络问题？换国内镜像源
npm config set registry https://registry.npmmirror.com

# 需要代理？（根据自己的代理端口修改）
export HTTP_PROXY="http://127.0.0.1:7897"
export HTTPS_PROXY="http://127.0.0.1:7897"
```

### 7.2 运行时问题

| 现象 | 原因 | 解决方法 |
|------|------|---------|
| Skill 调用格式不对 | 忘记命名空间前缀 | Plugin 内的 Skill 用 `/插件名:skill名` 格式 |
| Skill 未自动激活 | SKILL.md 的 description 描述太模糊 | 明确写出触发场景，如 "Use when reviewing code..." |
| Hook 脚本权限错误 | 脚本缺少执行权限 | `chmod +x hooks/my-hook.sh` |
| 配置修改不生效 | 未重启 Claude Code | 修改 Plugin 内容后需重启 |

### 7.3 开发调试

| 现象 | 原因 | 解决方法 |
|------|------|---------|
| `--plugin-dir` 加载失败 | 目录结构不正确 | 确认 `.claude-plugin/plugin.json` 存在于插件根目录 |
| Plugin 中 Skill 找不到 | skills/ 放错位置 | `skills/` 必须在插件根目录，不能在 `.claude-plugin/` 内 |
| plugin.json 解析报错 | JSON 格式错误 | 用在线 JSON 验证工具检查，确认冒号后有空格、无多余逗号 |

```bash
# 启用详细日志排查问题
export CLAUDE_LOG_LEVEL=debug
claude --plugin-dir ./my-plugin
```

---

## 8. 常见问题 FAQ

**Q1：Plugin 和 Skill 到底啥区别？**

一句话：**Skill 是能力本身，Plugin 是把能力打包成可安装、可分享的形式。** Plugin 可以包含 Skill，也可以包含 Commands、Hooks、MCP 配置等。

**Q2：Plugin 更新会覆盖我的配置吗？**

不会。更新时保留你的 `config.json`（个人配置），只更新 Plugin 代码文件（skills/、commands/ 等）。

**Q3：Plugin 开发需要懂编程吗？**

| 类型 | 需要编程？ | 说明 |
|------|----------|------|
| 只有 Commands 的 Plugin | ❌ 不需要 | 写 Markdown 就行 |
| 包含 Skills 的 Plugin | ❌ 不需要 | 写 Markdown（SKILL.md） |
| 带脚本的 Plugin | ✅ 需要 | Python/JavaScript 基础 |
| 带 MCP Server 的 Plugin | ✅ 需要 | Node.js/Python 开发经验 |

**Q4：Plugin 可以离线使用吗？**

安装在本地的 Plugin 可以离线使用。但如果 Plugin 内部调用了外部 API（如 GitHub API），那部分功能需要联网。

**Q5：Plugin 支持哪些编程语言？**

| 语言 | 支持度 | 适用场景 |
|------|-------|---------|
| JavaScript/TypeScript | ⭐⭐⭐⭐⭐ | MCP 集成、CLI 工具 |
| Python | ⭐⭐⭐⭐⭐ | 数据处理、AI 集成 |
| Shell Script | ⭐⭐⭐⭐ | 系统操作、自动化 |
| Go / Rust | ⭐⭐ | 高性能工具，需编译为可执行文件 |

**Q6：如何查看 Plugin 使用了多少 Token？**

可以通过 `/cost` 命令查看整体 Token 消耗。Plugin 本身不单独计费，Token 消耗取决于 Plugin 加载的提示词量和对话内容。

**Q7：可以同时安装多个版本的 Plugin 吗？**

不可以。同一个 Plugin 只能安装一个版本。如果需要在不同项目使用不同版本，可以用项目级安装（默认）隔离。

**Q8：Plugin 报错如何获取帮助？**

1. 查看 Plugin 的 README 和 GitHub Issues
2. 在 Anthropic Discord 的 `#claude-code-plugins` 频道提问
3. 提 GitHub Issue 时包含：系统环境、Claude Code 版本、Plugin 版本、完整错误信息

---

## 9. 总结

本章你已掌握：

1. **Plugin 本质**：可分享的 Commands + Skills + Hooks + MCP 打包体，一键安装、自动更新
2. **核心区别**：Plugin 是"超集"概念，解决了 Commands/Skills 无法便捷分享的痛点
3. **生态现状**：官方 Marketplace + 社区 200+ Plugin，按需选择
4. **安装管理**：`/plugin` 界面的 Discover / Installed / Marketplaces 三大标签页，全程图形化操作
5. **自定义开发**：`.claude-plugin/plugin.json` 清单 + `skills/` + `commands/` 标准结构，`--plugin-dir` 秒测试
6. **发布分享**：GitHub Release 发布 + 官方 Marketplace 表单提交，让全世界用上你的 Plugin

---

## 10. 参考资料

### Plugin 相关资源

- [Claude Code 官方 Plugin 文档](https://code.claude.com/docs/zh-CN/plugins)
- [知乎深度讲解：Claude Code Plugin 完整指南](https://zhuanlan.zhihu.com/p/1999547961936462596)
- [知乎教程：Claude Code Plugin 实战与最佳实践](https://zhuanlan.zhihu.com/p/1966486877088506681)
- [ClaudeCN 中文文档：Plugin 开发与使用](https://claudecn.com/docs/claude-code/plugins/)

---

## 下一步学习

| 章节 | 主题 | 你将学到 |
|------|------|---------|
| 第07章 | 从单兵到团队——企业级协作规范、CI/CD 与安全合规实战 | 团队协作规范、GitHub Actions CI/CD 集成、安全权限配置、成本控制 |
| 第08章 | 企业深水区——密钥安全、团队配置与合规审计全攻略 | 团队配置统一化、Git Hooks 自动审查、API Key 分级管理、GDPR/SOC2 合规、安全扫描 |
