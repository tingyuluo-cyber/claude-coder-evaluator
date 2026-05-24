# 版本历史

## 版本规划

### 已完成版本

#### v1.0 - 角色切换模式
**核心特性：**
- 单 agent 角色切换
- 同一个 LLM 扮演 Coder 和 Evaluator
- 基础的双层循环逻辑

**问题：**
- ❌ 存在确认偏差（同一个模型评估自己的代码）
- ❌ 上下文混杂（角色切换导致信息泄露）
- ❌ 无法真正客观评估

**文件结构：**
```
coder-evaluator-loop/
├── SKILL.md
├── config.json
├── references/
│   └── review_checklist.md
└── templates/
```

---

#### v2.0 - 真正的多 Agent 协作
**核心特性：**
- 三个独立 agent 实例（Master + Coder + Evaluator）
- 完全隔离的上下文
- 文件系统通信
- 独立的 agent 指令文件

**改进：**
- ✅ 消除确认偏差（独立 agent 实例）
- ✅ 上下文隔离（各 agent 只看到自己需要的信息）
- ✅ 支持并行派发（实验性）

**问题：**
- ⚠️ Master 权限过高（可以跳过循环、直接修改文件）
- ⚠️ 仍然是单模型（所有 agent 使用同一个 LLM）
- ⚠️ 文件过大（60KB）

**文件结构：**
```
coder-evaluator-loop/
├── SKILL.md (14.7 KB)
├── README.md (14.6 KB)
├── config.json (2.2 KB)
├── agents/
│   ├── master.md (19.3 KB)
│   ├── coder.md (3.4 KB)
│   └── evaluator.md (6.1 KB)
├── references/
│   └── review_checklist.md
├── templates/
└── examples/
```

---

#### v2.1 - 权限控制机制
**核心特性：**
- 严格的权限边界（strict/moderate/flexible）
- 审计日志（logs/audit.jsonl）
- 越权检测和处理
- 配置化的权限级别

**改进：**
- ✅ Master 不能跳过循环（strict 模式）
- ✅ Master 不能修改业务文件（strict 模式）
- ✅ Master 不能覆盖评估结果（strict 模式）
- ✅ 所有权限检查记录到审计日志

**问题：**
- ⚠️ 文件过大（60KB，存在冗余）
- ⚠️ 仍然是单模型

**文件结构：**
```
coder-evaluator-loop/
├── SKILL.md (14.7 KB)
├── README.md (14.6 KB)
├── config.json (2.2 KB)
├── agents/
│   ├── master.md (19.3 KB)
│   ├── coder.md (3.4 KB)
│   └── evaluator.md (6.1 KB)
├── references/
│   └── review_checklist.md
├── templates/
└── examples/
```

---

#### v2.2 - 精简优化
**核心特性：**
- 文件分层架构（渐进式加载）
- 精简 agents/ 指令文件
- 消除重复内容
- 目标：总大小 < 30 KB

**改进：**
- ✅ master.md 精简（19.3KB → < 5KB）
- ✅ SKILL.md 精简（14.7KB → < 8KB）
- ✅ README.md 精简（14.6KB → < 6KB）
- ✅ 详细逻辑移到 references/ 目录
- ✅ 创建版本历史文件（CHANGELOG.md）

**文件结构：**
```
coder-evaluator-loop/
├── SKILL.md (< 8 KB)
├── README.md (< 6 KB)
├── CHANGELOG.md (本文件)
├── config.json (2.2 KB)
├── agents/
│   ├── master.md (< 5 KB)
│   ├── coder.md (3.4 KB)
│   └── evaluator.md (6.1 KB)
├── references/
│   ├── review_checklist.md
│   ├── permission_rules.md (新增)
│   └── workflow_logic.md (新增)
├── templates/
└── examples/
```

---

### 计划中版本

#### v3.0 - Claude 模型切换（计划中）
**目标：** 支持不同 agent 使用不同的 Claude 模型

**计划特性：**
- 配置化的模型分配
- Master 使用 opus（最强，适合协调）
- Coder 使用 sonnet（平衡，适合编码）
- Evaluator 使用 haiku（快速，适合审查）

**配置示例：**
```json
{
  "agent_models": {
    "master": "opus",
    "coder": "sonnet",
    "evaluator": "haiku"
  }
}
```

**实现方式：**
```python
# 派发任务时指定模型
Agent({
  "subagent_type": "general-purpose",
  "model": "sonnet",  # 指定模型
  "prompt": coder_prompt
})
```

