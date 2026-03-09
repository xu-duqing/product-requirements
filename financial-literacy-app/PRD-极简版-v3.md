# 成长资本（Growth Capital）v3.0 - 极简版产品需求文档

**版本**: v3.0
**日期**: 2026-03-09
**优先级**: P0
**平台**: 移动端 H5

---

## 核心理念

**极简主义 - 做少，但做到极致**

不是模拟企业管理，而是简化家庭互动。

**核心关系：** 目标 → 完成 → 确认 → 记录

---

## 版本对比

### v2.0（复杂版）
- 流程：5步（发起→审核→执行→验收→发放）
- 页面：25个
- 功能：项目管理、账户系统、价值分配、成就系统...

### v3.0（极简版）
- 流程：3步（设置目标→完成→确认）
- 页面：8个
- 功能：目标、确认、记录、统计

---

## 产品定位

**产品名称**: 成长资本（Growth Capital）- 极简版

**核心理念**: **记录确认，不是审批管理**

**目标用户**:
- 家长端：父母
- 孩子端：6-12岁儿童

**核心价值**:
- 家长设置目标，孩子主动完成
- 一键确认，快速记录
- 轻松查看成长轨迹

---

## 核心流程

### 原流程（v2.0）- 复杂
```
孩子发起项目 → 家长审核 → 孩子执行 → 家长验收 → 发放价值
（5步，每步都要操作）
```

### 新流程（v3.0）- 极简
```
家长设置目标 → 孩子完成 → 家长确认 → 自动记录
（3步，聚焦核心）
```

---

## 功能需求

### 一、目标系统

#### 1.1 目标列表

**需求：**
- 显示所有目标
- 区分进行中/已完成
- 显示目标价值和进度

**接口：**
```
GET /api/goals
```

**数据模型：**
```typescript
interface Goal {
  id: string;
  title: string;
  description: string;
  value: number;
  status: 'active' | 'completed';
  createdAt: Date;
  completedAt?: Date;
}
```

---

#### 1.2 添加目标

**家长端：**
- 快速添加目标
- 填写标题、描述、价值
- 直接添加，不需要审核

**接口：**
```
POST /api/goals
```

**请求体：**
```json
{
  "title": "语文80分突破",
  "description": "语文考试达到80分",
  "value": 20
}
```

---

#### 1.3 完成目标

**孩子端：**
- 选择已完成的目标
- 点击完成
- 进入家长确认队列

**接口：**
```
POST /api/goals/:id/complete
```

---

### 二、确认系统

#### 2.1 确认列表

**家长端：**
- 显示待确认的目标
- 一键确认或拒绝
- 可以添加评价

**接口：**
```
GET /api/goals/pending
POST /api/goals/:id/confirm
POST /api/goals/:id/reject
```

**请求体（确认）：**
```json
{
  "confirmed": true,
  "comment": "做得很好！"
}
```

---

### 三、记录系统

#### 3.1 记录列表

**需求：**
- 显示所有价值记录
- 按时间排序
- 区分收入/支出

**接口：**
```
GET /api/records
```

**数据模型：**
```typescript
interface Record {
  id: string;
  type: 'income' | 'expense';
  amount: number;
  description: string;
  goalId?: string;
  createdAt: Date;
}
```

---

#### 3.2 添加支出

**需求：**
- 记录孩子的消费
- 简单描述和金额

**接口：**
```
POST /api/records/expense
```

**请求体：**
```json
{
  "amount": 30,
  "description": "购买零食"
}
```

---

### 四、统计系统

#### 4.1 简单统计

**需求：**
- 总收入、总支出、余额
- 本周/本月数据
- 简单的图表

**接口：**
```
GET /api/stats
```

**数据模型：**
```typescript
interface Stats {
  totalIncome: number;
  totalExpense: number;
  balance: number;
  weeklyIncome: number;
  weeklyExpense: number;
  monthlyIncome: number;
  monthlyExpense: number;
}
```

---

## 页面设计

### 家长端页面（5个）

#### 1. 首页 - 目标列表

```
┌─────────────────────────────────────────┐
│  < 目标列表               + 添加       │
├─────────────────────────────────────────┤
│  📊 总余额：530 元                       │
│  本周收入：560 元  支出：30 元          │
├─────────────────────────────────────────┤
│  进行中                                  │
│  ┌───────────────────────────────────┐  │
│  │ 语文80分突破           20 元      │  │
│  │ 英语100分             50 元      │  │
│  └───────────────────────────────────┘  │
├─────────────────────────────────────────┤
│  已完成                                  │
│  ┌───────────────────────────────────┐  │
│  ✓ 数学90分            30 元 ✓      │  │
│  └───────────────────────────────────┘  │
├─────────────────────────────────────────┤
│  [目标] [确认] [记录] [统计] [我的]      │
└─────────────────────────────────────────┘
```

