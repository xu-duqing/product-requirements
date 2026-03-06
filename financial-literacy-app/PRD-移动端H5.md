# 成长资本（Growth Capital）- 移动端 H5 产品需求文档

**版本**: v1.0
**日期**: 2026-03-06
**优先级**: P0
**平台**: 移动端 H5
**对应 PRD**: README-v2.md

---

## 文档概述

本文档是"成长资本"移动端 H5 版本的产品需求文档，基于 v2.0 产品理念，重点支持 Phase 1 引导期的实施。

**核心目标：**
- 支持孩子发起项目
- 支持家长审核项目
- 支持项目执行和验收
- 支持价值收入和账户管理

**适用场景：**
- 微信内嵌 H5
- 浏览器直接访问
- 临时轻量级使用

---

## 产品架构

### 技术栈

**前端：**
- HTML5 + CSS3
- Vue 3 + Vite（或原生 JS）
- 响应式设计（移动优先）
- 微信 JSSDK（支持微信支付、分享）

**后端：**
- Node.js + Express / Koa
- RESTful API
- JWT 认证
- WebSocket（实时推送）

**数据库：**
- PostgreSQL（主数据库）
- Redis（缓存）

**部署：**
- 云服务器（阿里云 / 腾讯云）
- Nginx 反向代理
- HTTPS

---

### 系统架构

```
┌─────────────────────────────────────────┐
│  客户端（H5）                            │
├─────────────────────────────────────────┤
│  家长端页面              孩子端页面        │
│  - 项目列表              - 我的项目        │
│  - 项目审核              - 发起项目        │
│  - 账户监控              - 价值收入        │
│  - 成长分析              - 账户管理        │
└─────────────────────────────────────────┘
            ↓ HTTPS
┌─────────────────────────────────────────┐
│  后端服务（Node.js）                     │
├─────────────────────────────────────────┤
│  API 网关                                 │
│  - 认证中间件                            │
│  - 权限控制                              │
└─────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────┐
│  业务服务层                              │
├─────────────────────────────────────────┤
│  - 用户服务              - 项目服务      │
│  - 账户服务              - 价值服务      │
│  - 成就服务              - 通知服务      │
└─────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────┐
│  数据层                                  │
├─────────────────────────────────────────┤
│  PostgreSQL              Redis           │
└─────────────────────────────────────────┘
```

---

## 功能需求

### 一、用户系统

#### 1.1 用户注册/登录

**需求：**
- 支持手机号注册
- 支持微信授权登录
- 支持家长和孩子双角色
- 支持绑定家庭关系

**接口：**
```
POST /api/auth/register
POST /api/auth/login
POST /api/auth/wechat-login
GET  /api/auth/me
```

**数据模型：**
```typescript
interface User {
  id: string;
  phone: string;
  nickname: string;
  avatar: string;
  role: 'parent' | 'child';
  familyId?: string;
  createdAt: Date;
  updatedAt: Date;
}

interface Family {
  id: string;
  parentId: string;
  childId: string;
  createdAt: Date;
}
```

---

#### 1.2 家庭关系绑定

**需求：**
- 家长可以邀请孩子加入家庭
- 孩子接受邀请后建立绑定
- 一个家庭包含一个家长和多个孩子

**接口：**
```
POST /api/family/invite
POST /api/family/accept
GET  /api/family/info
```

---

### 二、项目系统

#### 2.1 项目列表

**家长端：**
- 查看所有项目
- 筛选项目（类型、状态）
- 项目排序（时间、价值）

**孩子端：**
- 查看自己的项目
- 项目状态展示
- 项目进度显示

**接口：**
```
GET /api/projects
GET /api/projects/:id
```

**数据模型：**
```typescript
interface Project {
  id: string;
  name: string;
  description: string;
  type: 'learning' | 'innovation' | 'service' | 'investment';
  goal: string;
  budget: number;
  spent: number;
  value: number;
  status: 'pending' | 'approved' | 'in_progress' | 'completed' | 'rejected';
  deadline: Date;
  applicantId: string;
  approverId?: string;
  createdAt: Date;
  updatedAt: Date;
}
```

---

#### 2.2 项目发起（孩子端）

**需求：**
- 创建项目申请
- 填写项目信息
- 申请额外价值
- 提交审核

**界面：**
```
创建新项目
─────────────────────────────────────────
项目名称
[输入框]

项目描述
[多行文本框]

目标成果
[多行文本框]

所需资金
[数字输入]

完成时间
[日期选择器]

额外价值申请（可选）
[数字输入]

[提交申请]
```

**接口：**
```
POST /api/projects
```

