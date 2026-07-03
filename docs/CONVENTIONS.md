# 项目编码规范

> 本文档定义了项目的命名规则、import 规范、组件写法、TypeScript 规则等。AI 生成代码时，必须严格遵循本规范，确保代码风格一致。

## 一、命名规则

### 1.1 文件命名

| 类型 | 规范 | 正确示例 | 错误示例 |
|-----|------|---------|---------|
| React 组件 | PascalCase | `UserProfile.tsx` | `user-profile.tsx` |
| 自定义 Hook | camelCase，前缀 `use` | `useAuth.ts` | `UseAuth.ts` |
| 工具函数 | camelCase | `formatDate.ts` | `FormatDate.ts` |
| 常量定义 | camelCase | `apiErrors.ts` | `api-errors.ts` |
| 类型定义 | PascalCase，后缀 `Types` 或直接使用 | `UserTypes.ts` | `user-types.ts` |
| 样式文件 | 与组件同名 | `Button.module.css` | `button-style.css` |
| 测试文件 | 与被测文件同名，后缀 `.test.ts` | `utils.test.ts` | `test-utils.ts` |
| 配置文件 | kebab-case | `tailwind.config.ts` | `tailwindConfig.ts` |

### 1.2 变量与函数命名

| 类型 | 规范 | 正确示例 | 错误示例 |
|-----|------|---------|---------|
| 变量 | camelCase | `userName`, `isLoading` | `user_name`, `IsLoading` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` | `maxRetryCount` |
| 布尔值 | 前缀 `is`/`has`/`can`/`should` | `isActive`, `hasPermission` | `active`, `permission` |
| 函数 | camelCase，动词开头 | `getUserById`, `handleSubmit` | `userById`, `submitHandler` |
| 类 / 构造函数 | PascalCase | `UserService` | `userService` |
| 接口 / 类型 | PascalCase | `UserProfile`, `ApiResponse` | `userProfile`, `api_response` |
| 枚举 | PascalCase，成员 UPPER_SNAKE_CASE | `enum Status { ACTIVE }` | `enum status { active }` |
| 泛型参数 | 单个大写字母或 PascalCase | `T`, `TData`, `TProps` | `data`, `propsType` |

### 1.3 组件命名

```typescript
// ✅ 正确：PascalCase，语义清晰
function UserProfileCard() {}
function InvoiceListTable() {}

// ❌ 错误：非 PascalCase 或语义模糊
function userProfile() {}
function Component() {}
function List() {}        // 太泛化
```

### 1.4 路由与 API 命名

```typescript
// ✅ 正确：RESTful 风格，名词复数
GET    /api/v1/users           // 获取用户列表
GET    /api/v1/users/:id       // 获取单个用户
POST   /api/v1/users           // 创建用户
PUT    /api/v1/users/:id       // 全量更新用户
PATCH  /api/v1/users/:id       // 部分更新用户
DELETE /api/v1/users/:id       // 删除用户

// ❌ 错误：动词开头、单数名词
GET    /api/v1/getUsers
POST   /api/v1/createUser
GET    /api/v1/user/:id
```

## 二、Import 规范

### 2.1 Import 排序

按以下顺序分组，组间空一行：

1. **React / 框架内置**
2. **第三方库**
3. **内部包**（`@repo/ui`, `@repo/db` 等）
4. **相对路径导入**（`@/components`, `@/lib` 等）
5. **类型导入**（`import type { ... }`）
6. **样式导入**

```typescript
// ✅ 正确示例
import { useState, useEffect } from "react";
import { useRouter } from "next/navigation";

import { useQuery } from "@tanstack/react-query";
import { zodResolver } from "@hookform/resolvers/zod";

import { Button, Input } from "@repo/ui";
import { db } from "@repo/db";

import { useAuth } from "@/hooks/useAuth";
import { formatDate } from "@/lib/utils";

import type { User } from "@/types/user";

import "./styles.css";

// ❌ 错误示例：顺序混乱、未分组
import { Button } from "@repo/ui";
import { useState } from "react";
import { useAuth } from "@/hooks/useAuth";
import { useQuery } from "@tanstack/react-query";
import "./styles.css";
import type { User } from "@/types/user";
```

### 2.2 路径别名

```typescript
// ✅ 正确：使用路径别名
import { Button } from "@repo/ui";
import { db } from "@repo/db";
import { api } from "@/lib/api";
import { useAuth } from "@/hooks/useAuth";

// ❌ 错误：使用相对路径跨多层
import { Button } from "../../../../packages/ui/src/components/button";
import { db } from "../../../packages/db/src/client";
```

### 2.3 禁止循环依赖

```typescript
// ❌ 错误：A.ts 导入 B.ts，B.ts 又导入 A.ts
// A.ts
import { b } from "./B";
export const a = () => b();

// B.ts
import { a } from "./A";   // 循环依赖！
export const b = () => a();
```

## 三、组件写法

### 3.1 函数组件（优先）

```typescript
// ✅ 正确：使用函数声明，显式返回类型
interface UserCardProps {
  user: User;
  onEdit?: (id: string) => void;
}

