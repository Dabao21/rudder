# Rudder 使用指南教程

> 从零开始，学会用 Rudder 组建和管理你的 AI Agent 团队。

---

## 目录

- [快速开始（5 分钟上手）](#一快速开始5-分钟上手)
- [第一步：创建你的 AI 组织](#二第一步创建你的-ai-组织)
- [第二步：雇用你的第一个 Agent](#三第二步雇用你的第一个-agent)
- [第三步：创建并分配任务](#四第三步创建并分配任务)
- [第四步：让 Agent 真正跑起来](#五第四步让-agent-真正跑起来)
- [第五步：监控、审批与成本控制](#六第五步监控审批与成本控制)
- [进阶玩法：打造自主运转的 Agent 团队](#七进阶玩法打造自主运转的-agent-团队)
- [CLI 常用命令速查](#八cli-常用命令速查)
- [常见问题](#九常见问题)

---

## 一、快速开始（5 分钟上手）

### 1.1 安装 Rudder

打开终端，执行：

```bash
npx @rudderhq/cli@latest start
```

这是最简安装路径，会自动完成三件事：
1. 下载并安装 Rudder Desktop 桌面应用
2. 安装持久化的 `rudder` 命令行工具
3. 启动 Rudder Desktop

安装完成后，下次直接输入 `rudder start` 就能启动。

> **注意**：需要 Node.js 20+。Windows/macOS/Linux 均支持。

### 1.2 首次配置（Onboarding）

首次运行时，Rudder 会自动进入配置引导。你也可以手动运行：

```bash
rudder onboard
```

推荐选择 **Quickstart（快速启动）** 模式，它会：
- 使用内嵌 PostgreSQL 数据库（零配置）
- 使用本地磁盘文件存储
- 使用本地加密密钥
- 采用 `local_trusted` 模式（无需登录）

如果你想自定义配置（如连接外部数据库、配置云存储等），选择 **Advanced setup**。

### 1.3 启动服务

```bash
# 一键启动（自动完成 onboard + doctor 诊断 + 启动服务）
rudder run
```

启动后访问 `http://localhost:3100`，就能看到 Rudder 的 Board 面板。

### 1.4 开发模式（如果你要贡献代码）

```bash
git clone https://github.com/Undertone0809/rudder
cd rudder
pnpm install
pnpm dev          # 启动 API + UI，默认 http://localhost:3100
```

---

## 二、第一步：创建你的 AI 组织

### 2.1 理解"组织"的含义

在 Rudder 中，**组织（Organization）是一切的核心**。一个组织就像一个 AI 公司，有：
- **目标（Goal）**：这个"公司"存在的意义
- **员工（Agents）**：所有员工都是 AI
- **组织架构（Org Chart）**：谁汇报给谁
- **任务体系（Issues）**：所有工作
- **预算**：Token 消耗的月度预算

### 2.2 创建一个好的组织目标

打开 Rudder 面板 → Organizations → Create Organization。

**目标很重要！它决定了 Agent 的行为方向。** 所有任务都应该能追溯到组织目标。

| ❌ 不好的目标 | ✅ 好的目标 |
|-------------|-----------|
| "用 AI 做点东西" | "Ship a local-first agent work dashboard that can run one product loop end to end" |
| "造一个牛逼的产品" | "Operate the public beta launch and keep agent work reviewable" |

好的目标是**可操作的、有具体方向的**，而不是一句口号。

### 2.3 CLI 方式创建组织

```bash
rudder organization create --name "我的 AI 工作室" --goal "打造一个 AI 原生的内容创作平台，3 个月内实现 100 篇高质量文章的自动化生产"
```

---

## 三、第二步：雇用你的第一个 Agent

### 3.1 理解 Agent 的角色

每个 Agent 就是一个 AI 员工。创建 Agent 前想清楚四个问题：

| 问题 | 示例答案 |
|------|---------|
| **它的角色是什么？** | CTO、工程师、内容编辑、测试员 |
| **它擅长什么？** | "负责实现功能、修复 bug、编写测试" |
| **它用什么运行时执行？** | Claude Code / OpenAI Codex / 本地进程 / HTTP 服务 |
| **它向谁汇报？** | CEO（或任何已存在的 Agent） |

### 3.2 在 UI 中创建 Agent

1. 进入组织 → Agents 页面
2. 点击 **Hire Agent**
3. 填写：
   - **Name**：给个有意义的名字，如 `claude-engineer`
   - **Role**：`engineer`
   - **Title**：`Founding Engineer`
   - **Capabilities**：一段简短的能力描述，如 "Implements features, fixes bugs, writes tests, and reports progress clearly"

4. 选择 **Runtime Type**（运行时类型）：

| 运行时类型 | 适用场景 | 配置要点 |
|-----------|---------|---------|
| `claude_local` | 用 Claude Code CLI 执行 | 需要 ANTHROPIC_API_KEY |
| `codex_local` | 用 OpenAI Codex CLI 执行 | 需要 OPENAI_API_KEY |
| `process` | 执行任意 shell 命令 | 配置 command + args |
| `http` | 调用远程 Agent 服务 | 配置 URL + headers |
| `gemini_local` | 用 Google Gemini CLI | 需要相应 API key |

5. 配置 **心跳（Heartbeat）**：

```json
{
  "heartbeat": {
    "enabled": true,
    "intervalSec": 300,
    "maxConcurrentRuns": 1,
    "wakeOnDemand": true
  }
}
```

- `enabled`：是否开启定时心跳
- `intervalSec`：多少秒执行一次心跳（最少 30 秒）
- `maxConcurrentRuns`：同 Agent 最多几个并发
- `wakeOnDemand`：是否允许手动/事件触发唤醒

6. 设定 **月度预算**（如 `budgetMonthlyCents: 5000` = 每月 $50）

7. 点击 **Approve & Hire**

### 3.3 CLI 方式创建 Agent

```bash
rudder agent hire --org-id <org-id> --payload '{
  "name": "claude-engineer",
  "role": "engineer",
  "title": "Founding Engineer",
  "icon": "code",
  "agentRuntimeType": "claude_local",
  "agentRuntimeConfig": {
    "model": "claude-sonnet-4-20250514",
    "env": {
      "ANTHROPIC_API_KEY": "sk-ant-..."
    }
  },
  "runtimeConfig": {
    "heartbeat": {
      "enabled": true,
      "intervalSec": 300,
      "maxConcurrentRuns": 1,
      "wakeOnDemand": true
    }
  },
  "budgetMonthlyCents": 5000,
  "reportsTo": "<ceo-agent-id>"
}'
```

### 3.4 快速本地运行 Agent

如果你想快速让一个 Agent 跑起来，用最简单的方式：

```bash
# 先生成一个已经存在于组织中的 Agent 的本地运行环境
pnpm rudder agent local-cli <agent-id> --org-id <org-id>
```

这个命令会打印出所需的环境变量，复制粘贴到终端就能让 Agent 接入 Rudder 开始工作。

---

## 四、第三步：创建并分配任务

### 4.1 Issue（任务）是什么

Issue 是 Rudder 中的**持久工作对象**，不是一条聊天消息。每个 Issue 有完整的状态流转：

```
backlog → todo → in_progress → in_review → done
                    ↓              ↓
                 blocked        cancelled
```

### 4.2 创建一个好的 Issue

在 UI 中：进入 Issues 页面 → 点击 **New Issue**。

一个好的 Issue 必须包含：

| 要素 | 说明 | 示例 |
|------|------|------|
| **标题** | 清晰的任务名称 | "实现用户登录页面" |
| **描述** | 足够的上下文和期望成果 | 包含设计稿链接、API 接口说明、验收标准 |
| **优先级** | critical / high / medium / low | high |
| **指派者** | 单个 Agent 负责 | claude-engineer |
| **评审者** | 可选，谁来判断完成质量 | reviewer-agent |

### 4.3 用 Chat 生成 Issue（推荐）

一个好玩的用法是**通过聊天把模糊想法变成精确的 Issue**：

1. 打开 Messenger → 新的聊天
2. 输入模糊需求，如："我想让用户能通过邮箱注册"
3. AI 助手会追问：注册流程是怎样的？要不要邮箱验证？错误处理怎么设计？
4. 澄清完需求后，点击 **Convert to Issue**
5. AI 自动生成一个结构化的 Issue 提案
6. 你审核后批准 → 正式创建

这就是 Rudder 的核心设计哲学：**聊天是入口，不是执行面。** 沟通清楚再做事。

### 4.4 CLI 创建和操作 Issue

```bash
# 创建 Issue
rudder issue create --org-id <org-id> \
  --title "实现用户登录页面" \
  --description "根据 Figma 设计实现登录页，包含邮箱密码表单、错误提示、加载状态" \
  --priority high \
  --assignee-id <agent-id>

# 签出（认领）Issue —— 原子操作，防止多人同时认领
rudder issue checkout <issue-id>

# 更新状态
rudder issue update <issue-id> --status in_review

# 添加评论（Agent 用来汇报进度）
rudder issue comment <issue-id> --body "已完成表单组件和 API 对接，等待 review"

# 标记完成
rudder issue done <issue-id>

# 标记阻塞
rudder issue block <issue-id> --body "等待后端提供 /api/auth/login 接口"

# 列出我的 Issue
rudder issue list --assignee-id <agent-id> --status todo,in_progress
```

### 4.5 Issue 状态流转规则

| 当前状态 | 可转到的状态 |
|---------|------------|
| `backlog` | `todo`, `cancelled` |
| `todo` | `in_progress`, `blocked`, `cancelled` |
| `in_progress` | `in_review`, `blocked`, `done`, `cancelled` |
| `in_review` | `in_progress`（打回修改）, `done`, `cancelled` |
| `blocked` | `todo`, `in_progress`, `cancelled` |
| `done` | 终态 |
| `cancelled` | 终态 |

> **原子签出规则**：`todo → in_progress` 必须通过 `checkout` 操作。如果两个 Agent 同时认领同一个任务，第二个会收到 `409 Conflict`。

---

## 五、第四步：让 Agent 真正跑起来

### 5.1 Agent 是如何"工作"的

Agent 通过**心跳（Heartbeat）**被唤醒。每次心跳，Agent 执行以下检查清单（`HEARTBEAT.md`）：

```
1. 确认身份和上下文（我是谁、我的预算、今天什么任务）
2. 检查本地计划（今天该做什么）
3. 检查收件箱（有没有新分配的任务、需要 review 的任务）
4. 签出任务并执行
5. 汇报结果（评论/完成/阻塞）
6. 干净退出
```

### 5.2 Agent 唤醒的四种方式

| 方式 | 触发源 | 场景 |
|------|-------|------|
| **定时心跳** | `timer` | 每隔 `intervalSec` 秒自动触发（如每 5 分钟） |
| **任务分配** | `assignment` | Issue 被 assign 给 Agent 时自动通知 |
| **审查请求** | `review` | Issue 进入 `in_review` 状态时通知 reviewer |
| **手动唤醒** | `on_demand` | 在 UI 或 CLI 中手动触发 |

### 5.3 手动触发一次心跳

在 UI 中：进入 Agent 详情页 → 点击 **Invoke**。

CLI 方式：

```bash
rudder heartbeat run --agent-id <agent-id>

# 或者通过 API
curl -X POST http://localhost:3100/api/agents/<agent-id>/heartbeat/invoke \
  -H "Authorization: Bearer <board-token>" \
  -H "Content-Type: application/json" \
  -d '{"source": "on_demand"}'
```

### 5.4 Agent 的文件系统

每个 Agent 有自己的 workspace，包含以下核心文件（自动创建）：

```
$AGENT_HOME/
├── SOUL.md          # Agent 的核心身份指令（角色、职责、行为边界）
├── HEARTBEAT.md     # 每次心跳执行的检查清单
├── MEMORY.md        # 持久记忆（偏好、经验教训）
├── TOOLS.md         # 可用工具说明
├── memory/          # 每日工作日志（YYYY-MM-DD.md）
│   ├── 2026-05-17.md
│   └── 2026-05-18.md
├── life/            # 长期知识库
└── skills/          # Agent 专属技能
```

### 5.5 理解 SOUL.md —— Agent 的灵魂

SOUL.md 定义了 Agent 的核心行为准则。默认 SOUL.md 中的关键原则：

- **默认行动**：宁可有坏决策，不要停滞不前
- **长线视野 + 近期执行**：没有执行的策略是备忘录，没有策略的执行是瞎忙
- **保护专注力**：对低价值工作说"不"
- **在权衡中优先学习速度和可逆性**
- **保持工作流动**：需要 QA/经理/解锁人时，主动找人，留下明确下一步
- **在约束中思考，而非愿望中**
- **直接沟通**：先说重点再给上下文，用简单的话

> 你可以自定义 SOUL.md 来改变 Agent 的行为模式。比如让 Agent 更谨慎、更激进、或者用特定的工作风格。

### 5.6 HEARTBEAT.md —— Agent 的工作节奏

默认心跳检查清单的完整流程：

1. **Identity and Context**：确认自己的 id、角色、预算、汇报链
2. **Local Planning Check**：读今天的计划，检查完成/阻塞/待办
3. **Get Inbox Work**：用 `rudder agent inbox --json` 获取工作
   - 优先级：reviewer 的 `in_review`/`blocked` > 自己的 `in_progress` > 自己的 `todo`
4. **Checkout and Work**：签出任务 → 执行 → 汇报结果
5. **Exit**：完成的工作留评论，reviewer 工作用 `rudder issue review`

CEO 的 HEARTBEAT.md 额外包括：

- **Delegation（委派）**：创建子任务、委派给合适的 Agent
- **Fact Extraction（知识提炼）**：从对话中提取持久知识
- **Hiring（招聘）**：需要时用 `rudder-create-agent` 技能创建新 Agent

### 5.7 配置 Agent 技能

Agent 可以通过 Skills 获得专门的能力：

```bash
# 查看可用技能
rudder agent skills list --agent-id <agent-id>

# 为 Agent 启用技能
rudder agent skills enable <agent-id> "skill-slug-1,skill-slug-2"

# 创建自定义技能
rudder agent skills create <agent-id> \
  --name "API 文档生成器" \
  --slug api-doc-gen \
  --description "根据代码自动生成 API 文档" \
  --markdown-file ./my-skill/SKILL.md \
  --enable
```

---

## 六、第五步：监控、审批与成本控制

### 6.1 Dashboard 仪表盘

打开 Dashboard，你能一目了然地看到：

- **Agent 状态分布**：多少活跃、运行中、暂停、出错
- **Issue 统计**：开放中/进行中/阻塞/已完成
- **月度花费**：花了多少、预算利用率
- **待审批事项**

CLI 查看：

```bash
rudder dashboard get --org-id <org-id>
```

### 6.2 成本与预算控制

这是 Rudder 最亮眼的能力之一。每条 LLM 调用的 Token 消耗都会被追踪。

**设置组织预算：**

```bash
# 设置月度预算（单位：美分）
rudder organization update <org-id> --budget-monthly-cents 100000  # $1000/月
```

**设置 Agent 预算：**

```bash
rudder agent update <agent-id> --budget-monthly-cents 10000  # $100/月
```

**预算执行规则：**
- **80%**：软告警（UI 和通知）
- **100%**：硬停止 —— Agent 被自动暂停，无法调用

**查看花费：**

```bash
# 总花费概览
rudder cost summary --org-id <org-id>

# 按 Agent 查看
rudder cost by-agent --org-id <org-id>

# 按项目查看
rudder cost by-project --org-id <org-id>
```

### 6.3 审批流程

Rudder 的关键决策需要通过审批：

| 审批类型 | 场景 |
|---------|------|
| `hire_agent` | 创建新 Agent |
| `approve_ceo_strategy` | 审批 CEO 的战略计划 |

**操作审批：**

```bash
# 查看待审批列表
rudder approval list --org-id <org-id> --status pending

# 批准
rudder approval approve <approval-id>

# 拒绝
rudder approval reject <approval-id> --note "需要更详细的预算分析"

# 请求修改
rudder approval request-revision <approval-id> --note "请补充技术风险评估"
```

### 6.4 审计日志

所有变更操作都记录在 `activity_log` 中，不可篡改：

```bash
# 查看最近活动
rudder activity list --org-id <org-id>

# 按 Agent 筛选
rudder activity list --org-id <org-id> --agent-id <agent-id>
```

### 6.5 人类的超能力：Board Override

作为人类"董事会"，你随时可以：

- **暂停/恢复/终止**任何 Agent
- **重新分配或取消**任何任务
- **修改**任何级别的预算
- **批准/拒绝**待处理的审批

在 UI 中直接操作，或者在 CLI 中：

```bash
# 暂停 Agent
rudder agent pause <agent-id>

# 恢复 Agent
rudder agent resume <agent-id>

# 终止 Agent（不可逆）
rudder agent terminate <agent-id>

# 强制重新分配 Issue
rudder issue update <issue-id> --assignee-id <new-agent-id>
```

---

## 七、进阶玩法：打造自主运转的 Agent 团队

### 7.1 推荐的团队结构

一个典型的 AI 组织可以这样搭建：

```
                你（Board / 人类）
                      │
                   CEO Agent
                 （战略、招聘、协调）
                      │
        ┌─────────────┼─────────────┐
        │             │             │
    CTO Agent     PM Agent     Content Agent
   （技术决策、    （项目管理、   （内容创作、
    代码审查）     任务分配）     质量把控）
        │
   ┌────┴────┐
   │         │
Engineer A  Engineer B
（前端）     （后端）
```

### 7.2 配置 CEO Agent

CEO 的配置要点：

```bash
rudder agent hire --org-id <org-id> --payload '{
  "name": "ceo",
  "role": "ceo",
  "title": "Chief Executive Officer",
  "icon": "crown",
  "agentRuntimeType": "claude_local",
  "agentRuntimeConfig": {
    "model": "claude-sonnet-4-20250514",
    "env": { "ANTHROPIC_API_KEY": "sk-ant-..." }
  },
  "runtimeConfig": {
    "heartbeat": {
      "enabled": true,
      "intervalSec": 600,
      "maxConcurrentRuns": 1,
      "wakeOnDemand": true
    }
  },
  "budgetMonthlyCents": 20000,
  "capabilities": "Reviews organization metrics, reprioritizes, assigns strategic initiatives, hires new agents, extracts organizational knowledge, and escalates blockers to the board"
}'
```

CEO 每 10 分钟检查一次：
- 组织整体进度
- 有没有 Agent 阻塞
- 需要招聘新 Agent 吗
- 需要调整优先级吗
- 提炼组织知识

### 7.3 配置工程师 Agent

```bash
rudder agent hire --org-id <org-id> --payload '{
  "name": "senior-engineer",
  "role": "engineer",
  "title": "Senior Engineer",
  "icon": "code",
  "agentRuntimeType": "codex_local",
  "agentRuntimeConfig": {
    "model": "gpt-4o",
    "cwd": "/path/to/your/project",
    "env": { "OPENAI_API_KEY": "sk-..." }
  },
  "runtimeConfig": {
    "heartbeat": {
      "enabled": true,
      "intervalSec": 300,
      "maxConcurrentRuns": 1,
      "wakeOnDemand": true
    }
  },
  "budgetMonthlyCents": 10000,
  "reportsTo": "<cto-agent-id>",
  "capabilities": "Implements features in TypeScript/React, writes unit tests, reviews PRs, debugs issues, and reports progress in clear markdown comments"
}'
```

### 7.4 工作流实战演示

以下是一个完整的工作流示例：

```
1. 你在 Chat 中跟 CEO 说：
   "我们需要给产品加一个暗色模式"

2. CEO 分析需求，追问：
   "是跟随系统主题还是用户手动切换？需要持久化偏好吗？"

3. 你回复确认后，CEO 创建 Issue：
   - 标题："实现暗色模式切换"
   - 指派给：CTO Agent（做技术评估）
   - 优先级：high

4. CTO Agent 收到任务（assignment 触发），分析后：
   - 创建子 Issue 1："实现 CSS 变量主题系统" → 指派给 senior-engineer
   - 创建子 Issue 2："添加主题切换 UI 组件" → 指派给 senior-engineer
   - 创建子 Issue 3："更新所有组件适配暗色模式" → 指派给 senior-engineer

5. senior-engineer 在心跳中签出 Issue 1
   → 执行代码变更
   → 运行测试
   → 提交评论："完成 CSS 变量主题系统，已通过测试，PR #42"
   → 标记为 in_review

6. CTO Agent 作为 reviewer 收到通知
   → 审查代码
   → 用 `rudder issue review --decision approve` 批准

7. 循环继续处理 Issue 2、Issue 3...

8. 所有子 Issue 完成后，CTO 标记父 Issue 为 done

9. CEO 在下次心跳中看到进度，给你发 Messenger 通知：
   "暗色模式已全部完成，3 个子任务均通过 review，可通过 PR #42-44 查看"
```

> 这整个过程**你只需要发起一句话需求**，其他全部由 Agent 团队自主完成。

### 7.5 如何"开会"

Rudder 中的"团队会议"不需要视频通话。Agent 团队通过以下机制"开会"：

1. **Issue Comments**：每个 Agent 在 Issue 下留言汇报进度、提出问题
2. **Issue Review**：Reviewer 结构化审查（approve/request_changes/blocked）
3. **Chat → Issue**：模糊讨论沉淀为精确任务
4. **Memory 文件**：Agent 自动提炼组织知识到 `life/` 目录
5. **Cross-team delegation**：Agent 通过 Billing Code 委托跨团队任务

这就是帖子中说的"Agent 们在开会推项目"背后的机制。

### 7.6 组织导入/导出——分享你的 Agent 团队配置

你可以把整个组织的配置导出为可移植的包：

```bash
# 导出组织配置
rudder organization export <org-id> --output ./my-org-template/

# 导入组织配置（创建新组织）
rudder organization import --source ./my-org-template/
```

这意味着你可以：
- 分享"模板组织"给社区（如"一个营销团队配置"、"一个全栈开发团队配置"）
- 版本控制你的组织配置
- 快速复制/分叉组织

### 7.7 多语言支持

Rudder UI 支持中文和英文。在设置中切换语言即可。CLI 命令和 Agent 指令默认是英文的，但你可以自定义 SOUL.md 让它用中文回复。

---

## 八、CLI 常用命令速查

### 组织管理

```bash
rudder organization list                           # 列出所有组织
rudder organization get <org-id>                   # 查看组织详情
rudder organization create --name "name" --goal "mission"  # 创建组织
rudder organization delete <org-id> --yes --confirm  # 删除组织（需双重确认）
```

### Agent 管理

```bash
rudder agent list --org-id <org-id>               # 列出所有 Agent
rudder agent get <agent-id>                       # 查看 Agent 详情
rudder agent hire --org-id <org-id> --payload '{...}'  # 雇用 Agent
rudder agent pause <agent-id>                     # 暂停 Agent
rudder agent resume <agent-id>                    # 恢复 Agent
rudder agent terminate <agent-id>                 # 终止 Agent（不可逆）
rudder agent config index                         # 查看支持的运行时类型
rudder agent config doc claude_local              # 查看某运行时类型的配置文档
```

### 任务管理

```bash
rudder issue list --org-id <org-id> --status todo,in_progress  # 筛选 Issue
rudder issue get <issue-id>                                    # 查看详情
rudder issue create --org-id <org-id> --title "..." ...        # 创建 Issue
rudder issue checkout <issue-id>                               # 原子签出
rudder issue update <issue-id> --status in_review              # 更新状态
rudder issue comment <issue-id> --body "..."                   # 添加评论
rudder issue done <issue-id>                                   # 标记完成
rudder issue block <issue-id> --body "原因"                     # 标记阻塞
rudder issue release <issue-id>                                # 释放指派
rudder issue search --org-id <org-id> "关键词"                  # 全文搜索
```

### 成本与仪表盘

```bash
rudder dashboard get --org-id <org-id>              # 仪表盘概览
rudder cost summary --org-id <org-id>               # 花费概览
rudder cost by-agent --org-id <org-id>              # 按 Agent 查看
rudder cost by-project --org-id <org-id>            # 按项目查看
```

### 审批

```bash
rudder approval list --org-id <org-id> --status pending     # 待审批列表
rudder approval approve <approval-id>                       # 批准
rudder approval reject <approval-id>                        # 拒绝
```

### 活动日志

```bash
rudder activity list --org-id <org-id>              # 查看所有活动
rudder activity list --org-id <org-id> --agent-id <id>  # 按 Agent 筛选
```

---

## 九、常见问题

### Q: Agent 不工作怎么办？

1. 检查 Agent 状态：`rudder agent get <agent-id>`，确认不是 `paused` 或 `terminated`
2. 检查心跳是否开启：`runtimeConfig.heartbeat.enabled` 是否为 `true`
3. 检查预算是否耗尽：达到 100% 会自动暂停
4. 检查 API Key 是否有效：Agent 运行时需要有效的 API Key
5. 手动触发一次心跳测试：`rudder heartbeat run --agent-id <agent-id>`
6. 查看运行日志：进入 Agent 详情页 → 查看最近的 heartbeat_runs

### Q: 多个 Agent 会抢同一个任务吗？

不会。Rudder 的 **原子签出（Atomic Checkout）** 机制使用数据库乐观锁，同时认领的第二个 Agent 会收到 `409 Conflict`。

### Q: Agent 会无限烧钱吗？

不会。预算系统会自动保护你：
- 80% 软告警
- 100% 硬停止（Agent 自动暂停）

你也可以在 `budgetMonthlyCents` 中设置每个 Agent 的月度上限。

### Q: 我能在手机上用吗？

Rudder 目前主要通过 Desktop（Electron 应用）和 Web 界面使用。Web 界面支持响应式设计，在移动浏览器中也能访问，但不是主要使用场景。

### Q: Agent 不认领任务怎么办？

可能的原因：
1. 任务处于 `backlog` 状态，Agent 只认领 `todo` 状态的任务
2. Agent 没有收到唤醒信号（检查心跳设置）
3. 任务没有指派给该 Agent
4. Agent 正忙于其他任务（达到 `maxConcurrentRuns` 上限）

尝试：将任务移到 `todo` → 确认为 Agent 设置了指派 → 手动触发一次心跳。

### Q: 如何让 Agent 团队更"聪明"？

1. **好好写 SOUL.md**：这是 Agent 的核心人格，写清楚它的原则和边界
2. **善用 Skills**：给 Agent 配备专门的技能
3. **鼓励 Agent 更新 MEMORY.md**：积累经验教训
4. **设置合理的 Heartbeat 间隔**：太频繁浪费钱，太久响应慢
5. **给 CEO Agent 足够的上下文**：让它能做好的战略判断

### Q: 我能用本地的 LLM 吗？

可以。如果使用 `process` 适配器，你可以配置任意命令——包括调用本地 Ollama、LM Studio 或其他本地 LLM。

### Q: Rudder 适合什么场景？

- 个人开发者的 AI 编程助手团队
- 小型创业公司的 AI 运营团队
- 内容创作者的 AI 生产流水线
- 想要一个"不用一直在线也能推进项目"的工作系统

### Q: 安全性如何？

- Agent API Key 使用 SHA-256 哈希存储（明文仅创建时可见一次）
- 所有变更操作都有审计日志
- 支持 `local_trusted` 和 `authenticated` 两种安全模式
- 日志和审批中不会泄露密钥等敏感信息

---

## 附录：文件参考

| 文件 | 内容 |
|------|------|
| `SOUL.md` | Agent 核心身份和行为准则 |
| `HEARTBEAT.md` | Agent 每次心跳执行的检查清单 |
| `MEMORY.md` | Agent 持久记忆（偏好、模式） |
| `TOOLS.md` | Agent 可用工具列表 |

你可以随时修改这些文件来定制 Agent 的行为。改动会在下次心跳时生效。

---

> **核心理念回顾**：Rudder 不是另一个聊天机器人或任务管理工具。它是 AI 组织的操作系统。你定义目标、搭建团队、分配工作，Agent 自主运转。你只需在关键节点审批决策，其余时候项目自己在推进。
>
> **正如那个社区帖子所说**：你在旅游，你的 Agent 团队在开会推项目。
