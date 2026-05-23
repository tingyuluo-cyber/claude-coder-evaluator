# 使用示例

## 示例场景：开发一个用户认证模块

### 用户需求
"帮我开发一个用户认证模块，支持注册、登录、密码重置功能，使用 JWT token"

---

## 阶段 1：Coder 生成开发清单

**Coder 输出：**

```markdown
# 开发清单

## 需求概述
开发用户认证模块，包含注册、登录、密码重置功能，使用 JWT token 进行身份验证。

## 功能点列表

### 功能点 1: 用户注册
- **描述**: 实现用户注册功能，包括邮箱验证、密码加密存储
- **涉及文件**: 
  - src/auth/register.ts
  - src/models/user.ts
  - src/utils/validation.ts
- **预计复杂度**: 中
- **依赖**: 无
- **验收标准**:
  - [ ] 用户可以通过邮箱和密码注册
  - [ ] 密码使用 bcrypt 加密存储
  - [ ] 邮箱格式验证
  - [ ] 密码强度验证（至少 8 位，包含大小写和数字）
  - [ ] 防止重复注册

### 功能点 2: 用户登录
- **描述**: 实现用户登录功能，生成 JWT token
- **涉及文件**:
  - src/auth/login.ts
  - src/middleware/auth.ts
  - src/utils/jwt.ts
- **预计复杂度**: 中
- **依赖**: 功能点 1
- **验收标准**:
  - [ ] 用户可以通过邮箱和密码登录
  - [ ] 登录成功返回 JWT token
  - [ ] token 有效期 24 小时
  - [ ] 支持 token 刷新
  - [ ] 记录登录日志

### 功能点 3: 密码重置
- **描述**: 实现密码重置功能，通过邮件发送重置链接
- **涉及文件**:
  - src/auth/password-reset.ts
  - src/services/email.ts
  - src/utils/crypto.ts
- **预计复杂度**: 高
- **依赖**: 功能点 1
- **验收标准**:
  - [ ] 用户可以请求密码重置
  - [ ] 发送重置链接到用户邮箱
  - [ ] 重置链接有效期 1 小时
  - [ ] 用户可以通过链接重置密码
  - [ ] 重置后旧 token 失效

## 开发顺序建议

1. **功能点 1: 用户注册** - 理由: 基础功能，其他功能依赖用户模型
2. **功能点 2: 用户登录** - 理由: 依赖用户模型，是核心功能
3. **功能点 3: 密码重置** - 理由: 相对独立，可以在核心功能完成后实现

## 技术选型

- **框架**: Express.js + TypeScript
- **数据库**: PostgreSQL + Prisma ORM
- **认证**: JWT (jsonwebtoken)
- **加密**: bcrypt
- **邮件**: Nodemailer

## 风险识别

1. **邮件发送可靠性**
   - 影响: 密码重置功能可能无法正常工作
   - 应对方案: 使用可靠的邮件服务（如 SendGrid），添加重试机制

2. **JWT token 安全性**
   - 影响: token 泄露可能导致账户被盗
   - 应对方案: 使用 HTTPS，设置合理的过期时间，支持 token 刷新

## 预计工作量

- 功能点 1: 4 小时
- 功能点 2: 3 小时
- 功能点 3: 5 小时
- **总计**: 12 小时
```

---

## 阶段 2：Evaluator 评估清单

**Evaluator 输出：**

