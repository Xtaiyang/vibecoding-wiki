# AGENTS.md — AI 编辑器专属开发规范与文档导航地图

> 本文件是整个项目规范文档体系的**导航地图**。AI 编辑器（CodeBuddy / Cursor / Claude Code / Trae 等）在介入开发前，**必须首先阅读本文件**，再根据当前任务跳转到对应的规范文档。
>
> 文档之间通过锚点链接相互关联，形成逻辑闭环，避免 AI 生成孤立、脱节的代码。

---

## 一、阅读优先级（AI 介入顺序）

AI 在接收到任何开发任务时，按以下优先级加载上下文：

| 优先级 | 文档 | 何时阅读 | 核心目的 |
|--------|------|----------|----------|
| P0 | [README.md](./README.md) | **每次** | 理解项目定位、技术栈、启动方式 |
| P0 | [STRUCTURE.md](./STRUCTURE.md) | **每次** | 知道文件放哪、目录职责边界 |
| P0 | [CONVENTIONS.md](./CONVENTIONS.md) | **每次写代码** | 命名、import、组件、TS 规则 |
| P1 | [ARCHITECTURE.md](./ARCHITECTURE.md) | 改动架构 / 新增模块 | 理解分层、数据流、包依赖 |
| P1 | [CONCERNS.md](./CONCERNS.md) | 涉及认证 / 日志 / 缓存 / 错误 | 横切关注点统一处理 |
| P2 | [TECH-STACK.md](./TECH-STACK.md) | 选型决策 / 引入新依赖 | 理解为什么用、怎么用 |
| P2 | [INTEGRATIONS.md](./INTEGRATIONS.md) | 对接第三方服务 | 数据库 / 认证 / AI / 存储 集成规范 |
| P2 | [DESIGN.md](./docs/DESIGN.md) | 前端页面 / 视觉规范 | 通用设计规范、主题、响应式 |
| P2 | [anti-fraud-design.md](./docs/anti-fraud-design.md) | 反诈模块专用设计 | 公安蓝、风险三色、敏感词高亮等专用视觉规范 |
| P3 | [FEATURES.md](./FEATURES.md) | 新增 / 修改功能 | 业务边界、权限、交互逻辑 |
| P3 | [TESTING.md](./TESTING.md) | 写测试 | 测试策略、覆盖率、vitest 配置 |
| P4 | [USAGE.md](./USAGE.md) | 编写用户文档 | 功能概述与操作规范 |
| P4 | [TODO.md](./TODO.md) | 接续未完成任务 | 任务计划与完成状态 |
| P4 | [CHANGELOG.md](./CHANGELOG.md) | 发布版本 | 版本迭代记录 |
| P5 | [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) | 协作 / 贡献 | 行为准则 |
| P5 | [CLAUDE.md](./CLAUDE.md) | Claude Code 专属 | 附加指令（当前为空） |

---

## 二、任务→文档速查表

AI 根据任务类型，**必须先读对应文档再动手**：

| 任务类型 | 必读文档 | 关键约束 |
|----------|----------|----------|
| 新增 React 组件 | CONVENTIONS + STRUCTURE + DESIGN | 组件放 `packages/ui` 或 `apps/web/src/components`，遵循命名与 shadcn/ui 规范 |
| 新增 API 接口 | ARCHITECTURE + CONCERNS + INTEGRATIONS | 分层走 route → service → repository，错误统一拦截 |
| 新增数据库表 | TECH-STACK + INTEGRATIONS + STRUCTURE | 用 Drizzle schema，放 `packages/db/schema`，必须带 Zod 校验 |
| 新增业务功能 | FEATURES + CONVENTIONS | 先确认业务边界与权限要求，再开发 |
| 修复 Bug | CONVENTIONS + TESTING | 修复须补测试，遵循错误处理规范 |
| 引入新依赖 | TECH-STACK | 先论证选型，避免重复造轮子 |
| 写测试 | TESTING | 区分单元 / 集成 / E2E，满足覆盖率门槛 |
| 对接第三方 | INTEGRATIONS | 走统一适配层，配置走环境变量 |

---

## 三、AI 生成代码的强制红线

以下规则**任何情况下不可违反**：

1. **绝不生成与 [CONVENTIONS.md](./CONVENTIONS.md) 冲突的命名**。
2. **绝不绕过 [ARCHITECTURE.md](./ARCHITECTURE.md) 的分层**（如组件直接调数据库、route 层写业务逻辑）。
3. **绝不硬编码**密钥、连接串、第三方凭证，必须走 [INTEGRATIONS.md](./INTEGRATIONS.md) 定义的环境变量。
4. **绝不新增功能前不查** [FEATURES.md](./FEATURES.md)，避免重复实现或越界。
5. **任何代码改动必须同步更新**对应 MD 文件，保证规范与代码不脱节。

---

## 四、文档维护规则

- 文档与代码**同 PR 提交**，CI 校验改动文件对应的 MD 是否同步更新。
- 文档版本号与项目版本号对齐，记录于 [CHANGELOG.md](./CHANGELOG.md)。
- 新增文档须在本地图中登记，否则视为未纳入规范体系。

[DuMate AI生成]