# Coder + Evaluator 真正的多 Agent 协作工作流

## 🎯 核心升级：从角色切换到真正的独立 Agent

这个 skill 实现了**真正的多 agent 协作**，而非角色切换模式。

### 什么是真正的独立 Agent 协作？

**角色切换模式（旧）：**
```
同一个 Agent 实例
    ↓
切换到 Coder 角色 → 写代码
    ↓
切换到 Evaluator 角色 → 评估自己的代码
    ↓
问题：存在确认偏差，无法真正客观评估
```

**真正的独立 Agent 协作（新）：**
```
Master 协调器（当前会话）
    ├── 派发任务 → Coder Agent（独立实例）
    │                 ↓
    │            写代码到文件
    │                 ↓
    │            独立的上下文，专注编码
    │
    └── 派发任务 → Evaluator Agent（独立实例）
                      ↓
                 读取代码文件
                      ↓
                 独立的上下文，专注评估
                      ↓
                 输出评估报告
```

## 📁 目录结构

```
coder-evaluator-loop/
├── SKILL.md                    # 主文件：完整工作流定义
├── README.md                   # 本文件：快速概览
├── config.json                 # 配置文件
├── agents/                     # 🆕 独立 agent 指令
│   ├── master.md              # Master 协调器指令
│   ├── coder.md               # Coder agent 指令
│   └── evaluator.md           # Evaluator agent 指令
├── references/
│   └── review_checklist.md    # 评估清单
├── templates/
│   ├── development_checklist.md
│   ├── evaluation_report.md
│   └── completion_report.md
└── examples/
    └── usage_example.md       # 使用示例
```

## 🔄 工作流程

### 阶段 1：清单生成与评估循环（最多 3 次）

```
循环开始：
  1. Master → 派发任务给 Coder
     - 输入：用户需求
     - 输出：开发清单

  2. Master → 派发任务给 Evaluator
     - 输入：开发清单
     - 输出：评估报告

  3. Master → 判断是否通过
     - 通过 → 进入阶段 2
     - 未通过 → 提取反馈，继续循环

循环结束
```

### 阶段 2：功能点开发循环（每个功能点最多 3 次）

```
对每个功能点：
  循环开始：
    1. Master → 派发任务给 Coder
       - 输入：功能点描述 + 反馈（如果有）
       - 输出：代码实现

    2. Master → 派发任务给 Evaluator
       - 输入：代码实现
       - 输出：评估报告

    3. Master → 判断是否通过
       - 通过 → 标记完成，下一个功能点
       - 未通过 → 提取反馈，继续循环

  循环结束
```

## ✅ 真正独立 Agent 协作的优势

### 1. 消除确认偏差

| 场景 | 角色切换 | 独立 Agent |
|------|---------|-----------|
| 评估自己的代码 | ❌ 可能忽略自己的错误 | ✅ 客观独立评估 |
| 发现设计缺陷 | ❌ 受先入为主影响 | ✅ 全新视角审查 |
| 代码质量评分 | ❌ 可能偏高 | ✅ 更客观准确 |

### 2. 上下文隔离

| 场景 | 角色切换 | 独立 Agent |
|------|---------|-----------|
| 专注度 | ❌ 上下文混杂 | ✅ 只看到需要的信息 |
| 记忆干扰 | ❌ 可能混淆不同阶段 | ✅ 完全独立的记忆 |
| 角色认知 | ❌ 模糊 | ✅ 清晰的角色定义 |

### 3. 可扩展性

**当前架构（3 个 agent）：**
```
Master
├── Coder
└── Evaluator
```

**可扩展架构（N 个 agent）：**
```
Master
├── Coder
├── Evaluator
├── Security Auditor      # 🆕 安全审查
├── Performance Analyst   # 🆕 性能分析
├── Documentation Writer  # 🆕 文档编写
└── Test Engineer         # 🆕 测试工程师
```

### 4. 并行执行能力

**角色切换：** 必须串行执行
```
Coder 完成 → Evaluator 开始 → Coder 修改 → Evaluator 评估
```

**独立 Agent：** 可以并行派发
```
功能点 1 → Coder Agent 1
功能点 2 → Coder Agent 2  （并行）
功能点 3 → Coder Agent 3
```

## 📋 Agent 职责定义

### Master 协调器

**职责：**
- 接收用户需求
- 创建工作目录
- 派发任务给子 agent
- 读取 agent 输出
- 判断是否通过
- 管理循环逻辑
- 生成完成报告

**不参与：**
- 具体编码
- 代码评估

**指令文件：** `agents/master.md`

### Coder Agent

**职责：**
- 分析用户需求
- 生成开发清单
- 实现功能点代码
- 根据反馈修改代码

**独立性：**
- 独立的对话上下文
- 专注于编码实现
- 不知道 Evaluator 的存在

**指令文件：** `agents/coder.md`

### Evaluator Agent

**职责：**
- 评估开发清单
- 评估功能点实现
- 生成评估报告
- 提出改进建议

**独立性：**
- 独立的对话上下文
- 专注于评估审查
- 不知道 Coder 的存在

**指令文件：** `agents/evaluator.md`

## 📊 输出文件说明

### Master 输出

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
        └── timeline.jsonl         # 流程日志
```

### 通信文件格式

**Coder 输出（outputs/checklist.md）：**
```markdown
# 开发清单

## 需求概述
[用户需求描述]

## 功能点列表
### 功能点 1: [名称]
- 描述: [详细说明]
- 涉及文件: [文件列表]
- 验收标准:
  - [ ] [标准 1]
  - [ ] [标准 2]

## 开发顺序建议
1. [功能点 X] - 理由
...
```

**Evaluator 输出（outputs/evaluation.md）：**
```markdown
# 清单评估报告

