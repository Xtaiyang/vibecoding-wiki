# 项目更新日志

> 本文档记录各版本的功能优化、内容修改等迭代信息。每次发布新版本时必须更新。

## 版本规范

遵循 [Keep a Changelog](https://keepachangelog.com/) 规范，提交规范见 [CONVENTIONS.md](./CONVENTIONS.md) Git 章节。

版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)规范：
- **主版本号（X.y.z）**：不兼容的 API 修改
- **次版本号（x.Y.z）**：向下兼容的功能新增
- **修订号（x.y.Z）**：向下兼容的问题修复

## 版本格式说明

```
## [版本号] - 日期
### Added     新增功能
### Changed   变更 / 优化
### Deprecated 废弃
### Removed   移除
### Fixed     修复
### Security  安全相关
```

---

## [1.0.0] - 2026-06-26

### 新增

- 项目初始化，搭建 Monorepo 架构（Turborepo + pnpm）
- 前端项目搭建（Next.js 14 App Router + React 18）
- 后端项目搭建（Hono.js + tRPC）
- 数据库 Schema 设计（Drizzle ORM + PostgreSQL）
- 用户认证模块（注册、登录、JWT Token）
- 账套管理模块（创建、切换）
- 凭证管理模块（录入、列表、编辑、删除）
- 基础 UI 组件库搭建（Button、Input、Modal、Table 等）
- 项目核心规范文档体系建立（16 个规范文档）

---

## [0.9.0] - 2026-06-20

### 新增

- 技术选型调研与确定
- 项目架构设计文档
- 数据库 ER 图设计
- UI 设计规范制定

### 变更

- 前端框架从 Remix 调整为 Next.js（App Router 更契合需求）
- ORM 从 Prisma 调整为 Drizzle（更轻量、类型更安全）

---

## [0.1.0] - 2026-06-01

### 新增

- 项目立项
- 需求分析与功能规划
- 技术预研
