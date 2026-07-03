# 智慧共享财务平台

> 面向中小企业的智能化财务管理与共享服务平台，基于现代全栈技术栈构建。

## 一、项目介绍

### 1.1 产品定位

智慧共享财务平台是一款面向中小企业、代理记账公司及自由职业会计师的 SaaS 财务管理工具。平台提供从账套管理、凭证录入、发票管理到财务报表生成、税务申报辅助的完整财务工作流，支持多企业账套切换与团队协作。

### 1.2 核心价值

- **智能化录入**：AI 辅助识别发票、银行流水，自动生成凭证
- **多账套管理**：一个账号管理多个企业账套，一键切换
- **实时报表**：资产负债表、利润表、现金流量表实时生成
- **税务合规**：内置税务规则引擎，自动校验合规性
- **团队协作**：支持会计、出纳、老板多角色权限协作

### 1.3 技术亮点

- Monorepo 架构，前后端代码统一管理
- TypeScript 全栈类型安全
- 现代 React 生态（Server Components + TanStack Query）
- 原子化 UI 组件库，支持主题定制
- 自动化测试覆盖（单元 + 集成 + E2E）

## 二、目录结构

```
project-root/
├── apps/
│   ├── web/                    # 前端 Web 应用 (Next.js 14 App Router)
│   └── api/                    # 后端 API 服务 (Hono.js + Node.js)
├── packages/
│   ├── ui/                     # 共享 UI 组件库
│   ├── db/                     # 数据库 schema 与 Drizzle ORM 配置
│   ├── shared/                 # 共享工具函数、类型定义、常量
│   ├── config/                 # 共享配置（ESLint, TypeScript, Tailwind）
│   └── types/                  # 全局类型定义
├── docs/                       # 项目文档（设计稿、API 文档等）
├── scripts/                    # 构建、部署脚本
├── tests/                      # E2E 测试（Playwright）
├── package.json                # 根 package.json，定义 workspace
├── turbo.json                  # Turborepo 流水线配置
└── pnpm-workspace.yaml         # pnpm workspace 配置
```

> 完整目录结构详见 [STRUCTURE.md](./STRUCTURE.md)

## 三、技术栈

### 前端

| 技术 | 用途 | 版本 |
|-----|------|------|
| React | UI 框架 | ^18.3 |
| Next.js | 全栈框架（App Router） | ^14.2 |
| TypeScript | 类型安全 | ^5.4 |
| Tailwind CSS | 原子化 CSS | ^3.4 |
| TanStack Query | 服务端状态管理 | ^5.0 |
| TanStack Table | 表格组件 | ^8.0 |
| Zustand | 客户端状态管理 | ^4.5 |
| React Hook Form | 表单管理 | ^7.51 |
| Zod | 表单校验 | ^3.22 |
| Lucide React | 图标库 | ^0.400 |
| Recharts | 图表库 | ^2.12 |

### 后端

| 技术 | 用途 | 版本 |
|-----|------|------|
| Hono.js | 轻量 Web 框架 | ^4.0 |
| tRPC | 类型安全的 API 路由 | ^11.0 |
| Drizzle ORM | 数据库 ORM | ^0.30 |
| Zod | 输入校验 | ^3.22 |
| JWT | 认证令牌 | ^9.0 |
| bcryptjs | 密码加密 | ^2.4 |

### 基础设施

| 技术 | 用途 | 版本 |
|-----|------|------|
| PostgreSQL | 关系型数据库 | ^16 |
| Redis | 缓存 / Session | ^7 |
| MinIO | 对象存储（文件/发票图片） | latest |
| Turborepo | Monorepo 构建编排 | ^2.0 |
| pnpm | 包管理器 | ^9.0 |
| Docker | 容器化部署 | latest |
| Vitest | 单元测试 | ^1.6 |
| Playwright | E2E 测试 | ^1.44 |

> 技术选型详细说明见 [TECH-STACK.md](./TECH-STACK.md)

## 四、本地开发指南

### 4.1 环境要求

- Node.js >= 20.0.0
- pnpm >= 9.0.0
- Docker & Docker Compose（用于本地数据库）
- Git

### 4.2 快速开始

```bash
# 1. 克隆仓库
git clone https://github.com/company/finance-platform.git
cd finance-platform

# 2. 安装依赖
pnpm install

# 3. 启动本地基础设施（数据库、Redis、MinIO）
docker-compose -f docker-compose.dev.yml up -d

# 4. 复制环境变量模板
cp apps/web/.env.example apps/web/.env.local
cp apps/api/.env.example apps/api/.env.local

# 5. 执行数据库迁移
pnpm db:migrate

# 6. 启动开发服务器
pnpm dev
```

### 4.3 常用命令

```bash
# 启动所有应用（并行）
pnpm dev

# 仅启动前端
pnpm --filter web dev

# 仅启动后端
pnpm --filter api dev

# 执行数据库迁移
pnpm db:migrate

# 生成数据库迁移文件
pnpm db:generate

# 推送 schema 变更到数据库（开发环境）
pnpm db:push

# 运行所有测试
pnpm test

# 运行单元测试
pnpm test:unit

# 运行 E2E 测试
pnpm test:e2e

# 代码检查
pnpm lint

# 类型检查
pnpm typecheck

# 构建所有应用
pnpm build
```

### 4.4 环境变量说明

#### 前端 (apps/web/.env.local)

```env
NEXT_PUBLIC_API_URL=http://localhost:3001
NEXT_PUBLIC_APP_NAME=智慧共享财务平台
NEXT_PUBLIC_APP_VERSION=1.0.0
```

#### 后端 (apps/api/.env.local)

```env
# 服务器
PORT=3001
NODE_ENV=development

# 数据库
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/finance

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your-super-secret-key
JWT_EXPIRES_IN=7d

# MinIO
MINIO_ENDPOINT=localhost
MINIO_PORT=9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET=finance-platform

# 邮件服务（开发环境可配置为 mock）
SMTP_HOST=
SMTP_PORT=
SMTP_USER=
SMTP_PASS=
```

## 五、开发规范

- 编码规范：[CONVENTIONS.md](./CONVENTIONS.md)
- 架构说明：[ARCHITECTURE.md](./ARCHITECTURE.md)
- 测试策略：[TESTING.md](./TESTING.md)
- 功能清单：[FEATURES.md](./FEATURES.md)

## 六、部署说明

### 6.1 构建

```bash
# 生产构建
pnpm build

# 构建产物位于：
# - apps/web/.next/
# - apps/api/dist/
```

### 6.2 Docker 部署

```bash
# 构建生产镜像
docker-compose -f docker-compose.prod.yml build

# 启动生产环境
docker-compose -f docker-compose.prod.yml up -d
```

## 七、贡献指南

- 贡献者行为准则：[CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)
- 任务计划：[TODO.md](./TODO.md)
- 更新日志：[CHANGELOG.md](./CHANGELOG.md)

## 八、许可证

本项目采用 [MIT](./LICENSE) 许可证。
