# 工作流逻辑详细说明

## 整体流程

```
1. 初始化
   - 读取配置文件
   - 确定权限级别
   - 创建工作目录

2. 清单评估循环（第一层）
   - 最多 3 次
   - Coder 生成清单 → Evaluator 评估 → 判断是否通过
   - 不允许跳过

3. 功能点开发循环（第二层）
   - 对每个功能点
   - 最多 3 次
   - Coder 实现 → Evaluator 评估 → 判断是否通过
   - 不允许跳过

4. 完成
   - 生成完成报告
   - 向用户汇报
```

---

## 阶段 0：初始化

### 步骤

1. **读取配置文件**
   ```python
   config = read_file("config.json")
   permission_level = config.master_permissions.level
   max_review_cycles = config.max_review_cycles
   ```

2. **确定权限级别**
   ```python
   log(f"权限级别: {permission_level}")
   log(f"最大循环次数: {max_review_cycles}")
   ```

3. **创建工作目录**
   ```
   dev-tasks/
   └── [task-id]/
       ├── task.md
       ├── config.json
       ├── outputs/
       └── logs/
   ```

4. **写入任务描述**
   ```python
   write_file("task.md", user_requirements)
   ```

---

## 阶段 1：清单评估循环

### 循环逻辑

```python
for cycle in range(max_review_cycles):  # 默认 3 次
    log(f"清单评估循环，第 {cycle+1} 轮")

    # 1. 派发给 Coder 生成/修改清单
    if cycle == 0:
        coder_output = dispatch_to_coder("checklist", user_requirements)
    else:
        coder_output = dispatch_to_coder("checklist", user_requirements, feedback)

    # 2. 派发给 Evaluator 评估清单
    eval_result = dispatch_to_evaluator("checklist-evaluation", coder_output)

    # 3. 判断是否通过
    if eval_result.passed:
        log("清单通过评估")
        break
    else:
        feedback = eval_result.feedback
        log(f"清单需要修改，第 {cycle+1} 轮")
        log(f"问题: {eval_result.issues}")
else:
    # 循环 3 次仍未通过
    log("清单评估达到最大轮次，需要人工介入")
    notify_user("清单评估未通过，需要人工决策")
```

### 通过标准

**必须满足：**
- Evaluator 反馈"通过"
- 无"高"严重程度问题
- "中"严重程度问题 ≤ 2 个
- 需求覆盖度 ≥ 90%

**不通过条件：**
- Evaluator 反馈"需要修改"
- 存在"高"严重程度问题
- 存在超过 2 个"中"严重程度问题
- 需求覆盖度 < 90%

### 任务派发格式

**派发给 Coder：**
```python
dispatch_to_coder(
    task_type="checklist",
    requirements=user_requirements,
    feedback=feedback if cycle > 0 else None
)
```

**派发给 Evaluator：**
```python
dispatch_to_evaluator(
    task_type="checklist-evaluation",
    content=coder_output,
    evaluation_criteria="需求覆盖度、任务拆分、依赖关系、技术可行性、风险识别"
)
```

---

## 阶段 2：功能点开发循环

### 功能点提取

从通过的清单中提取功能点列表：

```python
feature_list = extract_features_from_checklist(checklist)
log(f"提取到 {len(feature_list)} 个功能点")
```

### 循环逻辑

```python
for feature in feature_list:
    log(f"开始开发功能点: {feature.name}")

    for cycle in range(max_review_cycles):  # 默认 3 次
        log(f"功能点 {feature.name} 开发循环，第 {cycle+1} 轮")

        # 1. 派发给 Coder 实现/修改功能点
        if cycle == 0:
            coder_output = dispatch_to_coder("implementation", feature)
        else:
            coder_output = dispatch_to_coder("modification", feature, feedback)

        # 2. 派发给 Evaluator 评估实现
        eval_result = dispatch_to_evaluator("implementation-evaluation", coder_output)

        # 3. 判断是否通过
        if eval_result.passed:
            log(f"功能点 {feature.name} 通过评估")
            mark_feature_complete(feature)
            break
        else:
            feedback = eval_result.feedback
            log(f"功能点 {feature.name} 需要修改，第 {cycle+1} 轮")
            log(f"问题: {eval_result.issues}")
    else:
        # 循环 3 次仍未通过
        log(f"功能点 {feature.name} 评估达到最大轮次")
        notify_user(f"功能点 {feature.name} 未通过，需要人工决策")
```