export function UserCard({ user, onEdit }: UserCardProps): JSX.Element {
  return (
    <div className="rounded-lg border p-4">
      <h3>{user.name}</h3>
      {onEdit && (
        <Button onClick={() => onEdit(user.id)}>编辑</Button>
      )}
    </div>
  );
}

// ❌ 错误：使用 React.FC（已废弃），或隐式返回类型
export const UserCard: React.FC<UserCardProps> = ({ user }) => {
  return <div>{user.name}</div>;
};
```

### 3.2 Props 定义

```typescript
// ✅ 正确：使用 interface，可选参数用 ?，有默认值的不标记可选
interface ButtonProps {
  children: React.ReactNode;
  variant?: "primary" | "secondary" | "danger";
  size?: "sm" | "md" | "lg";
  disabled?: boolean;
  onClick?: () => void;
}

export function Button({
  children,
  variant = "primary",
  size = "md",
  disabled = false,
  onClick,
}: ButtonProps): JSX.Element {
  // ...
}

// ❌ 错误：使用 type 定义 Props，或把所有字段都标记为可选
export type ButtonProps = {
  children?: React.ReactNode;   // children 不应该是可选的
  variant?: string;             // 应该用字面量联合类型
  // ...
};
```

### 3.3 状态管理

```typescript
// ✅ 正确：服务端状态用 TanStack Query，客户端状态用 Zustand
import { useQuery } from "@tanstack/react-query";
import { useUIStore } from "@/stores/uiStore";

export function UserList(): JSX.Element {
  // 服务端状态：数据获取
  const { data, isLoading, error } = useQuery({
    queryKey: ["users"],
    queryFn: fetchUsers,
  });

  // 客户端状态：UI 状态
  const { sidebarOpen, toggleSidebar } = useUIStore();

  // ...
}

// ❌ 错误：用 useState 管理服务端数据，或用 Context 管理频繁变化的状态
export function UserList(): JSX.Element {
  const [users, setUsers] = useState<User[]>([]);  // 应该用 useQuery
  const [sidebarOpen, setSidebarOpen] = useState(false);  // 如果跨组件共享，应该用 Zustand
  // ...
}
```

### 3.4 事件处理

```typescript
// ✅ 正确：事件处理函数以 handle 开头，内联函数简单清晰
export function LoginForm(): JSX.Element {
  const handleSubmit = async (values: LoginFormData): Promise<void> => {
    try {
      await login(values);
    } catch (error) {
      toast.error("登录失败，请检查账号密码");
    }
  };

  return (
    <form onSubmit={form.handleSubmit(handleSubmit)}>
      {/* ... */}
    </form>
  );
}

// ❌ 错误：事件函数命名不清晰，或直接在 JSX 写复杂逻辑
export function LoginForm(): JSX.Element {
  return (
    <form onSubmit={async (e) => {
      // ❌ 复杂逻辑不应该内联在 JSX 中
      e.preventDefault();
      const values = getValues();
      await login(values);
    }}>
      {/* ... */}
    </form>
  );
}
```

### 3.5 条件渲染

```typescript
// ✅ 正确：提前返回处理 loading / error，主逻辑保持清晰
export function UserProfile({ userId }: { userId: string }): JSX.Element | null {
  const { data, isLoading, error } = useUser(userId);

  if (isLoading) {
    return <UserProfileSkeleton />;
  }

  if (error) {
    return <ErrorMessage message="加载用户信息失败" />;
  }

  if (!data) {
    return <EmptyState message="用户不存在" />;
  }

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.email}</p>
    </div>
  );
}

// ❌ 错误：在 JSX 中嵌套三元运算符，逻辑不清晰
export function UserProfile({ userId }: { userId: string }): JSX.Element {
  const { data, isLoading, error } = useUser(userId);

  return (
    <div>
      {isLoading ? (
        <Skeleton />
      ) : error ? (
        <Error />
      ) : data ? (
        <div>{data.name}</div>
      ) : (
        <Empty />
      )}
    </div>
  );
}
```

## 四、TypeScript 规则

### 4.1 类型安全

```typescript
// ✅ 正确：显式定义类型，禁止 any
interface ApiResponse<T> {
  success: boolean;
  data: T;
  message?: string;
}

async function fetchUser(id: string): Promise<ApiResponse<User>> {
  const response = await fetch(`/api/users/${id}`);
  return response.json() as Promise<ApiResponse<User>>;
}

// ❌ 错误：使用 any 绕过类型检查
async function fetchUser(id: string): Promise<any> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();  // 隐式 any
}
```

### 4.2 泛型使用

```typescript
// ✅ 正确：有意义的泛型参数名
function useApi<TData, TError = ApiError>(
  url: string
): UseQueryResult<TData, TError> {
  return useQuery<TData, TError>({ queryKey: [url], queryFn: () => fetch(url) });
}

// ❌ 错误：无意义的泛型参数名
function useApi<T, E = Error>(url: string): UseQueryResult<T, E> {
  // ...
}
```

### 4.3 类型推断与显式注解

```typescript
// ✅ 正确：简单场景用类型推断，复杂场景显式注解
const userCount = 10;                          // 推断为 number
const users: User[] = [];                      // 显式注解数组元素类型
const fetchUser = async (id: string): Promise<User> => {  // 显式注解返回类型
  // ...
};

