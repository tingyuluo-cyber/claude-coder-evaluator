---
name: coder-evaluator-loop
description: |
  当用户提交功能开发需求时，使用此 skill 实现真正的多 agent 协作工作流。通过 Master 协调器协调独立的 Coder 和 Evaluator agent，实现双层循环开发评估。适用于需要结构化开发流程、代码质量保证、迭代式开发的场景。当用户说"开发这个功能"、"帮我写代码"、"实现这个需求"、"创建项目"等涉及功能开发的任务时触发此 skill。即使用户没有明确要求评估循环，只要任务涉及多步骤功能开发，都应该使用此 skill 来确保质量。
---

# Coder + Evaluator 真正的多 Agent 协作工作流

这是一个基于真正独立 agent 协作的开发工作流，通过 Master 协调器协调 Coder 和 Evaluator 两个独立 agent，实现双层循环开发评估。

## 🎯 核心特性：真正的独立 Agent 协作

**与角色切换模式的区别：**

| 特性 | 角色切换模式 | 真正的多 Agent 协作（本 skill） |
|------|-------------|-------------------------------|
| Agent 实例 | 1 个 | 3 个（Master + Coder + Evaluator） |
| 上下文隔离 | 无 | ✅ 完全隔离 |
| 并行执行 | 无法并行 | ✅ 可并行派发任务 |
| 确认偏差 | 存在 | ✅ 消除（独立审查） |
| 通信方式 | 内部切换 | 文件系统 + 任务通知 |

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
- 创建工作目录和通信文件
- 派发任务给 Coder 和 Evaluator agent
- 读取 agent 输出并判断是否通过
- 管理循环逻辑（最多 3 轮）
- 生成完成报告
- 向用户汇报结果

**不参与：**
- 具体编码（Coder 的工作）
- 代码评估（Evaluator 的工作）

**指令文件：** `agents/master.md`

### 2. Coder Agent（独立子 agent）

**职责：**
- 分析用户需求，生成开发清单
- 根据清单实现功能点代码
- 根据 Evaluator 反馈修改实现
- 遵循代码质量标准

**独立性：**
- 独立的对话上下文
- 专注于编码实现
- 不知道 Evaluator 的存在（只知道收到反馈）

**指令文件：** `agents/coder.md`

### 3. Evaluator Agent（独立子 agent）

**职责：**
- 评估开发清单的完整性、合理性和可行性
- 评估功能点实现的代码质量和需求覆盖度
- 生成结构化的评估报告
- 提出具体、可操作的改进建议

**独立性：**
- 独立的对话上下文
- 专注于评估审查
- 不知道 Coder 的存在（只看到需要评估的内容）

**指令文件：** `agents/evaluator.md`

## 工作流程

### 阶段 0：初始化

1. 接收用户需求
2. 创建任务 ID（格式：`task-YYYYMMDD-HHmmss`）
3. 创建工作目录结构：
   ```
   dev-tasks/
   └── [task-id]/
       ├── task.md
       ├── config.json
       ├── outputs/
       └── logs/
   ```
4. 将用户需求写入 `task.md`

### 阶段 1：清单生成与评估循环

**第一层循环（最多 3 次）：**

```
循环开始：
  1. Master 派发任务给 Coder
     - 任务类型：生成开发清单
     - 输入：用户需求（task.md）
     - 输出：outputs/checklist.md

  2. Master 等待 Coder 完成
     - 通过任务通知获取结果
     - 读取 outputs/checklist.md

  3. Master 派发任务给 Evaluator
     - 任务类型：评估清单
     - 输入：outputs/checklist.md
     - 输出：outputs/evaluation.md

  4. Master 等待 Evaluator 完成
     - 通过任务通知获取结果
     - 读取 outputs/evaluation.md

  5. Master 判断是否通过
     - 解析 evaluation.md 中的评估结果
     - 如果"通过" → 进入阶段 2
     - 如果"需要修改" → 提取反馈，进入下一次循环
     - 如果"需要重新设计" → 重新开始

循环结束
```

**通过标准：**
- 无"高"严重程度问题
- "中"严重程度问题 ≤ 3 个
- 需求覆盖度 ≥ 90%