**请求体：**
```json
{
  "name": "节日装饰项目",
  "description": "为家里制作节日装饰",
  "goal": "完成装饰布置，满意度 > 80%",
  "budget": 150,
  "deadline": "2026-03-15",
  "extraValue": 50
}
```

---

#### 2.3 项目审核（家长端）

**需求：**
- 查看项目申请详情
- 审核通过/拒绝
- 拒绝时提供原因
- 审核通过后注资

**接口：**
```
POST /api/projects/:id/approve
POST /api/projects/:id/reject
```

**请求体（审核通过）：**
```json
{
  "approved": true
}
```

**请求体（拒绝）：**
```json
{
  "approved": false,
  "reason": "预算过高"
}
```

---

#### 2.4 项目执行

**需求：**
- 查看项目详情
- 记录项目进度
- 上传证明材料
- 提交项目完成

**接口：**
```
POST /api/projects/:id/progress
POST /api/projects/:id/complete
POST /api/projects/:id/attachments
```

**请求体（进度更新）：**
```json
{
  "progress": 60,
  "note": "采购完成"
}
```

---

#### 2.5 项目验收（家长端）

**需求：**
- 查看项目成果
- 验收评分
- 发放价值回报

**接口：**
```
POST /api/projects/:id/verify
```

**请求体：**
```json
{
  "verified": true,
  "score": 90,
  "feedback": "完成得很好"
}
```

---

### 三、价值收入系统

#### 3.1 价值仪表盘

**需求：**
- 显示总价值收入
- 按类型分类显示
- 显示趋势图

**接口：**
```
GET /api/value/summary
GET /api/value/history
```

**数据模型：**
```typescript
interface ValueSummary {
  total: number;
  learning: number;
  innovation: number;
  service: number;
  investment: number;
}

interface ValueRecord {
  id: string;
  type: 'learning' | 'innovation' | 'service' | 'investment';
  amount: number;
  projectId: string;
  description: string;
  createdAt: Date;
}
```

---

#### 3.2 价值明细

**需求：**
- 显示价值收入历史
- 按时间排序
- 支持分页

**接口：**
```
GET /api/value/history?page=1&limit=20
```

---

### 四、账户管理系统

#### 4.1 账户余额

**需求：**
- 显示三账户余额
- 账户详情
- 账户操作历史

**接口：**
```
GET /api/accounts
GET /api/accounts/:type/history
```

**数据模型：**
```typescript
interface Account {
  type: 'spending' | 'savings' | 'investment';
  balance: number;
  totalIn: number;
  totalOut: number;
}

interface Transaction {
  id: string;
  type: 'in' | 'out';
  amount: number;
  accountType: 'spending' | 'savings' | 'investment';
  description: string;
  createdAt: Date;
}
```

---

#### 4.2 价值入账分配

**需求：**
- 收到价值时分配到不同账户
- 默认分配比例（40% + 40% + 20%）
- 支持自定义分配

**接口：**
```
POST /api/accounts/distribute
```

**请求体：**
```json
{
  "amount": 50,
  "distribution": {
    "spending": 20,
    "savings": 20,
    "investment": 10
  }
}
```

---

#### 4.3 消费

**需求：**
- 从消费账户支出
- 记录消费原因
- 生成消费记录

**接口：**
```
POST /api/accounts/spending/withdraw
```

**请求体：**
```json
{
  "amount": 30,
  "description": "购买零食"
}
```

---

#### 4.4 储蓄

**需求：**
- 存入储蓄账户
- 计算利息
- 到期赎回

**接口：**
```
POST /api/accounts/savings/deposit
POST /api/accounts/savings/withdraw
```

**请求体（存入）：**
```json
{
  "amount": 50,
  "duration": 12  // 月
}
```

---

#### 4.5 投资

**需求：**
- 从投资账户投资
- 记录投资项目
- 项目到期后分成

**接口：**
```
POST /api/accounts/investment/invest
POST /api/accounts/investment/settle
```

**请求体（投资）：**
```json
{
  "amount": 100,
  "projectId": "xxx",
  "expectedReturn": 70,
  "splitRatio": 0.7
}
```

---

### 五、成就系统

#### 5.1 成就列表

**需求：**
- 显示所有成就
- 成就进度
- 已达成标识

**接口：**
```
GET /api/achievements
```

**数据模型：**
```typescript
interface Achievement {
  id: string;
  name: string;
  description: string;
  icon: string;
  progress: number;
  target: number;
  unlocked: boolean;
  unlockedAt?: Date;
}
```

---

#### 5.2 成就统计

**需求：**
- 统计已解锁成就
- 显示成就徽章

**接口：**
```
GET /api/achievements/stats
```

---

### 六、通知系统

#### 6.1 通知列表

