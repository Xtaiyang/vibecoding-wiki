# 第三方集成方案说明

> 本文档定义了数据库、认证、AI、存储、邮件、支付等第三方服务的集成规范。AI 生成代码时，涉及第三方服务调用的逻辑，必须遵循本规范。

## 一、数据库集成（PostgreSQL）

### 1.1 连接配置

```typescript
// packages/db/src/client.ts
import { drizzle } from "drizzle-orm/node-postgres";
import { Pool } from "pg";
import * as schema from "./schema";

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,                    // 最大连接数
  idleTimeoutMillis: 30000,   // 空闲连接超时
  connectionTimeoutMillis: 5000, // 连接超时
});

export const db = drizzle(pool, { schema });
```

### 1.2 环境变量

```env
DATABASE_URL=postgresql://user:password@host:port/database?sslmode=require
```

### 1.3 使用规范

```typescript
// ✅ 正确：使用 Drizzle ORM 进行类型安全的数据库操作
import { db } from "@repo/db";
import { users } from "@repo/db/schema";
import { eq, desc, and } from "drizzle-orm";

// 查询
const user = await db.query.users.findFirst({
  where: eq(users.id, userId),
});

// 插入
const newUser = await db.insert(users)
  .values({ email, name, passwordHash })
  .returning();

// 更新
await db.update(users)
  .set({ name: newName })
  .where(eq(users.id, userId));

// 删除
await db.delete(users).where(eq(users.id, userId));

// 事务
await db.transaction(async (tx) => {
  const voucher = await tx.insert(vouchers).values(voucherData).returning();
  await tx.insert(voucherItems).values(items.map(item => ({
    ...item,
    voucherId: voucher[0].id,
  })));
});
```

## 二、缓存集成（Redis）

### 2.1 连接配置

```typescript
// apps/api/src/lib/redis.ts
import { Redis } from "ioredis";

export const redis = new Redis(process.env.REDIS_URL, {
  maxRetriesPerRequest: 3,
  enableReadyCheck: true,
});

redis.on("error", (err) => {
  logger.error({ message: "Redis connection error", error: err.message });
});
```

### 2.2 环境变量

```env
REDIS_URL=redis://localhost:6379/0
```

### 2.3 使用场景

- Session 存储
- JWT Refresh Token 存储
- 热点数据缓存（用户列表、报表数据）
- 限流计数器
- 分布式锁

## 三、对象存储集成（MinIO）

### 3.1 连接配置

```typescript
// apps/api/src/lib/minio.ts
import { Client } from "minio";

export const minioClient = new Client({
  endPoint: process.env.MINIO_ENDPOINT!,
  port: Number(process.env.MINIO_PORT!),
  useSSL: process.env.NODE_ENV === "production",
  accessKey: process.env.MINIO_ACCESS_KEY!,
  secretKey: process.env.MINIO_SECRET_KEY!,
});

export const BUCKET_NAME = process.env.MINIO_BUCKET!;
```

### 3.2 环境变量

```env
MINIO_ENDPOINT=localhost
MINIO_PORT=9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET=finance-platform
```

### 3.3 使用规范

```typescript
// ✅ 正确：文件上传
export async function uploadFile(file: Buffer, filename: string, contentType: string): Promise<string> {
  await minioClient.putObject(BUCKET_NAME, filename, file, file.length, {
    "Content-Type": contentType,
  });
  return `${process.env.MINIO_PUBLIC_URL}/${BUCKET_NAME}/${filename}`;
}

// ✅ 正确：生成预签名 URL（前端直接上传）
export async function getPresignedUploadUrl(filename: string, contentType: string): Promise<string> {
  return minioClient.presignedPutObject(BUCKET_NAME, filename, 3600, {
    "Content-Type": contentType,
  });
}

// ✅ 正确：生成预签名下载 URL
export async function getPresignedDownloadUrl(filename: string): Promise<string> {
  return minioClient.presignedGetObject(BUCKET_NAME, filename, 3600);
}
```

## 四、AI 服务集成

### 4.1 发票识别（OCR + AI）

```typescript
// apps/api/src/services/ai-service.ts
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface InvoiceOCRResult {
  invoiceNumber: string;
  invoiceDate: string;
  sellerName: string;
  buyerName: string;
  amount: number;
  taxAmount: number;
  totalAmount: number;
}

export async function recognizeInvoice(imageBase64: string): Promise<InvoiceOCRResult> {
  const response = await openai.chat.completions.create({
    model: "gpt-4o",
    messages: [
      {
        role: "system",
        content: "你是一个专业的发票识别助手。请从图片中提取发票信息，以 JSON 格式返回。",
      },
      {
        role: "user",
        content: [
          { type: "text", text: "请识别这张发票的信息" },
          { type: "image_url", image_url: { url: `data:image/jpeg;base64,${imageBase64}` } },
        ],
      },
    ],
    response_format: { type: "json_object" },
  });

  return JSON.parse(response.choices[0].message.content!);
}
```

