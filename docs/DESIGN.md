# 页面设计系统规范

> 基于 shadcn/ui + Tailwind CSS 构建的设计系统，为智慧共享财务平台提供统一的视觉语言与组件规范。

---

## 1. Design Philosophy（设计理念）

**核心原则**: 信息优先，操作高效 — 让财务数据一目了然，让关键操作触手可及。

**设计方向**: professional, data-dense, clarity-first, trustworthy

**个性特征**: precise, calm, dependable, efficient

**设计参考**: shadcn/ui（组件基线）+ Tailwind CSS（原子化工具）+ 金融行业标准视觉规范

**设计原则**:

1. **数据可读** — 财务数据是核心，表格、图表、金额等关键信息必须清晰可辨。数字使用等宽字体对齐，金额保留统一小数位。
2. **操作高效** — 高频操作（凭证录入、审批、导出）路径最短化，CTA 按钮目标 ≥ 36px，分布在自然操作路径上。
3. **状态明确** — 系统状态（加载、成功、失败、待处理）通过色彩 + 图标 + 文字三重编码，减少认知负担。
4. **长时间舒适** — 大面积中性色背景，避免高亮度色块；正文 16px + 1.6 行高，降低长时间操作疲劳。
5. **专业克制** — 不使用渐变、阴影克制、无装饰性插画。专业感来自精确的排版和清晰的信息层级。

---

## 2. Color Palette（调色板）

基于 shadcn/ui 的 CSS 变量体系，使用 HSL 格式定义，便于主题切换（Light / Dark）。

### 2.1 主题色定义

```css
:root {
  /* ===== 基础色 ===== */
  --background: 0 0% 100%;           /* 页面背景 */
  --foreground: 222.2 84% 4.9%;      /* 主文本 */

  --card: 0 0% 100%;                 /* 卡片背景 */
  --card-foreground: 222.2 84% 4.9%; /* 卡片文本 */

  --popover: 0 0% 100%;              /* 弹出层背景 */
  --popover-foreground: 222.2 84% 4.9%;

  --primary: 221.2 83.2% 53.3%;      /* 主色 — 可信蓝 */
  --primary-foreground: 210 40% 98%; /* 主色上文本（白） */

  --secondary: 210 40% 96.1%;        /* 次级背景 */
  --secondary-foreground: 222.2 47.4% 11.2%;

  --muted: 210 40% 96.1%;            /* 柔和背景 */
  --muted-foreground: 215.4 16.3% 46.9%;

  --accent: 210 40% 96.1%;           /* 强调背景 */
  --accent-foreground: 222.2 47.4% 11.2%;

  --destructive: 0 84.2% 60.2%;      /* 危险/删除色 */
  --destructive-foreground: 210 40% 98%;

  /* ===== 边框与输入 ===== */
  --border: 214.3 31.8% 91.4%;
  --input: 214.3 31.8% 91.4%;
  --ring: 221.2 83.2% 53.3%;

  /* ===== 圆角 ===== */
  --radius: 0.5rem;
}

.dark {
  --background: 222.2 84% 4.9%;
  --foreground: 210 40% 98%;

  --card: 222.2 84% 4.9%;
  --card-foreground: 210 40% 98%;

  --popover: 222.2 84% 4.9%;
  --popover-foreground: 210 40% 98%;

  --primary: 217.2 91.2% 59.8%;
  --primary-foreground: 222.2 47.4% 11.2%;

  --secondary: 217.2 32.6% 17.5%;
  --secondary-foreground: 210 40% 98%;

  --muted: 217.2 32.6% 17.5%;
  --muted-foreground: 215 20.2% 65.1%;

  --accent: 217.2 32.6% 17.5%;
  --accent-foreground: 210 40% 98%;

  --destructive: 0 62.8% 30.6%;
  --destructive-foreground: 210 40% 98%;

  --border: 217.2 32.6% 17.5%;
  --input: 217.2 32.6% 17.5%;
  --ring: 224.3 76.3% 48%;
}
```

