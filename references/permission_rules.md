# 权限规则详细说明

## 权限级别定义

### Strict 模式（默认）

**适用场景：** 生产环境、严格流程、需要高度可追溯性

**权限矩阵：**

| 操作类型 | 权限 | 说明 |
|---------|------|------|
| 写入 logs/ | ✅ 允许 | 只能写入日志文件 |
| 写入 task.md | ✅ 允许 | 任务描述文件 |
| 写入 config.json | ✅ 允许 | 配置文件 |
| 写入 outputs/completion-report.md | ✅ 允许 | 最终完成报告 |
| 写入 outputs/checklist.md | ❌ 禁止 | Coder 的工作 |
| 写入 outputs/evaluation.md | ❌ 禁止 | Evaluator 的工作 |
| 写入 outputs/implementation-*.md | ❌ 禁止 | Coder 的工作 |
| 写入 outputs/modification-*.md | ❌ 禁止 | Coder 的工作 |
| 跳过循环 | ❌ 禁止 | 必须严格执行 |
| 覆盖评估结果 | ❌ 禁止 | 必须以 Evaluator 判断为准 |
| 直接修改 agent 输出 | ❌ 禁止 | 所有修改必须派发给对应 agent |

**行为规范：**
- Master 只能协调，不能执行具体工作
- 所有循环必须严格执行，不能跳过
- 所有评估必须以 Evaluator 判断为准
- 所有修改必须派发给对应的 agent

---

### Moderate 模式

**适用场景：** 大多数场景、需要一定灵活性

**权限矩阵：**

| 操作类型 | 权限 | 说明 |
|---------|------|------|
| 写入 logs/ | ✅ 允许 | 日志文件 |
| 写入 task.md | ✅ 允许 | 任务描述文件 |
| 写入 config.json | ✅ 允许 | 配置文件 |
| 写入 outputs/completion-report.md | ✅ 允许 | 最终完成报告 |
| 写入 outputs/checklist.md | ❌ 禁止 | Coder 的工作 |
| 写入 outputs/evaluation.md | ❌ 禁止 | Evaluator 的工作 |
| 写入 outputs/implementation-*.md | ❌ 禁止 | Coder 的工作 |
| 写入 outputs/modification-*.md | ❌ 禁止 | Coder 的工作 |
| 跳过循环 | ⚠️ 需要确认 | 需要用户确认 |
| 覆盖评估结果 | ⚠️ 需要确认 | 需要用户确认 |
| 直接修改 agent 输出 | ❌ 禁止 | 所有修改必须派发给对应 agent |

**行为规范：**
- Master 可以跳过循环，但需要用户确认
- Master 可以覆盖评估结果，但需要用户确认
- 所有越权操作必须记录到审计日志
- 必须等待用户确认后才能执行

---

### Flexible 模式

**适用场景：** 原型开发、快速迭代、实验性项目

**权限矩阵：**

| 操作类型 | 权限 | 说明 |
|---------|------|------|
| 写入 logs/ | ✅ 允许 | 日志文件 |
| 写入 task.md | ✅ 允许 | 任务描述文件 |
| 写入 config.json | ✅ 允许 | 配置文件 |
| 写入 outputs/completion-report.md | ✅ 允许 | 最终完成报告 |
| 写入 outputs/checklist.md | ✅ 允许 | 但记录审计日志 |
| 写入 outputs/evaluation.md | ✅ 允许 | 但记录审计日志 |
| 写入 outputs/implementation-*.md | ✅ 允许 | 但记录审计日志 |
| 写入 outputs/modification-*.md | ✅ 允许 | 但记录审计日志 |
| 跳过循环 | ✅ 允许 | 但记录审计日志 |
| 覆盖评估结果 | ✅ 允许 | 但记录审计日志 |
| 直接修改 agent 输出 | ✅ 允许 | 但记录审计日志 |

**行为规范：**
- Master 可以执行任何操作
- 所有操作必须记录到审计日志
- 用户可以选择是否需要确认
- 保留完整的操作历史

---

## 权限检查流程

### 检查逻辑

```python
def check_permission(action, target=None):
    """
    检查操作权限

    参数:
        action: 操作类型
            - write_file: 写入文件
            - skip_cycle: 跳过循环
            - override_eval: 覆盖评估结果
            - modify_output: 修改 agent 输出
        target: 目标文件路径（write_file 时需要）

    返回:
        allowed: 是否允许
        requires_approval: 是否需要用户确认
        reason: 说明原因
    """
    config = load_config()
    level = config.master_permissions.level

    # 根据权限级别判断
    if level == "strict":
        return check_strict_permission(action, target)
    elif level == "moderate":
        return check_moderate_permission(action, target)
    elif level == "flexible":
        return check_flexible_permission(action, target)
```

### Strict 模式检查

