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
- Vue 3 + Vite
- 响应式设计（移动优先）
- 微信 JSSDK（支持微信支付、分享）
- **部署：Cloudflare Pages**

**后端：**
- **Cloudflare Workers**（边缘运行时）
- Hono / Itty-Router（轻量级路由）
- D1 Database（边缘数据库）
- KV（缓存和会话管理）
- R2（文件存储，可选）

**数据库：**
- **Cloudflare D1**（基于 SQLite 的边缘数据库）
- **Cloudflare KV**（缓存和会话）

**部署：**
- **Cloudflare Pages**（前端）
- **Cloudflare Workers**（后端 API）
- **Cloudflare D1**（数据库）
- 全自动 CI/CD

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
│  Cloudflare Pages（前端）               │
├─────────────────────────────────────────┤
│  - Vue 3 SPA                            │
│  - 全局 CDN 分发                        │
│  - 自动 HTTPS                           │
└─────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────┐
│  Cloudflare Workers（后端 API）          │
├─────────────────────────────────────────┤
│  - 边缘运行时（全球节点）                │
│  - Hono / Itty-Router                   │
│  - 认证中间件                            │
│  - 权限控制                              │
└─────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────┐
│  Cloudflare Data Layer                  │
├─────────────────────────────────────────┤
│  D1 Database            KV Cache         │
│  - 用户数据             - 会话缓存        │
│  - 项目数据             - 热点数据        │
│  - 账户数据             - 限流计数        │
│  R2 Storage（可选）                     │
│  - 文件上传                               │
└─────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────┐
│  第三方服务                              │
├─────────────────────────────────────────┤
│  - 微信支付                              │
│  - 微信登录                              │
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

### D1 数据库表设计

> Cloudflare D1 基于 SQLite，使用标准 SQL 语法。

#### users 表
```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  phone TEXT UNIQUE NOT NULL,
  nickname TEXT,
  avatar TEXT,
  role TEXT NOT NULL CHECK (role IN ('parent', 'child')),
  family_id INTEGER,
  wechat_openid TEXT UNIQUE,
  created_at TEXT DEFAULT (datetime('now')),
  updated_at TEXT DEFAULT (datetime('now')),
  FOREIGN KEY (family_id) REFERENCES families(id)
);

CREATE INDEX idx_users_phone ON users(phone);
CREATE INDEX idx_users_wechat_openid ON users(wechat_openid);
CREATE INDEX idx_users_family_id ON users(family_id);
```

#### families 表
```sql
CREATE TABLE families (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  parent_id INTEGER NOT NULL,
  child_id INTEGER NOT NULL,
  created_at TEXT DEFAULT (datetime('now')),
  FOREIGN KEY (parent_id) REFERENCES users(id),
  FOREIGN KEY (child_id) REFERENCES users(id)
);

CREATE INDEX idx_families_parent ON families(parent_id);
CREATE INDEX idx_families_child ON families(child_id);
```

#### projects 表
```sql
CREATE TABLE projects (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  description TEXT,
  type TEXT NOT NULL CHECK (type IN ('learning', 'innovation', 'service', 'investment')),
  goal TEXT,
  budget REAL NOT NULL,
  spent REAL DEFAULT 0,
  value REAL DEFAULT 0,
  status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'approved', 'in_progress', 'completed', 'rejected')),
  deadline TEXT,
  applicant_id INTEGER NOT NULL,
  approver_id INTEGER,
  extra_value REAL DEFAULT 0,
  created_at TEXT DEFAULT (datetime('now')),
  updated_at TEXT DEFAULT (datetime('now')),
  FOREIGN KEY (applicant_id) REFERENCES users(id),
  FOREIGN KEY (approver_id) REFERENCES users(id)
);

CREATE INDEX idx_projects_applicant ON projects(applicant_id);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_projects_type ON projects(type);
```

