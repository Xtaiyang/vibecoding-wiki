# 项目横切关注点说明

> 本文档定义了认证、日志、缓存、错误处理、安全等通用模块规范。AI 生成代码时，涉及这些横切关注点的逻辑，必须遵循本规范。

## 一、认证与授权

### 1.1 认证架构

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    User     │────▶│   Login     │────▶│   JWT       │
│  (Browser)  │     │   API       │     │   Issued    │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
┌─────────────┐     ┌─────────────┐            │
│   Access    │◀────│  API Call   │◀───────────┘
│   Resource  │     │ (Bearer Token)
└─────────────┘     └─────────────┘
```

### 1.2 JWT 规范

```typescript
// Token 结构
interface JWTPayload {
  sub: string;          // 用户 ID
  email: string;        // 用户邮箱
  role: string;         // 角色
  accountId?: string;   // 当前账套 ID
  iat: number;          // 签发时间
  exp: number;          // 过期时间
}

// Token 配置
const JWT_CONFIG = {
  accessTokenSecret: process.env.JWT_ACCESS_SECRET!,
  refreshTokenSecret: process.env.JWT_REFRESH_SECRET!,
  accessTokenExpiresIn: "15m",    // Access Token 15 分钟
  refreshTokenExpiresIn: "7d",    // Refresh Token 7 天
} as const;
```

### 1.3 认证流程

```typescript
// 1. 登录签发 Token
async function login(email: string, password: string): Promise<AuthResult> {
  const user = await authenticateUser(email, password);
  const accessToken = await jwt.sign(
    { sub: user.id, email: user.email, role: user.role },
    JWT_CONFIG.accessTokenSecret,
    { expiresIn: JWT_CONFIG.accessTokenExpiresIn }
  );
  const refreshToken = await jwt.sign(
    { sub: user.id },
    JWT_CONFIG.refreshTokenSecret,
    { expiresIn: JWT_CONFIG.refreshTokenExpiresIn }
  );
  // 刷新令牌存入 Redis（支持后端吊销）
  await redis.set(`refresh:${user.id}`, refreshToken, "EX", 7 * 24 * 3600);
  return { accessToken, refreshToken, user };
}

// 2. 请求时校验 Token
// middleware/auth.ts
export async function authMiddleware(c: Context, next: Next) {
  const token = c.req.header("Authorization")?.replace("Bearer ", "");
  if (!token) {
    throw new UnauthorizedError("缺少认证令牌");
  }
  try {
    const payload = await jwt.verify(token, JWT_CONFIG.accessTokenSecret);
    c.set("user", payload);
    await next();
  } catch {
    throw new UnauthorizedError("认证令牌无效或已过期");
  }
}

// 3. Token 刷新
async function refreshToken(refreshToken: string): Promise<AuthResult> {
  const payload = await jwt.verify(refreshToken, JWT_CONFIG.refreshTokenSecret);
  const stored = await redis.get(`refresh:${payload.sub}`);
  if (stored !== refreshToken) {
    throw new UnauthorizedError("刷新令牌无效");
  }
  // 签发新 Token
  return login(payload.sub);
}
```

### 1.4 权限控制（RBAC）

```typescript
// roles.ts
export enum Role {
  OWNER = "owner",           // 企业主
  ACCOUNTANT = "accountant", // 会计
  CASHIER = "cashier",       // 出纳
  VIEWER = "viewer",         // 查看者
}

// permissions.ts
export enum Permission {
  VOUCHER_CREATE = "voucher:create",
  VOUCHER_UPDATE = "voucher:update",
  VOUCHER_DELETE = "voucher:delete",
  VOUCHER_VIEW = "voucher:view",
  INVOICE_MANAGE = "invoice:manage",
  REPORT_VIEW = "report:view",
  SETTINGS_MANAGE = "settings:manage",
  USER_MANAGE = "user:manage",
}