## 基本信息
- 评估结果: [通过/需要修改]

## 详细评估
### 需求覆盖度
- 评分: 8/10
- 状态: ✓ 通过

## 问题列表
### 高严重程度问题
1. **[问题标题]**
   - 位置: [文件:行号]
   - 建议: [如何修改]

## 结论
[最终结论]
```

## ⚙️ 配置参数

编辑 `config.json`：

```json
{
  "max_review_cycles": 3,           // 最大评估轮次
  "require_tests": true,            // 是否要求测试
  "evaluation_strictness": "medium", // low/medium/high
  "parallel_dispatch": false,       // 是否并行派发（实验性）
  "timeout_minutes": 10             // agent 超时时间
}
```

## 🚀 使用示例

### 场景：开发用户认证模块

**用户说：**
"帮我开发一个用户认证模块，支持注册、登录、密码重置功能"

**执行流程：**

1. **Master 初始化**
   - 创建任务 ID: `task-20240101-120000`
   - 创建工作目录
   - 将需求写入 `task.md`

2. **清单评估循环**
   - 第 1 轮：
     - Coder 生成清单
     - Evaluator 评估：需要修改（缺少登出功能）
   - 第 2 轮：
     - Coder 修改清单
     - Evaluator 评估：通过

3. **功能点开发循环**
   - 功能点 1（用户注册）：
     - Coder 实现
     - Evaluator 评估：需要修改（数据库连接未关闭）
     - Coder 修改
     - Evaluator 评估：通过
   - 功能点 2（用户登录）：
     - Coder 实现
     - Evaluator 评估：通过（1 轮）
   - 功能点 3（密码重置）：
     - Coder 实现
     - Evaluator 评估：通过（2 轮）
   - 功能点 4（用户登出）：
     - Coder 实现
     - Evaluator 评估：通过（1 轮）

4. **Master 生成完成报告**
   - 总耗时：2.5 小时
   - 平均评估轮次：1.5 轮
   - 代码质量评分：87 分

## 🔧 技术实现

### Agent 派发方式

使用 Agent 工具派发任务：

```python
# 派发给 Coder
Agent({
  "subagent_type": "general-purpose",
  "description": "Coder agent 生成开发清单",
  "prompt": "读取 agents/coder.md 了解你的角色和职责。\n\n任务类型: checklist\n\n用户需求:\n[用户需求内容]\n\n输出要求:\n- 将结果写入 outputs/checklist.md\n- 遵循 coder.md 中定义的输出格式",
  "run_in_background": True
})

# 派发给 Evaluator
Agent({
  "subagent_type": "general-purpose",
  "description": "Evaluator agent 评估清单",
  "prompt": "读取 agents/evaluator.md 了解你的角色和职责。\n\n任务类型: checklist-evaluation\n\n待评估内容:\n[outputs/checklist.md 内容]\n\n输出要求:\n- 将评估报告写入 outputs/evaluation.md\n- 遵循 evaluator.md 中定义的输出格式",
  "run_in_background": True
})
```

### 文件系统通信

所有 agent 通过文件系统进行通信：
- Master 写入需求文件 → Coder/Evaluator 读取
- Coder/Evaluator 写入输出文件 → Master 读取

### 任务通知

子 agent 完成时，Master 收到通知：
```
Agent task completed
Total tokens: 84852
Duration: 23332ms
```

## 📚 参考资料

### 指令文件
- `agents/master.md` - Master 协调器详细指令
- `agents/coder.md` - Coder agent 详细指令
- `agents/evaluator.md` - Evaluator agent 详细指令

### 参考文件
- `references/review_checklist.md` - 详细评估清单
- `templates/` - 各类报告模板
- `examples/usage_example.md` - 完整使用示例

### 外部参考
- skill-creator 的多 agent 实现：`/Users/xialiang/Library/Application Support/CherryStudio/Data/Skills/skill-creator/`
- Claude Code 的 Agent 工具文档

## 🎓 最佳实践

### 1. 清晰的需求描述
- 提供足够的上下文
- 明确验收标准
- 说明技术约束

### 2. 合理的循环次数
- 默认 3 次，通常足够
- 如果连续 3 次未通过，考虑人工介入

### 3. 关注评估反馈
- 优先处理"高"严重程度问题
- 合理采纳改进建议
- 不要忽视"中"严重程度问题

### 4. 保持 agent 独立性
- 不要在派发任务时透露其他 agent 的输出
- 让每个 agent 独立工作
- 遵循通信协议

## 🔮 未来扩展

### 可添加的新 agent

1. **Security Auditor**
   - 专注于安全审查
   - 检查 SQL 注入、XSS、认证漏洞等

2. **Performance Analyst**
   - 专注于性能分析
   - 检查时间复杂度、空间复杂度、缓存策略等

3. **Documentation Writer**
   - 专注于文档编写
   - 生成 API 文档、用户手册、架构文档等

4. **Test Engineer**
   - 专注于测试
   - 编写单元测试、集成测试、端到端测试等

### 并行执行优化

当功能点之间无依赖时，可以并行派发：

```
功能点 1 → Coder Agent 1  ──┐
功能点 2 → Coder Agent 2  ──┤ 并行
功能点 3 → Coder Agent 3  ──┘
                              ↓
                      Evaluator Agent（串行评估）
```

## 📝 更新日志

- **v2.0** (2024-01-01)
  - 从角色切换模式升级为真正的多 agent 协作
  - 添加独立的 agent 指令文件
  - 支持并行派发（实验性）
  - 改进错误处理和日志记录

- **v1.0** (2023-12-01)
  - 初始版本
  - 基于角色切换模式

## 🤝 贡献

欢迎提出改进建议和 bug 报告！

## 📄 许可证

MIT License