### 2.2 语义色扩展

在 shadcn/ui 基础上，为财务场景扩展的语义色：

| Token               | Light HEX | Dark HEX  | CSS Variable                | Usage                     |
| ------------------- | --------- | --------- | --------------------------- | ------------------------- |
| Success             | #16A34A   | #22C55E   | --color-success             | 操作成功、审核通过        |
| Success-BG          | #F0FDF4   | #052E16   | --color-success-bg          | 成功浅色背景              |
| Warning             | #CA8A04   | #EAB308   | --color-warning             | 警告提示、待处理          |
| Warning-BG          | #FEFCE8   | #422006   | --color-warning-bg          | 警告浅色背景              |
| Danger              | #DC2626   | #EF4444   | --color-danger              | 错误、逾期、删除          |
| Danger-BG           | #FEF2F2   | #450A0A   | --color-danger-bg           | 错误浅色背景              |
| Info                | #2563EB   | #3B82F6   | --color-info                | 信息提示、说明            |
| Info-BG             | #EFF6FF   | #172554   | --color-info-bg             | 信息浅色背景              |
| Money-Positive      | #16A34A   | #22C55E   | --color-money-positive      | 收入、正向金额            |
| Money-Negative      | #DC2626   | #EF4444   | --color-money-negative      | 支出、负向金额            |
| Highlight-BG        | #FEF9C3   | #713F12   | --color-highlight-bg        | 搜索高亮、关键数据标注    |
| AI-Accent           | #7C3AED   | #A78BFA   | --color-ai-accent           | AI 生成内容标识色（紫色）  |
| AI-BG               | #F5F3FF   | #2E1065   | --color-ai-bg               | AI 推理过程区域底色        |

### 2.3 色彩使用规则

1. **主色（Primary）用于主要 CTA** — 确认、提交、创建等主操作按钮，以及链接、活跃态。
2. **Destructive 仅用于破坏性操作** — 删除、撤销等不可逆操作。普通错误提示用 Warning。
3. **语义色全局一致** — Success/Warning/Danger/Info 的含义贯穿所有组件，不可混用。
4. **金额色遵循财务惯例** — 收入绿色、支出红色，不可反转。
5. **AI 内容使用紫色标识** — 与主色蓝色区分，明确标记 AI 生成内容。

---

## 3. Typography（排版）

### 3.1 字体栈

```css
/* 正文 — 中文优先，跨平台兼容 */
--font-sans: "Inter", "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei",
             "Source Han Sans SC", "Noto Sans SC", system-ui, sans-serif;

/* 等宽字体 — 金额、编号、数据 */
--font-mono: "JetBrains Mono", "SF Mono", "Cascadia Code", "Source Code Pro",
             Consolas, "Microsoft YaHei", monospace;
```

> **选型理由**: Inter 作为西文主字体，x-height 高，数字等宽（tabular figures），适合数据密集的财务界面。中文字体栈覆盖 macOS（PingFang SC）、Windows（Microsoft YaHei）和跨平台 fallback，无需加载 Web 字体。等宽字体用于金额、编号等对齐要求高的场景。

### 3.2 字号层级

| Level   | Tailwind Class | Size   | Weight | Line-height | Usage                     |
| ------- | -------------- | ------ | ------ | ----------- | ------------------------- |
| H1      | text-3xl       | 30px   | 700    | 1.2         | 页面主标题                |
| H2      | text-2xl       | 24px   | 600    | 1.3         | 区块标题                  |
| H3      | text-xl        | 20px   | 600    | 1.35        | 子区块标题、卡片标题      |
| H4      | text-lg        | 18px   | 600    | 1.4         | 小标题、表头              |
| Body-L  | text-base+     | 16px   | 500    | 1.6         | 关键内容、金额、重要说明  |
| Body    | text-base      | 16px   | 400    | 1.6         | 默认正文                  |
| Body-S  | text-sm        | 14px   | 400    | 1.5         | 辅助文本、列表项          |
| Small   | text-xs        | 12px   | 400    | 1.5         | 标签、元信息、时间戳      |
| Caption | text-[11px]    | 11px   | 400    | 1.4         | 最次级注释、免责提示      |

