# 项目完整目录结构说明

> 本文档定义了项目的完整目录树，明确每个目录的职责。AI 生成新文件时，必须先确认目标目录，禁止越界放置。

## 一、顶层目录

```
project-root/
├── apps/                       # 应用层：可独立部署的应用程序
├── packages/                   # 包层：共享的模块与配置
├── docs/                       # 文档层：设计稿、API 文档、需求文档
├── scripts/                    # 脚本层：构建、部署、数据迁移脚本
├── tests/                      # 测试层：E2E 测试与集成测试
├── tools/                      # 工具层：自定义 CLI 工具、代码生成器
├── .github/                    # GitHub 配置：CI/CD 工作流、Issue 模板
├── .husky/                     # Git Hooks：提交前检查
├── package.json                # 根 package.json，定义 workspaces
├── turbo.json                  # Turborepo 构建流水线配置
├── pnpm-workspace.yaml         # pnpm workspace 配置
├── tsconfig.json               # 根 TypeScript 配置
└── README.md                   # 项目总览
```

## 二、apps/ 目录

`apps/` 目录存放可独立部署的应用程序。每个应用都是完整的、可运行的单元。

### 2.1 apps/web/ —— 前端 Web 应用

基于 Next.js 14 (App Router) 构建的客户端应用。

```
apps/web/
├── src/
│   ├── app/                    # Next.js App Router 路由
│   │   ├── (auth)/             # 认证路由组（登录、注册、找回密码）
│   │   │   ├── login/
│   │   │   ├── register/
│   │   │   └── forgot-password/
│   │   ├── (dashboard)/        # 控制台路由组（需要认证）
│   │   │   ├── layout.tsx      # 控制台布局（侧边栏 + 顶部导航）
│   │   │   ├── page.tsx        # 控制台首页（数据概览）
│   │   │   ├── accounts/       # 账套管理
│   │   │   ├── vouchers/       # 凭证管理
│   │   │   ├── invoices/       # 发票管理
│   │   │   ├── reports/        # 财务报表
│   │   │   ├── settings/       # 系统设置
│   │   │   └── ...
│   │   ├── api/                # Next.js API Routes（仅用于文件上传等）
│   │   ├── layout.tsx          # 根布局（Provider、全局样式）
│   │   ├── page.tsx            # 落地页（Landing Page）
│   │   ├── loading.tsx         # 全局 loading 状态
│   │   ├── error.tsx           # 全局 error 边界
│   │   └── not-found.tsx       # 404 页面
│   ├── components/             # 业务组件（仅当前应用使用）
│   │   ├── layout/             # 布局组件（Sidebar, Header, Breadcrumb）
│   │   ├── dashboard/          # 控制台专属组件
│   │   ├── forms/              # 业务表单组件
│   │   └── charts/             # 图表组件（基于 Recharts 封装）
│   ├── hooks/                  # 自定义 React Hooks
│   ├── stores/                 # Zustand 状态管理
│   ├── lib/                    # 工具函数（仅限 web 应用）
│   ├── styles/                 # 全局样式、主题配置
│   ├── types/                  # 前端专属类型定义
│   └── middleware.ts           # Next.js 中间件（认证检查、路由守卫）
├── public/                     # 静态资源（图片、字体、favicon）
├── .env.example                # 环境变量模板
├── .env.local                  # 本地环境变量（gitignore）
├── next.config.js              # Next.js 配置
├── tailwind.config.ts          # Tailwind CSS 配置
├── tsconfig.json               # TypeScript 配置
└── package.json                # 应用依赖
```

### 2.2 apps/api/ —— 后端 API 服务

基于 Hono.js + tRPC 构建的 API 服务。

