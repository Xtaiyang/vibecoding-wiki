# 项目技术栈选型说明

> 本文档解释了 React、TanStack、Drizzle 等核心技术的选型原因与使用规范。AI 生成代码时，必须遵循已选技术栈，不得引入未经评估的新技术。

## 一、前端技术栈

### 1.1 React 18

**选型原因**：
- 生态最成熟，组件库、工具链丰富
- Concurrent Features（并发特性）提升用户体验
- Server Components 支持，减少客户端 bundle 体积
- 团队熟悉度高，学习成本低

**使用规范**：
- 优先使用函数组件 + Hooks
- 默认使用 Server Components（Next.js App Router）
- 仅在需要客户端交互时添加 `"use client"` 指令
- 禁止使用 Class Components（ legacy 代码除外）

### 1.2 Next.js 14 (App Router)

**选型原因**：
- App Router 提供原生布局、加载状态、错误边界支持
- Server Components 直接获取数据，减少 API 调用层级
- 内置图片优化、字体优化、SEO 支持
- 中间件支持路由守卫、A/B 测试等场景

**使用规范**：
- 使用 App Router，不使用 Pages Router
- 页面组件默认是 Server Component
- 数据获取在 Server Component 中直接进行
- 交互逻辑封装为 Client Component，通过 props 传入数据

```typescript
// ✅ 正确：Server Component 直接获取数据
// app/(dashboard)/vouchers/page.tsx
export default async function VouchersPage(): Promise<JSX.Element> {
  const vouchers = await getVouchers();  // 服务端直接查询
  return <VoucherList data={vouchers} />;
}

// ✅ 正确：交互逻辑用 Client Component
// components/voucher-form.tsx
"use client";
export function VoucherForm(): JSX.Element {
  const [isOpen, setIsOpen] = useState(false);
  // ...
}
```

### 1.3 TypeScript 5.4

**选型原因**：
- 编译时类型检查，减少运行时错误
- IDE 智能提示，提升开发效率
- 团队协作时接口契约清晰
- 现代 TypeScript 支持 `satisfies`、改进的类型推断

**使用规范**：
- 严格模式开启（`strict: true`）
- 禁止使用 `any` 类型
- 所有函数参数和返回值必须显式注解类型
- 使用 `interface` 定义对象类型，`type` 定义联合类型/工具类型

### 1.4 Tailwind CSS 3.4

**选型原因**：
- 原子化 CSS，无需命名烦恼
- 构建时 purge，生产环境体积极小
- Design Token 通过配置统一管控
- 响应式、暗黑模式、任意值支持完善

**使用规范**：
- 使用 `cn()` 工具函数合并类名（基于 clsx + tailwind-merge）
- 自定义样式通过 `tailwind.config.ts` 的 theme 扩展
- 不使用 `@apply` 抽取样式（优先用组件封装）
- 颜色、间距使用 Design Token，不硬编码数值

```typescript
// ✅ 正确
import { cn } from "@repo/ui/utils";

<button className={cn(
  "px-4 py-2 rounded-md font-medium",
  variant === "primary" && "bg-primary text-white hover:bg-primary-dark",
  variant === "danger" && "bg-error text-white hover:bg-error-dark",
  disabled && "opacity-50 cursor-not-allowed"
)}>
  {children}
</button>

// ❌ 错误：硬编码颜色值、不使用 cn 合并
<button className="px-4 py-2 bg-[#1677FF] text-white">
  {children}
</button>
```

### 1.5 TanStack Query 5

**选型原因**：
- 服务端状态管理的事实标准
- 自动缓存、后台刷新、去重请求
- 支持 SSR、Suspense 集成
- 开发工具完善，调试方便

**使用规范**：
- 所有服务端数据获取必须使用 TanStack Query
- `queryKey` 必须语义化、结构化
- 突变（Mutation）成功后手动失效相关查询缓存
- 全局配置重试、缓存时间策略

```typescript
// ✅ 正确
const { data, isLoading, error } = useQuery({
  queryKey: ["vouchers", accountId, { page, pageSize, filters }],
  queryFn: () => fetchVouchers(accountId, { page, pageSize, filters }),
  staleTime: 1000 * 60 * 5,  // 5 分钟内不重新获取
});

const mutation = useMutation({
  mutationFn: createVoucher,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ["vouchers"] });
    toast.success("凭证创建成功");
  },
});
```

### 1.6 Zustand 4.5

**选型原因**：
- API 极简，boilerplate 远少于 Redux
- 支持 TypeScript，无需额外类型封装
- 持久化、中间件扩展方便
- 适合中小规模全局状态

**使用规范**：
- 仅用于客户端全局状态（UI 状态、用户偏好设置）
- 服务端状态不要用 Zustand 管理
- Store 按领域拆分，不创建巨型 Store

