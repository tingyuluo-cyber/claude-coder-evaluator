# 以下都为AI总结,看看就好,也别太当回事
# Coder + Evaluator 真正的多 Agent 协作工作流

## 核心特性

**真正的独立 Agent 协作：**
- 3 个独立 agent 实例（Master + Coder + Evaluator）
- 完全隔离的上下文
- 消除确认偏差
- 严格的权限控制

**双层循环开发评估：**
- 清单评估循环（最多 3 次）
- 功能点开发循环（每个功能点最多 3 次）
- 不允许跳过循环

## 快速开始

### 触发条件

当你说以下内容时，会自动触发此 skill：
- "帮我开发一个用户认证模块"
- "实现一个文件上传功能"
- "创建一个 REST API"
- 任何涉及多步骤功能开发的任务

### 执行流程

1. **Master 初始化**
   - 创建任务目录
   - 读取配置文件

2. **清单评估循环**
   - Coder 生成开发清单
   - Evaluator 评估清单
   - 循环直到通过（最多 3 次）

3. **功能点开发循环**
   - 对每个功能点：
     - Coder 实现功能点
     - Evaluator 评估实现
     - 循环直到通过（最多 3 次）

4. **完成**
   - 生成完成报告
   - 向用户汇报

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

## 文件结构

```
coder-evaluator-loop/
├── SKILL.md                    # 主文件：核心指令 (< 8 KB)
├── README.md                   # 本文件：快速概览 (< 6 KB)
├── CHANGELOG.md                # 版本历史
├── config.json                 # 配置文件
├── agents/                     # Agent 指令
│   ├── master.md               # Master 协调器 (< 5 KB)
│   ├── coder.md                # Coder agent
│   └── evaluator.md            # Evaluator agent
├── references/                 # 详细文档
│   ├── permission_rules.md     # 权限规则详解
│   ├── workflow_logic.md       # 工作流逻辑详解
│   └── review_checklist.md     # 审查清单
├── templates/                  # 报告模板
└── examples/                   # 使用示例
```

## 配置示例

```json
{
  "max_review_cycles": 3,
  "require_tests": true,
  "evaluation_strictness": "medium",
  "auto_fix_minor_issues": false,
  "strictness_levels": {
    "low": { "max_cycles": 5, "require_tests": false, "allow_minor_issues": true },
    "medium": { "max_cycles": 3, "require_tests": true, "allow_minor_issues": false },
    "high": { "max_cycles": 2, "require_tests": true, "allow_minor_issues": false }
  },
  "master_permissions": {
    "level": "strict",
    "can_skip_cycles": false,
    "can_modify_files": false,
    "can_override_evaluator": false,
    "require_approval_for_skip": true,
    "require_approval_for_override": true,
    "audit_all_operations": true,
    "allowed_write_paths": ["logs/", "task.md", "config.json", "outputs/completion-report.md"],
    "denied_write_paths": ["outputs/checklist.md", "outputs/evaluation.md", "outputs/implementation-*.md"]
  },
  "audit": {
    "enabled": true,
    "log_file": "logs/audit.jsonl",
    "log_permission_checks": true,
    "log_permission_violations": true,
    "log_user_approvals": true
  }
}
```

## 输出示例

```
dev-tasks/
└── task-20240101-120000/
    ├── task.md
    ├── outputs/
    │   ├── checklist.md
    │   ├── evaluation.md
    │   ├── implementation-用户注册.md
    │   ├── evaluation-用户注册.md
    │   └── completion-report.md
    └── logs/
        ├── timeline.jsonl
        └── audit.jsonl
```

## 最佳实践

1. **默认使用 strict 模式**
   - 确保流程严格执行
   - 防止 Master 越权操作

2. **不要跳过循环**
   - 即使代码看起来已经很好
   - 确保每个功能点都经过完整评估

3. **尊重 Evaluator 判断**
   - 不要覆盖评估结果
   - 以 Evaluator 的判断为准

4. **查看审计日志**
   - 定期检查 `logs/audit.jsonl`
   - 确保没有越权操作

## 文档导航

**快速了解：**
- 本文件（README.md）- 快速概览

**核心指令：**
- `SKILL.md` - 核心指令（精简版）

**Agent 指令：**
- `agents/master.md` - Master 协调器（精简版）
- `agents/coder.md` - Coder agent
- `agents/evaluator.md` - Evaluator agent

**详细参考：**
- `references/permission_rules.md` - 权限规则详解
- `references/workflow_logic.md` - 工作流逻辑详解
- `references/review_checklist.md` - 审查清单

**版本历史：**
- `CHANGELOG.md` - 版本历史和未来规划

## 更新日志

- **v2.2**: 精简优化，文件分层架构（渐进式加载）
  - master.md 精简（19.3KB → < 5KB）
  - SKILL.md 精简（14.7KB → < 8KB）
  - README.md 精简（14.6KB → < 6KB）
  - 新增权限规则和工作流逻辑参考文档
- **v2.1**: 新增严格的权限控制机制
- **v2.0**: 从角色切换模式升级为真正的多 agent 协作
- **v1.0**: 初始版本，基于角色切换模式

**完整历史：** 参考 `CHANGELOG.md`

## 许可证

MIT License
