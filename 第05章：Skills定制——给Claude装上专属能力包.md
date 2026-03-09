# 第五章：Skills定制——给Claude装上专属能力包

## 目录

- [1. 前言](#1-前言)
- [2. Skills 核心概念](#2-skills-核心概念)
  - [2.1 Skills 是什么](#21-skills-是什么)
  - [2.2 Skills vs Commands](#22-skills-vs-commands)
  - [2.3 渐进式披露原理](#23-渐进式披露原理)
- [3. 准备工作](#3-准备工作)
  - [3.1 账号要求](#31-账号要求)
  - [3.2 环境要求](#32-环境要求)
- [4. 官方 Skills 速查](#4-官方-skills-速查)
- [5. 安装 Skills](#5-安装-skills)
  - [5.1 官方/第三方 Skills 安装](#51-官方第三方-skills-安装)
  - [5.2 手动安装skills 实战](#52-手动安装skills-实战)
  - [5.3 自定义 Skill 开发（本地安装）](#53-自定义-skill-开发本地安装)
- [6. 创建第一个自定义 Skill](#6-创建第一个自定义-skill)
  - [6.1 目录结构](#61-目录结构)
  - [6.2 SKILL.md 配置详解](#62-skillmd-配置详解)
  - [6.3 实战：5分钟创建代码注释 Skill](#63-实战5分钟创建代码注释-skill)
  - [6.4 测试 Skill](#64-测试-skill)
- [7. Skills 使用方法](#7-skills-使用方法)
- [8. 进阶使用](#8-进阶使用)
  - [8.1 提示词组织技巧](#81-提示词组织技巧)
  - [8.2 Python 脚本集成](#82-python-脚本集成)
  - [8.3 多步骤工作流](#83-多步骤工作流)
- [9. 故障排查](#9-故障排查)
- [10. 总结](#10-总结)
- [11. 参考资料](#11-参考资料)
- [下一步学习](#下一步学习)

---

## 1. 前言

学完 Commands，你已经能把重复提示词压缩成一个命令。

但有个更深的问题——**Commands 是一次性触发，没有记忆，不能积累知识**。每次换个项目、换个场景，还得从零教 Claude。

**Skills 就是突破这个边界的答案：**

| 对比 | Commands | Skills |
|------|---------|--------|
| 定位 | 触发器（按钮） | 能力包（APP） |
| 知识容量 | 几百到几千字 | 可达数万字 |
| 触发方式 | 必须显式输入 `/命令` | 自动识别 + 显式调用 |
| 脚本集成 | 有限 | 支持 Python/JS 脚本 |
| 可维护性 | 简单直接 | 模块化分层 |

> 类比：Commands 是快捷键，Skills 是手机里的 APP——装上之后，Claude 瞬间变成该领域专家。

---

## 2. Skills 核心概念

### 2.1 Skills 是什么

Skills 是 Claude Code 的**"能力 APP"**——把特定领域的知识、规则、工具打包成可复用的模块。

**没有 Skills 的困境**：

```
你：帮我写一篇公众号文章
Claude：好的，请问什么风格？字数多少？有什么特殊要求？

...下次对话...

你：再帮我写一篇
Claude：好的，请问什么风格？（又从零开始）
```

**有了 Skills 之后**：

```
你：帮我写一篇公众号文章
Claude：[自动加载公众号写作 Skill，读取风格规范、爆款公式]
       好的！基于你的写作规范，我来帮你...
```

Skills 的核心价值：

| 场景 | 效果 |
|------|------|
| 领域专业化 | 预置大量领域知识，即用即专业 |
| 团队协作 | 一次配置，全员共享标准 |
| 知识积累 | 集中管理，版本可控 |
| 质量一致 | 标准化流程，输出稳定 |

### 2.2 Skills vs Commands

**一句话区分**：Commands 是"入口"，Skills 是"能力"，两者通常配合使用。

| 对比维度 | Commands（斜杠命令） | Skills（能力包） |
|---------|-------------------|----------------|
| 文件结构 | 单个 `.md` 文件 | 多文件目录 |
| 触发方式 | 必须输入 `/命令名` | 关键词自动识别 |
| 状态管理 | 无状态 | 可维护配置和状态 |
| 工具集成 | 写在 `.md` 里 | 可调用外部脚本 |
| 适用场景 | 单一任务 | 复杂工作流 |

**协作关系**：

```
用户输入
   │
   ├──→ Commands（触发层）：/write 公众号文章
   │          │
   │          ▼
   └──→ Skills（能力层）：自动加载写作规范、调用脚本
```

最佳实践：
- 简单任务 → 直接用 Command
- 复杂工作流 → Command 作入口 + Skill 提供能力

### 2.3 渐进式披露原理

Skills 采用**按需加载**设计，核心思想：只在用户需要时才展示复杂功能，避免内存浪费。

| 层级 | 内容 | 何时加载 | 内存占用 |
|------|------|---------|---------|
| 第一层：元数据 | YAML Frontmatter 的 `name` 和 `description` | 始终常驻 | 极小（<100字节） |
| 第二层：指令 | SKILL.md 的 Markdown Body | Skill 激活时 | 按需加载 |
| 第三层：资源 | `scripts/`、`templates/`、`config/` | 调用时才读取 | 用完即释放 |

> 这就是为什么有几十个 Skills 也不会拖慢 Claude——元数据极小，详细内容按需加载。

---

## 3. 准备工作

### 3.1 账号要求

| 账户类型 | Skills 支持 | 说明 |
|---------|-----------|------|
| Claude Pro | ✅ 支持 | 推荐，功能最完整 |
| Claude Teams | ✅ 支持 | 企业用户推荐 |
| Claude Enterprise | ✅ 支持 | 企业级，功能最强 |
| 免费版 | ❌ 不支持 | 暂无此功能 |
| 中转站 (API) | ✅ 支持 | 国内用户推荐 |

> **中转站用户**：通过 API 密钥使用 Claude Code 的中转站用户也支持 Skills。推荐使用 AnyRouter、gemai 等稳定的中转站。

### 3.2 环境要求

**必需工具**：

- **Claude Desktop**（网页版或桌面版均可）
  - 下载地址：https://www.anthropic.com/claude-desktop
  - 最低版本：2.0+（推荐最新版）

- **Claude Code CLI**（用于本地开发）
  ```bash
  # 验证安装
  claude --version
  
  # 如未安装，按第一章步骤重新安装
  ```

- **Python 3.8+**（如需使用脚本）
  ```bash
  python --version
  ```

---

## 4. 官方 Skills 速查

Anthropic 官方提供的核心技能包：

> **官方资源**：
> - [Anthropic 官方 Claude Plugins 仓库](https://github.com/anthropics/claude-plugins-official)
> - [SkillsMP - Skills 市场](https://skillsmp.com/zh)
> - [ComposioHQ - Awesome Claude Skills 合集](https://github.com/ComposioHQ/awesome-claude-skills)
> - [sickn33 - Antigravity Awesome Skills](https://github.com/sickn33/antigravity-awesome-skills)

| 技能包                  | 功能         | 核心能力                                    | 适用场景                         |
| ----------------------- | ------------ | ------------------------------------------- | -------------------------------- |
| **document-skills**     | 文档处理     | 解析 Excel/Word/PPT/PDF，提取数据，生成报告 | 数据分析、报告自动生成、内容提取 |
| **example-skills**      | 技能开发示例 | Skill 创建模板、MCP 构建示例                | 学习如何开发自定义 Skill         |
| **planning-with-files** | 文件规划     | 项目文档整理、任务分解、甘特图生成          | 项目管理、任务规划               |
| **frontend-design**     | 前端设计     | UI/UX 设计指导、代码生成、组件库推荐        | 前端开发、设计稿转代码           |

**推荐使用顺序**：

1. 先装 `example-skills` 学习 Skill 开发
2. 根据需求装 `document-skills`（文档处理最常用）
3. 按场景选装其他技能包

---

## 5. 安装 Skills

### 5.1 官方/第三方 Skills 安装

**方式一：网页版/桌面版安装（最简单）**

```
1. 打开 Claude 官网或桌面版
2. 进入"设置" → "功能" → "Skills"
3. 找到要启用的官方技能，点击开关启用
4. 或点击"上传技能"，选择 .skill 文件上传第三方技能
```

**方式二：Claude Code CLI 安装（推荐开发者）**

Step 1：添加官方技能市场 或者通过skills查找网站进行搜索自己需要的 自行安装

[Agent Skills 市场 - Claude、Codex 和 ChatGPT Skills | SkillsMP](https://skillsmp.com/zh)

```bash
/plugin marketplace add anthropics/skills
```

![image-20260303172407830](D:\Desktop\cc\assets\image-20260303172407830.png)

或者通过原空间安装

![image-20260303153725885](D:\Desktop\cc\assets\image-20260303153725885.png)

Step 2：安装官方技能包（可选多个）

```bash
# 文档处理技能（Excel、Word、PPT、PDF 等）
/plugin install document-skills@anthropic-agent-skills

# 示例技能包（学习自定义 Skill 开发的好素材）
/plugin install example-skills@anthropic-agent-skills
```

![image-20260303172114170](D:\Desktop\cc\assets\image-20260303172114170.png)

| 选项                                                         | 通俗解释                                                     | 适用场景                                                 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------------------- |
| Install for you (user scope)                                 | **用户级安装**：插件仅对你当前的账号生效，不管你打开哪个仓库 / 项目，都能使用这个插件 | 个人使用、想在所有项目中用这个示例插件学习               |
| Install for all collaborators on this repository (project scope) | **项目级安装**：插件仅对当前这个代码仓库生效，且仓库的所有协作者都能看到 / 使用这个插件 | 团队协作、仅需要在特定项目中测试示例技能                 |
| Install for you, in this repo only (local scope)             | **本地仓库级安装**：插件仅对你自己生效，且仅在当前这个仓库中可用 | 只想在某个项目中测试，不想影响其他项目，也不想让团队看到 |
| Back to plugin list                                          | 放弃安装，返回插件列表                                       | 暂时不想装这个插件                                       |

或者在安装了元空间的 仓库下 /plugin Discover  选择自己需要安装的插件skills

![image-20260303163731631](D:\Desktop\cc\assets\image-20260303163731631.png)

Step 3：重启 Claude Code 完成激活

```bash
# 退出并重新启动
claude
```

**方式三：手动安装（进阶用户）**

将 Skill 文件夹放置在以下任一位置，Claude 会自动识别：

官方安装的会在plugins/marketplaces/官方/skills 下

```
# 个人级（所有项目可用）
~/.claude/skills/your-skill-name/

# 项目级（仅当前项目可用）
/path/to/project/.claude/skills/your-skill-name/
```

### 5.2 手动安装skills 实战

```
# 创建插件目录（建议放在用户目录下，方便查找）
mkdir -p ~/claude-plugins/connect-apps-plugin
# 进入该目录
cd ~/claude-plugins/connect-apps-plugin
# 克隆插件仓库（如果没有git，先安装：brew install git / apt install git）
git clone https://github.com/ComposioHQ/awesome-claude-skills.git .
# cmd 命令拷贝到 .claude\skills\ 目录下
xcopy "%USERPROFILE%\claude-plugins\connect-apps-plugin" "%USERPROFILE%\.claude\skills\" /E /H /Y
# 检查是否安装
claude
/skills
```

![image-20260304101146734](D:\Desktop\cc\assets\image-20260304101146734.png)

![image-20260304101121234](D:\Desktop\cc\assets\image-20260304101121234.png)

![image-20260304101211232](D:\Desktop\cc\assets\image-20260304101211232.png)

**powershell**

```
# 1. 确保目标目录存在（不存在则创建）
New-Item -Path "~\.claude\skills" -ItemType Directory -Force

# 2. 复制源目录下的所有文件/子文件夹到目标目录（关键：末尾加 \*）
Copy-Item -Path "~\claude-plugins\connect-apps-plugin\*" -Destination "~\.claude\skills" -Recurse -Force
```

![image-20260304101729390](D:\Desktop\cc\assets\image-20260304101729390.png)

### 5.3 自定义 Skill 开发（本地安装）

一旦你创建了自定义 Skill，可以通过以下方式安装：

**放入标准目录**

```bash
# macOS/Linux
mkdir -p ~/.claude/skills/my-skill
cp -r ./my-skill/* ~/.claude/skills/my-skill/

# Windows PowerShell
New-Item -ItemType Directory -Path "$env:USERPROFILE\.claude\skills\my-skill" -Force
Copy-Item -Recurse ".\my-skill\*" "$env:USERPROFILE\.claude\skills\my-skill\" -Force
```

---

---

## 6. 创建第一个自定义 Skill

### 6.1 目录结构

每个 Skill 是 `.claude/skills/` 下的独立目录：

```
.claude/skills/[skill-name]/
├── SKILL.md          # [必需] 核心定义文件（YAML元数据 + Markdown指令）
├── scripts/          # [可选] 工具脚本（Python/JS）
├── templates/        # [可选] 输出模板
├── config/           # [可选] 配置文件
└── data/             # [可选] 静态数据
```

| 文件/目录 | 是否必需 | 用途 |
|---------|---------|------|
| `SKILL.md` | ✅ 必需 | Skill 的身份证 + 说明书 |
| `scripts/` | 可选 | 可执行的自动化脚本 |
| `templates/` | 可选 | 输出格式模板 |
| `config/` | 可选 | 运行时配置参数 |

> **命名规范**：目录名只能用小写字母、数字、连字符（`-`），不能有空格或下划线。

### 6.2 SKILL.md 配置详解

`SKILL.md` 由两部分组成：**YAML Frontmatter**（机器读）+ **Markdown Body**（AI 读）。

**YAML Frontmatter 字段表**（必填字段只有 2 个）：

| 字段 | 类型 | 是否必填 | 说明 |
|------|------|---------|------|
| `name` | string | ✅ | Skill 名称，必须与目录名一致 |
| `description` | string | ✅ | 触发场景描述，Claude 用此判断是否自动激活 |
| `version` | string | 可选 | 版本号（如 1.0.0） |
| `author` | string | 可选 | 作者名称 |

**最小配置示例**：

```markdown
---
name: code-commenter
description: 当用户要求"添加注释"、"代码注释"或"注释代码"时激活
---

# 代码注释生成器

## 角色定义
你是一位代码审查专家，擅长编写清晰的中文注释。

## 注释原则
- 解释"为什么"而不是"是什么"
- 使用简洁的中文
- 专业术语保持英文（如 API、JWT）
```

**description 写法对比**：

| 写法 | 效果 |
|------|------|
| ✅ `当用户要求"添加注释"或"代码注释"时激活` | 明确触发词，自动识别准确 |
| ❌ `代码注释工具` | 太模糊，自动激活不准 |

**Markdown Body 推荐结构**：

```markdown
# Skill 名称

## 一、角色定义
[AI 扮演的角色和专业背景]

## 二、核心能力
[列出 3-5 个核心能力]

## 三、工作流程
### 步骤1：[名称]
[详细说明]

## 四、规则约束
[必须遵守的规则]

## 五、输出格式
[输出结构定义]
```

### 6.3 实战：5分钟创建代码注释 Skill

**Step 1：建目录**

```bash
# macOS / Linux
mkdir -p .claude/skills/code-commenter

# Windows PowerShell
New-Item -ItemType Directory -Path ".claude\skills\code-commenter" -Force
```

**Step 2：创建 SKILL.md**

创建文件 `.claude/skills/code-commenter/SKILL.md`：

```markdown
---
name: code-commenter
description: 当用户要求"添加注释"、"给代码加注释"或"注释代码"时，自动为代码添加清晰的中文注释
---

# 代码注释生成器

## 角色定义
你是一位经验丰富的代码审查专家，擅长编写清晰、准确、有价值的代码注释。

## 何时激活
当用户说以下内容时激活本 Skill：
- "帮我添加注释"
- "给这段代码加注释"
- "comment this code"

## 注释原则

### 1. 解释"为什么"而不是"是什么"
```python
# ❌ 差：循环遍历列表
for item in items:
    process(item)

# ✅ 好：过滤已过期订单，避免重复发货
for item in items:
    process(item)
```

![image-20260304111955336](D:\Desktop\cc\assets\image-20260304111955336.png)

![image-20260304112023280](D:\Desktop\cc\assets\image-20260304112023280.png)

### 2. 注释格式规范

- **函数/方法**：说明功能、参数、返回值
- **复杂逻辑**：解释业务背景
- **魔法数字**：说明数值含义（如 86400 = 24小时）

### 3. 语言要求
- 使用简洁中文
- 专业术语保持英文（如 API、JWT、JSON）

### 6.4 测试 Skill

**Step 1：启动 Claude Code**

```bash
claude
```

**Step 2：测试触发**

在对话中输入：

```
帮我给这段代码添加注释

def calculate_discount(price, user_level):
    if user_level == "vip":
        return price * 0.8
    elif user_level == "svip":
        return price * 0.7
    else:
        return price
```

**Step 3：验证预期响应**

Claude 应该自动激活 `code-commenter` Skill，并返回带详细中文注释的代码：

```python
def calculate_discount(price, user_level):
    """
    根据用户等级计算折扣后的价格

    Args:
        price: 原始价格
        user_level: 用户等级（vip/svip/普通）

    Returns:
        折扣后的价格
    """
    # VIP用户享受8折优惠
    if user_level == "vip":
        return price * 0.8
    # SVIP用户享受7折优惠
    elif user_level == "svip":
        return price * 0.7
    # 普通用户无折扣
    else:
        return price
```

![image-20260304113812089](D:\Desktop\cc\assets\image-20260304113812089.png)

**🎯 Hot Reloading 体验**

修改 `SKILL.md` 后，**无需重启 Claude Code**，下次对话时修改会自动生效！

尝试：
1. 编辑 `.claude/skills/code-commenter/SKILL.md`，修改注释原则
2. 回到 Claude Code，继续对话（不用重启）
3. 观察新的规则是否立即应用

---

## 7. Skills 使用方法

### 7.1 自动激活 vs 手动触发

**自动激活（推荐）**：

Claude 自动识别用户输入中的触发关键词，激活对应的 Skill：

```
用户：帮我给这段代码添加注释
Claude：[自动识别"添加注释" → 激活 code-commenter Skill]
```

关键是写好 `description` 字段，列举明确的触发词：

```yaml
---
name: code-commenter
description: 当用户要求"添加注释"、"给代码加注释"或"代码注释"时激活
---
```

**手动触发**：

用户显式在开始时声明使用某个 Skill：

```
我需要用 code-commenter Skill，帮我给这段代码添加注释
def calculate():
    ...
```

### 7.2 Skill 交互模式

| 模式 | 使用场景 | 示例 |
|------|---------|------|
| **单词触发** | 一句话激活，继续对话 | "帮我添加注释" → 输入代码 → 自动应用 |
| **链式调用** | 多个 Skill 协作 | 先用 title-generator → 再用 content-writer |
| **反复迭代** | 同一 Skill 多次使用 | 生成标题 → 用户反馈 → 再生成 |

### 7.3 与 Commands 的配合

**最佳实践**：Command 作为入口，Skill 提供能力

```
命令层：/write 《标题》
   ↓
Skill 层：自动加载 title-generator 和 content-writer
   ↓
输出：完整文章
```

在 Commands 的 `.md` 中调用 Skill：

```markdown
# /write 命令

帮我写一篇关于「{topic}」的公众号文章。

> 使用 Skill：title-generator + gongzhonghao-writer
```

---

## 8. 进阶使用

### 8.1 提示词组织技巧

本节目的：掌握在 SKILL.md 中组织提示词的最佳实践。

> **2.10+ 重大变化**：所有提示词内容统一写在 SKILL.md 的 Markdown Body 中，无需单独的 `prompts/` 文件夹。

#### 8.1.1 提示词组织方式

推荐**按功能分章节**组织，保持清晰的层次：

```markdown
# [Skill名称]

## 一、角色定义
[AI扮演的角色]

## 二、核心能力
[3-5个核心能力列表]

## 三、工作流程
### 步骤1：...
### 步骤2：...

## 四、规则约束
[必须遵守的规则]

## 五、示例展示
[好/坏示例对比]

## 六、输出格式
[输出结构定义]
```

#### 8.1.2 章节命名规范

| 命名模式 | 示例 | 说明 |
|---------|------|------|
| `## 一、xxx` | `## 一、角色定义` | 主要章节 |
| `## {功能}规范` | `## 标题生成规范` | 功能说明 |
| `### {子功能}` | `### 公式1：工具推荐型` | 子功能细节 |

**最佳实践**：保持 3 级以内的层次，使用清晰中文标题。

#### 8.1.3 提示词结构模板

```markdown
---
name: skill-name
description: 当用户[具体场景]时激活
version: 1.0.0
---

# Skill 名称

## 一、角色定义
你是一位[专业背景]的专家...

## 二、核心能力
1. [能力1]
2. [能力2]
3. [能力3]

## 三、工作流程
### 步骤1：[分析/准备]
### 步骤2：[执行]
### 步骤3：[验证]

## 四、规则约束
- 必须遵守的规则1
- 必须遵守的规则2

## 五、示例展示
✅ **好的示例**
❌ **差的示例**
```

#### 8.1.4 提示词版本管理

在 SKILL.md 开头记录版本历史：

```markdown
---
name: my-skill
version: 2.0.0
---

## 版本历史
### V2.0.0 (2025-01)
- 新增：XXX
- 优化：YYY

### V1.0.0 (2024-12)
- 初版发布
```

#### 8.1.5 提示词优化技巧

**1. 用代码块强调公式**
```markdown
### 标题公式
[品牌词] + [数字] + [推荐词]
```

**2. 用表格对比版本**
| 版本 | 改进点 |
|------|--------|
| V1.0 | 基础功能 |
| V2.0 | 新增XXX |

**3. 用强指令替代建议**
- ❌ "建议使用中文"
- ✅ "必须使用简洁中文"

### 8.2 Python 脚本集成

**什么时候需要脚本？**

| 任务类型 | Claude 原生 | 脚本增强 |
|---------|------------|---------|
| 文本分析 | 模糊判断 | 精确 NLP 分析 |
| 数字计算 | 可能出错 | 100% 准确 |
| 文件批处理 | 效率低 | 高效批量处理 |
| 复杂校验 | 难以一致 | 确定性校验 |

**标准脚本模板**（放入 `scripts/` 目录）：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""脚本功能简述"""
import sys
import json

def process(input_data: str) -> dict:
    """核心处理逻辑"""
    # 在这里实现具体功能
    return {
        "success": True,
        "data": {"result": input_data},
        "message": "处理成功"
    }

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print(json.dumps({"success": False, "message": "缺少参数"}))
        sys.exit(1)

    result = process(sys.argv[1])
    print(json.dumps(result, ensure_ascii=False, indent=2))
    sys.exit(0 if result["success"] else 1)
```

**在 SKILL.md 中调用脚本**：

```markdown
## 工具调用
- 质量检测：`python scripts/quality_detector.py "文章内容" --json`
- 标题生成：`python scripts/title_generator.py "主题关键词"`
```

**参数传递方式速查**：

| 方式 | 适用场景 | 示例 |
|------|---------|------|
| 命令行参数 | 简单参数 | `python script.py "topic"` |
| 标准输入 | 大量文本 | `echo "content" \| python script.py` |
| 文件传递 | 复杂数据 | `python script.py --input file.json` |

### 8.3 多步骤工作流

在 SKILL.md 的 Markdown Body 中用自然语言描述完整工作流：

```markdown
## 工作流程

### 步骤1：选题过滤
判断三个维度：时效性、爆款潜力、是否值得写
- 不值得写 → 直接给出建议，结束流程
- 值得写 → 继续下一步

### 步骤2：信息收集（按需）
如果主题需要最新数据：
- 使用 WebSearch 搜索"主题 最新资讯 2025"
- 整理关键数据和观点

### 步骤3：创作文章
应用写作风格规范，控制字数 1500-2000 字

### 步骤4：质量检测
调用 `scripts/quality_detector.py` 检测 AI 腔、自然度
- 检测通过 → 保存文章
- 检测失败 → 按建议修改后重新检测
```

> Commands 可作为工作流入口，Skill 提供底层能力：`/write 主题` → 触发 Command → Command 调用公众号写作 Skill。

---

## 9. 故障排查

| 现象 | 原因 | 解决方法 |
|------|------|---------|
| Skill 未自动激活 | `description` 描述不清晰 | 在 description 中明确列出触发关键词，如"当用户提到"或"当用户要求" |
| AI 回复不符合规范 | Markdown Body 指令太模糊 | 用 `必须`、`禁止` 等强指令替代 `建议`，提高执行一致性 |
| 官方 Skill 安装失败 | 网络问题或账户权限不足 | 检查网络连接、确认是付费账户，尝试重启 Claude Code |
| 脚本执行失败 | Python 路径或权限问题 | `python --version` 验证，macOS/Linux 加 `chmod +x` 执行权限 |
| YAML 解析报错 | 冒号后无空格或 `---` 缺失 | 检查格式：`name: skill-name`，注意冒号后有空格 |
| 文件路径问题 | 目录名与 `name` 字段不一致 | 确认目录名和 YAML 中的 `name` 字段完全一致 |
| 中转站用户 Skills 不可用 | 中转站不支持 Skills 功能 | 更换支持 Skills 的中转站（如 AnyRouter、gemai），或使用官方账户 |

**YAML 格式快速验证**：

```bash
# 用 Python 验证 SKILL.md 的 YAML 部分
python3 -c "
import yaml
with open('.claude/skills/你的skill/SKILL.md', 'r', encoding='utf-8') as f:
    content = f.read()
# 提取 --- 包裹的 YAML 部分
parts = content.split('---')
yaml.safe_load(parts[1])
print('✅ YAML 格式正确')
"
```

---

## 10. 总结

本章你已掌握：

1. **Skills 本质**：能力 APP，封装领域知识和工具，即装即用
2. **安装方式**：官方 Skill 3 种安装方法（网页/CLI/手动），自定义 Skill 本地开发
3. **官方资源**：document-skills 用于文档处理，example-skills 用于学习开发
4. **SKILL.md 结构**：YAML Frontmatter（`name` + `description`）+ Markdown Body（详细指令）
5. **自动激活原理**：Claude 根据 `description` 自动识别触发场景，自动加载 Skill
6. **脚本集成**：通过 `scripts/` 目录接入 Python 脚本，实现精确计算和自动化处理

---

## 11. 参考资料

**官方文档与教程**：
- [Claude Code 官方 Skills 文档](https://code.claude.com/docs/zh-CN/skills)
- [知乎深度讲解：Claude Code Skills 定制完整指南](https://zhuanlan.zhihu.com/p/2000662940370626160)
- [Claude 中文文档：Skills 开发与使用](https://claudecn.com/docs/claude-code/advanced/skills/)

---

## 下一步学习

| 章节 | 主题 | 你将学到 |
|------|------|---------|
| 第06章 | Plugins全攻略——一键安装海量扩展，还能自己造轮子 | 插件安装、配置与自定义开发 |
| 第07章 | 从单兵到团队——企业级协作规范、CI/CD与安全合规实战 | 团队规范、CI/CD 配置、权限管理 |
| 第08章 | 企业深水区——密钥安全、团队配置与合规审计全攻略 | 密钥管理、合规审计、安全最佳实践 |