### 3.3 字重使用规则

| Weight | Tailwind    | Usage                      |
| ------ | ----------- | -------------------------- |
| 400    | font-normal | 正文、说明文字             |
| 500    | font-medium | 标签、按钮文字、强调正文   |
| 600    | font-semibold | 区块标题、表头、重要数据  |
| 700    | font-bold   | 页面标题、关键数字         |

### 3.4 排版特殊规则

- **数字等宽**: 所有金额、编号使用 `font-variant-numeric: tabular-nums`（Tailwind: `tabular-nums`），避免数字跳动
- **金额格式**: 统一保留两位小数，千位使用逗号分隔（如 `1,234.56`），负数用括号或减号
- **中英文混排**: 英文/数字与中文之间不加额外空格，由排版引擎自动处理
- **字间距**: 中文不设 letter-spacing，英文标题可设 `-0.01em` 微调

---

## 4. Component Styles（组件样式）

所有组件基于 shadcn/ui，通过 Tailwind CSS 类名 + CSS 变量定制。不使用自定义 CSS 类，保持与 shadcn/ui 生态一致。

### 4.1 按钮（Button）

使用 `shadcn/ui Button` 组件，支持以下 variant：

| Variant     | 使用场景                   | 示例                              |
| ----------- | -------------------------- | --------------------------------- |
| default     | 主操作（提交、保存、创建） | 新建凭证、保存、确认              |
| secondary   | 次级操作（导出、返回）     | 导出 Excel、返回列表              |
| destructive | 破坏性操作（删除、撤销）   | 删除凭证、撤销审核                |
| outline     | 边框按钮（筛选、切换）     | 高级筛选、切换视图                |
| ghost       | 幽灵按钮（取消、关闭）     | 取消、关闭弹窗                    |
| link        | 链接按钮                   | 查看详情、了解更多                |

**尺寸规范**:

| Size   | 高度  | 内边距         | 字号   | 使用场景          |
| ------ | ----- | -------------- | ------ | ----------------- |
| sm     | 32px  | h-8 px-3       | 13px   | 表格内操作、标签  |
| default | 36px | h-9 px-4       | 14px   | 默认按钮          |
| lg     | 40px  | h-10 px-6      | 15px   | 页面主 CTA        |
| icon   | 36px  | h-9 w-9        | —      | 图标按钮          |

**交互规范**:
- 所有按钮必须包含 hover / focus-visible / disabled / active 状态
- CTA 按钮目标尺寸 ≥ 36px
- 图标按钮使用 Lucide React 图标，16px / 20px
- 加载中状态使用 `Button` + `Loader2` 图标旋转动画，禁用点击

### 4.2 表单输入

使用 `shadcn/ui Input` / `Textarea` / `Select` / `DatePicker` 组件。

| 组件       | 使用场景                           | 特殊要求                        |
| ---------- | ---------------------------------- | ------------------------------- |
| Input      | 单行文本、搜索、金额              | 金额输入右对齐，tabular-nums    |
| Textarea   | 多行文本、备注、描述              | 可选字数统计                    |
| Select     | 下拉选择（科目、客户、状态）      | 支持搜索过滤                    |
| DatePicker | 日期选择（记账日期、到期日）      | 支持快捷选项（今天、本月）      |
| Combobox   | 可搜索下拉（大量选项）            | 支持键盘导航                    |
| Checkbox   | 多选（凭证标记、权限勾选）        | —                               |
| RadioGroup | 单选（报表类型、导出格式）        | —                               |
| Switch     | 开关（启用/禁用、自动/手动）      | —                               |