```typescript
// ✅ 正确：按领域拆分 Store
// stores/uiStore.ts
export const useUIStore = create<UIState>((set) => ({
  sidebarOpen: true,
  theme: "light",
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
  setTheme: (theme) => set({ theme }),
}));

// stores/accountStore.ts
export const useAccountStore = create<AccountState>((set) => ({
  currentAccountId: null,
  setCurrentAccount: (id) => set({ currentAccountId: id }),
}));
```

### 1.7 React Hook Form + Zod

**选型原因**：
- React Hook Form：性能优秀，非受控组件，减少重渲染
- Zod：Schema 校验，类型安全，前后端可共享 schema
- 两者集成：`@hookform/resolvers/zod` 提供无缝衔接

**使用规范**：
- 所有表单必须使用 React Hook Form
- 校验 Schema 使用 Zod，定义在 `packages/shared/src/validators/`
- 表单提交错误通过 `react-hot-toast` 或 `sonner` 提示

```typescript
// ✅ 正确
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { loginSchema, type LoginInput } from "@repo/shared";

export function LoginForm(): JSX.Element {
  const form = useForm<LoginInput>({
    resolver: zodResolver(loginSchema),
    defaultValues: { email: "", password: "" },
  });

  const onSubmit = async (data: LoginInput) => {
    await login(data);
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <input {...form.register("email")} />
      {form.formState.errors.email && (
        <span>{form.formState.errors.email.message}</span>
      )}
    </form>
  );
}
```

## 二、后端技术栈

### 2.1 Hono.js 4

**选型原因**：
- 极轻量，性能优于 Express
- 支持多种运行时（Node.js, Deno, Bun, Cloudflare Workers）
- 中间件模型简洁，与 Express 兼容
- TypeScript 原生支持

**使用规范**：
- 路由定义使用 Hono 的链式 API
- 中间件按顺序注册，注意执行顺序
- 错误统一在 `app.onError` 中处理

```typescript
// ✅ 正确
import { Hono } from "hono";

const app = new Hono();

// 中间件顺序：限流 → 日志 → 认证 → 路由
app.use(rateLimitMiddleware);
app.use(loggerMiddleware);
app.use(authMiddleware);

// 路由
app.route("/api/v1/auth", authRoutes);
app.route("/api/v1/users", userRoutes);

// 全局错误处理
app.onError(errorHandler);
```

### 2.2 tRPC 11

**选型原因**：
- 端到端类型安全，前端调用 API 时自动获得类型提示
- 无需手动维护 API 文档和类型定义
- 支持批量请求、请求去重、订阅
- 与 React Query 集成完善

**使用规范**：
- 所有 API 路由通过 tRPC 定义
- Input/Output 使用 Zod Schema 校验
- 前端通过 `createTRPCReact` 创建类型安全客户端

```typescript
// ✅ 正确：tRPC Router 定义
// apps/api/src/trpc/router.ts
import { router, publicProcedure, protectedProcedure } from "./trpc";
import { z } from "zod";

export const appRouter = router({
  user: router({
    getById: protectedProcedure
      .input(z.object({ id: z.string().uuid() }))
      .query(async ({ input, ctx }) => {
        return ctx.services.user.getById(input.id);
      }),
    update: protectedProcedure
      .input(userUpdateSchema)
      .mutation(async ({ input, ctx }) => {
        return ctx.services.user.update(input);
      }),
  }),
});

export type AppRouter = typeof appRouter;
```

### 2.3 Drizzle ORM 0.30

**选型原因**：
- 类型安全：Schema 定义即 TypeScript 类型
- SQL-like API：贴近原生 SQL，学习成本低
- 轻量：体积小，性能优于 Prisma
- 迁移支持：Drizzle Kit 提供完善的迁移管理

**使用规范**：
- Schema 定义在 `packages/db/src/schema/`
- 使用关系查询避免 N+1 问题
- 复杂查询使用 `sql` 模板标签
- 所有数据库操作通过 Drizzle Client

```typescript
// ✅ 正确：Schema 定义
// packages/db/src/schema/users.ts
import { pgTable, uuid, varchar, timestamp } from "drizzle-orm/pg-core";

export const users = pgTable("users", {
  id: uuid("id").primaryKey().defaultRandom(),
  email: varchar("email", { length: 255 }).notNull().unique(),
  name: varchar("name", { length: 100 }).notNull(),
  createdAt: timestamp("created_at").defaultNow().notNull(),
});

// ✅ 正确：关系查询
const vouchersWithItems = await db.query.vouchers.findMany({
  with: {
    items: true,
    creator: { columns: { id: true, name: true } },
  },
});
```

