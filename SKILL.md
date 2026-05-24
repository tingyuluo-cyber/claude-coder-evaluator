---
name: coder-evaluator-loop
description: |
  当用户提交功能开发需求时，使用此 skill 实现真正的多 agent 协作工作流。通过 Master 协调器协调独立的 Coder 和 Evaluator agent，实现双层循环开发评估。适用于需要结构化开发流程、代码质量保证、迭代式开发的场景。当用户说"开发这个功能"、"帮我写代码"、"实现这个需求"、"创建项目"等涉及功能开发的任务时触发此 skill。即使用户没有明确要求评估循环，只要任务涉及多步骤功能开发，都应该使用此 skill 来确保质量。
---

# Coder + Evaluator 真正的多 Agent 协作工作流

这是一个基于真正独立 agent 协作的开发工作流，通过 Master 协调器协调 Coder 和 Evaluator 两个独立 agent，实现双层循环开发评估。

## 核心特性

**与角色切换模式的区别：**

| 特性 | 角色切换模式 | 真正的多 Agent 协作（本 skill） |
|------|-------------|-------------------------------|
| Agent 实例 | 1 个 | 3 个（Master + Coder + Evaluator） |
| 上下文隔离 | 无 | ✅ 完全隔离 |
| 并行执行 | 无法并行 | ✅ 可并行派发任务 |
| 确认偏差 | 存在 | ✅ 消除（独立审查） |
| 权限控制 | 无 | ✅ 严格权限边界 |

## 架构概述

```
用户需求
    ↓
[Master 协调器]
    ├── 派发任务 ──→ [Coder Agent] (独立实例)
    │                    ↓
    │               输出到文件
    │                    ↓
    ├── 读取输出 ←───────┘
    │
    ├── 派发任务 ──→ [Evaluator Agent] (独立实例)
    │                    ↓
    │               输出到文件
    │                    ↓
    ├── 读取评估 ←───────┘
    │
    └── 判断是否通过
           ↓
       [通过?]
           ↓ 是
       进入下一阶段
           ↓ 否
       派发修改任务给 Coder
```

## 三个独立 Agent 的职责

### 1. Master 协调器（当前会话 agent）

**职责：**
- 接收用户需求
- 创建工作目录
- 派发任务给 Coder 和 Evaluator agent
- 读取 agent 输出并判断是否通过
- 管理循环逻辑（最多 3 轮）
- 生成完成报告

**不参与：**
- 具体编码（Coder 的工作）
- 代码评估（Evaluator 的工作）

**指令文件：** `agents/master.md`

### 2. Coder Agent（独立子 agent）

**职责：**
- 分析用户需求，生成开发清单
- 根据清单实现功能点代码
- 根据 Evaluator 反馈修改实现

**独立性：**
- 独立的对话上下文
- 专注于编码实现

**指令文件：** `agents/coder.md`

### 3. Evaluator Agent（独立子 agent）

**职责：**
- 评估开发清单的完整性、合理性
- 评估功能点实现的代码质量
- 生成结构化的评估报告

**独立性：**
- 独立的对话上下文
- 专注于评估审查

**指令文件：** `agents/evaluator.md`

## 权限控制

**三种权限级别：**

| 级别 | 跳过循环 | 修改文件 | 覆盖评估 | 适用场景 |
|------|---------|---------|---------|---------|
| **strict** | ❌ | ❌ | ❌ | 生产环境（默认） |
| **moderate** | ⚠️ 需确认 | ❌ | ⚠️ 需确认 | 大多数场景 |
| **flexible** | ✅ | ✅ | ✅ | 原型开发 |

**默认行为（strict 模式）：**
- Master 不能跳过循环
- Master 不能修改业务文件
- Master 不能覆盖评估结果
- 所有操作记录到审计日志

**详细规则：** 参考 `references/permission_rules.md`

## 工作流程

### 阶段 1：清单生成与评估循环（最多 3 次）

```
循环开始：
  1. Master → 派发任务给 Coder（生成清单）
  2. Master → 派发任务给 Evaluator（评估清单）
  3. Master → 判断是否通过
     - 通过 → 进入阶段 2
     - 未通过 → 提取反馈，继续循环
循环结束
```

### 阶段 2：功能点开发循环（每个功能点最多 3 次）

```
对每个功能点：
  循环开始：
    1. Master → 派发任务给 Coder（实现功能点）
    2. Master → 派发任务给 Evaluator（评估实现）
    3. Master → 判断是否通过
       - 通过 → 标记完成，下一个功能点
       - 未通过 → 提取反馈，继续循环
  循环结束
```

**详细流程：** 参考 `references/workflow_logic.md`

## 输出文件结构

```
dev-tasks/
└── [task-id]/
    ├── task.md                    # 用户需求
    ├── config.json                # 任务配置
    ├── outputs/
    │   ├── checklist.md           # 开发清单
    │   ├── evaluation.md          # 清单评估报告
    │   ├── implementation-*.md    # 功能点实现
    │   ├── evaluation-*.md        # 功能点评估报告
    │   └── completion-report.md   # 完成报告
    └── logs/
        ├── timeline.jsonl         # 流程日志
        └── audit.jsonl            # 审计日志
```

## 配置参数

编辑 `config.json`：

```json
{
  "max_review_cycles": 3,
  "require_tests": true,
  "evaluation_strictness": "medium",
  "master_permissions": {
    "level": "strict",
    "can_skip_cycles": false,
    "can_modify_files": false,
    "can_override_evaluator": false,
    "audit_all_operations": true
  }
}
```

## 参考文件

**指令文件：**
- `agents/master.md` - Master 协调器指令
- `agents/coder.md` - Coder agent 指令
- `agents/evaluator.md` - Evaluator agent 指令

**详细文档：**
- `references/permission_rules.md` - 权限规则详细说明
- `references/workflow_logic.md` - 工作流逻辑详细说明
- `references/review_checklist.md` - 评估清单

**模板和示例：**
- `templates/` - 各类报告模板
- `examples/usage_example.md` - 使用示例

**版本历史：**
- `CHANGELOG.md` - 版本历史和未来规划

## 更新日志

- **v2.2**: 精简优化，文件分层架构
- **v2.1**: 新增严格的权限控制机制
- **v2.0**: 从角色切换模式升级为真正的多 agent 协作
- **v1.0**: 初始版本，基于角色切换模式

**完整历史：** 参考 `CHANGELOG.md`