**表单布局规则**:
- 标签在输入框上方（垂直布局），不使用行内标签
- 必填字段标签后加 `*` 标记，红色
- 验证错误信息在输入框下方显示，红色文本
- 使用 `React Hook Form` + `Zod` 进行表单校验

### 4.3 数据表格（DataTable）

使用 `TanStack Table` + `shadcn/ui Table` 组件封装。

| 特性         | 规范                                                |
| ------------ | --------------------------------------------------- |
| 行高         | 紧凑 36px / 默认 44px / 宽松 52px，默认使用 44px    |
| 表头         | 背景色 muted，文本 font-medium text-sm              |
| 金额列       | 右对齐，tabular-nums，正数绿色/负数红色              |
| 排序         | 表头可点击排序，显示排序方向图标                      |
| 筛选         | 列头下拉筛选或顶部筛选栏                              |
| 分页         | 底部分页器，默认每页 20 条，可切换 10/20/50/100      |
| 空状态       | 显示空状态插图 + 提示文字 + 操作按钮                  |
| 加载状态     | 表格行 skeleton 动画                                  |
| 行操作       | 最右侧操作列，hover 时显示，不超过 3 个操作           |
| 行选择       | 左侧 Checkbox 列，支持全选和批量操作                  |

### 4.4 卡片（Card）

使用 `shadcn/ui Card` 组件。

| 变体    | 样式                                | 使用场景               |
| ------- | ----------------------------------- | ---------------------- |
| default | 白色背景 + 1px border              | 数据展示、表单容器     |
| elevated | 白色背景 + shadow-sm              | 首页统计卡片、摘要     |
| outlined | 透明背景 + 1px border             | 列表项、嵌套卡片       |

**卡片内容结构**:
```
┌─────────────────────────────┐
│ CardHeader                   │
│   CardTitle  +  CardAction  │
│   CardDescription            │
├─────────────────────────────┤
│ CardContent                  │
│   主要内容区域               │
├─────────────────────────────┤
│ CardFooter (可选)            │
│   操作按钮 / 补充说明        │
└─────────────────────────────┘
```

### 4.5 状态标签 / 徽章（Badge）

使用 `shadcn/ui Badge` 组件。

| Variant     | 颜色    | 使用场景                         |
| ----------- | ------- | -------------------------------- |
| default     | Primary | 默认标签、编号                   |
| secondary   | Muted   | 辅助标签、类型标注               |
| destructive | Red     | 逾期、异常、紧急                 |
| outline     | Border  | 待处理、中性状态                 |
| success     | Green   | 已完成、审核通过、正常           |
| warning     | Amber   | 待审核、即将到期、提醒           |

### 4.6 对话框（Dialog / Sheet）

使用 `shadcn/ui Dialog`（居中弹窗）和 `Sheet`（侧边抽屉）。

| 组件   | 使用场景                           | 尺寸                        |
| ------ | ---------------------------------- | --------------------------- |
| Dialog | 确认操作、表单弹窗、详情查看       | sm/md/lg/full，默认 md      |
| Sheet  | 筛选面板、详情面板、快速编辑       | top/right/bottom/left       |

**规则**:
- 破坏性操作确认使用 Dialog，标题为操作描述，包含明确后果说明
- 右侧 Sheet 宽度默认 400px，用于详情查看和快速编辑
- Dialog 内表单提交后自动关闭，提交失败保持打开

### 4.7 Toast 提示

使用 `shadcn/ui Sonner`（基于 Sonner 封装）。

| 类型    | 颜色      | 自动消失 | 使用场景               |
| ------- | --------- | -------- | ---------------------- |
| success | Green     | 3s       | 操作成功               |
| error   | Red       | 不消失   | 操作失败、网络异常     |
| warning | Amber     | 5s       | 数据异常、即将过期     |
| info    | Blue      | 5s       | 信息提示、操作引导     |