```python
def check_strict_permission(action, target):
    # 写入文件检查
    if action == "write_file":
        if target in ALLOWED_PATHS:
            return True, False, "允许写入"
        else:
            return False, False, f"strict 模式禁止写入 {target}"

    # 跳过循环检查
    if action == "skip_cycle":
        return False, False, "strict 模式禁止跳过循环"

    # 覆盖评估结果检查
    if action == "override_eval":
        return False, False, "strict 模式禁止覆盖评估结果"

    # 修改 agent 输出检查
    if action == "modify_output":
        return False, False, "strict 模式禁止修改 agent 输出"

    return True, False, "操作允许"
```

### Moderate 模式检查

```python
def check_moderate_permission(action, target):
    # 写入文件检查
    if action == "write_file":
        if target in ALLOWED_PATHS:
            return True, False, "允许写入"
        else:
            return False, False, f"moderate 模式禁止写入 {target}"

    # 跳过循环检查
    if action == "skip_cycle":
        return True, True, "moderate 模式跳过循环需要用户确认"

    # 覆盖评估结果检查
    if action == "override_eval":
        return True, True, "moderate 模式覆盖评估结果需要用户确认"

    # 修改 agent 输出检查
    if action == "modify_output":
        return False, False, "moderate 模式禁止修改 agent 输出"

    return True, False, "操作允许"
```

### Flexible 模式检查

```python
def check_flexible_permission(action, target):
    # 所有操作都允许，但记录审计日志
    return True, False, f"flexible 模式允许 {action}"
```

---

## 审计日志

### 日志格式

```json
{
  "timestamp": "2024-01-01T12:00:00Z",
  "event": "permission_check",
  "action": "skip_cycle",
  "target": null,
  "actor": "master",
  "permission_level": "strict",
  "allowed": false,
  "requires_approval": false,
  "reason": "strict 模式下 Master 禁止跳过循环",
  "user_response": null,
  "final_decision": "denied"
}
```

### 日志事件类型

- `permission_check`: 权限检查
- `permission_violation`: 权限违规
- `user_approval_request`: 请求用户确认
- `user_approval_response`: 用户确认响应
- `operation_executed`: 操作执行
- `operation_denied`: 操作拒绝

### 日志文件位置

- 文件路径：`logs/audit.jsonl`
- 格式：JSON Lines（每行一个 JSON 对象）
- 编码：UTF-8

---

## 允许/禁止路径列表

### 允许写入的路径（所有模式）

```json
{
  "allowed_write_paths": [
    "logs/",
    "task.md",
    "config.json",
    "outputs/completion-report.md"
  ]
}
```

### 禁止写入的路径（strict 模式）

```json
{
  "denied_write_paths": [
    "outputs/checklist.md",
    "outputs/evaluation.md",
    "outputs/implementation-*.md",
    "outputs/modification-*.md"
  ]
}
```

---

## 越权处理流程

### 检测到越权操作

1. **记录到审计日志**
   ```json
   {
     "event": "permission_violation",
     "action": "skip_cycle",
     "allowed": false,
     "reason": "strict 模式禁止跳过循环"
   }
   ```

2. **通知用户（如果需要确认）**
   ```
   检测到越权操作：跳过循环
   当前权限级别：moderate
   需要用户确认：是

   请确认是否允许执行此操作？
   ```

3. **等待用户决策**
   - 用户确认 → 执行操作，记录审计日志
   - 用户拒绝 → 取消操作，记录审计日志
   - 用户无响应 → 默认拒绝，记录审计日志

4. **不自动执行**
   - 任何越权操作都不能自动执行
   - 必须等待用户明确确认
   - 如果无法获取用户确认，默认拒绝

---

## 最佳实践

### 1. 默认使用 Strict 模式

```json
{
  "master_permissions": {
    "level": "strict"
  }
}
```

**理由：**
- 确保流程的严格执行
- 防止 Master 越权操作
- 完整的审计日志

### 2. 谨慎使用 Moderate/Flexible 模式

**适用场景：**
- 原型开发
- 快速迭代
- 实验性项目

**注意事项：**
- 确保有用户确认流程
- 定期检查审计日志
- 不要在生产环境使用 Flexible 模式

### 3. 定期检查审计日志

```bash
# 查看所有权限违规
grep "permission_violation" logs/audit.jsonl

# 查看所有用户确认请求
grep "user_approval_request" logs/audit.jsonl

# 统计越权尝试次数
grep -c "permission_violation" logs/audit.jsonl
```

### 4. 理解权限边界

**Master 应该做的：**
- 派发任务给子 agent
- 读取子 agent 的输出
- 判断是否通过（基于 Evaluator 评估结果）
- 管理循环逻辑
- 生成完成报告

**Master 不应该做的：**
- 直接写代码
- 直接修改清单
- 跳过循环
- 覆盖评估结果
- 修改 agent 输出