```
apps/api/
├── src/
│   ├── routes/                 # API 路由定义
│   │   ├── auth.ts             # 认证路由（登录、注册、Token 刷新）
│   │   ├── users.ts            # 用户管理路由
│   │   ├── accounts.ts         # 账套管理路由
│   │   ├── vouchers.ts         # 凭证管理路由
│   │   ├── invoices.ts         # 发票管理路由
│   │   ├── reports.ts          # 报表路由
│   │   ├── settings.ts         # 系统设置路由
│   │   └── index.ts            # 路由聚合入口
│   ├── middleware/             # 中间件
│   │   ├── auth.ts             # JWT 认证中间件
│   │   ├── rbac.ts             # 权限校验中间件
│   │   ├── error-handler.ts    # 全局错误处理中间件
│   │   ├── logger.ts           # 请求日志中间件
│   │   └── rate-limiter.ts     # 限流中间件
│   ├── services/               # 业务服务层
│   │   ├── auth-service.ts
│   │   ├── user-service.ts
│   │   ├── account-service.ts
│   │   ├── voucher-service.ts
│   │   ├── invoice-service.ts
│   │   ├── report-service.ts
│   │   └── ai-service.ts       # AI 识别服务
│   ├── lib/                    # 工具函数
│   │   ├── jwt.ts              # JWT 工具
│   │   ├── password.ts         # 密码加密工具
│   │   ├── response.ts         # 统一响应格式
│   │   └── errors.ts           # 自定义错误类
│   ├── types/                  # 后端专属类型
│   └── index.ts                # 应用入口
├── .env.example
├── .env.local
├── tsconfig.json
└── package.json
```

## 三、packages/ 目录

`packages/` 目录存放共享的模块与配置，供 `apps/` 中的应用引用。

### 3.1 packages/ui/ —— 共享 UI 组件库

基于 React + Tailwind CSS + Radix UI 构建的原子化组件库。

```
packages/ui/
├── src/
│   ├── components/             # 组件目录
│   │   ├── button/             # 按钮组件
│   │   │   ├── button.tsx
│   │   │   ├── button.types.ts
│   │   │   └── button.stories.tsx
│   │   ├── input/              # 输入框组件
│   │   ├── select/             # 选择器组件
│   │   ├── table/              # 表格组件
│   │   ├── modal/              # 弹窗组件
│   │   ├── form/               # 表单相关组件
│   │   ├── date-picker/        # 日期选择器
│   │   ├── chart/              # 图表容器组件
│   │   └── index.ts            # 组件统一导出
│   ├── hooks/                  # 通用 Hooks
│   ├── utils/                  # 样式工具函数（cn 合并）
│   ├── theme/                  # 主题配置（Design Token）
│   └── index.ts                # 包入口
├── tsconfig.json
├── tailwind.config.ts
└── package.json
```

**职责边界**：
- `packages/ui`：通用、无业务逻辑的纯 UI 组件（Button, Input, Modal 等）
- `apps/web/components`：包含业务逻辑的业务组件（DashboardCard, VoucherForm 等）

### 3.2 packages/db/ —— 数据库 Schema 与 ORM

```
packages/db/
├── src/
│   ├── schema/                 # Drizzle ORM Schema 定义
│   │   ├── users.ts            # 用户表
│   │   ├── accounts.ts         # 账套表
│   │   ├── companies.ts        # 企业表
│   │   ├── vouchers.ts         # 凭证表
│   │   ├── voucher-items.ts    # 凭证明细表
│   │   ├── invoices.ts         # 发票表
│   │   ├── subjects.ts         # 会计科目表
│   │   ├── reports.ts          # 报表数据表
│   │   ├── settings.ts         # 系统设置表
│   │   └── index.ts            # Schema 聚合导出
│   ├── migrations/             # 数据库迁移文件
│   ├── seed/                   # 种子数据脚本
│   ├── client.ts               # Drizzle Client 配置
│   └── index.ts                # 包入口
├── drizzle.config.ts           # Drizzle Kit 配置
├── tsconfig.json
└── package.json
```

### 3.3 packages/shared/ —— 共享工具与常量