#### value_records 表
```sql
CREATE TABLE value_records (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  type TEXT NOT NULL CHECK (type IN ('learning', 'innovation', 'service', 'investment')),
  amount REAL NOT NULL,
  project_id INTEGER,
  description TEXT,
  created_at TEXT DEFAULT (datetime('now')),
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (project_id) REFERENCES projects(id)
);

CREATE INDEX idx_value_records_user ON value_records(user_id);
CREATE INDEX idx_value_records_created ON value_records(created_at);
```

#### accounts 表
```sql
CREATE TABLE accounts (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  type TEXT NOT NULL CHECK (type IN ('spending', 'savings', 'investment')),
  balance REAL DEFAULT 0,
  created_at TEXT DEFAULT (datetime('now')),
  updated_at TEXT DEFAULT (datetime('now')),
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_accounts_user ON accounts(user_id);
CREATE INDEX idx_accounts_type ON accounts(type);
```

#### transactions 表
```sql
CREATE TABLE transactions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  account_type TEXT NOT NULL CHECK (account_type IN ('spending', 'savings', 'investment')),
  type TEXT NOT NULL CHECK (type IN ('in', 'out')),
  amount REAL NOT NULL,
  description TEXT,
  created_at TEXT DEFAULT (datetime('now')),
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_transactions_user ON transactions(user_id);
CREATE INDEX idx_transactions_created ON transactions(created_at);
```

#### achievements 表
```sql
CREATE TABLE achievements (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  name TEXT NOT NULL,
  description TEXT,
  icon TEXT,
  progress INTEGER DEFAULT 0,
  target INTEGER NOT NULL,
  unlocked INTEGER DEFAULT 0,
  unlocked_at TEXT,
  created_at TEXT DEFAULT (datetime('now')),
  updated_at TEXT DEFAULT (datetime('now')),
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_achievements_user ON achievements(user_id);
```

#### notifications 表
```sql
CREATE TABLE notifications (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  type TEXT NOT NULL CHECK (type IN ('project_approved', 'project_completed', 'value_received')),
  title TEXT NOT NULL,
  content TEXT,
  read INTEGER DEFAULT 0,
  created_at TEXT DEFAULT (datetime('now')),
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_notifications_user ON notifications(user_id);
CREATE INDEX idx_notifications_read ON notifications(read);
CREATE INDEX idx_notifications_created ON notifications(created_at);
```

### D1 数据库操作

```typescript
// Cloudflare Workers 中使用 D1
export interface Env {
  DB: D1Database;
  KV: KVNamespace;
}

// 查询示例
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    // 创建用户
    await env.DB.prepare(`
      INSERT INTO users (phone, nickname, role)
      VALUES (?, ?, ?)
    `).bind('13800138000', '测试用户', 'parent').run();

    // 查询用户
    const user = await env.DB.prepare(`
      SELECT * FROM users WHERE phone = ?
    `).bind('13800138000').first();

    // 更新用户
    await env.DB.prepare(`
      UPDATE users SET nickname = ? WHERE id = ?
    `).bind('新昵称', 1).run();

    return Response.json(user);
  }
};
```

---

## Cloudflare Workers 实现

### 项目结构

```
growth-capital/
├── workers/                 # Cloudflare Workers API
│   ├── src/
│   │   ├── index.ts        # 入口文件
│   │   ├── routes/         # 路由定义
│   │   ├── middleware/     # 中间件
│   │   ├── controllers/    # 控制器
│   │   └── utils/          # 工具函数
│   ├── schema.sql          # D1 数据库表结构
│   └── wrangler.toml       # Workers 配置
├── frontend/               # Vue 3 前端
│   ├── src/
│   ├── public/
│   └── package.json
└── README.md
```

### Workers 配置（wrangler.toml）

```toml
name = "growth-capital-api"
main = "src/index.ts"
compatibility_date = "2024-01-01"

# D1 数据库绑定
[[d1_databases]]
binding = "DB"
database_name = "growth-capital-db"
database_id = "your-database-id"

# KV 缓存绑定
[[kv_namespaces]]
binding = "KV"
id = "your-kv-namespace-id"

# 环境变量
[vars]
ENVIRONMENT = "production"
JWT_SECRET = "your-jwt-secret"

# 开发环境配置
[env.development]
vars = { ENVIRONMENT = "development" }

[[env.development.d1_databases]]
binding = "DB"
database_name = "growth-capital-db-dev"
database_id = "your-dev-database-id"
```