**需求：**
- 显示所有通知
- 按时间排序
- 未读标识

**接口：**
```
GET /api/notifications
```

**数据模型：**
```typescript
interface Notification {
  id: string;
  type: 'project_approved' | 'project_completed' | 'value_received';
  title: string;
  content: string;
  read: boolean;
  createdAt: Date;
}
```

---

#### 6.2 标记已读

**需求：**
- 标记单个通知已读
- 标记所有通知已读

**接口：**
```
POST /api/notifications/:id/read
POST /api/notifications/read-all
```

---

## 交互设计

### 一、家长端页面

#### 1.1 首页（项目列表）

```
┌─────────────────────────────────────────┐
│  < 成长资本              搜索 🔍      │
├─────────────────────────────────────────┤
│  📊 本周概览                             │
│  价值收入：560 元  项目：3 个           │
├─────────────────────────────────────────┤
│  项目列表                                │
│  ┌───────────────────────────────────┐  │
│  │ 语文80分突破      进行中  20 元   │  │
│  │ 英语100分         已完成  50 元   │  │
│  │ 数学90分          待验收  30 元   │  │
│  └───────────────────────────────────┘  │
│                   [审核新项目 +]        │
├─────────────────────────────────────────┤
│  [项目] [账户] [分析] [我的]            │
└─────────────────────────────────────────┘
```

---

#### 1.2 项目审核页

```
┌─────────────────────────────────────────┐
│  < 项目审核                              │
├─────────────────────────────────────────┤
│  项目：节日装饰项目                      │
│  申请人：小宝贝                          │
│  申请时间：2026-03-06                   │
├─────────────────────────────────────────┤
│  项目描述：                              │
│  我想为家里制作节日装饰...               │
├─────────────────────────────────────────┤
│  目标成果：                              │
│  完成节日装饰布置，满意度 > 80%         │
├─────────────────────────────────────────┤
│  所需资金：150 元                       │
│  完成时间：2026-03-15                   │
├─────────────────────────────────────────┤
│  额外价值：50 元                        │
├─────────────────────────────────────────┤
│  [拒绝] [批准 + 注资 150 元]            │
└─────────────────────────────────────────┘
```

---

#### 1.3 账户监控页

```
┌─────────────────────────────────────────┐
│  < 账户监控                              │
├─────────────────────────────────────────┤
│  💳 消费账户：  200 元  余额充足        │
│  🏦 储蓄账户：  300 元  增长中          │
│  📊 投资账户：   60 元  投资中          │
├─────────────────────────────────────────┤
│  最近记录                                │
│  03-06  语文80分突破  +20              │
│  03-06  英语100分      +50             │
│  03-05  购买零食       -30             │
│  03-03  存款利息       +10             │
└─────────────────────────────────────────┘
```

---

### 二、孩子端页面

#### 2.1 首页（我的项目）

```
┌─────────────────────────────────────────┐
│  < 我的价值收入        💰 560 元       │
├─────────────────────────────────────────┤
│  📚 学习项目  (3个)                     │
│  ┌───────────────────────────────────┐  │
│  │ 每天专注学习 2 小时               │  │
│  │ 进行中  ⏰ 2/2 小时  10 元       │  │
│  └───────────────────────────────────┘  │
├─────────────────────────────────────────┤
│  🚀 创新项目  (1个)                     │
│  ┌───────────────────────────────────┐  │
│  │ AI机器人 - 第2阶段               │  │
│  │ 进度：2/4 完成  50 元            │  │
│  └───────────────────────────────────┘  │
├─────────────────────────────────────────┤
│  [项目] [收入] [账户] [我的]            │
└─────────────────────────────────────────┘
```

---

#### 2.2 发起项目页

```
┌─────────────────────────────────────────┐
│  < 发起新项目                            │
├─────────────────────────────────────────┤
│  项目名称                                │
│  [例如：为家里做一个节日装饰            ]  │
├─────────────────────────────────────────┤
│  项目描述                                │
│  [我想为家里制作节日装饰，包括...       ]  │
├─────────────────────────────────────────┤
│  目标成果                                │
│  [完成节日装饰布置，满意度 > 80%       ]  │
├─────────────────────────────────────────┤
│  所需资金                                │
│  [150 元                                ]  │
├─────────────────────────────────────────┤
│  完成时间                                │
│  [2026-03-15 ▼                         ]  │
├─────────────────────────────────────────┤
│  额外价值申请（可选）                    │
│  [50 元                                 ]  │
├─────────────────────────────────────────┤
│           [提交申请]                     │
└─────────────────────────────────────────┘
```

---

#### 2.3 项目详情页