```
packages/shared/
├── src/
│   ├── constants/              # 全局常量
│   │   ├── roles.ts            # 角色常量定义
│   │   ├── permissions.ts      # 权限常量定义
│   │   ├── voucher-types.ts    # 凭证类型常量
│   │   └── api-errors.ts       # API 错误码常量
│   ├── utils/                  # 通用工具函数
│   │   ├── date.ts             # 日期处理
│   │   ├── number.ts           # 数字/金额格式化
│   │   ├── validate.ts         # 通用校验函数
│   │   └── crypto.ts           # 加密/哈希工具
│   ├── validators/             # Zod 校验 schema
│   │   ├── auth.ts
│   │   ├── user.ts
│   │   ├── voucher.ts
│   │   └── invoice.ts
│   └── index.ts
├── tsconfig.json
└── package.json
```

### 3.4 packages/config/ —— 共享配置

```
packages/config/
├── eslint/                     # ESLint 配置包
│   ├── base.js
│   ├── next.js
│   └── node.js
├── typescript/                 # TypeScript 配置包
│   ├── base.json
│   ├── next.json
│   └── node.json
├── tailwind/                   # Tailwind 配置包
│   └── tailwind.config.ts
└── package.json
```

## 四、docs/ 目录

```
docs/
├── design/                     # 设计稿与原型
│   ├── ui-kit.figma
│   └── prototype.rp
├── api/                        # API 文档（自动生成）
│   └── openapi.json
├── requirements/               # 需求文档
│   ├── prd-v1.0.md
│   └── feature-specs/
└── diagrams/                   # 架构图、流程图
    ├── system-architecture.png
    └── data-flow.png
```

## 五、scripts/ 目录

```
scripts/
├── dev-setup.sh                # 开发环境初始化脚本
├── db-migrate.sh               # 数据库迁移脚本
├── db-seed.sh                  # 数据库种子数据脚本
├── db-backup.sh                # 数据库备份脚本
├── build-docker.sh             # Docker 镜像构建脚本
└── deploy.sh                   # 部署脚本
```

## 六、tests/ 目录

```
tests/
├── e2e/                        # Playwright E2E 测试
│   ├── auth.spec.ts            # 认证流程测试
│   ├── voucher-flow.spec.ts    # 凭证流程测试
│   └── invoice-flow.spec.ts    # 发票流程测试
├── fixtures/                   # 测试数据与 fixtures
└── playwright.config.ts        # Playwright 配置
```

## 七、目录职责对照表

| 目录 | 职责 | 什么该放 | 什么不该放 |
|-----|------|---------|-----------|
| `apps/web/src/app/` | Next.js 路由页面 | 页面组件、布局、loading/error 状态 | 通用 UI 组件、工具函数 |
| `apps/web/src/components/` | 业务组件 | 带业务逻辑的组件、页面级组件 | 纯 UI 原子组件 |
| `apps/web/src/hooks/` | 前端 Hooks | 数据获取、DOM 操作、状态逻辑 | 服务端逻辑、数据库操作 |
| `apps/web/src/stores/` | 客户端状态 | Zustand store、全局 UI 状态 | 服务端状态、敏感数据 |
| `apps/api/src/routes/` | API 路由 | 路由定义、请求校验、参数解析 | 业务逻辑、数据库操作 |
| `apps/api/src/services/` | 业务服务 | 业务逻辑、数据编排、外部调用 | HTTP 请求处理、路由定义 |
| `apps/api/src/middleware/` | 中间件 | 认证、日志、错误处理、限流 | 业务逻辑 |
| `packages/ui/` | 通用 UI | 原子组件、复合组件、主题 | 业务逻辑、API 调用 |
| `packages/db/` | 数据库 | Schema、迁移、Client 配置 | 业务逻辑、API 路由 |
| `packages/shared/` | 共享工具 | 常量、工具函数、校验 schema | 组件、数据库操作 |

## 八、命名约定补充

目录与文件命名规则详见 [CONVENTIONS.md](./CONVENTIONS.md)「命名规则」章节，此处强调目录级约定：

- **功能模块目录**：kebab-case，如 `reimbursement/`
- **组件文件**：PascalCase，如 `InvoiceTable.tsx`
- **路由文件**：TanStack Router 约定（`_auth` 前缀为布局路由，`__root` 为根）
- **测试文件**：与源文件同目录，`.test.ts(x)` 后缀