### Workers API 实现（使用 Hono）

```typescript
// src/index.ts
import { Hono } from 'hono'
import { cors } from 'hono/cors'
import { logger } from 'hono/logger'
import { authRoutes } from './routes/auth'
import { projectRoutes } from './routes/projects'
import { accountRoutes } from './routes/accounts'

export interface Env {
  DB: D1Database;
  KV: KVNamespace;
  JWT_SECRET: string;
}

const app = new Hono<{ Bindings: Env }>()

// 中间件
app.use('*', cors())
app.use('*', logger())

// 路由
app.route('/api/auth', authRoutes)
app.route('/api/projects', projectRoutes)
app.route('/api/accounts', accountRoutes)

// 健康检查
app.get('/health', (c) => {
  return c.json({ status: 'ok', timestamp: Date.now() })
})

export default app
```

### 用户认证路由

```typescript
// src/routes/auth.ts
import { Hono } from 'hono'
import { sign, verify } from 'hono/jwt'

export const authRoutes = new Hono<{ Bindings: Env }>()

// 注册
authRoutes.post('/register', async (c) => {
  const { phone, nickname, role } = await c.req.json()

  // 检查用户是否已存在
  const existing = await c.env.DB.prepare(
    'SELECT id FROM users WHERE phone = ?'
  ).bind(phone).first()

  if (existing) {
    return c.json({ error: '用户已存在' }, 400)
  }

  // 创建用户
  const result = await c.env.DB.prepare(`
    INSERT INTO users (phone, nickname, role)
    VALUES (?, ?, ?)
  `).bind(phone, nickname, role).run()

  const userId = result.meta.last_row_id

  // 生成 JWT
  const token = await sign(
    { userId, phone, role },
    c.env.JWT_SECRET
  )

  return c.json({
    success: true,
    token,
    user: { id: userId, phone, nickname, role }
  })
})

// 登录
authRoutes.post('/login', async (c) => {
  const { phone } = await c.req.json()

  const user = await c.env.DB.prepare(
    'SELECT * FROM users WHERE phone = ?'
  ).bind(phone).first()

  if (!user) {
    return c.json({ error: '用户不存在' }, 404)
  }

  const token = await sign(
    { userId: user.id, phone: user.phone, role: user.role },
    c.env.JWT_SECRET
  )

  return c.json({
    success: true,
    token,
    user
  })
})

// 微信登录
authRoutes.post('/wechat-login', async (c) => {
  const { code } = await c.req.json()

  // 调用微信 API 获取 openid
  const wechatResponse = await fetch(
    `https://api.weixin.qq.com/sns/jscode2session?appid=${process.env.WECHAT_APPID}&secret=${process.env.WECHAT_SECRET}&js_code=${code}&grant_type=authorization_code`
  )
  const wechatData = await wechatResponse.json()

  if (!wechatData.openid) {
    return c.json({ error: '微信授权失败' }, 400)
  }

  // 查找或创建用户
  let user = await c.env.DB.prepare(
    'SELECT * FROM users WHERE wechat_openid = ?'
  ).bind(wechatData.openid).first()

  if (!user) {
    const result = await c.env.DB.prepare(`
      INSERT INTO users (wechat_openid, nickname, role)
      VALUES (?, ?, ?)
    `).bind(wechatData.openid, '微信用户', 'child').run()

    user = {
      id: result.meta.last_row_id,
      wechat_openid: wechatData.openid,
      nickname: '微信用户',
      role: 'child'
    }
  }

  const token = await sign(
    { userId: user.id, role: user.role },
    c.env.JWT_SECRET
  )

  return c.json({
    success: true,
    token,
    user
  })
})