### 4.2 智能凭证生成

```typescript
export async function generateVoucherSuggestion(
  description: string,
  amount: number,
  type: "income" | "expense"
): Promise<{ subjectId: string; debit: number; credit: number }[]> {
  const response = await openai.chat.completions.create({
    model: "gpt-4o-mini",
    messages: [
      {
        role: "system",
        content: `根据交易描述和金额，生成会计分录建议。返回 JSON 数组，每个元素包含 subjectId、debit、credit。`,
      },
      {
        role: "user",
        content: `描述: ${description}, 金额: ${amount}, 类型: ${type}`,
      },
    ],
    response_format: { type: "json_object" },
  });

  return JSON.parse(response.choices[0].message.content!).entries;
}
```

### 4.3 环境变量

```env
OPENAI_API_KEY=sk-...
OPENAI_BASE_URL=https://api.openai.com/v1  # 可配置为代理地址
```

### 4.4 降级策略

```typescript
// ✅ 正确：AI 服务故障时降级为手动输入
export async function recognizeInvoiceWithFallback(imageBase64: string): Promise<InvoiceOCRResult | null> {
  try {
    return await recognizeInvoice(imageBase64);
  } catch (error) {
    logger.warn({ event: "invoice_recognition_failed", error: error.message });
    // 返回 null，前端提示用户手动输入
    return null;
  }
}
```

## 五、邮件服务集成

### 5.1 SMTP 配置

```typescript
// apps/api/src/lib/email.ts
import nodemailer from "nodemailer";

const transporter = nodemailer.createTransporter({
  host: process.env.SMTP_HOST,
  port: Number(process.env.SMTP_PORT),
  secure: true,
  auth: {
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS,
  },
});

export async function sendEmail(options: SendEmailOptions): Promise<void> {
  await transporter.sendMail({
    from: `"智慧共享财务平台" <${process.env.SMTP_USER}>`,
    to: options.to,
    subject: options.subject,
    html: options.html,
  });
}
```

### 5.2 环境变量

```env
SMTP_HOST=smtp.example.com
SMTP_PORT=465
SMTP_USER=noreply@finance-platform.com
SMTP_PASS=your-smtp-password
```

### 5.3 使用场景

- 注册验证码邮件
- 密码重置邮件
- 月度财务报表邮件推送
- 系统通知邮件

## 六、短信服务集成（预留）

### 6.1 配置

```env
SMS_PROVIDER=aliyun  # 或 tencent
SMS_ACCESS_KEY_ID=...
SMS_ACCESS_KEY_SECRET=...
SMS_SIGN_NAME=智慧共享财务
```

### 6.2 使用场景

- 登录二次验证
- 重要操作通知（大额转账、密码修改）

## 七、支付服务集成（预留）

### 7.1 配置

```env
PAYMENT_PROVIDER=alipay  # 或 wechat
PAYMENT_APP_ID=...
PAYMENT_PRIVATE_KEY=...
PAYMENT_PUBLIC_KEY=...
```

### 7.2 使用场景

- SaaS 订阅付费
- 增值服务购买

## 八、第三方集成通用规范

### 8.1 密钥管理

- 所有第三方服务的密钥必须通过环境变量配置
- 禁止将密钥硬编码在代码中
- 生产环境密钥与开发环境密钥分离
- 定期轮换密钥

### 8.2 超时与重试

```typescript
// ✅ 正确：所有外部调用必须设置超时和重试
import axios from "axios";

const apiClient = axios.create({
  timeout: 10000,  // 10 秒超时
});

// 重试配置
async function callWithRetry<T>(fn: () => Promise<T>, retries = 3): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    if (retries <= 0) throw error;
    await new Promise(resolve => setTimeout(resolve, 1000 * (4 - retries)));
    return callWithRetry(fn, retries - 1);
  }
}
```

### 8.3 降级与熔断

```typescript
// ✅ 正确：第三方服务不可用时优雅降级
class CircuitBreaker {
  private failures = 0;
  private lastFailureTime?: number;
  private readonly threshold = 5;
  private readonly timeout = 60000; // 1 分钟

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.isOpen()) {
      throw new Error("Circuit breaker is open");
    }
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private isOpen(): boolean {
    if (this.failures < this.threshold) return false;
    if (!this.lastFailureTime) return false;
    return Date.now() - this.lastFailureTime < this.timeout;
  }

  private onSuccess(): void {
    this.failures = 0;
    this.lastFailureTime = undefined;
  }

  private onFailure(): void {
    this.failures++;
    this.lastFailureTime = Date.now();
  }
}
```

## 九、关联文档

- [ARCHITECTURE.md](./ARCHITECTURE.md) - 架构分层说明
- [CONCERNS.md](./CONCERNS.md) - 横切关注点（安全、日志）
- [TECH-STACK.md](./TECH-STACK.md) - 技术选型说明
