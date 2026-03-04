# 第四章：让Claude连上一切——MCP与Hooks实战

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
- [4. 总结](#4-总结)
- [5. 参考资料](#5-参考资料)
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

## 4. 总结

本章你已掌握：

1. **MCP 本质**：AI 连接外部工具的 USB 标准，`.mcp.json` 配置，`npx` 启动
2. **常用服务器**：filesystem / github / sqlite / context7 等 10 个主流 Server 速查
3. **三大作用域**：Local（私密）> Project（团队）> User（全局），按需选择
4. **Commands 联动**：`mcp__服务器名__工具名` 在 Commands 中直接调用 MCP
5. **Hooks 本质**：事件驱动的 shell 脚本，PreToolUse / PostToolUse / Stop 三大时机
6. **Hooks 实战**：自动格式化、安全日志、完成通知，一次配置永久生效

---

## 5. 参考资料

### MCP 相关资源

- [Claude Code 官方 MCP 文档](https://code.claude.com/docs/zh-CN/mcp)
- [知乎深度讲解：MCP 协议与应用实战](https://zhuanlan.zhihu.com/p/1963592231739957552)
- [CSDN 教程：Claude Code MCP 集成完整指南](https://blog.csdn.net/m0_74837192/article/details/150616899)

### Hooks 相关资源

- [Claude Code 官方 Hooks 文档](https://code.claude.com/docs/zh-CN/hooks)
- [知乎深度讲解：Claude Hooks 自动化工作流完全指南](https://zhuanlan.zhihu.com/p/1950634615065809103)
- [CSDN 教程：Claude Hooks 实战与最佳实践](https://blog.csdn.net/qq_20042935/article/details/156891507)

---

## 下一步学习

| 章节 | 主题 | 你将学到 |
|------|------|---------|
| 第05章 | Skills 定制——给Claude装上专属能力包 | Skill 开发、自动激活机制、复杂提示词组织 |
| 第06章 | CLAUDE.md 完整配置指南 | 项目规范管理、长期记忆维护、团队协作 |
| 第07章 | Plugins 生态完整指南 | 插件安装、配置与自定义开发 |