// 认证中间件
export const authMiddleware = async (c: any, next: any) => {
  const authHeader = c.req.header('Authorization')

  if (!authHeader) {
    return c.json({ error: '未授权' }, 401)
  }

  const token = authHeader.replace('Bearer ', '')

  try {
    const payload = await verify(token, c.env.JWT_SECRET)
    c.set('user', payload)
    await next()
  } catch (error) {
    return c.json({ error: '无效的 token' }, 401)
  }
}
```

### 项目管理路由

```typescript
// src/routes/projects.ts
import { Hono } from 'hono'
import { authMiddleware } from '../routes/auth'

export const projectRoutes = new Hono<{ Bindings: Env }>()

// 所有路由需要认证
projectRoutes.use('*', authMiddleware)

// 获取项目列表
projectRoutes.get('/', async (c) => {
  const user = c.get('user')
  const { type, status } = c.req.query()

  let query = 'SELECT * FROM projects WHERE applicant_id = ?'
  const params = [user.userId]

  if (type) {
    query += ' AND type = ?'
    params.push(type)
  }

  if (status) {
    query += ' AND status = ?'
    params.push(status)
  }

  query += ' ORDER BY created_at DESC'

  const projects = await c.env.DB.prepare(query)
    .bind(...params)
    .all()

  return c.json({ success: true, projects: projects.results })
})

// 创建项目
projectRoutes.post('/', async (c) => {
  const user = c.get('user')
  const { name, description, type, goal, budget, deadline, extraValue } = await c.req.json()

  const result = await c.env.DB.prepare(`
    INSERT INTO projects (name, description, type, goal, budget, deadline, applicant_id, extra_value)
    VALUES (?, ?, ?, ?, ?, ?, ?, ?)
  `).bind(name, description, type, goal, budget, deadline, user.userId, extraValue || 0).run()

  const projectId = result.meta.last_row_id

  return c.json({
    success: true,
    project: { id: projectId, name, status: 'pending' }
  })
})

// 审核项目
projectRoutes.post('/:id/approve', async (c) => {
  const user = c.get('user')
  const projectId = c.req.param('id')
  const { approved, reason } = await c.req.json()

  if (user.role !== 'parent') {
    return c.json({ error: '只有家长可以审核项目' }, 403)
  }

  if (approved) {
    await c.env.DB.prepare(`
      UPDATE projects
      SET status = 'approved', approver_id = ?, updated_at = datetime('now')
      WHERE id = ?
    `).bind(user.userId, projectId).run()
  } else {
    await c.env.DB.prepare(`
      UPDATE projects
      SET status = 'rejected', updated_at = datetime('now')
      WHERE id = ?
    `).bind(projectId).run()

    // 记录拒绝原因
    await c.env.DB.prepare(`
      INSERT INTO project_rejections (project_id, reason)
      VALUES (?, ?)
    `).bind(projectId, reason).run()
  }

  return c.json({ success: true })
})

// 完成项目
projectRoutes.post('/:id/complete', async (c) => {
  const user = c.get('user')
  const projectId = c.req.param('id')
  const { attachments, note } = await c.req.json()

  // 更新项目状态
  await c.env.DB.prepare(`
    UPDATE projects
    SET status = 'completed', updated_at = datetime('now')
    WHERE id = ? AND applicant_id = ?
  `).bind(projectId, user.userId).run()

  return c.json({ success: true })
})

// 获取项目详情
projectRoutes.get('/:id', async (c) => {
  const projectId = c.req.param('id')

  const project = await c.env.DB.prepare(`
    SELECT * FROM projects WHERE id = ?
  `).bind(projectId).first()

  if (!project) {
    return c.json({ error: '项目不存在' }, 404)
  }

  return c.json({ success: true, project })
})
```

### 账户管理路由

```typescript
// src/routes/accounts.ts
import { Hono } from 'hono'
import { authMiddleware } from '../routes/auth'

export const accountRoutes = new Hono<{ Bindings: Env }>()

accountRoutes.use('*', authMiddleware)

// 获取账户余额
accountRoutes.get('/', async (c) => {
  const user = c.get('user')

  const accounts = await c.env.DB.prepare(`
    SELECT * FROM accounts WHERE user_id = ?
  `).bind(user.userId).all()

  return c.json({ success: true, accounts: accounts.results })
})