---

#### 2. 确认列表

```
┌─────────────────────────────────────────┐
│  < 确认列表                              │
├─────────────────────────────────────────┤
│  待确认 (2)                              │
│  ┌───────────────────────────────────┐  │
│  │ 语文80分突破           20 元      │  │
│  │ 3分钟前                         │  │
│  │         [确认] [拒绝]              │  │
│  ├───────────────────────────────────┤  │
│  │ 英语100分             50 元      │  │
│  │ 10分钟前                        │  │
│  │         [确认] [拒绝]              │  │
│  └───────────────────────────────────┘  │
├─────────────────────────────────────────┤
│  [目标] [确认] [记录] [统计] [我的]      │
└─────────────────────────────────────────┘
```

---

#### 3. 记录列表

```
┌─────────────────────────────────────────┐
│  < 记录列表                              │
├─────────────────────────────────────────┤
│  03-09  +20  语文80分突破               │
│  03-09  +50  英语100分                  │
│  03-08  +30  数学90分                   │
│  03-08  -30  购买零食                   │
│  03-07  +10  存款利息                   │
├─────────────────────────────────────────┤
│           + 添加支出                    │
├─────────────────────────────────────────┤
│  [目标] [确认] [记录] [统计] [我的]      │
└─────────────────────────────────────────┘
```

---

#### 4. 统计

```
┌─────────────────────────────────────────┐
│  < 统计                                  │
├─────────────────────────────────────────┤
│  💰 总余额：530 元                       │
├─────────────────────────────────────────┤
│  总收入：560 元                          │
│  总支出：30 元                           │
├─────────────────────────────────────────┤
│  本周收入：560 元                        │
│  本周支出：30 元                         │
├─────────────────────────────────────────┤
│  本月收入：1200 元                       │
│  本月支出：100 元                        │
├─────────────────────────────────────────┤
│  [目标] [确认] [记录] [统计] [我的]      │
└─────────────────────────────────────────┘
```

---

#### 5. 我的

```
┌─────────────────────────────────────────┐
│  < 我的                                  │
├─────────────────────────────────────────┤
│  👤 家长账号                             │
│  手机：138****8000                       │
├─────────────────────────────────────────┤
│  👶 孩子：小宝贝                         │
├─────────────────────────────────────────┤
│  ⚙️  设置                               │
│  编辑昵称                               │
│  更换手机号                             │
├─────────────────────────────────────────┤
│  📋  使用说明                           │
│  关于我们                               │
├─────────────────────────────────────────┤
│  🚪 退出登录                            │
├─────────────────────────────────────────┤
│  [目标] [确认] [记录] [统计] [我的]      │
└─────────────────────────────────────────┘
```

---

### 孩子端页面（3个）

#### 1. 首页 - 目标列表

```
┌─────────────────────────────────────────┐
│  < 我的目标              我的余额:530元 │
├─────────────────────────────────────────┤
│  进行中                                  │
│  ┌───────────────────────────────────┐  │
│  │ 语文80分突破           20 元      │  │
│  │                     [完成 ✓]       │  │
│  ├───────────────────────────────────┤  │
│  │ 英语100分             50 元      │  │
│  │                     [完成 ✓]       │  │
│  └───────────────────────────────────┘  │
├─────────────────────────────────────────┤
│  已完成                                  │
│  ┌───────────────────────────────────┐  │
│  ✓ 数学90分            30 元 ✓      │  │
│  └───────────────────────────────────┘  │
├─────────────────────────────────────────┤
│  [目标] [记录] [我的]                   │
└─────────────────────────────────────────┘
```

---

#### 2. 记录列表

```
┌─────────────────────────────────────────┐
│  < 我的记录                              │
├─────────────────────────────────────────┤
│  我的余额：530 元                        │
├─────────────────────────────────────────┤
│  03-09  +20  语文80分突破               │
│  03-09  +50  英语100分                  │
│  03-08  +30  数学90分                   │
│  03-08  -30  购买零食                   │
│  03-07  +10  存款利息                   │
├─────────────────────────────────────────┤
│  [目标] [记录] [我的]                   │
└─────────────────────────────────────────┘
```

---

#### 3. 我的

```
┌─────────────────────────────────────────┐
│  < 我的                                  │
├─────────────────────────────────────────┤
│  👶 小宝贝                               │
│  我的余额：530 元                        │
├─────────────────────────────────────────┤
│  ⚙️  设置                               │
│  编辑昵称                               │
├─────────────────────────────────────────┤
│  📋  使用说明                           │
│  关于我们                               │
├─────────────────────────────────────────┤
│  [目标] [记录] [我的]                   │
└─────────────────────────────────────────┘
```

---

## 技术架构

