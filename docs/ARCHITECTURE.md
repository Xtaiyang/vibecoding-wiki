# 项目整体架构说明

> 本文件定义项目的**架构分层、数据流、包依赖关系**。AI 在新增模块或改动分层时必读，违反分层将导致代码无法维护，所以必须遵循架构分层约束，不得破坏层次边界。目录职责详见 [STRUCTURE.md](./STRUCTURE.md)，编码规则详见 [CONVENTIONS.md](./CONVENTIONS.md)。

---

## 一、架构总览

项目采用 **Monorepo + 分层架构**：

```
┌─────────────────────────────────────────────────────────┐
│                    apps/web  apps/mobile                │  表现层（UI）
│   routes → features → components → hooks(stores)        │
└────────────┬──────────────────────────┬─────────────────┘
             │ TanStack Query (HTTP)    │ Zustand (本地状态)
             ▼                          ▼
┌─────────────────────────────────────────────────────────┐
│                      apps/server                         │  应用层（API）
│   routes(路由/入参校验) → services(业务) → repositories(数据)│
└────────────┬──────────────────────────┬─────────────────┘
             │ Drizzle ORM              │ 横切：auth/log/error
             ▼                          ▼
┌─────────────────────────────────────────────────────────┐
│ packages/db (schema)  packages/shared (类型/常量/工具)    │  共享层
└─────────────────────────────────────────────────────────┘
```

---

## 二、分层职责与规则

### 2.1 表现层（apps/web / apps/mobile）

| 子层 | 职责 | 禁止 |
|------|------|------|
| `routes/` | 路由定义、页面级布局、loader 数据预取 | 写业务逻辑 |
| `features/` | 功能模块组装（调用 hook + 渲染组件） | 直接调 API（走 hook） |
| `components/` | 纯展示组件 | 包含数据请求 |
| `hooks/` | 封装 TanStack Query 调用 + 客户端逻辑 | 写 JSX 渲染 |
| `stores/` | Zustand 客户端状态（UI 状态为主） | 存服务端数据（用 Query） |

**数据获取唯一入口**：`hooks/` 中的 `useXxxQuery`，底层走 TanStack Query → HTTP → server routes。

### 2.2 应用层（apps/server）

严格三层，**单向依赖**，禁止逆向：

```
routes  ──调用──▶  services  ──调用──▶  repositories  ──调用──▶  packages/db
  │                  │                                        │
  ▼                  ▼                                        ▼
入参校验(Zod)     业务逻辑/编排                             Drizzle 查询
权限校验          领域错误抛出                               表操作封装
```

| 层 | 职责 | 禁止 |
|----|------|------|
| `routes/` | 接收请求、Zod 入参校验、权限校验、调用 service、返回响应 | 直接操作数据库、写业务逻辑 |
| `services/` | 业务逻辑编排、事务控制、领域错误抛出 | 直接处理 HTTP req/res、直接写裸 SQL |
| `repositories/` | 数据访问封装、Drizzle 查询构造 | 含业务规则判断、感知 HTTP 上下文 |

```typescript
// ✅ 正确分层示例
// routes/invoice.ts
router.post('/invoice', authMiddleware, async (ctx) => {
  const dto = createInvoiceSchema.parse(ctx.body);     // 入参校验
  const result = await invoiceService.create(dto);     // 调 service
  return ctx.json(result);
});

// services/invoice.ts
export const invoiceService = {
  async create(dto: CreateInvoiceDTO) {
    await validateContractExists(dto.contractId);       // 业务编排
    const invoice = await invoiceRepo.insert(dto);      // 调 repo
    await auditLog.record('invoice.create', invoice.id);// 横切
    return invoice;
  }
};

// repositories/invoice.ts
export const invoiceRepo = {
  insert(dto: CreateInvoiceDTO) {
    return db.insert(invoiceTable).values(dto).returning(); // 纯数据操作
  }
};
```