### 通过标准

**必须满足：**
- Evaluator 反馈"通过"
- 功能正确性 100% 通过
- 无"高"严重程度问题
- "中"严重程度问题 ≤ 2 个
- 代码质量评分 ≥ 80 分

**不通过条件：**
- Evaluator 反馈"需要修改"
- 功能正确性 < 100%
- 存在"高"严重程度问题
- 存在超过 2 个"中"严重程度问题
- 代码质量评分 < 80 分

### 任务派发格式

**派发给 Coder：**
```python
dispatch_to_coder(
    task_type="implementation",
    feature=feature,
    feedback=feedback if cycle > 0 else None
)
```

**派发给 Evaluator：**
```python
dispatch_to_evaluator(
    task_type="implementation-evaluation",
    content=coder_output,
    evaluation_criteria="功能正确性、代码质量、项目一致性、测试覆盖"
)
```

---

## 阶段 3：完成与总结

### 完成报告生成

```python
completion_report = generate_completion_report(
    task_id=task_id,
    start_time=start_time,
    end_time=datetime.now(),
    checklist_evaluations=checklist_evaluations,
    feature_evaluations=feature_evaluations,
    output_files=output_files
)

write_file("outputs/completion-report.md", completion_report)
```

### 向用户汇报

```python
notify_user(f"""
开发任务完成！

任务 ID: {task_id}
总耗时: {duration}
功能点: {len(feature_list)} 个
平均评估轮次: {avg_evaluation_rounds}

详细报告: outputs/completion-report.md
""")
```

---

## 任务派发机制

### 派发给 Coder

```python
def dispatch_to_coder(task_type, content, feedback=None):
    """
    派发任务给 Coder agent

    参数:
        task_type: 任务类型
            - checklist: 生成开发清单
            - implementation: 实现功能点
            - modification: 修改实现
        content: 任务内容（用户需求或功能点描述）
        feedback: Evaluator 的反馈（如果有）

    返回:
        coder_output: Coder 的输出
    """
    prompt = f"""
读取 agents/coder.md 了解你的角色和职责。

任务类型: {task_type}

用户需求/功能点描述:
{content}

{"Evaluator 反馈:" + feedback if feedback else ""}

输出要求:
- 将结果写入 outputs/[对应文件].md
- 遵循 coder.md 中定义的输出格式

工作目录: {task_path}
"""

    # 派发任务
    agent_result = Agent({
        "subagent_type": "general-purpose",
        "description": f"Coder agent {task_type}",
        "prompt": prompt,
        "run_in_background": True
    })

    # 等待完成
    wait_for_completion(agent_result)

    # 读取输出
    coder_output = read_file(f"outputs/{get_output_filename(task_type)}")

    return coder_output
```

### 派发给 Evaluator

```python
def dispatch_to_evaluator(task_type, content):
    """
    派发任务给 Evaluator agent

    参数:
        task_type: 任务类型
            - checklist-evaluation: 评估清单
            - implementation-evaluation: 评估实现
        content: 待评估的内容

    返回:
        eval_result: 评估结果
            - passed: 是否通过
            - feedback: 反馈信息
            - issues: 问题列表
    """
    prompt = f"""
读取 agents/evaluator.md 了解你的角色和职责。

任务类型: {task_type}

待评估内容:
{content}

评估标准:
{get_evaluation_criteria(task_type)}

输出要求:
- 将评估报告写入 outputs/evaluation.md
- 遵循 evaluator.md 中定义的输出格式

工作目录: {task_path}
"""

    # 派发任务
    agent_result = Agent({
        "subagent_type": "general-purpose",
        "description": f"Evaluator agent {task_type}",
        "prompt": prompt,
        "run_in_background": True
    })

    # 等待完成
    wait_for_completion(agent_result)

    # 读取输出
    eval_output = read_file("outputs/evaluation.md")

    # 解析结果
    eval_result = parse_evaluation_result(eval_output)

    return eval_result
```

