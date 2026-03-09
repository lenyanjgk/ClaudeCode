# 第七章：从单兵到团队——企业级协作规范、CI/CD 与安全合规实战

## 目录

- [1. 前言](#1-前言)
- [2. 团队协作规范](#2-团队协作规范)
  - [2.1 为什么需要团队规范](#21-为什么需要团队规范)
  - [2.2 项目结构标准化](#22-项目结构标准化)
  - [2.3 CLAUDE.md 三层配置体系](#23-claudemd-三层配置体系)
  - [2.4 代码审查命令](#24-代码审查命令)
  - [2.5 新成员入职清单](#25-新成员入职清单)
- [3. CI/CD 集成](#3-cicd-集成)
  - [3.1 GitHub Actions 基础配置](#31-github-actions-基础配置)
  - [3.2 多场景工作流](#32-多场景工作流)
  - [3.3 完整流水线示例](#33-完整流水线示例)
- [4. 安全与合规](#4-安全与合规)
  - [4.1 权限系统配置](#41-权限系统配置)
  - [4.2 allowedTools 白名单](#42-allowedtools-白名单)
  - [4.3 审计日志](#43-审计日志)
- [5. 性能与成本优化](#5-性能与成本优化)
  - [5.1 上下文管理](#51-上下文管理)
  - [5.2 成本控制](#52-成本控制)
  - [5.3 verbose 调试](#53-verbose-调试)
- [6. 故障排查](#6-故障排查)
- [7. 总结](#7-总结)
- [8. 参考资料](#8-参考资料)
- [下一步学习](#下一步学习)

---

## 1. 前言

学完前六章，你已经能把 Claude Code 玩得很溜——Commands 定制、MCP 扩展、Plugins 一键装、Skills 打包复用，个人工作流基本满足了。

但问题来了：**换个角色，从个人开发者变成带 5 人、10 人团队的技术负责人，Claude Code 怎么用？**

典型的团队混乱场景，你可能中枪过：

| 混乱现象 | 根因 | 后果 |
|---------|------|------|
| 每人的 CLAUDE.md 各不相同 | 没有统一规范 | AI 产出的代码风格乱成一锅粥 |
| 敏感密钥被 AI 意外提交到仓库 | 没有权限限制 | 安全事故，老板找你谈话 |
| 代码审查全靠人工盯 | 没接入 CI/CD | 效率低，问题漏网 |
| 换个项目所有配置从头再来 | 没有标准化结构 | 重复劳动，新人懵圈 |
| 团队月账单翻倍超预期 | 没有成本管控 | 财务部门找过来了 |

**本章解决的就是这些问题。**

从团队规范到 CI/CD 集成，从安全合规到成本控制，一章讲完企业级 Claude Code 部署的全部要点。3 人团队能用，100 人团队也能用。

---

## 2. 团队协作规范

### 2.1 为什么需要团队规范

3 人以下团队，靠沟通凑合；超过 5 人，没有规范就是灾难的开始。

规范要解决三个核心问题：

| 问题 | 解决方案 |
|------|---------|
| AI 产出质量不一致 | 统一 CLAUDE.md，全员共享同一套规则 |
| 配置无法传承 | 配置文件入库，随代码一起版本管理 |
| 新人上手慢 | 标准化目录结构 + 入职清单，1 天内跑通 |

### 2.2 项目结构标准化

**推荐的企业级目录结构：**

```
project-root/
├── .claude/                      # Claude Code 专用配置（全部入库）
│   ├── settings.json             # 团队统一权限与工具配置
│   ├── settings.local.json       # 个人本地覆盖（禁止入库！）
│   ├── commands/                 # 团队共享 Slash 命令
│   │   ├── code-review.md        # 代码审查命令
│   │   ├── security-check.md     # 安全检查命令
│   │   └── deploy.md             # 部署命令
│   └── skills/                   # 项目专属 Skill
│       └── project-skill/
│           └── SKILL.md
├── .github/
│   └── workflows/
│       └── claude-review.yml     # CI/CD 自动审查
├── docs/
│   └── ai-context/               # 给 AI 看的项目说明书
│       ├── project-structure.md
│       ├── coding-standards.md
│       └── architecture.md
├── src/
├── tests/
├── CLAUDE.md                     # 项目主配置（入库）
├── .mcp.json                     # MCP 服务器配置（入库）
└── .gitignore
```

**什么该入库，什么不该入库——一张表说清楚：**

| 文件/目录 | 入库策略 | 原因 |
|----------|---------|------|
| `.claude/settings.json` | ✅ 必须入库 | 团队统一权限配置 |
| `.claude/settings.local.json` | ❌ 禁止入库 | 个人偏好，可能含本地路径 |
| `.claude/commands/` | ✅ 必须入库 | 团队共享命令 |
| `.claude/skills/` | ✅ 必须入库 | 项目专属能力 |
| `CLAUDE.md` | ✅ 必须入库 | 团队共享项目规范 |
| `.mcp.json` | ✅ 必须入库（不含密钥） | 团队共享 MCP 配置 |
| `.env` / `.env.local` | ❌ 禁止入库 | 含密钥等敏感信息 |

**在 `.gitignore` 中明确写出来，不要靠记忆：**

```gitignore
# 禁止入库
.claude/settings.local.json
.env
.env.local
.env.*.local

# 确保这些不被其他规则误排除
!.claude/settings.json
!.claude/commands/
!.claude/skills/
!CLAUDE.md
!.mcp.json
```

**`docs/ai-context/` 是干嘛的？**

这个目录专门存放给 AI 看的项目上下文，不是给人看的 README，而是帮助 Claude 快速理解项目的「说明书」：

```markdown
# docs/ai-context/project-structure.md 示例

## 技术栈
- 前端：React 18 + TypeScript + Vite
- 后端：Node.js 20 + Fastify
- 数据库：PostgreSQL + Prisma ORM

## 核心模块
### 用户模块 (src/modules/user/)
负责注册、登录、权限管理，依赖 JWT + bcrypt

### 订单模块 (src/modules/order/)
负责下单、支付、状态管理，依赖用户模块和支付网关

## 代码约定
- 所有 API 响应格式：{ data, error, meta }
- 数据库操作必须使用 Prisma Client
- 日期时间统一 UTC
```

把详细内容放这里，CLAUDE.md 只写「详见 docs/ai-context/」，既让 AI 能找到，又不撑爆主配置文件的 token。

### 2.3 CLAUDE.md 三层配置体系

Claude Code 支持三层 CLAUDE.md，优先级从低到高：

| 层级 | 文件位置 | 作用范围 | 入库建议 |
|------|---------|---------|---------|
| 全局配置 | `~/.claude/CLAUDE.md` | 所有项目 | 个人文件，不入库 |
| 项目配置 | 项目根 `CLAUDE.md` | 当前项目全团队 | ✅ 必须入库 |
| 模块配置 | `src/legacy/CLAUDE.md` | 特定子目录 | ✅ 按需入库 |

**项目 CLAUDE.md 模板（精简版，控制在 1000 tokens 以内）：**

```markdown
# [项目名] - Claude Code 配置

## 1. 项目概览
- **技术栈**：React 18 + Node.js 20 + PostgreSQL
- **当前阶段**：开发中
- **详细上下文**：见 docs/ai-context/

## 2. 代码规范
- 语言：TypeScript，禁止使用 any（特殊情况需注释说明）
- 命名：文件 kebab-case，类 PascalCase，变量 camelCase，常量 UPPER_SNAKE_CASE
- 函数 ≤50 行，类 ≤300 行
- 所有公共 API 必须有 JSDoc 注释，注释使用中文

## 3. 安全规则
- 禁止硬编码 API 密钥、密码等敏感信息
- 禁止提交 .env 文件
- 数据库查询必须参数化，禁止字符串拼接

## 4. 测试要求
- 新功能必须有单元测试，核心逻辑覆盖率 >80%
- 集成测试必须覆盖主要用户流程

## 5. Git 规范
提交格式：`<type>(<scope>): <description>`
类型：feat / fix / docs / refactor / test / chore

## 6. 项目特殊注意
[此处填写项目特有的规则，如遗留代码处理方式、禁用的第三方库等]
```

> **精简原则**：CLAUDE.md 目标控制在 <1,000 tokens。超长的配置文件每次请求都会消耗大量 token，得不偿失。详细内容一律放进 `docs/ai-context/`。

**全局 CLAUDE.md 模板（`~/.claude/CLAUDE.md`）：**

```markdown
# 全局 Claude Code 配置

## 个人偏好
- 使用中文回复
- 代码注释使用中文
- 偏好简洁直接的代码风格

## 安全底线（所有项目通用）
- 永远不在代码中包含真实的 API 密钥
- 不自动执行 rm -rf、DROP TABLE 等危险命令，执行前必须确认
- 不强制推送到 main/master 分支
```

### 2.4 代码审查命令

把代码审查流程标准化，创建团队共享命令 `.claude/commands/code-review.md`：

```markdown
对当前代码变更进行全面审查。

参数：$ARGUMENTS（可选，指定文件路径；不填则审查 git diff 的所有变更）

## 审查维度

### 1. 代码质量
- 命名是否表意，是否有重复代码
- 函数/类是否超出长度限制

### 2. 安全性
- 是否有 SQL 注入、XSS 风险
- 是否有硬编码的敏感信息
- 敏感数据处理是否安全

### 3. 性能
- 是否有 N+1 查询
- 是否有循环内的不必要计算

### 4. 测试
- 新增功能是否有对应测试
- 边界情况是否覆盖

## 输出格式

### 必须修改（Blocking）
- 问题描述 + 代码位置 + 修复建议

### 建议修改（Suggestion）
- 问题描述 + 改进建议

### 做得好的地方（Praise）
- 值得保留的优秀实践
```

使用方式：

```
/code-review              # 审查所有最近变更
/code-review src/auth/    # 只审查指定目录
```

### 2.5 新成员入职清单

```markdown
## Claude Code 新成员入职清单

### 第1步：环境准备
- [ ] 安装 Claude Code CLI（参考第一章）
- [ ] 配置全局 ~/.claude/CLAUDE.md（写入个人偏好）
- [ ] 获取 API 密钥并配置到环境变量

### 第2步：项目配置
- [ ] 克隆项目仓库
- [ ] 进入项目目录，运行 `claude` 初始化
- [ ] 输入 `/mcp` 确认 MCP 服务器正常连接
- [ ] 创建个人 .claude/settings.local.json（覆盖个人偏好，不入库）

### 第3步：熟悉规范
- [ ] 阅读项目 CLAUDE.md 和 docs/ai-context/
- [ ] 输入 `/help` 查看团队自定义命令列表
- [ ] 运行一次 `/code-review` 体验审查流程

### 第4步：验证配置
- [ ] 提交一个测试 PR，确认 CI 自动审查触发正常
- [ ] 检查权限配置：尝试执行一条危险命令验证是否被拦截
```

---

## 3. CI/CD 集成

### 3.1 GitHub Actions 基础配置

Anthropic 提供了官方 GitHub Action：`anthropics/claude-code-action`，直接在 PR 上触发 Claude Code 审查，零代码配置。

**准备工作——先在 GitHub 仓库添加 Secret：**

1. 进入仓库 **Settings → Secrets and variables → Actions**
2. 点击 **New repository secret**
3. 名称：`ANTHROPIC_API_KEY`，值：你的 API Key

**最简配置**（`.github/workflows/claude-review.yml`）：

```yaml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize, reopened]
  issue_comment:
    types: [created]

permissions:
  contents: read
  pull-requests: write
  issues: write

jobs:
  claude-review:
    if: |
      github.event_name == 'pull_request' ||
      (github.event_name == 'issue_comment' &&
       contains(github.event.comment.body, '@claude'))
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0   # 获取完整历史，用于 diff 比较

      - name: Run Claude Code Review
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          model: claude-sonnet-4-6-20250929
          max_tokens: 4096
          timeout: 300
```

配置完成后：
- 每次提 PR，Claude 自动审查并在 PR 下方留评论
- 在 PR 或 Issue 评论中 `@claude xxx`，Claude 响应交互式指令

### 3.2 多场景工作流

一个完整的团队工作流通常需要三类并行 Job：

| Job | 触发时机 | 功能 |
|-----|---------|------|
| `review` | PR 创建/更新 | 代码质量审查 |
| `security-scan` | PR 创建/更新 | 安全漏洞扫描 |
| `interactive` | 评论含 `@claude` | 响应交互式指令 |

```yaml
name: Claude Code CI

on:
  pull_request:
    types: [opened, synchronize, reopened]
  issue_comment:
    types: [created]

permissions:
  contents: read
  pull-requests: write
  issues: write

env:
  CLAUDE_MODEL: claude-sonnet-4-6-20250929

jobs:
  # ===== 代码审查 =====
  review:
    name: Code Review
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Claude Code Review
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          model: ${{ env.CLAUDE_MODEL }}
          max_tokens: 4096
          prompt: |
            请审查这次 PR 的代码变更，重点关注：
            1. 代码质量和可维护性
            2. 潜在的 bug 和安全问题
            3. 性能考量
            4. 测试覆盖建议
            请用中文回复，按「必须修改 / 建议修改 / 优点」三部分输出。

  # ===== 安全扫描 =====
  security-scan:
    name: Security Scan
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Claude Security Scan
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          model: ${{ env.CLAUDE_MODEL }}
          max_tokens: 2048
          prompt: |
            请对本次 PR 进行安全扫描，检查：
            1. 硬编码的敏感信息（API Key、密码、token 等）
            2. SQL 注入、XSS、命令注入风险
            3. 不安全的依赖版本
            4. 权限配置问题
            发现高危问题请在开头标注 [SECURITY-CRITICAL]。

  # ===== 交互式指令 =====
  interactive:
    name: Interactive Commands
    if: |
      github.event_name == 'issue_comment' &&
      contains(github.event.comment.body, '@claude')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Process Command
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          model: ${{ env.CLAUDE_MODEL }}
          prompt: ${{ github.event.comment.body }}
```

**交互式指令使用示例（在评论中）：**

```
@claude 帮我分析这段代码的性能瓶颈
@claude 生成这个模块的单元测试
@claude /code-review src/payment/
```

### 3.3 完整流水线示例

加上 lint、测试、构建、部署的完整 6 阶段流水线：

```yaml
name: Full CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  lint:
    name: Lint & Format
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci && npm run lint && npm run format:check

  test:
    name: Unit Tests
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci && npm test -- --coverage

  claude-review:
    name: Claude Review
    if: github.event_name == 'pull_request'
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          model: claude-sonnet-4-6-20250929
          max_tokens: 4096
          prompt: 请审查 PR 代码变更，输出质量评估、安全检查、改进建议三部分结果（中文）。

  build:
    name: Build
    needs: [test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/

  deploy-staging:
    name: Deploy Staging
    if: github.ref == 'refs/heads/develop'
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-output
      - run: echo "部署到 Staging 环境..."

  deploy-production:
    name: Deploy Production
    if: github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-output
      - run: echo "部署到 Production 环境..."
```

> **注意**：`claude-review` Job 只在 PR 事件时触发，push 到 main/develop 时不触发，避免浪费 token。

---

## 4. 安全与合规

### 4.1 权限系统配置

Claude Code 采用 **allow / deny 双列表**权限模型：

| 级别 | 行为 | 适用操作 |
|------|------|---------|
| Allow | 直接执行，无需确认 | 低风险高频操作（读文件、搜索） |
| Ask（默认） | 执行前弹出确认框 | 有一定影响的操作（编辑文件） |
| Deny | 完全拒绝，无法绕过 | 高危操作（删除、强推、sudo） |

**`.claude/settings.json` 权限配置示例：**

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "WebFetch",
      "WebSearch",
      "Bash(npm test *)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(git status)"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(rm -rf ~)",
      "Bash(git push --force)",
      "Bash(git reset --hard)",
      "Bash(sudo *)",
      "Bash(curl * | bash)",
      "Read(./.env)",
      "Read(./secrets/**)"
    ]
  }
}
```

> **规则说明**：未出现在任一列表中的工具和命令默认走 ask（弹出确认）。deny 优先级高于 allow，支持通配符匹配。

### 4.2 allowedTools 白名单

比 permissions 更细粒度的工具控制，支持路径限制：

```json
{
  "allowedTools": [
    "Read",
    "Glob",
    "Grep",
    "Edit(src/**)",
    "Write(src/**)",
    "Write(tests/**)",
    "Bash(npm *)",
    "Bash(git status)",
    "Bash(git diff *)",
    "Bash(git add *)",
    "Bash(git commit *)",
    "mcp__context7__*",
    "mcp__github__*"
  ],
  "disallowedTools": [
    "Bash(rm -rf *)",
    "Bash(sudo *)",
    "Write(.env*)",
    "Write(**/secrets/*)"
  ]
}
```

**白名单匹配规则速查：**

| 写法 | 含义 | 示例 |
|------|------|------|
| `"Read"` | 精确匹配工具名 | 只允许 Read 工具 |
| `"Bash(npm *)"` | 通配符匹配命令参数 | 允许所有 npm 子命令 |
| `"Edit(src/**)"` | 路径通配符 | 只允许编辑 src 目录下的文件 |
| `"mcp__context7__*"` | MCP 工具通配符 | 允许 context7 的所有功能 |

**分环境配置建议：**

| 环境 | 策略 | 说明 |
|------|------|------|
| 开发环境 | 宽松，但保留危险命令 deny | 便于迭代 |
| Staging | 中等，限制 Write 范围 | 模拟生产 |
| CI/CD | 严格，只保留必要工具 | 最小权限原则 |

### 4.3 审计日志

**启用审计日志配置（`.claude/settings.json`）：**

```json
{
  "audit": {
    "enabled": true,
    "logPath": "./logs/claude-audit/",
    "retention": "90d",
    "events": [
      "tool_use",
      "file_access",
      "command_execution",
      "permission_change",
      "error"
    ]
  }
}
```

**审计日志格式示例：**

```json
{
  "entries": [
    {
      "timestamp": "2026-01-15T10:30:45.123Z",
      "userId": "dev@company.com",
      "event": "tool_use",
      "tool": "Bash",
      "command": "npm test",
      "status": "success",
      "duration": 5432
    },
    {
      "timestamp": "2026-01-15T10:31:00.456Z",
      "userId": "dev@company.com",
      "event": "permission_denied",
      "tool": "Read",
      "path": ".env.production",
      "reason": "Path in denied list",
      "status": "blocked"
    }
  ]
}
```

**企业合规检查清单：**

| 合规项 | 检查内容 | 通过条件 |
|--------|---------|---------|
| 密钥安全 | API 密钥通过环境变量传入 | 代码中无硬编码密钥 |
| 敏感文件保护 | .env 类文件在 deny 列表中 | AI 无法读取 |
| 危险命令拦截 | rm -rf / sudo 被明确禁止 | deny 列表覆盖 |
| 审计追踪 | 所有工具调用有完整日志 | `audit.enabled = true` |
| 日志保留 | 保留期符合公司/行业政策 | `retention >= 90d` |
| 权限最小化 | 每个角色只有必要权限 | allowedTools 明确限定范围 |

---

## 5. 性能与成本优化

### 5.1 上下文管理

**上下文窗口的构成（以 Claude Sonnet 4.6 为例，上限 200K tokens）：**

| 组成部分 | 典型大小 | 优化优先级 |
|---------|---------|---------|
| 系统提示词 | ~2,000 tokens | 无法优化 |
| CLAUDE.md | 1,000–5,000 tokens | ⭐⭐⭐ 重点优化 |
| 对话历史 | 动态增长 | ⭐⭐⭐ 定期清理 |
| 工具返回结果 | 动态增长 | ⭐⭐ 控制读取量 |
| 当前消息 | 用户输入 | ⭐ 精简描述 |

**三个实用优化策略：**

**策略一：精简 CLAUDE.md**

```markdown
# ❌ 冗长写法（每次浪费几千 token）
我们的团队使用以下代码规范。首先，所有代码必须使用 TypeScript 编写。
其次，我们要求所有函数都有 JSDoc 注释。另外，变量命名必须遵循
camelCase 规范……（继续 500 字）

# ✅ 精简写法（同样信息量）
## 代码规范
- 语言：TypeScript，禁用 any
- 注释：JSDoc 必需（中文）
- 命名：变量 camelCase，类 PascalCase
- 行数：函数 ≤50，类 ≤300
```

**策略二：大任务分步拆解**

```
# ❌ 一次性大任务（容易超限，Claude 也容易出错）
"重构整个项目的用户模块、订单模块、支付模块..."

# ✅ 分步骤执行
步骤1：分析用户模块现状
步骤2：提出重构方案
步骤3：执行用户模块重构，验证结果
步骤4：继续订单模块
```

**策略三：分层 CLAUDE.md**

```
项目根/CLAUDE.md           # 全局规则（目标 <1,000 tokens）
├── src/CLAUDE.md          # 源码特定规则（<500 tokens）
├── tests/CLAUDE.md        # 测试规则（<300 tokens）
└── src/legacy/CLAUDE.md   # 遗留代码专用规则（<300 tokens）
```

**常用对话管理命令：**

| 命令 | 时机 | 效果 |
|------|------|------|
| `/clear` | 任务完成后，切换新任务前 | 清空对话历史，释放上下文 |
| `/compact` | 对话过长但需要保留上下文 | 压缩历史为摘要，节省 token |
| `Shift + Enter` | 需要输入多行指令时 | 换行不发送，整理好再提交 |

### 5.2 成本控制

**各模型价格参考（2026 年）：**

| 模型 | 输入 | 输出 | 推荐场景 |
|------|------|------|---------|
| Claude Haiku 4.5 | $0.25/M | $1.25/M | 简单代码生成、批量文件处理 |
| Claude Sonnet 4.6 | $3/M | $15/M | 日常开发、代码审查（默认推荐） |
| Claude Opus 4.5 | $15/M | $75/M | 架构设计、复杂推理任务 |

**单次对话成本估算：**

```
成本 = (输入 tokens / 1M × 输入价格) + (输出 tokens / 1M × 输出价格)

示例（Sonnet 4.6，10K 输入 + 2K 输出）：
= (10,000 / 1M × 3) + (2,000 / 1M × 15)
= $0.03 + $0.03 = $0.06/次
```

**成本控制配置（`.claude/settings.json`）：**

```json
{
  "costControl": {
    "dailyLimit": 10.0,
    "monthlyLimit": 200.0,
    "alertThreshold": 0.8,
    "actions": {
      "onDailyLimitReached": "warn",
      "onMonthlyLimitReached": "block"
    }
  }
}
```

**五条省钱建议：**

| 建议 | 效果 |
|------|------|
| 简单任务用 Haiku，复杂任务用 Opus | 成本最多降低 80% |
| 批量处理：一次审查多个文件而不是逐个审查 | 减少重复初始化开销 |
| 用 `/compact` 压缩长对话 | 降低历史 token 累积 |
| 精简 CLAUDE.md（控制在 1K tokens） | 减少每次请求的固定成本 |
| CI 中设置 `max_tokens` 上限 | 防止单次超支 |

### 5.3 verbose 调试

开启 verbose 模式，能看到每次请求的完整 token 消耗和工具调用过程，是定位性能瓶颈最直接的工具：

```bash
# 单次开启
claude --verbose

# 持久开启（写入配置）
# .claude/settings.json
{
  "verbose": true
}
```

**关键日志信息解读：**

```
[DEBUG] CLAUDE.md tokens: 1,234          → CLAUDE.md 占用了多少 token
[DEBUG] Context size: 8,432 tokens       → 当前上下文总大小
[DEBUG] Available context: 191,568 tokens → 还剩多少可用空间
[DEBUG] Tool call: Glob → 12 files found  → 某次工具调用的结果
[DEBUG] Input tokens: 8,011              → 本次请求发送了多少 token
[DEBUG] Output tokens: 1,234            → 本次响应返回了多少 token
[DEBUG] Latency: 5,000ms                → 接口响应延迟
[DEBUG] Cost: $0.0234                    → 本次调用花了多少钱
```

**常见性能问题速查：**

| 症状 | 可能原因 | 解决方案 |
|------|---------|---------|
| 响应慢（>10s） | 上下文过大 | `/clear` 清空对话，精简 CLAUDE.md |
| 频繁超 token 限制 | 单次任务过大 | 分解成多个步骤执行 |
| 成本快速上涨 | 无效迭代多 | 优化提示词，减少来回修改次数 |
| 工具调用失败 | MCP 服务器连接问题 | 检查 `.mcp.json` 配置和网络连通性 |

---

## 6. 故障排查

### 6.1 团队协作问题

| 现象 | 原因 | 解决方法 |
|------|------|---------|
| 不同成员 AI 产出风格差异大 | CLAUDE.md 未统一 | 确认 `CLAUDE.md` 和 `settings.json` 已入库，团队成员拉取最新版本 |
| CI 自动审查未触发 | Secrets 未配置或 Action 版本太旧 | 检查 GitHub Secrets 中的 `ANTHROPIC_API_KEY`，升级 Action 到最新版 |
| `settings.local.json` 被意外入库 | `.gitignore` 缺失对应规则 | 补充 `.gitignore`，并从仓库历史中清除 |
| 团队成员的 `/code-review` 命令找不到 | commands/ 目录未入库 | 确认 `.gitignore` 没有错误排除 `.claude/commands/` |

### 6.2 安全权限问题

| 现象 | 原因 | 解决方法 |
|------|------|---------|
| 危险命令没有被拦截 | deny 列表未配置 | 在 `settings.json` 的 `permissions.deny` 中补充高危命令 |
| allowedTools 设置不生效 | JSON 格式错误（多余逗号等） | 用 JSON 在线验证工具检查格式 |
| 审计日志目录不存在报错 | 目录未预先创建 | `mkdir -p ./logs/claude-audit/` |
| CI 中安全扫描漏掉了高危问题 | prompt 描述不够具体 | 在 prompt 中明确列出要检查的安全类别 |

### 6.3 性能与成本问题

| 现象 | 原因 | 解决方法 |
|------|------|---------|
| 月账单远超预期 | 没有成本上限配置 | 配置 `costControl.monthlyLimit` 并设置告警 |
| CLAUDE.md 加载慢，每次响应都慢 | 配置文件过大 | 精简到 <1,000 tokens，详情移到 `docs/ai-context/` |
| CI 中 Claude 调用偶发超时 | 没有设置 timeout 和 max_tokens | 在 Action 中配置 `timeout: 300` 和 `max_tokens: 4096` |
| verbose 日志看到 CLAUDE.md tokens 很高 | CLAUDE.md 内容过多 | 审查 CLAUDE.md，删除冗余内容，提取到 ai-context 目录 |

---

## 7. 总结

本章你已掌握：

1. **团队规范**：标准化目录结构、CLAUDE.md 三层配置体系、配置入库策略、新成员入职 SOP
2. **CI/CD 集成**：GitHub Actions 自动审查（`anthropics/claude-code-action`）、多场景工作流（代码审查 + 安全扫描 + 交互指令）、完整 6 阶段流水线
3. **安全合规**：allow/deny 权限模型、allowedTools 白名单精细控制、审计日志配置与合规检查清单
4. **性能优化**：上下文管理三策略、verbose 模式 token 分析、常见性能瓶颈定位
5. **成本控制**：按场景选模型、批量处理、成本预警配置、五条省钱建议

**一句话总结**：个人用 Claude Code 靠感觉，团队用 Claude Code 靠规范。把本章的配置落地到你的项目里，10 人团队也能用得既高效又安全。

---

## 8. 参考资料

- [Claude Code 官方文档](https://code.claude.com/docs/zh-CN)
- [GitHub：anthropics/claude-code-action 官方 Action](https://github.com/anthropics/claude-code-action)
- [知乎：Claude Code 企业级最佳实践完整指南](https://zhuanlan.zhihu.com/p/1963592231739957552)
- [ClaudeCN 中文文档：安全与权限配置](https://claudecn.com/docs/claude-code/security/)

---

## 下一步学习

| 章节 | 主题 | 你将学到 |
|------|------|---------|
| 第08章 | 企业深水区——密钥安全、团队配置与合规审计全攻略 | 团队配置统一化、Git Hooks 自动审查、API Key 分级管理、GDPR/SOC2 合规、安全扫描与事故响应 |