### 技术栈

**前端：**
- Vue 3 + Vite
- Tailwind CSS
- Cloudflare Pages

**后端：**
- Cloudflare Workers
- Hono
- D1 Database

**数据库：**
- D1（SQLite）

---

### D1 数据库表设计

#### goals 表
```sql
CREATE TABLE goals (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT NOT NULL,
  description TEXT,
  value REAL NOT NULL,
  status TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'completed')),
  created_at TEXT DEFAULT (datetime('now')),
  completed_at TEXT,
  confirmed INTEGER DEFAULT 0,
  comment TEXT
);

CREATE INDEX idx_goals_status ON goals(status);
```

#### records 表
```sql
CREATE TABLE records (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  type TEXT NOT NULL CHECK (type IN ('income', 'expense')),
  amount REAL NOT NULL,
  description TEXT,
  goal_id INTEGER,
  created_at TEXT DEFAULT (datetime('now')),
  FOREIGN KEY (goal_id) REFERENCES goals(id)
);

CREATE INDEX idx_records_type ON records(type);
CREATE INDEX idx_records_created ON records(created_at);
```

---

## Cloudflare Workers 实现

### 路由实现

```typescript
// src/index.ts
import { Hono } from 'hono'
import { cors } from 'hono/cors'
import { goalRoutes } from './routes/goals'
import { recordRoutes } from './routes/records'
import { statsRoutes } from './routes/stats'

export interface Env {
  DB: D1Database;
  JWT_SECRET: string;
}

const app = new Hono<{ Bindings: Env }>()

app.use('*', cors())

app.route('/api/goals', goalRoutes)
app.route('/api/records', recordRoutes)
app.route('/api/stats', statsRoutes)

export default app
```

---

### 目标路由

```typescript
// src/routes/goals.ts
import { Hono } from 'hono'
import { authMiddleware } from './auth'

export const goalRoutes = new Hono<{ Bindings: Env }>()

goalRoutes.use('*', authMiddleware)

// 获取目标列表
goalRoutes.get('/', async (c) => {
  const { status } = c.req.query()

  let query = 'SELECT * FROM goals'
  if (status) {
    query += ` WHERE status = '${status}'`
  }
  query += ' ORDER BY created_at DESC'

  const goals = await c.env.DB.prepare(query).all()

  return c.json({ success: true, goals: goals.results })
})

// 添加目标（家长）
goalRoutes.post('/', async (c) => {
  const { title, description, value } = await c.req.json()

  const result = await c.env.DB.prepare(`
    INSERT INTO goals (title, description, value)
    VALUES (?, ?, ?)
  `).bind(title, description, value).run()

  return c.json({ success: true, id: result.meta.last_row_id })
})

// 完成目标（孩子）
goalRoutes.post('/:id/complete', async (c) => {
  const goalId = c.req.param('id')

  await c.env.DB.prepare(`
    UPDATE goals
    SET status = 'completed', completed_at = datetime('now')
    WHERE id = ?
  `).bind(goalId).run()

  return c.json({ success: true })
})

// 确认目标（家长）
goalRoutes.post('/:id/confirm', async (c) => {
  const goalId = c.req.param('id')
  const { confirmed, comment } = await c.req.json()

  if (confirmed) {
    // 获取目标信息
    const goal = await c.env.DB.prepare(
      'SELECT * FROM goals WHERE id = ?'
    ).bind(goalId).first()

    // 更新目标
    await c.env.DB.prepare(`
      UPDATE goals
      SET confirmed = 1, comment = ?
      WHERE id = ?
    `).bind(comment || '', goalId).run()

    // 创建收入记录
    await c.env.DB.prepare(`
      INSERT INTO records (type, amount, description, goal_id)
      VALUES ('income', ?, ?, ?)
    `).bind(goal.value, goal.title, goalId).run()
  } else {
    // 拒绝，恢复目标状态
    await c.env.DB.prepare(`
      UPDATE goals
      SET status = 'active', completed_at = NULL
      WHERE id = ?
    `).bind(goalId).run()
  }

  return c.json({ success: true })
})

// 获取待确认目标
goalRoutes.get('/pending', async (c) => {
  const goals = await c.env.DB.prepare(`
    SELECT * FROM goals
    WHERE status = 'completed' AND confirmed = 0
    ORDER BY completed_at DESC
  `).all()

  return c.json({ success: true, goals: goals.results })
})
```

---

### 记录路由