```markdown
# 清单评估报告

## 总体评价
**需要修改**

清单整体结构清晰，功能点划分合理，但存在一些需要改进的地方。

## 详细评估

### 需求覆盖度
- **评分**: 8/10
- **状态**: ✓ 通过
- **说明**: 覆盖了主要功能需求，但缺少登出功能和 token 黑名单机制

### 任务拆分合理性
- **评分**: 9/10
- **状态**: ✓ 通过
- **说明**: 功能点粒度合适，职责单一

### 依赖关系
- **评分**: 9/10
- **状态**: ✓ 通过
- **说明**: 依赖关系清晰，顺序合理

### 技术可行性
- **评分**: 8/10
- **状态**: ✓ 通过
- **说明**: 技术选型合理，但需要补充环境配置说明

### 风险识别
- **评分**: 7/10
- **状态**: ✗ 未通过
- **说明**: 遗漏了数据库连接失败、并发注册等风险

## 问题列表

### 中严重程度问题

1. **缺少登出功能**
   - **位置**: 功能点列表
   - **描述**: 需求中提到的认证模块应包含登出功能
   - **影响**: 用户无法主动失效 token
   - **建议**: 添加功能点 4: 用户登出，实现 token 黑名单

2. **缺少环境配置说明**
   - **位置**: 技术选型部分
   - **描述**: 没有说明需要哪些环境变量（数据库连接、JWT 密钥等）
   - **影响**: 开发者可能不清楚如何配置
   - **建议**: 添加环境配置章节

### 低严重程度问题

1. **风险识别不完整**
   - **位置**: 风险识别部分
   - **描述**: 缺少数据库连接失败、并发注册等常见风险
   - **影响**: 可能导致开发中遇到未预见的问题
   - **建议**: 补充更多风险点

## 优点

- 功能点划分清晰，职责单一
- 技术选型合理，使用了成熟的技术栈
- 验收标准明确，便于测试

## 改进建议

### 必须修改
1. 添加功能点 4: 用户登出（包含 token 黑名单）
2. 添加环境配置说明

### 建议修改
1. 补充风险识别（数据库连接失败、并发注册等）
2. 考虑添加 rate limiting 防止暴力破解

## 下一步行动

- [ ] 添加用户登出功能点
- [ ] 补充环境配置说明
- [ ] 完善风险识别
- [ ] 重新提交评估
```

---

## 阶段 3：Coder 修改清单并重新评估

**Coder 修改后重新提交：**

（添加了功能点 4、环境配置说明、完善了风险识别）

**Evaluator 第二次评估：**

```markdown
# 清单评估报告

## 总体评价
**通过**

清单已完善，满足所有评估标准。

## 详细评估

### 需求覆盖度
- **评分**: 10/10
- **状态**: ✓ 通过
- **说明**: 覆盖所有功能需求，包括登出和 token 管理

### 任务拆分合理性
- **评分**: 9/10
- **状态**: ✓ 通过
- **说明**: 功能点划分合理

### 依赖关系
- **评分**: 9/10
- **状态**: ✓ 通过
- **说明**: 依赖关系清晰

### 技术可行性
- **评分**: 9/10
- **状态**: ✓ 通过
- **说明**: 技术选型合理，环境配置清晰

### 风险识别
- **评分**: 9/10
- **状态**: ✓ 通过
- **说明**: 风险识别完整，有应对方案

## 问题列表

无

## 优点

- 功能完整，覆盖所有需求
- 环境配置清晰
- 风险识别全面

## 结论

清单通过评估，可以开始开发。
```

---

## 阶段 4：功能点开发循环（以功能点 1 为例）

### Coder 实现功能点 1

**Coder 输出：**

```markdown
# 功能点 1 实现: 用户注册

## 实现概述
已实现用户注册功能，包括邮箱验证、密码加密存储、防重复注册等。

## 修改的文件

### src/models/user.ts
```typescript
import { PrismaClient } from '@prisma/client';
import bcrypt from 'bcrypt';

const prisma = new PrismaClient();

export interface CreateUserInput {
  email: string;
  password: string;
  name?: string;
}

export async function createUser(input: CreateUserInput) {
  // 检查邮箱是否已注册
  const existingUser = await prisma.user.findUnique({
    where: { email: input.email }
  });
  
  if (existingUser) {
    throw new Error('Email already registered');
  }
  
  // 加密密码
  const saltRounds = 10;
  const hashedPassword = await bcrypt.hash(input.password, saltRounds);
  
  // 创建用户
  const user = await prisma.user.create({
    data: {
      email: input.email,
      password: hashedPassword,
      name: input.name
    }
  });
  
  return { id: user.id, email: user.email, name: user.name };
}
```

### src/utils/validation.ts
```typescript
export function validateEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

export function validatePassword(password: string): { valid: boolean; message?: string } {
  if (password.length < 8) {
    return { valid: false, message: 'Password must be at least 8 characters' };
  }
  
  if (!/[a-z]/.test(password)) {
    return { valid: false, message: 'Password must contain lowercase letter' };
  }
  
  if (!/[A-Z]/.test(password)) {
    return { valid: false, message: 'Password must contain uppercase letter' };
  }
  
  if (!/[0-9]/.test(password)) {
    return { valid: false, message: 'Password must contain number' };
  }
  
  return { valid: true };
}
```

### src/auth/register.ts
```typescript
import { Router, Request, Response } from 'express';
import { createUser, CreateUserInput } from '../models/user';
import { validateEmail, validatePassword } from '../utils/validation';