**规则**:
- 错误 Toast 不自动消失，需用户手动关闭
- 成功操作提供 Toast 反馈（保存成功、删除成功等）
- 批量操作完成时，Toast 显示操作摘要（如"已删除 3 条记录"）

### 4.8 导航

使用 `shadcn/ui` + 自定义布局组件。

| 组件          | 使用场景                   | 规范                          |
| ------------- | -------------------------- | ----------------------------- |
| Sidebar       | 主导航                     | 可折叠，展开 240px / 收起 64px |
| Breadcrumb    | 页面路径                   | H2 标题上方                   |
| Tabs          | 页面内标签切换             | 凭证列表/草稿箱、本月/上月    |

---

## 5. Layout（布局）

### 5.1 布局框架

```
┌──────────────────────────────────────────────────┐
│ TopBar (56px, sticky)                             │
│   Logo | 搜索 | 通知 | 用户头像                    │
├────────┬─────────────────────────────────────────┤
│        │                                         │
│ Side   │  Main Content                           │
│ bar    │                                         │
│ (240px │  ┌─────────────────────────────────┐    │
│ /64px) │  │ Breadcrumb                       │    │
│        │  │ H1 页面标题 + 操作按钮            │    │
│ 导航   │  ├─────────────────────────────────┤    │
│ 菜单   │  │                                   │    │
│        │  │  页面内容区                         │    │
│        │  │  (max-width: 1400px, 居中)        │    │
│        │  │                                   │    │
│        │  └─────────────────────────────────┘    │
│        │                                         │
└────────┴─────────────────────────────────────────┘
```

### 5.2 栅格与间距

```css
--container-max-width: 1400px;
--container-padding: 24px;          /* 内容区左右内边距 */
--sidebar-width: 240px;             /* 侧边栏展开宽度 */
--sidebar-collapsed: 64px;          /* 侧边栏收起宽度 */
--topbar-height: 56px;              /* 顶栏高度 */
```

**间距体系**（4px 基准网格）:

| Token       | Value | Tailwind  | Usage                     |
| ----------- | ----- | --------- | ------------------------- |
| --space-xs  | 4px   | p-1/gap-1 | 图标与文字间距            |
| --space-sm  | 8px   | p-2/gap-2 | 紧凑元素间距              |
| --space-md  | 16px  | p-4/gap-4 | 默认组件间距              |
| --space-lg  | 24px  | p-6/gap-6 | 区块间距、容器内边距      |
| --space-xl  | 32px  | p-8/gap-8 | 大区块间距                |
| --space-2xl | 48px  | p-12      | 页面分区间距              |

### 5.3 仪表盘布局

```
┌─────────────────────────────────────────────┐
│  统计卡片行 (grid: 4列)                       │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐       │
│  │ 总收入 │ │ 总支出 │ │ 净利润 │ │ 待审核 │       │
│  └──────┘ └──────┘ └──────┘ └──────┘       │
│                                               │
│  ┌──────────────────┐ ┌────────────────┐     │
│  │ 收支趋势图表       │ │ 待办事项列表    │     │
│  │ (flex: 2)         │ │ (flex: 1)      │     │
│  └──────────────────┘ └────────────────┘     │
│                                               │
│  ┌──────────────────┐ ┌────────────────┐     │
│  │ 最近凭证           │ │ 税务提醒        │     │
│  │ (flex: 2)         │ │ (flex: 1)      │     │
│  └──────────────────┘ └────────────────┘     │
└─────────────────────────────────────────────┘
```

---

## 6. Depth & Elevation（深度与层级）

### 6.1 阴影体系