```typescript
// src/routes/records.ts
import { Hono } from 'hono'
import { authMiddleware } from './auth'

export const recordRoutes = new Hono<{ Bindings: Env }>()

recordRoutes.use('*', authMiddleware)

// 获取记录列表
recordRoutes.get('/', async (c) => {
  const records = await c.env.DB.prepare(`
    SELECT * FROM records
    ORDER BY created_at DESC
  `).all()

  return c.json({ success: true, records: records.results })
})

// 添加支出
recordRoutes.post('/expense', async (c) => {
  const { amount, description } = await c.req.json()

  await c.env.DB.prepare(`
    INSERT INTO records (type, amount, description)
    VALUES ('expense', ?, ?)
  `).bind(amount, description).run()

  return c.json({ success: true })
})
```

---

### 统计路由

```typescript
// src/routes/stats.ts
import { Hono } from 'hono'
import { authMiddleware } from './auth'

export const statsRoutes = new Hono<{ Bindings: Env }>()

statsRoutes.use('*', authMiddleware)

// 获取统计
statsRoutes.get('/', async (c) => {
  const { DB } = c.env

  // 总收入、总支出
  const totalResult = await DB.prepare(`
    SELECT
      SUM(CASE WHEN type = 'income' THEN amount ELSE 0 END) as totalIncome,
      SUM(CASE WHEN type = 'expense' THEN amount ELSE 0 END) as totalExpense
    FROM records
  `).first()

  const stats: any = {
    totalIncome: totalResult?.totalIncome || 0,
    totalExpense: totalResult?.totalExpense || 0,
    balance: (totalResult?.totalIncome || 0) - (totalResult?.totalExpense || 0)
  }

  // 本周统计
  const weekAgo = new Date()
  weekAgo.setDate(weekAgo.getDate() - 7)

  const weekResult = await DB.prepare(`
    SELECT
      SUM(CASE WHEN type = 'income' THEN amount ELSE 0 END) as weeklyIncome,
      SUM(CASE WHEN type = 'expense' THEN amount ELSE 0 END) as weeklyExpense
    FROM records
    WHERE created_at > ?
  `).bind(weekAgo.toISOString()).first()

  stats.weeklyIncome = weekResult?.weeklyIncome || 0
  stats.weeklyExpense = weekResult?.weeklyExpense || 0

  // 本月统计
  const monthAgo = new Date()
  monthAgo.setDate(monthAgo.getDate() - 30)

  const monthResult = await DB.prepare(`
    SELECT
      SUM(CASE WHEN type = 'income' THEN amount ELSE 0 END) as monthlyIncome,
      SUM(CASE WHEN type = 'expense' THEN amount ELSE 0 END) as monthlyExpense
    FROM records
    WHERE created_at > ?
  `).bind(monthAgo.toISOString()).first()

  stats.monthlyIncome = monthResult?.monthlyIncome || 0
  stats.monthlyExpense = monthResult?.monthlyExpense || 0

  return c.json({ success: true, stats })
})
```

---

## 开发计划

### Phase 1：MVP 开发（1周）

**Day 1-2：**
- [x] 项目初始化
- [x] D1 数据库设计和初始化
- [ ] 用户系统（注册/登录）
- [ ] 目标系统基础功能

**Day 3-4：**
- [ ] 确认系统
- [ ] 记录系统
- [ ] 统计系统

**Day 5：**
- [ ] 界面开发
- [ ] 测试和部署

---

## 技术要求

### 性能要求
- 首屏加载 < 1s
- API 响应 < 300ms
- Cloudflare 全球节点

### 兼容性要求
- iOS 13+
- Android 8+
- 微信浏览器
- Safari / Chrome

---

## 成功指标

### 短期（1周）
- 完成核心功能
- 家长和孩子都能使用
- 操作流畅，无卡顿

### 中期（1个月）
- 完成多个目标
- 累计价值 > 500 元
- 家长和孩子都满意

### 长期（3个月）
- 建立稳定的使用习惯
- 孩子主动完成目标
- 理解"能力→回报"的关系

---

## 备注

**极简主义原则：**

1. **最少操作** - 每个功能都在3步以内
2. **最快记录** - 一键确认，自动记录
3. **最轻负担** - 不占用家长时间
4. **聚焦核心** - 只做"目标→确认→记录"

**去掉的功能：**
- ❌ 项目申请和审核
- ❌ 价值分配比例
- ❌ 三账户体系
- ❌ 投资和储蓄
- ❌ 成就系统
- ❌ 通知系统

**保留的功能：**
- ✅ 目标设置
- ✅ 完成确认
- ✅ 价值记录
- ✅ 简单统计

**迭代原则：**
- 先跑通核心流程
- 再优化细节体验
- 最后考虑扩展功能

---

## 版本历史

### v3.0（极简版）- 当前版本
**核心变化：**
- 流程从5步简化到3步
- 页面从25个减少到8个
- 功能极度简化
- 聚焦"目标→完成→确认→记录"

### v2.0（复杂版）
- 企业化管理流程
- 25个页面
- 完整的项目管理系统

### v1.0（初始版）
- 基础财商教育框架