### 阶段 2：功能点开发循环

**对每个功能点，执行第二层循环（最多 3 次）：**

```
功能点列表（从清单中提取）：
  - 功能点 1
  - 功能点 2
  - ...

对每个功能点：
  循环开始：
    1. Master 派发任务给 Coder
       - 任务类型：实现功能点
       - 输入：功能点描述、验收标准、反馈（如果有）
       - 输出：outputs/implementation-[功能点名].md

    2. Master 等待 Coder 完成
       - 读取 outputs/implementation-[功能点名].md

    3. Master 派发任务给 Evaluator
       - 任务类型：评估功能点实现
       - 输入：outputs/implementation-[功能点名].md
       - 输出：outputs/evaluation-[功能点名].md

    4. Master 等待 Evaluator 完成
       - 读取 outputs/evaluation-[功能点名].md

    5. Master 判断是否通过
       - 如果"通过" → 标记完成，进入下一个功能点
       - 如果"需要修改" → 提取反馈，进入下一次循环

  循环结束
```

**通过标准：**
- 功能正确性 100% 通过
- 无"高"严重程度问题
- "中"严重程度问题 ≤ 2 个
- 代码质量评分 ≥ 80 分

### 阶段 3：完成与总结

1. Master 生成完成报告
2. 记录所有循环和评估结果
3. 向用户汇报：
   - 完成的功能点列表
   - 质量统计（平均评估轮次、最严格功能点等）
   - 输出文件位置
   - 后续建议

## 通信协议

### 文件系统通信

所有 agent 通过文件系统进行通信：

**Coder 输出文件：**
- `outputs/checklist.md` - 开发清单
- `outputs/implementation-[name].md` - 功能点实现
- `outputs/modification-[name].md` - 修改说明

**Evaluator 输出文件：**
- `outputs/evaluation.md` - 清单评估报告
- `outputs/evaluation-[name].md` - 功能点评估报告

**Master 管理文件：**
- `task.md` - 用户需求
- `config.json` - 任务配置
- `logs/timeline.jsonl` - 流程日志
- `outputs/completion-report.md` - 完成报告

### 任务派发格式

Master 派发任务时，使用以下格式：

```
读取 agents/[coder/evaluator].md 了解你的角色和职责。

任务类型: [checklist/implementation/checklist-evaluation/implementation-evaluation]

用户需求/待评估内容:
[具体内容或文件路径]

输出要求:
- 将结果写入 outputs/[对应文件].md
- 遵循 [coder/evaluator].md 中定义的输出格式

工作目录: [task-path]
```

## 配置参数

编辑 `config.json` 进行自定义：

```json
{
  "max_review_cycles": 3,
  "require_tests": true,
  "evaluation_strictness": "medium",
  "parallel_dispatch": false,
  "timeout_minutes": 10
}
```

### 参数说明

- `max_review_cycles`: 最大评估轮次（默认 3）
- `require_tests`: 是否要求测试（默认 true）
- `evaluation_strictness`: 评估严格程度（low/medium/high）
- `parallel_dispatch`: 是否并行派发 Coder 和 Evaluator（实验性）
- `timeout_minutes`: agent 超时时间（默认 10 分钟）

## 使用指南

### 何时使用此 skill
- 开发新功能或模块
- 重构现有代码
- 修复复杂 bug（涉及多文件修改）
- 任何需要结构化开发流程的任务

### 何时不使用此 skill
- 简单的单文件修改
- 明确的单行 bug 修复
- 仅文档更新
- 简单的配置更改

### 自定义选项
- 可以调整最大循环次数（默认 3 次）
- 可以启用/禁用测试要求
- 可以自定义评估标准
- 可以调整功能点粒度

## 与角色切换模式的优势

### 1. 消除确认偏差
- **角色切换**：同一个 LLM 既写代码又评估，可能忽略自己犯的错误
- **独立 Agent**：Evaluator 是完全独立的实例，没有"先入为主"的偏见

### 2. 真正的并行执行
- **角色切换**：必须串行执行（先写完再评估）
- **独立 Agent**：可以并行派发多个功能点给不同 agent