```typescript
// ❌ 错误：route 层直接操作数据库
router.post('/invoice', async (ctx) => {
  const data = ctx.body; // 无校验
  const result = await db.insert(invoiceTable).values(data); // 越层
  return ctx.json(result);
});
```

### 2.3 共享层（packages/*）

| 包 | 职责 | 依赖方向 |
|----|------|----------|
| `packages/shared` | 类型、常量、纯工具 | 被所有层依赖，自身不依赖业务包 |
| `packages/db` | schema + Drizzle 客户端 | 依赖 `shared`，被 `server` 依赖 |
| `packages/ui` | 组件库 | 依赖 `shared`，被 `apps/web` / `apps/mobile` 依赖 |
| `packages/config` | 工程配置 | 独立，被构建工具消费 |

> **关键**：`packages/shared` 是类型契约的唯一来源，前后端共享 DTO 定义，保证接口一致。

---

## 三、数据流

### 3.1 读数据流（查询）

```
用户操作 → TanStack Query 缓存命中？
  ├─ 命中 → 直接渲染（零请求）
  └─ 未命中 → fetch HTTP → server route → service → repository → DB
              ← 数据返回 → Query 缓存 → 组件渲染
```

### 3.2 写数据流（变更）

```
表单提交 → useXxxMutation
  → 乐观更新（onMutate 更新 Query 缓存）
  → HTTP → server route（Zod 校验 + 权限）
  → service（业务逻辑 + 事务）
  → repository → DB
  ← 返回结果 → onSuccess 使相关 queryKey 失效 → UI 刷新
```

### 3.3 状态归属

| 状态类型 | 存放 | 示例 |
|----------|------|------|
| 服务端数据 | TanStack Query 缓存 | 发票列表、用户信息 |
| 客户端 UI 状态 | Zustand / 组件 useState | 侧边栏开关、筛选弹窗 |
| URL 状态 | TanStack Router 搜索参数 | 当前页码、筛选条件（可分享、可刷新） |

> **规则**：能放 URL 的状态不放 Zustand（URL 状态可分享、可刷新恢复），详见 [TECH-STACK.md](./TECH-STACK.md) 的 TanStack Router 规范。

---

## 四、包依赖关系图

```
packages/config ──────── (被构建工具消费，无运行时依赖)
packages/shared ──────── (零业务依赖)
packages/db ───依赖──▶ packages/shared
packages/ui ───依赖──▶ packages/shared
apps/web ───依赖──▶ packages/ui, packages/shared
apps/mobile ─依赖──▶ packages/ui, packages/shared
apps/server ──依赖──▶ packages/db, packages/shared
```

**禁止环形依赖**：`apps/*` 之间互不依赖，`packages/*` 之间仅单向依赖。

---

## 五、新增模块的标准流程

AI 接到「新增 XX 功能」任务时，按以下顺序创建文件（以「发票导出」为例）：

1. **数据层**：`packages/db/src/schema/invoice.ts` 已有则跳过；新增字段需加 Zod。
2. **共享类型**：`packages/shared/src/types/invoice.ts` 补充 DTO。
3. **Repository**：`apps/server/src/repositories/invoice.ts` 新增导出查询方法。
4. **Service**：`apps/server/src/services/invoice.ts` 新增导出业务逻辑。
5. **Route**：`apps/server/src/routes/invoice.ts` 新增导出端点 + 权限校验。
6. **Hook**：`apps/web/src/features/invoice/hooks/useExportInvoice.ts`。
7. **组件 / 页面**：`apps/web/src/features/invoice/ExportButton.tsx`。
8. **测试**：service 层单元测试 + route 层集成测试。
9. **更新文档**：[FEATURES.md](./FEATURES.md) 登记功能 + [CHANGELOG.md](./CHANGELOG.md) 记录。

> 每一步对应目录见 [STRUCTURE.md](./STRUCTURE.md)「文件放置速查表」。