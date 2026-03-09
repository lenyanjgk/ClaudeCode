# 第四章：让Claude连上一切——MCP、Hooks与Subagent实战

## 目录

- [1. 前言](#1-前言)
- [2. MCP 集成](#2-mcp-集成)
  - [2.1 MCP 是什么](#21-mcp-是什么)
  - [2.2 配置方式](#22-配置方式)
  - [2.3 常用服务器速查](#23-常用服务器速查)
  - [2.4 三大作用域](#24-三大作用域)
  - [2.5 在 Commands 中调用 MCP](#25-在-commands-中调用-mcp)
  - [2.6 故障排查](#26-故障排查)
  - [2.7 参考资料](#27-参考资料)
- [3. Hooks 系统](#3-hooks-系统)
  - [3.1 Hooks 是什么](#31-hooks-是什么)
  - [3.2 配置方式](#32-配置方式)
  - [3.3 实战示例](#33-实战示例)
  - [3.4 可用环境变量](#34-可用环境变量)
  - [3.5 常用场景速查](#35-常用场景速查)
  - [3.6 故障排查](#36-故障排查)
- [4. Subagent 子代理体系](#4-subagent-子代理体系)
  - [4.1 Subagent 是什么](#41-subagent-是什么)
  - [4.2 快速安装](#42-快速安装)
  - [4.3 /agents 交互式创建（从零搭建）](#43-agents-交互式创建从零搭建)
  - [4.4 代理目录总览](#44-代理目录总览)
  - [4.5 并行调用多个专家](#45-并行调用多个专家)
  - [4.6 故障排查](#46-故障排查)
- [5. 总结](#5-总结)
- [6. 参考资料](#6-参考资料)
- [下一步学习](#下一步学习)

---

## 1. 前言

学完 Commands，你已经能把重复提示词压缩成一个词。

但 Claude 本身有能力边界——它不能直接操作数据库、不能搜网页、不能推代码到 GitHub。

**MCP 和 Hooks 就是突破这个边界的两把钥匙：**

| 工具 | 解决什么问题 | 类比 |
|------|------------|------|
| MCP | 让 Claude 调用外部服务（数据库/GitHub/搜索） | 给 Claude 装"插件" |
| Hooks | 让 Claude 在操作前后自动触发脚本（lint/测试/通知） | 给 Claude 设"自动触发器" |

---

## 2. MCP 集成

### 2.1 MCP 是什么

MCP（Model Context Protocol）是 AI 连接外部工具的**"USB 标准"**——统一协议，任何 MCP Server 都能即插即用地被 Claude 调用。

| 对比 | 没有 MCP | 有 MCP |
|------|---------|--------|
| 对接成本 | 每个工具单独开发 | 一次开发，处处复用 |
| 兼容性 | 各平台不通用 | 开放标准，跨平台 |
| 生态 | 各自为战 | 社区共建，直接拿来用 |

### 2.2 配置方式

**方式一：配置文件（推荐团队）**

在项目根目录创建 `.mcp.json`：

```json
{
  "mcpServers": {
    "服务器名": {
      "command": "npx",
      "args": ["-y", "包名", "附加参数"],
      "env": {
        "API_KEY": "${环境变量名}"
      }
    }
  }
}
```

> `${GITHUB_TOKEN}` 会自动读取同名环境变量，**不要把密钥硬写进文件**。

**方式二：CLI 快速添加（个人测试）**

```bash
claude mcp add <具体mcp地址>   # 添加
claude mcp list                          # 查看所有
claude mcp remove <名称>                 # 删除
```

```
claude mcp add filesystem -s user -- npx -y @modelcontextprotocol/server-filesystem D:/Desktop D:/develop
```

![image-20260303093940759](D:\Desktop\cc\assets\image-20260303093940759.png)

```
claude mcp list
或者/mcp
```

![image-20260303094059462](D:\Desktop\cc\assets\image-20260303094059462.png)

```
claude mcp remove filesystem -s user
```

![image-20260303094135986](D:\Desktop\cc\assets\image-20260303094135986.png)

删除成功：![image-20260303094218308](D:\Desktop\cc\assets\image-20260303094218308.png)

**作用域指定：**

```bash
claude mcp add --scope project <名称> ...  # 项目级（默认）
claude mcp add --scope user <名称> ...     # 用户级（全局）
```

### 2.3 常用服务器速查

| 分类 | 服务器 | 包名 | 需 API Key | 推荐 |
|------|--------|------|-----------|------|
| 文件 | filesystem | `@modelcontextprotocol/server-filesystem` | ❌ | ⭐⭐⭐ |
| 文件 | memory | `@modelcontextprotocol/server-memory` | ❌ | ⭐⭐ |
| 数据库 | sqlite | `@modelcontextprotocol/server-sqlite` | ❌ | ⭐⭐⭐ |
| 数据库 | postgres | `@modelcontextprotocol/server-postgres` | ❌（需连接串） | ⭐⭐ |
| 开发 | github | `@modelcontextprotocol/server-github` | ✅ | ⭐⭐⭐ |
| 开发 | git | `@modelcontextprotocol/server-git` | ❌ | ⭐⭐ |
| 搜索 | brave-search | `@modelcontextprotocol/server-brave-search` | ✅ | ⭐⭐⭐ |
| 搜索 | fetch | `@modelcontextprotocol/server-fetch` | ❌ | ⭐⭐ |
| 知识 | context7 | `@upstash/context7-mcp` | ❌ | ⭐⭐⭐ |
| 自动化 | puppeteer | `@modelcontextprotocol/server-puppeteer` | ❌ | ⭐⭐ |

**典型配置示例（Filesystem + GitHub）：**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "."],
      "env": {}
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" }
    }
  }
}
```

启动 `claude` 时看到 `✓ filesystem` `✓ github` 即配置成功，C盘目录下 用户名下的 .claude.json文件。

![image-20260303094322453](D:\Desktop\cc\assets\image-20260303094322453.png)

### 2.4 三大作用域

| 作用域 | 存储位置 | 优先级 | 适用场景 |
|--------|---------|--------|---------|
| Local | `~/.claude.json`（项目条目） | 最高 | 含 API Key，绝不提交 Git |
| Project | 项目根 `.mcp.json` | 中 | 团队共享，版本控制 |
| User | `~/.claude.json`（全局） | 最低 | 个人通用工具，跨项目可用 |

> **选择原则**：含密钥 → Local；团队必需 → Project；个人常用 → User。

### 2.5 在 Commands 中调用 MCP

第三章的 `allowed-tools` 直接支持 MCP 工具，格式：`mcp__服务器名__工具名`

```markdown
---
allowed-tools:
  - mcp__github__create_issue
  - mcp__filesystem__read_file
  - mcp__brave-search__brave_web_search
---
```

这样 Commands 就能直接驱动 GitHub、文件系统、搜索等外部服务，形成完整自动化工作流。

![image-20260303092442716](D:\Desktop\cc\assets\image-20260303092442716.png)

### 2.6 故障排查

| 现象 | 原因 | 解决方法 |
|------|------|---------|
| 启动时看不到 MCP 服务器 | 配置文件路径或格式错误 | 确认 `.mcp.json` 在项目根目录，JSON 格式正确 |
| 服务器启动失败 | Node.js 版本过低 | 升级到 v18+ |
| `${VAR}` 环境变量未生效 | 变量未导出或终端未重启 | `echo $VAR` 验证，重启终端 |
| 工具调用被拒 | `allowed-tools` 未声明 | 在 Commands frontmatter 加入工具名 |
| npx 下载超时 | 网络问题 | `npm config set registry https://registry.npmmirror.com` |

---

## 3. Hooks 系统

### 3.1 Hooks 是什么

Hooks 是**事件驱动的自动化触发器**——Claude 执行特定操作时，自动运行你预设的 shell 脚本。

比如可以在ClaudeCode 写完代码后，自动执行某些命令的格式化，以便让最终的代码更加美观，更加符合我们的需求

| Hook 时机 | 触发事件 | 典型用途 |
|-----------|---------|---------|
| `PreToolUse` | Claude 调用工具**之前** | 安全拦截危险命令、参数校验 |
| `PostToolUse` | Claude 调用工具**之后** | 自动 lint / 测试 / 格式化 |
| PostToolUseFailure | Claude 调用工具失败后 | 错误日志记录、重试机制触发、失败原因分析 |
| `Notification` | Claude 发送通知时 | 桌面弹窗、音效提醒 |
| `Stop` | Claude 完成一轮回复后 | 自动保存、发送完成通知 |

> 类比：MCP 是给 Claude 装插件，Hooks 是给 Claude 装"自动触发器"——它做完某件事就自动帮你跑一段脚本。

![image-20260303094903495](D:\Desktop\cc\assets\image-20260303094903495.png)

### 3.2 配置方式

Hooks 写在 `settings.json` 的 `hooks` 字段，支持两个位置：

| 位置 | 路径 | 生效范围 |
|------|------|---------|
| 项目级 | `.claude/settings.json` | 仅当前项目，可提交 Git |
| 用户级 | `~/.claude/settings.json` | 所有项目 |

**配置格式：**

```json
{
  "hooks": {
    "事件名": [
      {
        "matcher": "工具名正则",
        "hooks": [
          {
            "type": "command",
            "command": "要执行的 shell 命令"
          }
        ]
      }
    ]
  }
}
```

**`matcher` 说明**：匹配工具名，支持正则。`Write|Edit` 同时匹配两个工具，留空则匹配所有工具。

### 3.3 实战示例

**写完文件自动格式化（最常用）**提取路径 + 格式化文件

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' I xargs prettier --write"
          }
        ]
      }
    ]
  }
}
```

### 3.4 可用环境变量

Hook 脚本执行时，Claude 会注入以下变量：

| 变量 | 含义 | 示例值 |
|------|------|--------|
| `$CLAUDE_TOOL_NAME` | 当前调用的工具名 | `Write` |
| `$CLAUDE_FILE_PATHS` | 被操作的文件路径（空格分隔） | `src/app.ts src/utils.ts` |
| `$CLAUDE_PROJECT_DIR` | 项目根目录 | `/Users/me/project` |
| `$CLAUDE_TOOL_INPUT` | 工具的完整输入（JSON） | `{"command":"ls -la"}` |

### 3.5 常用场景速查

| 场景 | 事件 | 工具 matcher | 脚本示意 |
|------|------|-------------|---------|
| 保存后自动 prettier | PostToolUse | `Write\|Edit` | `prettier --write $CLAUDE_FILE_PATHS` |
| 保存后自动 ESLint | PostToolUse | `Write\|Edit` | `eslint --fix $CLAUDE_FILE_PATHS` |
| 写完跑单元测试 | PostToolUse | `Write` | `npm test -- --passWithNoTests` |
| 记录所有 Bash 命令 | PreToolUse | `Bash` | `echo $CLAUDE_TOOL_INPUT >> audit.log` |
| 完成后播放音效 | Stop | （空） | `afplay /System/Library/Sounds/Glass.aiff` |
| 完成后桌面通知 | Stop | （空） | `osascript -e 'display notification...'` |

> **注意**：`PreToolUse` Hook 退出码为 `2` 时会**阻止**该工具调用，其余退出码不会阻断执行。

### 3.6 故障排查

| 现象 | 原因 | 解决方法 |
|------|------|---------|
| Hook 没有触发 | settings.json 路径或格式错误 | 确认文件位置，JSON 格式用在线工具验证 |
| 脚本报"命令未找到" | 脚本用了 shell 别名或 PATH 不完整 | 写完整路径，如 `/usr/local/bin/prettier` |
| PreToolUse 误拦截 | matcher 正则范围太宽 | 精确化正则，如 `^Bash$` 而非 `Bash` |
| 文件路径变量为空 | 工具不涉及文件操作 | 改用 `$CLAUDE_TOOL_INPUT` 获取完整输入 |

---

## 4. Subagent 子代理体系

### 4.1 Subagent 是什么

学完 MCP 和 Hooks，你已经能扩展 Claude 的能力边界——让它能调用外部服务、自动运行脚本。

但还有个更深的问题——**一个 Claude 怎么同时精通 100+ 个专业领域？**

**Subagent（子代理）**就是答案：Claude Code 通过 Task 工具启动的**专业化 AI 代理**，每个都是该领域的专家。

| 工具 | 解决什么问题 | 类比 |
|------|------------|------|
| MCP | 让 Claude 调用外部服务 | 给 Claude 装"插件" |
| Hooks | 让 Claude 自动运行脚本 | 给 Claude 设"自动触发器" |
| **Subagent** | **让 Claude 调动专家团队** | **给 Claude 配"顾问团队"** |

#### Subagent 的核心优势

**并行协作**：同时启动多个专家代理，处理不同任务，效率翻倍。

```
单个 Claude 顺序处理：
  代码审查 → 性能优化 → 安全审计 → 文档生成（耗时）

多个 Subagent 并行处理：
  code-reviewer  ─────┐
  performance-pro ────┼──→ 同时并行，结果聚合（快速）
  security-auditor ───┤
  doc-writer  ────────┘
```

**领域专业化**：每个 Subagent 都是特定领域的深度专家，质量远高于通用 Claude。

**按需扩展**：100+ 专家代理库，你只需说一句话，Claude 自动调用最合适的专家。

> **⚠️ 重要提醒**：使用多 Agent 并行调用的 token 消耗量巨大。每启动一个子代理都会消耗独立的上下文窗口。**建议按需调用，不要一次性启动太多代理。**

---

### 4.2 快速安装

#### 方法一：交互式脚本安装（推荐）

```bash
claude plugin marketplace add VoltAgent/awesome-claude-code-subagents
```

```
claude plugin install <plugin-name>
```

或者/plugin Installed界面自行安装

#### 方法二：手动安装（完全控制）

1. **确定存放路径**：
   - 全局可用：`~/.claude/agents/`
   - 项目专用：`.claude/agents/`（项目根目录）

2. **复制文件**：从仓库的 `categories/` 文件夹中找到你需要的 `.md` 代理定义文件，复制到上述路径即可。

**验证安装**：
```bash
# 查看已启用的子代理
claude /agents
```

### 故障排查与相关截图：

如果是网络问题需要检查自己的代理软件（🪜）

可能安装过程需要代理软件，看自己的个人配置（自己的代理接口）或者手动安装

```
$env:HTTP_PROXY = "http://127.0.0.1:7897"
$env:HTTPS_PROXY = "http://127.0.0.1:7897"
```

![image-20260306143259905](D:\Desktop\cc\assets\image-20260306143259905.png)

![image-20260306143333962](D:\Desktop\cc\assets\image-20260306143333962.png)

![image-20260306144520059](D:\Desktop\cc\assets\image-20260306144520059.png)

---

### 4.3 /agents 交互式创建（从零搭建）

除了安装现成的代理库，你还可以用 `/agents` 命令**从零创建自己的 Subagent**——整个过程全程交互式引导，不需要手写任何配置文件。

#### 第一步：输入 `/agents`

在 Claude Code 对话中输入 `/agents`，你会看到当前已有的代理列表，底部有 **Create new agent** 选项：

>  ![image-20260309105503911](D:\Desktop\cc\assets\image-20260309105503911.png)

#### 第二步：选择存放位置

| 选项 | 说明 | 适合场景 |
|------|------|---------|
| `Project (.claude/agents/)` | 仅当前项目可用 | 项目专用代理 |
| `Personal (~/.claude/agents/)` | 全局可用 | 通用代理，跨项目复用 |

>  ![image-20260309105529950](D:\Desktop\cc\assets\image-20260309105529950.png)

#### 第三步：选择创建方式

| 方式 | 说明 | 适合 |
|------|------|------|
| **Generate with Claude（推荐）** | 你只需描述需求，Claude 自动生成完整配置 | 新手首选 |
| Manual configuration | 手动填写每个字段 | 需要精确控制的场景 |

>  ![image-20260309105555441](D:\Desktop\cc\assets\image-20260309105555441.png)

#### 第四步：描述代理职责

用自然语言描述这个代理应该做什么、什么时候被调用。**写得越详细，生成效果越好：**

```
这是一个用于代码审核的 SubAgent。在用户要求"代码审核"的时候调用它。
```

> ![image-20260309105637688](D:\Desktop\cc\assets\image-20260309105637688.png)

#### 第五步：等待 Claude 生成

Claude 会根据你的描述自动生成代理的名称、系统提示词、工具选择等配置：

> ![image-20260309105649445](D:\Desktop\cc\assets\image-20260309105649445.png)

#### 第六步：选择工具权限

选择这个代理可以使用哪些工具。根据代理职责按需勾选：

| 工具类别 | 包含工具 | 说明 |
|---------|---------|------|
| Read-only tools | Glob、Grep、Read 等 | 只读，安全无副作用 |
| Edit tools | Edit、Write 等 | 可修改文件 |
| Execution tools | Bash 等 | 可执行命令 |
| MCP tools | 已配置的 MCP 服务 | 调用外部服务 |

> ![image-20260309105659270](D:\Desktop\cc\assets\image-20260309105659270.png) 

#### 第七步：选择模型

| 模型 | 特点 | 建议场景 |
|------|------|---------|
| **Sonnet** | 性能均衡，速度与质量兼顾 | **日常代理首选** |
| Opus | 最强推理能力，成本最高 | 复杂架构 / 深度分析 |
| Haiku | 速度快、成本低 | 简单任务 / 批量处理 |
| Inherit from parent | 继承主会话模型 | 跟随当前设置 |

> ![image-20260309105726331](D:\Desktop\cc\assets\image-20260309105726331.png)

#### 第八步：选择颜色标识

给代理设置一个背景色，方便在会话中一眼识别它的输出（比如代码审核用绿色、安全审计用红色）。

> ![image-20260309105738430](D:\Desktop\cc\assets\image-20260309105738430.png) 

#### 第九步：配置记忆

| 选项 | 说明 | 建议 |
|------|------|------|
| **Enable (.claude/agent-memory/)** | 项目级记忆，代理会记住历史上下文 | **推荐** |
| None | 无持久记忆 | 一次性任务 |
| User scope | 全局记忆（`~/.claude/agent-memory/`） | 跨项目通用代理 |
| Local scope | 本地记忆，不提交 Git | 含敏感信息的场景 |

> ![image-20260309105810187](D:\Desktop\cc\assets\image-20260309105810187.png) 

#### 第十步：确认保存

最终会展示代理的完整配置，确认无误后按 `s` 或 `Enter` 保存：

```
Name:        code-reviewer
Location:    .claude/agents/code-reviewer.md
Tools:       Glob, Grep, Read, WebFetch, WebSearch
Model:       Sonnet
Memory:      Project (.claude/agent-memory/)

Description: Use this agent when the user explicitly requests a
             '代码审核' (code review), '审查代码', '检查代码'...

System prompt:
你是一位资深代码审核专家，拥有超过15年的软件工程经验，
精通多种编程语言和架构模式...
```

> ![image-20260309105823738](D:\Desktop\cc\assets\image-20260309105823738.png)

![image-20260309105839854](D:\Desktop\cc\assets\image-20260309105839854.png)

#### 实战调用

保存完成后，直接在对话里说 **"给我做下代码审核"**——Claude 会自动识别并调用刚刚创建的 `code-reviewer` 代理。

你会看到**绿色标识**，表示 Subagent 正在独立执行审核任务：

> ![image-20260309105848512](D:\Desktop\cc\assets\image-20260309105848512.png)

审核完成后，代理会返回完整的审核报告，收工：

> ![image-20260309105902528](D:\Desktop\cc\assets\image-20260309105902528.png)

> **凯神提醒**：`/agents` 交互式创建是目前最推荐的方式——不需要手写 `.md` 文件，Claude 帮你自动生成完整配置（名称、描述、系统提示词、工具权限全都有），新手 2 分钟就能搞定一个专属代理。

---

### 4.4 代理目录总览

VoltAgent 提供的 10 大类专家代理（100+ 代理），按需安装：

#### 1️⃣ 核心开发 (Core Development)
api-designer / backend-developer / electron-pro / frontend-developer / fullstack-developer / graphql-architect / microservices-architect / mobile-developer / ui-designer / websocket-engineer / wordpress-master

#### 2️⃣ 语言专家 (Language Specialists)
typescript-pro / python-pro / rust-engineer / golang-pro / java-architect / javascript-pro / react-expert / vue-expert / angular-architect / nextjs-developer / swift-expert / kotlin-expert / cpp-pro / csharp-developer / php-pro / sql-pro / django-developer / laravel-expert / rails-expert / spring-boot-engineer / flutter-expert / elixir-expert / dotnet-core-expert / powershell-pro

#### 3️⃣ 基础设施 (Infrastructure)
cloud-architect / devops-engineer / kubernetes-expert / terraform-engineer / database-admin / sre / deployment-engineer / azure-infra-engineer / network-engineer / platform-engineer / security-engineer / incident-responder / windows-infra-admin

#### 4️⃣ 质量与安全 (Quality & Security)
code-reviewer / security-auditor / qa-automation-engineer / performance-engineer / debugging-expert / error-detective / penetration-tester / architecture-reviewer / accessibility-tester / chaos-engineer / compliance-auditor / testing-automation-expert

#### 5️⃣ 数据与人工智能 (Data & AI)
ai-engineer / llm-architect / ml-engineer / data-engineer / data-scientist / data-analyst / database-optimizer / postgres-pro / mlops-engineer / nlp-engineer / prompt-engineer

#### 6️⃣ 开发者体验 (Developer Experience)
refactoring-expert / documentation-engineer / git-workflow-manager / legacy-code-modernizer / mcp-developer / build-engineer / cli-developer / dependency-manager / dx-optimizer / tooling-engineer

#### 7️⃣ 专业领域 (Specialized Domains)
blockchain-developer / game-developer / fintech-engineer / iot-engineer / embedded-systems-engineer / api-documenter / seo-specialist / mobile-app-developer / m365-admin

#### 8️⃣ 业务与产品 (Business & Product)
product-manager / business-analyst / project-manager / scrum-master / technical-writer / ux-researcher / customer-success-manager / sales-engineer / legal-advisor / content-marketing-specialist

#### 9️⃣ 元数据与编排 (Meta & Orchestration)
multi-agent-coordinator / workflow-orchestrator / agent-organizer / agent-installer / context-manager / task-dispatcher / error-coordinator / performance-monitor / knowledge-synthesizer / it-ops-orchestrator

**推荐新手起点**：
- `code-reviewer`（代码审查）
- `debugging-expert`（调试）
- `refactoring-expert`（重构）
- `documentation-engineer`（文档）

---

### 4.5 并行调用多个专家

#### 自动识别调用

当你的描述符合某个子代理的专业领域时，Claude 会**自动调用**对应的专家：

```
你：这段代码有性能问题
Claude：[自动识别] 调用 performance-engineer 代理...
        基于性能优化的最佳实践，我来分析瓶颈...

你：帮我检查代码质量
Claude：[自动识别] 调用 code-reviewer 代理...
        我注意到以下代码质量问题...
```

#### 显式调用多个专家

对于复杂任务，**显式并行调用**多个专家：

```
并行调用各个专家查看/解决 XXXX 问题

需要：
- code-reviewer 检查代码质量
- performance-engineer 分析性能
- security-auditor 审计安全漏洞
```

Claude 会同时启动 3 个子代理，返回综合报告。

#### 常见场景示例

| 场景 | 调用代理 | 效果 |
|------|---------|------|
| **代码审查** | code-reviewer | 快速找出质量问题 |
| **性能优化** | performance-engineer + database-optimizer | 全面分析瓶颈 |
| **安全审计** | security-auditor + penetration-tester | 深度安全检查 |
| **新项目启动** | architecture-reviewer + project-manager + tech-writer | 快速生成架构文档 |
| **重构遗留代码** | legacy-code-modernizer + refactoring-expert | 系统现代化 |
| **API 设计** | api-designer + api-documenter | 完整的设计方案 |

---

### 4.6 故障排查

| 现象 | 原因 | 解决方法 |
|------|------|---------|
| `/agents` 看不到任何代理 | 安装路径错误或仓库未克隆 | 确认代理文件在 `~/.claude/agents/` 或 `.claude/agents/` |
| 代理安装成功但不被调用 | Subagent 定义文件格式错误 | 检查 `.md` 文件的 frontmatter 格式 |
| 并行调用代理导致超时 | 启动的代理太多，超过 token 限制 | 减少并行代理数量，改为分批调用 |
| 代理返回结果不符合预期 | 代理描述匹配度不高 | 在调用时更明确地说明需求，或显式指定代理 |
| Token 消耗过多 | 每个子代理都有独立上下文 | 按需调用，避免一次性启动太多代理 |

---

## 5. 总结

本章你已掌握：

1. **MCP 本质**：AI 连接外部工具的 USB 标准，`.mcp.json` 配置，`npx` 启动
2. **常用服务器**：filesystem / github / sqlite / context7 等 10 个主流 Server 速查
3. **三大作用域**：Local（私密）> Project（团队）> User（全局），按需选择
4. **Commands 联动**：`mcp__服务器名__工具名` 在 Commands 中直接调用 MCP
5. **Hooks 本质**：事件驱动的 shell 脚本，PreToolUse / PostToolUse / Stop 三大时机
6. **Hooks 实战**：自动格式化、安全日志、完成通知，一次配置永久生效
7. **Subagent 本质**：Claude 调动的专家团队，100+ 代理库，按需并行协作
8. **/agents 创建**：交互式引导从零搭建专属代理，2 分钟完成，不需手写配置
9. **Subagent 实战**：快速安装、自动识别、显式并行调用，提升开发效率 10 倍

---

## 6. 参考资料

### MCP 相关资源

- [Claude Code 官方 MCP 文档](https://code.claude.com/docs/zh-CN/mcp)
- [知乎深度讲解：MCP 协议与应用实战](https://zhuanlan.zhihu.com/p/1963592231739957552)
- [CSDN 教程：Claude Code MCP 集成完整指南](https://blog.csdn.net/m0_74837192/article/details/150616899)

### Hooks 相关资源

- [Claude Code 官方 Hooks 文档](https://code.claude.com/docs/zh-CN/hooks)
- [知乎深度讲解：Claude Hooks 自动化工作流完全指南](https://zhuanlan.zhihu.com/p/1950634615065809103)
- [CSDN 教程：Claude Hooks 实战与最佳实践](https://blog.csdn.net/qq_20042935/article/details/156891507)

### Subagent 相关资源

- [Claude Code 官方 Subagent 文档](https://code.claude.com/docs/zh-CN/sub-agents)
- [CSDN 深度教程：Subagent 子代理完整指南](https://blog.csdn.net/qq_20042935/article/details/156897940)
- [ClaudeCN 中文文档：Subagent 高级用法](https://claudecn.com/docs/claude-code/advanced/subagents/)

---

## 下一步学习

| 章节 | 主题 | 你将学到 |
|------|------|---------|
| 第05章 | Skills 定制——给Claude装上专属能力包 | Skill 开发、自动激活机制、复杂提示词组织 |
| 第06章 | CLAUDE.md 完整配置指南 | 项目规范管理、长期记忆维护、团队协作 |
| 第07章 | Plugins 生态完整指南 | 插件安装、配置与自定义开发 |