## 三、基础设施技术栈

### 3.1 PostgreSQL 16

**选型原因**：
- 关系型数据库，ACID 保证，适合财务数据
- JSONB 支持灵活的数据结构
- 窗口函数、CTE 支持复杂报表查询
- 开源、稳定、社区活跃

**使用规范**：
- 核心财务数据使用标准化表结构
- 日志、审计数据可使用 JSONB 字段
- 敏感字段（如密码）使用加密存储
- 建立合适的索引（B-tree、GIN）

### 3.2 Redis 7

**选型原因**：
- 高性能 KV 存储，读写性能优异
- 支持多种数据结构（String、Hash、Set、Sorted Set）
- 支持持久化、集群、哨兵模式
- 广泛应用于缓存、Session、限流场景

### 3.3 MinIO

**选型原因**：
- 兼容 S3 API，可无缝迁移到阿里云 OSS、AWS S3
- 开源、轻量，本地开发方便
- 支持预签名 URL，前端直传文件

## 四、开发工具技术栈

### 4.1 pnpm

**选型原因**：
- 磁盘空间高效（硬链接共享依赖）
- 安装速度快
- Monorepo workspace 支持完善
- 严格的依赖管理（防止 phantom dependencies）

**使用规范**：
- 使用 `pnpm-workspace.yaml` 定义 workspace
- 共享依赖安装在 root，`--filter` 安装到指定包
- 使用 `pnpm exec` 运行本地二进制

### 4.2 Turborepo

**选型原因**：
- 智能缓存，只构建变更的部分
- 并行任务执行，提升构建速度
- Pipeline 可视化，依赖关系清晰
- 远程缓存支持团队协作

**使用规范**：
- 在 `turbo.json` 中定义 task pipeline
- 合理利用 `dependsOn` 定义任务依赖
- 开启远程缓存加速 CI/CD

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "dist/**"]
    },
    "test": {
      "dependsOn": ["build"]
    },
    "lint": {},
    "typecheck": {
      "dependsOn": ["^build"]
    }
  }
}
```

### 4.3 ESLint + Prettier

**选型原因**：
- ESLint：代码质量检查，发现潜在问题
- Prettier：代码格式化，统一风格
- 两者配合，质量与风格双保障

**使用规范**：
- 使用 `@repo/config/eslint` 共享配置
- Prettier 配置统一，不允许项目级覆盖
- 提交前通过 husky + lint-staged 自动检查

## 五、测试技术栈

### 5.1 Vitest

**选型原因**：
- 原生 ESM 支持，与 Vite 生态一致
- 速度快，支持多线程
- 兼容 Jest API，迁移成本低
- 内置 TypeScript 支持

### 5.2 Playwright

**选型原因**：
- 多浏览器支持（Chromium、Firefox、WebKit）
- 自动等待、可靠的重试机制
- 录制回放、Trace Viewer 调试方便
- 与 CI/CD 集成完善

## 六、技术栈决策记录

| 决策 | 选中方案 | 放弃方案 | 决策原因 |
|-----|---------|---------|---------|
| 前端框架 | React 18 | Vue 3, Svelte | 生态成熟，团队熟悉 |
| 全栈框架 | Next.js 14 | Remix, Nuxt | App Router + Server Components |
| 状态管理（服务端） | TanStack Query | SWR, Apollo | 功能完善，生态活跃 |
| 状态管理（客户端） | Zustand | Redux, Jotai | 简洁够用，TS 友好 |
| ORM | Drizzle | Prisma, TypeORM | 轻量、类型安全、SQL-like |
| API 框架 | Hono + tRPC | Express, NestJS | 轻量、类型安全、高性能 |
| CSS 方案 | Tailwind CSS | CSS Modules, Styled Components | 原子化、统一管控 |
| 表单方案 | RHF + Zod | Formik, Yup | 性能 + 类型安全 |
| 包管理器 | pnpm | npm, yarn | Monorepo 友好、速度快 |
| 构建工具 | Turborepo | Nx, Rush | 轻量、缓存智能 |
| 单元测试 | Vitest | Jest | ESM 原生、速度快 |
| E2E 测试 | Playwright | Cypress, Selenium | 多浏览器、可靠性 |

## 七、技术引入审批流程

如需引入新的技术栈或依赖，必须经过以下流程：

1. **技术调研**：对比至少 2 个同类方案
2. **影响评估**：评估对现有架构的影响、学习成本、维护成本
3. **PoC 验证**：在独立分支进行概念验证
4. **团队评审**：技术团队评审通过
5. **文档更新**：更新 TECH-STACK.md 和 ARCHITECTURE.md
6. **逐步落地**：从非核心模块开始引入