```
┌─────────────────────────────────────────┐
│  < 节日装饰项目                          │
├─────────────────────────────────────────┤
│  状态：进行中  🟢                       │
│  预算：150 元  已用：80 元              │
│  完成时间：2026-03-15                   │
├─────────────────────────────────────────┤
│  目标成果：                              │
│  ✅ 完成节日装饰布置                    │
│  ⬜ 家人满意度 > 80%                    │
├─────────────────────────────────────────┤
│  进度：██████░░░░░░░ 3/5               │
├─────────────────────────────────────────┤
│  任务清单：                              │
│  ✅ 采购材料       (80 元)              │
│  ✅ 设计方案                             │
│  🔄 完成布置       (进行中)             │
│  ⬜ 验收                                 │
│  ⬜ 提交报告                             │
├─────────────────────────────────────────┤
│  [采购记录] [进度更新] [完成项目]       │
└─────────────────────────────────────────┘
```

---

## 数据模型

### 数据库表设计

#### users 表
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  phone VARCHAR(20) UNIQUE NOT NULL,
  nickname VARCHAR(50),
  avatar VARCHAR(255),
  role VARCHAR(10) NOT NULL CHECK (role IN ('parent', 'child')),
  family_id INTEGER REFERENCES families(id),
  wechat_openid VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### families 表
```sql
CREATE TABLE families (
  id SERIAL PRIMARY KEY,
  parent_id INTEGER REFERENCES users(id),
  child_id INTEGER REFERENCES users(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### projects 表
```sql
CREATE TABLE projects (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  type VARCHAR(20) NOT NULL CHECK (type IN ('learning', 'innovation', 'service', 'investment')),
  goal TEXT,
  budget DECIMAL(10,2) NOT NULL,
  spent DECIMAL(10,2) DEFAULT 0,
  value DECIMAL(10,2) DEFAULT 0,
  status VARCHAR(20) NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'approved', 'in_progress', 'completed', 'rejected')),
  deadline DATE,
  applicant_id INTEGER REFERENCES users(id),
  approver_id INTEGER REFERENCES users(id),
  extra_value DECIMAL(10,2) DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### value_records 表
```sql
CREATE TABLE value_records (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  type VARCHAR(20) NOT NULL CHECK (type IN ('learning', 'innovation', 'service', 'investment')),
  amount DECIMAL(10,2) NOT NULL,
  project_id INTEGER REFERENCES projects(id),
  description TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### accounts 表
```sql
CREATE TABLE accounts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  type VARCHAR(20) NOT NULL CHECK (type IN ('spending', 'savings', 'investment')),
  balance DECIMAL(10,2) DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### transactions 表
```sql
CREATE TABLE transactions (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  account_type VARCHAR(20) NOT NULL CHECK (account_type IN ('spending', 'savings', 'investment')),
  type VARCHAR(10) NOT NULL CHECK (type IN ('in', 'out')),
  amount DECIMAL(10,2) NOT NULL,
  description TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 开发计划

### Phase 1：MVP 开发（2周）

**Week 1：**
- [x] 项目初始化
- [x] 数据库设计
- [ ] 用户系统（注册/登录）
- [ ] 家庭关系绑定

**Week 2：**
- [ ] 项目系统基础功能
- [ ] 项目列表和详情
- [ ] 项目发起和审核

---

### Phase 2：核心功能（2周）

**Week 3：**
- [ ] 项目执行和验收
- [ ] 价值收入系统
- [ ] 账户管理系统

**Week 4：**
- [ ] 成就系统
- [ ] 通知系统
- [ ] 界面优化

---

### Phase 3：测试上线（1周）

**Week 5：**
- [ ] 内部测试
- [ ] Bug 修复
- [ ] 部署上线

---

## 技术要求

### 性能要求
- 首屏加载 < 2s
- 接口响应 < 500ms
- 支持 1000 并发

### 兼容性要求
- iOS 13+
- Android 8+
- 微信浏览器
- Safari / Chrome

### 安全要求
- HTTPS 加密
- JWT 认证
- SQL 注入防护
- XSS 防护

---

## 成功指标

### 短期（1个月）
- 完成第 1 个项目
- 孩子理解项目流程
- 家长和孩子都满意

### 中期（3个月）
- 完成多个项目
- 孩子独立发起项目
- 价值收入稳定增长

### 长期（6个月）
- 建立稳定思维模式
- 孩子主动发现机会
- 可能真正创业

---

## 备注

**核心原则：做少，但做到极致。**

**技术优先级：**
- 用户体验 > 功能完整性
- 稳定性 > 新功能
- 简洁性 > 复杂性

**迭代原则：**
- 先跑通核心流程
- 再优化细节体验
- 最后扩展新功能