// 角色权限映射
const ROLE_PERMISSIONS: Record<Role, Permission[]> = {
  [Role.OWNER]: Object.values(Permission),
  [Role.ACCOUNTANT]: [
    Permission.VOUCHER_CREATE,
    Permission.VOUCHER_UPDATE,
    Permission.VOUCHER_DELETE,
    Permission.VOUCHER_VIEW,
    Permission.INVOICE_MANAGE,
    Permission.REPORT_VIEW,
  ],
  [Role.CASHIER]: [
    Permission.VOUCHER_CREATE,
    Permission.VOUCHER_VIEW,
    Permission.INVOICE_MANAGE,
  ],
  [Role.VIEWER]: [
    Permission.VOUCHER_VIEW,
    Permission.REPORT_VIEW,
  ],
};

// middleware/rbac.ts
export function requirePermission(...requiredPermissions: Permission[]) {
  return async (c: Context, next: Next) => {
    const user = c.get("user") as JWTPayload;
    const permissions = ROLE_PERMISSIONS[user.role as Role] || [];
    const hasPermission = requiredPermissions.every(p => permissions.includes(p));
    if (!hasPermission) {
      throw new ForbiddenError("权限不足");
    }
    await next();
  };
}
```

### 1.5 前端认证状态

```typescript
// stores/authStore.ts
import { create } from "zustand";

interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  login: (credentials: LoginInput) => Promise<void>;
  logout: () => void;
  refreshToken: () => Promise<void>;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  isAuthenticated: false,
  login: async (credentials) => {
    const { user, accessToken, refreshToken } = await api.auth.login(credentials);
    localStorage.setItem("accessToken", accessToken);
    localStorage.setItem("refreshToken", refreshToken);
    set({ user, isAuthenticated: true });
  },
  logout: () => {
    localStorage.removeItem("accessToken");
    localStorage.removeItem("refreshToken");
    set({ user: null, isAuthenticated: false });
  },
  refreshToken: async () => {
    const refreshToken = localStorage.getItem("refreshToken");
    if (!refreshToken) return;
    const { accessToken } = await api.auth.refresh({ refreshToken });
    localStorage.setItem("accessToken", accessToken);
  },
}));
```

## 二、日志规范

### 2.1 日志级别

| 级别 | 用途 | 输出位置 |
|-----|------|---------|
| `error` | 系统错误、异常、需要立即处理的问题 | 控制台 + 文件 + 告警系统 |
| `warn` | 潜在问题、非预期但可恢复的情况 | 控制台 + 文件 |
| `info` | 关键业务流程记录（登录、创建凭证等） | 控制台 + 文件 |
| `debug` | 开发调试信息 | 仅控制台（开发环境） |

### 2.2 日志内容规范

```typescript
// ✅ 正确：结构化日志，包含上下文
logger.info({
  event: "voucher_created",
  userId: "user-123",
  accountId: "account-456",
  voucherId: "voucher-789",
  amount: 10000.00,
  timestamp: new Date().toISOString(),
});

// ❌ 错误：无意义的日志
console.log("ok");
console.log(data);  // 直接打印对象，无上下文
```

### 2.3 禁止行为

- ❌ 禁止使用 `console.log` / `console.error` 等原生方法
- ❌ 禁止在日志中输出密码、Token、身份证号等敏感信息
- ❌ 禁止在循环中打印日志（造成日志风暴）

## 三、缓存策略

### 3.1 缓存分层

| 层级 | 技术 | 用途 | 过期时间 |
|-----|------|------|---------|
| 浏览器缓存 | HTTP Cache-Control | 静态资源、图片 | 1d ~ 30d |
| CDN 缓存 | CDN 边缘节点 | 静态资源 | 1h ~ 24h |
| 应用缓存 | Redis | Session、Token、热点数据 | 15m ~ 7d |
| 数据库缓存 | PostgreSQL Buffer | 查询结果 | 由数据库管理 |

### 3.2 Redis 使用规范

```typescript
// ✅ 正确：使用前缀命名空间，避免 Key 冲突
const CACHE_KEYS = {
  user: (id: string) => `user:${id}`,
  account: (id: string) => `account:${id}`,
  voucherList: (accountId: string) => `vouchers:${accountId}:list`,
  report: (accountId: string, type: string) => `report:${accountId}:${type}`,
  session: (token: string) => `session:${token}`,
  rateLimit: (ip: string) => `rate_limit:${ip}`,
} as const;