// ❌ 错误：过度注解或完全不注解
const userCount: number = 10;                  // 多余
const users = [];                              // 推断为 never[]，有问题
const fetchUser = async (id) => {              // 参数和返回类型都是 any
  // ...
};
```

### 4.4 严格空值检查

```typescript
// ✅ 正确：处理 null / undefined
function getUserName(user: User | null): string {
  return user?.name ?? "匿名用户";
}

// ❌ 错误：忽略空值可能性
function getUserName(user: User): string {
  return user.name;  // 如果 user 可能为 null，会报错
}
```

## 五、服务端代码规范

### 5.1 API 响应格式

```typescript
// ✅ 正确：统一响应格式
interface ApiResponse<T> {
  success: boolean;
  data: T;
  message?: string;
  timestamp: string;
  requestId: string;
}

// 成功响应
{
  "success": true,
  "data": { "id": "1", "name": "张三" },
  "timestamp": "2026-06-26T10:00:00Z",
  "requestId": "req-123456"
}

// 错误响应
{
  "success": false,
  "data": null,
  "message": "用户不存在",
  "timestamp": "2026-06-26T10:00:00Z",
  "requestId": "req-123456"
}
```

### 5.2 错误处理

```typescript
// ✅ 正确：使用自定义错误类，统一错误处理
class AppError extends Error {
  constructor(
    public code: string,
    message: string,
    public statusCode: number = 500
  ) {
    super(message);
    this.name = "AppError";
  }
}

class NotFoundError extends AppError {
  constructor(resource: string) {
    super("NOT_FOUND", `${resource} 不存在`, 404);
  }
}

// 在 service 层抛出
async function getUserById(id: string): Promise<User> {
  const user = await db.query.users.findFirst({ where: eq(users.id, id) });
  if (!user) {
    throw new NotFoundError("用户");
  }
  return user;
}

// 在 middleware 统一捕获
app.onError((err, c) => {
  if (err instanceof AppError) {
    return c.json({ success: false, message: err.message }, err.statusCode);
  }
  return c.json({ success: false, message: "服务器内部错误" }, 500);
});
```

### 5.3 数据库查询

```typescript
// ✅ 正确：使用 Drizzle ORM，避免 N+1 问题
async function getVouchersWithItems(accountId: string): Promise<Voucher[]> {
  return db.query.vouchers.findMany({
    where: eq(vouchers.accountId, accountId),
    with: {
      items: true,  // 一次性关联查询
      creator: true,
    },
    orderBy: desc(vouchers.createdAt),
  });
}

// ❌ 错误：循环中查询数据库（N+1 问题）
async function getVouchersWithItems(accountId: string): Promise<Voucher[]> {
  const vouchers = await db.select().from(vouchers).where(eq(vouchers.accountId, accountId));
  for (const voucher of vouchers) {
    voucher.items = await db.select().from(voucherItems).where(eq(voucherItems.voucherId, voucher.id));
    // 每次循环都查询数据库！
  }
  return vouchers;
}
```

## 六、代码组织

### 6.1 文件长度

- 单个文件不超过 **300 行**
- 单个函数不超过 **50 行**
- 单个组件不超过 **200 行**
- 超过限制必须拆分

### 6.2 注释规范

```typescript
// ✅ 正确：解释「为什么」而不是「做了什么」
// 由于税务规则要求，金额必须保留两位小数
const amount = Number(value.toFixed(2));

// ❌ 错误：注释重复代码内容
// 设置用户名为张三
const userName = "张三";

// ✅ 正确：JSDoc 注释公共 API
/**
 * 根据 ID 获取用户信息
 * @param id - 用户唯一标识
 * @returns 用户对象，若不存在则抛出 NotFoundError
 * @throws {NotFoundError} 用户不存在时抛出
 */
export async function getUserById(id: string): Promise<User> {
  // ...
}
```

### 6.3 魔法数字与字符串

```typescript
// ✅ 正确：使用常量
const MAX_VOUCHER_ITEMS = 50;
const DEFAULT_PAGE_SIZE = 20;
const TOKEN_EXPIRES_IN = "7d";

if (items.length > MAX_VOUCHER_ITEMS) {
  throw new Error(`凭证明细不能超过 ${MAX_VOUCHER_ITEMS} 条`);
}

// ❌ 错误：魔法数字/字符串
if (items.length > 50) {
  throw new Error("凭证明细不能超过 50 条");
}
```

## 七、Git 提交规范

遵循 Conventional Commits：

| 类型 | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(invoice): 新增发票 OCR 识别` |
| `fix` | 修复 | `fix(contract): 修复合同状态流转异常` |
| `refactor` | 重构 | `refactor(db): 重构发票表索引` |
| `style` | 格式 | `style: 统一缩进` |
| `docs` | 文档 | `docs: 更新 CONVENTIONS` |
| `test` | 测试 | `test(invoice): 补充发票列表单测` |
| `chore` | 杂项 | `chore: 升级依赖` |