| Level    | Tailwind       | Value                              | Usage                 |
| -------- | -------------- | ---------------------------------- | --------------------- |
| Flat     | shadow-none    | none                               | 默认卡片（边框定义）  |
| Raised   | shadow-sm      | 0 1px 2px rgba(0,0,0,0.05)         | 卡片 hover、输入聚焦  |
| Floating | shadow-md      | 0 4px 6px rgba(0,0,0,0.07)         | 下拉菜单、弹出层      |
| Overlay  | shadow-lg      | 0 10px 15px rgba(0,0,0,0.1)        | Dialog、Sheet         |
| Modal    | shadow-xl      | 0 20px 25px rgba(0,0,0,0.1)        | 全屏弹窗              |

**规则**: 默认使用边框定义卡片边界，不使用阴影。仅在交互态（hover、浮动、弹窗）引入阴影。

### 6.2 Z-index 层级

| Token        | Value | Tailwind         | Usage           |
| ------------ | ----- | ---------------- | --------------- |
| --z-base     | 0     | z-0              | 默认内容层      |
| --z-sticky   | 10    | z-10             | 顶栏、侧边栏    |
| --z-dropdown | 20    | z-20             | 下拉菜单        |
| --z-sticky   | 30    | z-30             | 粘性元素        |
| --z-modal    | 40    | z-40             | Dialog / Sheet  |
| --z-toast    | 50    | z-50             | Toast 提示      |
| --z-overlay  | 60    | z-50             | 全屏遮罩        |

---

## 7. Cautions（注意事项）

### Never Do（禁止）

1. **禁止覆盖 shadcn/ui 的 CSS 变量语义** — 不重新定义 `--primary` 等变量的用途，只在值上定制。
2. **禁止使用自定义 CSS 类** — 所有样式通过 Tailwind 工具类实现，保持与 shadcn/ui 一致。
3. **禁止语义色混用** — Success 仅用于成功，Danger 仅用于危险/删除，不可互换。
4. **禁止正文使用 12px 以下字号** — 最小可读字号 12px（仅限标签/徽章），正文最小 14px。
5. **禁止低对比度组合** — 所有文本/背景组合必须通过 WCAG AA（4.5:1 正文，3:1 大文本）。
6. **禁止自动消失的错误 Toast** — 错误提示必须由用户手动关闭，数据安全优先。
7. **禁止丢失用户输入** — 任何异常状态下，表单已填内容必须保留。
8. **禁止内联 style 属性** — 除非动态计算的值（如图表尺寸），一律使用 Tailwind 类。

### Prefer（推荐）

1. **用边框替代阴影** — 默认使用 border 定义卡片边界。
2. **色彩 + 图标 + 文字三重编码** — 状态不仅用颜色，同时配合图标和文字，确保色盲用户可识别。
3. **数字右对齐 + 等宽** — 表格中的金额、数量列使用 `text-right tabular-nums`。
4. **操作反馈** — 所有写操作（增删改）完成后提供 Toast 反馈。
5. **空状态设计** — 列表、表格为空时显示引导性空状态，而非空白页面。
6. **AI 内容标识** — AI 生成内容区域使用紫色标识 + "AI" 标签，与人工内容区分。
7. **Skeleton 加载** — 数据加载中使用 skeleton 占位，避免布局跳动。

---

## 8. Responsive Behavior（响应式行为）

### 8.1 断点定义

遵循 Tailwind CSS 默认断点：

| Name      | Tailwind | Width    | 行为                             |
| --------- | -------- | -------- | -------------------------------- |
| sm        | sm:      | ≥ 640px  | 手机横屏 / 小平板                |
| md        | md:      | ≥ 768px  | 平板                              |
| lg        | lg:      | ≥ 1024px | 小桌面（侧边栏收起）             |
| xl        | xl:      | ≥ 1280px | 标准桌面                          |
| 2xl       | 2xl:     | ≥ 1536px | 大桌面                            |

### 8.2 适配规则