---

## 循环控制规则

### 规则 1：不允许跳过循环

**原因：**
- 确保每个功能点都经过完整评估
- 防止质量问题遗漏
- 保持流程的一致性

**实现：**
```python
# 禁止跳过循环
allowed, requires_approval, reason = check_permission("skip_cycle")
if not allowed:
    log(f"权限检查：{reason}")
    # 继续执行循环，不跳过
```

### 规则 2：必须基于 Evaluator 判断

**原因：**
- 确保评估的客观性
- 防止 Master 主观判断
- 保持多 agent 协作的独立性

**实现：**
```python
# 严格遵循 Evaluator 判断
if eval_result.passed:
    log("通过评估")
else:
    log("需要修改")
    feedback = eval_result.feedback
```

### 规则 3：最大循环次数限制

**原因：**
- 防止无限循环
- 促使问题及时解决
- 保持开发效率

**实现：**
```python
for cycle in range(max_review_cycles):  # 默认 3 次
    # 循环逻辑
    if passed:
        break
else:
    # 超过最大次数
    notify_user("需要人工介入")
```

---

## 输出文件管理

### 文件命名规则

```
outputs/
├── checklist.md                    # 开发清单
├── evaluation.md                   # 清单评估报告
├── implementation-功能点名称.md     # 功能点实现
├── evaluation-功能点名称.md         # 功能点评估报告
└── completion-report.md            # 完成报告
```

### 文件内容规范

**开发清单（checklist.md）：**
- 需求概述
- 功能点列表
- 开发顺序建议
- 技术选型
- 风险识别

**评估报告（evaluation.md）：**
- 评估结果（通过/需要修改）
- 详细评估（各维度评分）
- 问题列表（按严重程度）
- 改进建议

**功能点实现（implementation-*.md）：**
- 实现概述
- 修改的文件
- 新增的文件
- 验收标准完成情况
- 测试建议

**完成报告（completion-report.md）：**
- 任务概述
- 执行摘要
- 质量统计
- 输出文件列表
- 后续建议

---

## 错误处理

### Agent 超时

```python
try:
    agent_result = Agent({
        "prompt": prompt,
        "timeout": timeout_minutes * 60 * 1000
    })
except TimeoutError:
    log("Agent 超时")
    notify_user("Coder/Evaluator agent 超时，请检查")
    # 等待用户决策
```

### Agent 失败

```python
try:
    agent_result = Agent({"prompt": prompt})
except AgentError as e:
    log(f"Agent 失败: {e}")
    notify_user(f"Coder/Evaluator agent 失败: {e}")
    # 等待用户决策
```

### 输出格式错误

```python
try:
    eval_result = parse_evaluation_result(eval_output)
except ParseError:
    log("输出格式错误")
    # 重新派发任务
    dispatch_to_evaluator(task_type, content)
```

---

## 日志记录

### 日志格式

```json
{
  "timestamp": "2024-01-01T12:00:00Z",
  "event": "cycle_start",
  "phase": "checklist_evaluation",
  "cycle": 1,
  "details": {}
}
```

### 事件类型

- `task_start`: 任务开始
- `task_complete`: 任务完成
- `coder_dispatched`: 任务派发给 Coder
- `coder_completed`: Coder 完成任务
- `evaluator_dispatched`: 任务派发给 Evaluator
- `evaluator_completed`: Evaluator 完成评估
- `cycle_start`: 循环开始
- `cycle_end`: 循环结束
- `phase_complete`: 阶段完成
- `permission_check`: 权限检查
- `permission_violation`: 权限违规
- `error`: 错误发生

### 日志文件

- 文件路径：`logs/timeline.jsonl`
- 格式：JSON Lines
- 编码：UTF-8