### 3. 上下文隔离
- **角色切换**：所有上下文混在一起，容易混淆
- **独立 Agent**：每个 agent 只看到自己需要的信息，专注度更高

### 4. 可扩展性
- **角色切换**：难以扩展新角色
- **独立 Agent**：可以轻松添加新的 agent 类型（如 Security Auditor、Performance Analyst）

## 错误处理

### Agent 超时
- 记录错误到日志
- 通知用户
- 等待用户决策

### 循环次数超限
- 记录到日志
- 通知用户，说明当前状态
- 提供所有中间结果供用户参考
- 等待用户决策

### Agent 输出格式错误
- 记录到日志
- 尝试解析，如果失败则重新派发任务
- 如果连续 2 次格式错误，通知用户

## 输出示例

### 完成报告示例

```markdown
# 开发任务完成报告

## 任务概述
- **任务 ID**: task-20240101-120000
- **任务描述**: 开发用户认证模块
- **开始时间**: 2024-01-01 12:00:00
- **结束时间**: 2024-01-01 14:30:00
- **总耗时**: 2.5 小时

## 执行摘要

### 清单评估阶段
- **循环次数**: 2 次
- **最终结果**: 通过
- **关键修改**: 添加了登出功能、补充了环境配置说明

### 功能点开发阶段
- **功能点总数**: 4 个
- **通过功能点**: 4 个
- **需要人工介入**: 0 个
- **评估循环统计**:
  - 用户注册: 2 轮通过
  - 用户登录: 1 轮通过
  - 密码重置: 2 轮通过
  - 用户登出: 1 轮通过

## 质量统计
- **平均评估轮次**: 1.5 轮
- **最严格功能点**: 用户注册，2 轮
- **代码质量评分**: 87 分

## 输出文件
- 开发清单: dev-tasks/task-20240101-120000/outputs/checklist.md
- 功能点实现: 
  - dev-tasks/task-20240101-120000/outputs/implementation-用户注册.md
  - dev-tasks/task-20240101-120000/outputs/implementation-用户登录.md
  - dev-tasks/task-20240101-120000/outputs/implementation-密码重置.md
  - dev-tasks/task-20240101-120000/outputs/implementation-用户登出.md
- 评估报告: 
  - dev-tasks/task-20240101-120000/outputs/evaluation.md
  - dev-tasks/task-20240101-120000/outputs/evaluation-用户注册.md
  - ...
- 完成报告: dev-tasks/task-20240101-120000/outputs/completion-report.md

## 建议
1. 考虑添加 rate limiting 防止暴力破解
2. 建议添加登录日志审计功能
3. 可以考虑实现 OAuth2.0 支持
```

## 参考文件

- `agents/master.md` - Master 协调器指令
- `agents/coder.md` - Coder agent 指令
- `agents/evaluator.md` - Evaluator agent 指令
- `references/review_checklist.md` - 详细评估清单
- `templates/` - 各类报告模板
- `examples/usage_example.md` - 使用示例

## 技术实现细节

### Agent 派发方式

使用 Agent 工具派发任务：

```python
# 派发给 Coder
Agent({
  "subagent_type": "general-purpose",
  "description": "Coder agent 生成开发清单",
  "prompt": coder_prompt,
  "run_in_background": True
})

# 派发给 Evaluator
Agent({
  "subagent_type": "general-purpose",
  "description": "Evaluator agent 评估清单",
  "prompt": evaluator_prompt,
  "run_in_background": True
})
```

### 并行执行（实验性）

当 `parallel_dispatch` 为 true 时，可以并行派发多个功能点：

```python
# 同时派发多个功能点给 Coder
for feature in feature_list:
    Agent({
      "subagent_type": "general-purpose",
      "description": f"Coder agent 实现功能点 {feature.name}",
      "prompt": feature_prompt,
      "run_in_background": True
    })
```

**注意：** 并行执行需要谨慎使用，因为：
1. 功能点之间可能有依赖关系
2. 需要确保文件不会冲突
3. 调试和追踪更复杂

## 更新日志

- **v2.0** (2024-01-01): 从角色切换模式升级为真正的多 agent 协作
- **v1.0** (2023-12-01): 初始版本，基于角色切换模式