// 价值入账分配
accountRoutes.post('/distribute', async (c) => {
  const user = c.get('user')
  const { amount, distribution } = await c.req.json()

  // 更新消费账户
  await c.env.DB.prepare(`
    UPDATE accounts
    SET balance = balance + ?, updated_at = datetime('now')
    WHERE user_id = ? AND type = 'spending'
  `).bind(distribution.spending, user.userId).run()

  // 更新储蓄账户
  await c.env.DB.prepare(`
    UPDATE accounts
    SET balance = balance + ?, updated_at = datetime('now')
    WHERE user_id = ? AND type = 'savings'
  `).bind(distribution.savings, user.userId).run()

  // 更新投资账户
  await c.env.DB.prepare(`
    UPDATE accounts
    SET balance = balance + ?, updated_at = datetime('now')
    WHERE user_id = ? AND type = 'investment'
  `).bind(distribution.investment, user.userId).run()

  return c.json({ success: true })
})

// 消费
accountRoutes.post('/spending/withdraw', async (c) => {
  const user = c.get('user')
  const { amount, description } = await c.req.json()

  // 检查余额
  const account = await c.env.DB.prepare(`
    SELECT balance FROM accounts WHERE user_id = ? AND type = 'spending'
  `).bind(user.userId).first()

  if (!account || account.balance < amount) {
    return c.json({ error: '余额不足' }, 400)
  }

  // 扣除余额
  await c.env.DB.prepare(`
    UPDATE accounts
    SET balance = balance - ?, updated_at = datetime('now')
    WHERE user_id = ? AND type = 'spending'
  `).bind(amount, user.userId).run()

  // 记录交易
  await c.env.DB.prepare(`
    INSERT INTO transactions (user_id, account_type, type, amount, description)
    VALUES (?, 'spending', 'out', ?, ?)
  `).bind(user.userId, amount, description).run()

  return c.json({ success: true })
})
```

### D1 数据库初始化

```bash
# 创建 D1 数据库
npx wrangler d1 create growth-capital-db

# 初始化表结构
npx wrangler d1 execute growth-capital-db --local --file=./workers/schema.sql

# 本地开发
npx wrangler dev

# 部署
npx wrangler deploy
```

### Cloudflare Pages 部署配置

```toml
# frontend/wrangler.toml
name = "growth-capital-frontend"
compatibility_date = "2024-01-01"

[vars]
API_URL = "https://growth-capital-api.workers.dev"
```

```bash
# 部署前端到 Cloudflare Pages
cd frontend
npm run build
npx wrangler pages deploy dist
```

## 开发计划

### Phase 1：MVP 开发（2周）

**Week 1：**
- [x] Cloudflare Workers 项目初始化
- [x] D1 数据库设计和初始化
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
- [ ] 本地测试（wrangler dev）
- [ ] Cloudflare 预览环境测试
- [ ] 正式部署
- [ ] 监控和日志

## 技术要求

### 性能要求
- 首屏加载 < 2s（Cloudflare CDN）
- API 响应 < 500ms（边缘计算）
- 支持全球访问（300+ 节点）
- D1 查询 < 100ms

### 免费额度
- Workers: 100,000 请求/天
- Pages: 500 构建次数/月
- D1: 5GB 存储 / 2500 万次读取/月
- KV: 100,000 次读取/天

### 兼容性要求
- iOS 13+
- Android 8+
- 微信浏览器
- Safari / Chrome

### 安全要求
- HTTPS（自动）
- JWT 认证
- CORS 配置
- 输入验证
- SQL 注入防护（D1 参数化查询）

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

## 备注

**核心原则：做少，但做到极致。**

**Cloudflare 优势：**
- 全球 CDN，极致性能
- 边缘计算，低延迟
- 免费额度充足
- 自动 HTTPS
- 零运维成本
- 无限扩展能力

**技术优先级：**
- 用户体验 > 功能完整性
- 稳定性 > 新功能
- 简洁性 > 复杂性

**迭代原则：**
- 先跑通核心流程
- 再优化细节体验
- 最后扩展新功能