// 写入缓存
await redis.setex(CACHE_KEYS.user(userId), 3600, JSON.stringify(user));

// 读取缓存
const cached = await redis.get(CACHE_KEYS.user(userId));
if (cached) {
  return JSON.parse(cached);
}

// 删除缓存（数据更新时）
await redis.del(CACHE_KEYS.user(userId));
```

### 3.3 缓存更新策略

```typescript
// Cache-Aside 模式（旁路缓存）
async function getUserWithCache(userId: string): Promise<User> {
  // 1. 先读缓存
  const cached = await redis.get(CACHE_KEYS.user(userId));
  if (cached) {
    return JSON.parse(cached);
  }
  // 2. 缓存未命中，读数据库
  const user = await db.query.users.findFirst({
    where: eq(users.id, userId),
  });
  if (!user) {
    throw new NotFoundError("用户");
  }
  // 3. 写入缓存
  await redis.setex(CACHE_KEYS.user(userId), 3600, JSON.stringify(user));
  return user;
}

// 更新数据时同步清除缓存
async function updateUser(userId: string, data: UpdateUserInput): Promise<User> {
  const updated = await db.update(users)
    .set(data)
    .where(eq(users.id, userId))
    .returning();
  // 清除缓存，下次读取时重建
  await redis.del(CACHE_KEYS.user(userId));
  return updated[0];
}
```

## 四、错误处理

### 4.1 错误分类

| 错误类型 | HTTP 状态码 | 场景 |
|---------|-----------|------|
| `ValidationError` | 400 | 参数校验失败 |
| `UnauthorizedError` | 401 | 未认证、Token 过期 |
| `ForbiddenError` | 403 | 权限不足 |
| `NotFoundError` | 404 | 资源不存在 |
| `ConflictError` | 409 | 资源冲突（如重复创建） |
| `TooManyRequestsError` | 429 | 请求过于频繁 |
| `InternalError` | 500 | 服务器内部错误 |

### 4.2 错误类定义

```typescript
// lib/errors.ts
export class AppError extends Error {
  constructor(
    public code: string,
    message: string,
    public statusCode: number = 500
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class ValidationError extends AppError {
  constructor(message: string) {
    super("VALIDATION_ERROR", message, 400);
  }
}

export class UnauthorizedError extends AppError {
  constructor(message: string = "未认证") {
    super("UNAUTHORIZED", message, 401);
  }
}

export class ForbiddenError extends AppError {
  constructor(message: string = "权限不足") {
    super("FORBIDDEN", message, 403);
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super("NOT_FOUND", `${resource} 不存在`, 404);
  }
}

export class ConflictError extends AppError {
  constructor(message: string) {
    super("CONFLICT", message, 409);
  }
}
```

### 4.3 全局错误处理

```typescript
// middleware/error-handler.ts
export function errorHandler(err: Error, c: Context) {
  // 记录错误日志
  if (err instanceof AppError) {
    logger.error({
      code: err.code,
      message: err.message,
      statusCode: err.statusCode,
      stack: err.stack,
      path: c.req.path,
      method: c.req.method,
    });
    return c.json(
      { success: false, message: err.message, code: err.code },
      err.statusCode as StatusCode
    );
  }

  // 未知错误
  logger.error({
    message: err.message,
    stack: err.stack,
    path: c.req.path,
    method: c.req.method,
  });
  return c.json(
    { success: false, message: "服务器内部错误", code: "INTERNAL_ERROR" },
    500
  );
}
```

### 4.4 前端错误处理

```typescript
// lib/api-client.ts
import { toast } from "sonner";

apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    const status = error.response?.status;
    const message = error.response?.data?.message || "请求失败";

    switch (status) {
      case 400:
        toast.error(`参数错误: ${message}`);
        break;
      case 401:
        toast.error("登录已过期，请重新登录");
        useAuthStore.getState().logout();
        window.location.href = "/login";
        break;
      case 403:
        toast.error("权限不足，无法执行此操作");
        break;
      case 404:
        toast.error("请求的资源不存在");
        break;
      case 429:
        toast.error("请求过于频繁，请稍后再试");
        break;
      case 500:
        toast.error("服务器内部错误，请联系管理员");
        break;
      default:
        toast.error(message);
    }
    return Promise.reject(error);
  }
);
```

## 五、安全规范

### 5.1 输入校验

```typescript
// ✅ 正确：所有外部输入必须经过 Zod 校验
import { z } from "zod";