const router = Router();

router.post('/register', async (req: Request, res: Response) => {
  try {
    const { email, password, name }: CreateUserInput = req.body;
    
    // 验证邮箱格式
    if (!validateEmail(email)) {
      return res.status(400).json({ error: 'Invalid email format' });
    }
    
    // 验证密码强度
    const passwordValidation = validatePassword(password);
    if (!passwordValidation.valid) {
      return res.status(400).json({ error: passwordValidation.message });
    }
    
    // 创建用户
    const user = await createUser({ email, password, name });
    
    res.status(201).json({ message: 'User registered successfully', user });
  } catch (error) {
    if (error.message === 'Email already registered') {
      return res.status(409).json({ error: error.message });
    }
    res.status(500).json({ error: 'Internal server error' });
  }
});

export default router;
```

## 测试情况

已编写单元测试，覆盖：
- 正常注册流程
- 重复邮箱注册
- 无效邮箱格式
- 弱密码
- 密码加密验证

## 验收标准完成情况

- [x] 用户可以通过邮箱和密码注册
- [x] 密码使用 bcrypt 加密存储
- [x] 邮箱格式验证
- [x] 密码强度验证（至少 8 位，包含大小写和数字）
- [x] 防止重复注册
```

---

### Evaluator 评估功能点 1

**Evaluator 输出：**

```markdown
# 功能点评估报告: 用户注册

## 实现状态
**需要修改**

功能基本实现，但存在一些代码质量问题。

## 检查结果

### 功能正确性
- [✓] 核心功能实现
- [✓] 边界条件处理
- [✓] 错误处理

### 代码质量
- [✓] 命名规范
- [✗] 代码结构
- [✓] 注释完整性
- [✓] 性能考虑

### 需求覆盖
- [✓] 验收标准 1: 用户可以通过邮箱和密码注册
- [✓] 验收标准 2: 密码使用 bcrypt 加密存储
- [✓] 验收标准 3: 邮箱格式验证
- [✓] 验收标准 4: 密码强度验证
- [✓] 验收标准 5: 防止重复注册

## 问题列表

### 中严重程度问题

1. **数据库连接未关闭**
   - **位置**: src/models/user.ts:4
   - **描述**: PrismaClient 实例在模块级别创建，但没有关闭逻辑
   - **影响**: 可能导致数据库连接泄漏
   - **建议**: 添加连接池管理或在应用关闭时关闭连接

2. **错误处理不够精细**
   - **位置**: src/auth/register.ts:25
   - **描述**: 捕获所有错误并返回 500，没有区分不同类型的错误
   - **影响**: 难以调试问题
   - **建议**: 添加更具体的错误类型和日志

### 低严重程度问题

1. **魔法数字**
   - **位置**: src/models/user.ts:15
   - **描述**: saltRounds = 10 是魔法数字
   - **影响**: 代码可读性
   - **建议**: 提取为常量

## 改进建议

### 必须修改
1. 添加数据库连接管理
2. 改进错误处理

### 建议修改
1. 提取魔法数字为常量
2. 添加请求日志

## 测试建议

- 测试并发注册同一邮箱
- 测试数据库连接失败情况
- 测试超长输入

## 结论

需要修改后重新评估。
```

---

## 阶段 5：Coder 修改并重新评估

**Coder 修改后重新提交，Evaluator 通过：**

```markdown
# 功能点评估报告: 用户注册

## 实现状态
**通过**

已修复所有问题，代码质量良好。

## 检查结果

### 功能正确性
- [✓] 核心功能实现
- [✓] 边界条件处理
- [✓] 错误处理

### 代码质量
- [✓] 命名规范
- [✓] 代码结构
- [✓] 注释完整性
- [✓] 性能考虑

### 需求覆盖
- [✓] 所有验收标准通过

## 问题列表

无

## 结论

功能点 1 通过评估，可以进入下一个功能点。
```

---

## 阶段 6：继续开发其他功能点

重复阶段 4-5，直到所有功能点完成。

---

## 阶段 7：生成完成报告

**Coder 输出完成报告**（使用 completion_report.md 模板）

---

## 关键要点

1. **双层循环**：清单层 + 功能点层，确保整体和细节都经过评估
2. **结构化反馈**：Evaluator 使用标准化的评估报告格式
3. **迭代改进**：根据反馈持续改进，直到通过
4. **质量保证**：每个环节都有明确的验收标准
5. **文档完整**：所有决策和修改都有记录