**预期效果：**
- 不同模型有不同的"思维方式"
- 进一步减少确认偏差
- 发挥各模型的优势

**技术依赖：**
- Agent 工具的 `model` 参数（已支持）
- 不需要修改框架

**预计工作量：**
- 修改 config.json：添加 agent_models 配置
- 修改 master.md：派发任务时读取模型配置
- 测试不同模型组合的效果

---

#### v4.0 - 跨提供商多模型协作（未来）
**目标：** 支持不同 agent 使用不同提供商的模型

**计划特性：**
- 支持 Claude、GPT-4、本地模型等
- 真正的异构 agent 协作
- 发挥各模型的独特优势

**配置示例：**
```json
{
  "agent_models": {
    "master": "mimo:mimo-v2.5-pro",
    "coder": "openai:gpt-4",
    "evaluator": "lmstudio:google/gemma-4-e2b"
  }
}
```

**实现方式：**
```python
# 派发任务时指定完整模型 ID
Agent({
  "subagent_type": "general-purpose",
  "model_id": "openai:gpt-4",  # 跨提供商
  "prompt": coder_prompt
})
```

**预期效果：**
- 真正消除确认偏差（不同模型有不同的偏见）
- 发挥各模型的独特优势
- 支持本地模型（隐私保护、成本控制）

**技术依赖：**
- 扩展 Agent 工具支持 `model_id` 参数
- 支持跨提供商的模型调用
- 可能需要 agent 发现机制

**预计工作量：**
- 修改 CherryStudio 框架（Agent 工具）
- 实现跨提供商的模型调用
- 测试不同模型组合的效果

---

#### v5.0 - 智能 Agent 协作（远期愿景）
**目标：** Agent 之间可以动态协商和调整

**计划特性：**
- Agent 可以根据任务复杂度选择协作策略
- 动态调整循环次数和评估标准
- Agent 之间可以请求帮助或提供建议

**示例场景：**
```
Coder: "这个功能点比较复杂，建议 Security Auditor 参与审查"
Master: "同意，派发任务给 Security Auditor"
Security Auditor: "发现 2 个高危漏洞，建议修改"
Coder: "已修复，请求 Evaluator 重新评估"
Evaluator: "通过，但建议添加更多测试用例"
```

**技术依赖：**
- Agent 间实时通信协议
- 动态 agent 发现和调度
- 智能决策引擎

**预计工作量：**
- 设计 Agent 通信协议
- 实现动态调度系统
- 开发智能决策引擎

---

## 版本对比

| 版本 | Agent 实例 | 模型 | 权限控制 | 文件大小 | 确认偏差 |
|------|-----------|------|---------|---------|---------|
| v1.0 | 1 个 | 单模型 | 无 | ~20 KB | ❌ 存在 |
| v2.0 | 3 个 | 单模型 | 无 | ~60 KB | ✅ 减少 |
| v2.1 | 3 个 | 单模型 | 严格 | ~60 KB | ✅ 减少 |
| v2.2 | 3 个 | 单模型 | 严格 | < 30 KB | ✅ 减少 |
| v3.0 | 3 个 | 多 Claude | 严格 | < 30 KB | ✅✅ 进一步减少 |
| v4.0 | 3 个 | 多提供商 | 严格 | < 30 KB | ✅✅✅ 真正消除 |
| v5.0 | N 个 | 多提供商 | 动态 | < 30 KB | ✅✅✅ 真正消除 |

---

## 开发优先级

### P0（必须做）
- ✅ v2.2 精简优化（当前）

### P1（应该做）
- ⏳ v3.0 Claude 模型切换

### P2（可以做）
- ⏳ v4.0 跨提供商多模型协作

### P3（未来做）
- ⏳ v5.0 智能 Agent 协作

---

## 技术债务

### 当前技术债务
1. **文件过大**：master.md 包含大量伪代码，应该移到 references/
2. **内容重复**：README.md 和 SKILL.md 有大量重复
3. **缺少渐进式加载**：所有内容都在一个文件中

### 已解决的技术债务
1. ✅ 角色切换问题（v2.0 解决）
2. ✅ 权限控制问题（v2.1 解决）

### 未来技术债务
1. ⏳ 单模型限制（v3.0 解决）
2. ⏳ 跨提供商限制（v4.0 解决）

---

## 贡献者

- **mimo**：架构设计、实现、文档

---

## 许可证

MIT License