const loginSchema = z.object({
  email: z.string().email("邮箱格式不正确"),
  password: z.string().min(8, "密码至少 8 位").max(128),
});

export async function login(c: Context) {
  const body = await c.req.json();
  const result = loginSchema.safeParse(body);
  if (!result.success) {
    throw new ValidationError(result.error.errors[0].message);
  }
  // ...
}
```

### 5.2 SQL 注入防护

```typescript
// ✅ 正确：使用 Drizzle ORM 参数化查询
await db.select().from(users).where(eq(users.email, email));

// ❌ 错误：字符串拼接 SQL
await db.execute(`SELECT * FROM users WHERE email = '${email}'`);
```

### 5.3 XSS 防护

```typescript
// ✅ 正确：前端输出时转义（React 默认转义）
// React 自动转义 JSX 中的变量，不需要手动处理

// ❌ 错误：使用 dangerouslySetInnerHTML
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// 如果必须使用，先进行 DOMPurify 消毒
import DOMPurify from "dompurify";
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />
```

### 5.4 CSRF 防护

```typescript
// 使用 SameSite Cookie
Set-Cookie: sessionId=xxx; SameSite=Strict; Secure; HttpOnly

// API 校验 Origin/Referer
export function csrfProtection(c: Context, next: Next) {
  const origin = c.req.header("Origin");
  const allowedOrigins = ["https://finance-platform.com"];
  if (origin && !allowedOrigins.includes(origin)) {
    throw new ForbiddenError("非法来源");
  }
  return next();
}
```

### 5.5 敏感数据处理

```typescript
// ✅ 正确：密码使用 bcrypt 加密
import bcrypt from "bcryptjs";

const SALT_ROUNDS = 12;

export async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, SALT_ROUNDS);
}

export async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return bcrypt.compare(password, hash);
}

// ✅ 正确：返回数据时过滤敏感字段
export function sanitizeUser(user: User): SafeUser {
  const { passwordHash, refreshToken, ...safeUser } = user;
  return safeUser;
}
```

### 5.6 文件上传安全

```typescript
// ✅ 正确：限制文件类型、大小，重命名文件名
const ALLOWED_MIME_TYPES = ["image/jpeg", "image/png", "application/pdf"];
const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB

export async function uploadFile(file: File): Promise<string> {
  if (!ALLOWED_MIME_TYPES.includes(file.type)) {
    throw new ValidationError("不支持的文件类型");
  }
  if (file.size > MAX_FILE_SIZE) {
    throw new ValidationError("文件大小超过限制");
  }
  // 使用 UUID 重命名，避免路径遍历
  const ext = path.extname(file.name);
  const filename = `${uuidv4()}${ext}`;
  // 上传到 MinIO...
}
```

## 六、限流策略

```typescript
// middleware/rate-limiter.ts
import { Ratelimit } from "@upstash/ratelimit";

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, "10 s"), // 每 10 秒 10 次
});

export async function rateLimitMiddleware(c: Context, next: Next) {
  const ip = c.req.header("x-forwarded-for") || "anonymous";
  const { success, limit, remaining, reset } = await ratelimit.limit(ip);

  c.header("X-RateLimit-Limit", limit.toString());
  c.header("X-RateLimit-Remaining", remaining.toString());
  c.header("X-RateLimit-Reset", reset.toString());

  if (!success) {
    throw new TooManyRequestsError("请求过于频繁，请稍后再试");
  }

  await next();
}
```

## 七、关联文档

- [ARCHITECTURE.md](./ARCHITECTURE.md) - 架构分层说明
- [INTEGRATIONS.md](./INTEGRATIONS.md) - 第三方服务集成（Redis、MinIO 配置）
- [CONVENTIONS.md](./CONVENTIONS.md) - 编码规范