- **桌面优先**: 财务平台核心用户为桌面端，设计基准 1440×900。最小支持宽度 1024px。
- **侧边栏**: ≥1280px 展开 240px，<1280px 收起 64px（图标模式），<768px 隐藏（点击汉堡菜单打开 overlay）。
- **统计卡片**: ≥1280px 四列，1024-1279px 两列，<1024px 单列。
- **表格**: 窄屏时横向滚动，关键列（名称、金额、状态）固定，次要列可隐藏。
- **表单**: <1024px 时单列布局，≥1024px 部分字段可双列。

### 8.3 打印适配

```css
@media print {
  /* 隐藏导航和操作元素 */
  nav, .sidebar, button, .no-print { display: none !important; }

  /* 表格保留 */
  table { break-inside: auto; }
  tr    { break-inside: avoid; }

  /* 去除背景色 */
  * { background: white !important; color: black !important; }
}
```

---

## 9. Dark Mode（暗色模式）

基于 shadcn/ui 的 CSS 变量体系，暗色模式通过 `.dark` 类切换。

### 规则

1. **暗色模式为可选功能** — 财务平台默认亮色，暗色为用户偏好设置。
2. **所有颜色使用 CSS 变量** — 组件不硬编码颜色值，确保暗色模式自动适配。
3. **图表适配** — Recharts 图表在暗色模式下调整网格线、标签色、背景色。
4. **金额色保持一致** — 暗色模式下收入仍为绿色、支出仍为红色，仅亮度调整。
5. **打印始终亮色** — 无论用户选择何种模式，打印输出使用亮色背景。

---

## 10. Agent Prompt Guide（Agent 生成指南）

### 10.1 关键注意事项

1. **使用 shadcn/ui 组件** — 所有 UI 组件优先使用 `@repo/ui` 中的 shadcn/ui 封装，不手写基础组件。
2. **Tailwind 类名** — 样式只使用 Tailwind 工具类，不写自定义 CSS。使用项目定义的 CSS 变量（`hsl(var(--primary))`）。
3. **字号** — 正文 16px（text-base），金额/关键数据可加大。按钮文字 14px，标签 12px。
4. **字体** — 使用 `--font-sans` 和 `--font-mono` 变量，不加载额外 Web 字体。
5. **金额格式** — 右对齐 + tabular-nums + 千位逗号 + 两位小数。正数绿色、负数红色。
6. **状态编码** — 状态展示使用色彩 + 图标 + 文字三重编码，不依赖单一颜色。
7. **间距** — 使用 4px 基准网格（Tailwind 的 p-1/p-2/p-4/p-6/p-8/p-12），不使用任意值。
8. **圆角** — 默认 `--radius: 0.5rem`（8px），通过 shadcn/ui 的 rounded 变量控制。
9. **无渐变** — 所有背景使用纯色，不使用 linear-gradient。
10. **表单校验** — 使用 React Hook Form + Zod，错误信息在字段下方显示。
11. **表格** — 使用 TanStack Table + shadcn/ui Table，金额列右对齐 + tabular-nums。
12. **AI 内容** — AI 生成内容使用紫色（--color-ai-accent）标识 + "AI" 标签。

### 10.2 快速 Tailwind 类名参考

```
/* 布局 */
container mx-auto max-w-screen-xl px-6

/* 卡片 */
rounded-lg border bg-card text-card-foreground shadow-sm

/* 金额 */
font-mono tabular-nums text-right

/* 状态 */
text-success / text-warning / text-destructive
bg-success/10 / bg-warning/10 / bg-destructive/10

/* 按钮 */
h-9 px-4 rounded-md bg-primary text-primary-foreground hover:bg-primary/90

/* 表格 */
text-sm font-medium text-muted-foreground (表头)
text-sm (单元格)
```

### 10.3 图标

- 统一用 `lucide-react`（shadcn 默认图标库），禁止混用多套图标。
- 图标尺寸统一 `size-4`（16px）/ `size-5`（20px），用 Tailwind 尺寸类。
- 业务状态图标与颜色令牌联动，如成功状态用 `CheckCircle` + `success` 色。