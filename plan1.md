# 仓储与销售管理系统详细开发计划 (plan1.md)

## 1. 系统概述 (System Overview)

本系统旨在构建一个集成的仓储与销售管理平台，专注于库存的实时监控、销售流程的闭环管理以及与 WooCommerce 的无缝对接。系统将部署在 Vercel 上，使用 Next.js 进行开发，Supabase 作为后端数据库，并通过 REST API 与 WooCommerce 交互。

### 技术栈 (Tech Stack)
*   **Frontend**: Next.js (App Router), React, Tailwind CSS.
*   **Backend/Database**: Supabase (PostgreSQL, Auth, Realtime).
*   **Integration**: WooCommerce REST API (v3).
*   **Deployment**: Vercel.
*   **PDF Generation**: `@react-pdf/renderer`.

---

## 2. 数据库设计 (Database Schema - Supabase)

### 2.1 用户与权限 (Users & Roles)
*   **Table**: `profiles` (extends `auth.users`)
    *   `id` (UUID, PK)
    *   `email` (Text)
    *   `role` (Enum: 'admin', 'sales', 'warehouse', 'accountant')
    *   `full_name` (Text) - 用于显示在 Invoice 上的名字
    *   `phone` (Text) - 联系电话

### 2.2 产品缓存 (Product Cache)
*   *设计策略*: 产品主数据在 WooCommerce，本系统定期同步或实时拉取。Supabase 中存储必要的补充信息。
*   **Table**: `products`
    *   `wc_id` (Int, PK, WooCommerce ID)
    *   `sku` (Text)
    *   `name` (Text)
    *   `category` (Text) - 父子类拼接
    *   `price` (Decimal) - 当前售价
    *   `stock_quantity` (Int) - 当前库存
    *   `other_config` (Text) - **Supabase 独有字段**，用户手动补充
    *   `image_url` (Text)
    *   `last_synced_at` (Timestamp)

### 2.3 客户管理 (Customers)
*   **Table**: `customers`
    *   `id` (UUID, PK)
    *   `name` (Text)
    *   `address` (Text)
    *   `phone` (Text)
    *   `email` (Text)

### 2.4 销售发票 (Sales Invoices)
*   **Table**: `sales_invoices`
    *   `id` (UUID, PK)
    *   `quotation_no` (Text, Unique) - 格式: AFLX + Date + Sequence
    *   `user_id` (UUID, FK -> profiles) - 提交人
    *   `customer_id` (UUID, FK -> customers)
    *   `invoice_for` (Text) - 客户名或公司名
    *   `project_name` (Text)
    *   `payment_terms` (Enum: 'Cash', 'MOMO', 'Check', 'To Be Decide', 'Other')
    *   `status` (Enum: 'Draft', 'Pending', 'Approved', 'Rejected')
    *   `total_amount` (Decimal)
    *   `currency` (Default: 'GHS')
    *   `remarks` (Text) - 默认包含固定的条款，但存入库以便历史追溯
    *   `created_at` (Timestamp)
    *   `approved_at` (Timestamp, Nullable)
    *   `sales_manager_name` (Text) - 快照
    *   `sales_manager_email` (Text) - 快照
    *   `sales_manager_contact` (Text) - 快照

*   **Table**: `invoice_items`
    *   `id` (UUID, PK)
    *   `invoice_id` (UUID, FK)
    *   `product_wc_id` (Int, FK)
    *   `product_name` (Text) - 快照
    *   `sku` (Text) - 快照
    *   `quantity` (Int)
    *   `unit_price` (Decimal) - 销售时的单价
    *   `adjustment` (Decimal) - 价格调整金额
    *   `adjustment_note` (Text) - 调整原因
    *   `total_line_price` (Decimal) - (Unit * Qty) + Adj

### 2.5 库存调整申请 (Inventory Adjustment Requests)
*   **Table**: `inventory_adjustments`
    *   `id` (UUID, PK)
    *   `user_id` (UUID, FK)
    *   `product_wc_id` (Int, FK)
    *   `current_qty` (Int) - 申请时的库存
    *   `adjust_qty` (Int) - 调整数量 (+/-)
    *   `reason` (Text)
    *   `status` (Enum: 'Pending', 'Approved', 'Rejected')
    *   `rejection_reason` (Text)
    *   `created_at` (Timestamp)

---

## 3. 角色权限矩阵 (RBAC Matrix)

| 功能页面 (Page Tabs) | 管理员 (Admin/GM) | 销售 (Sales) | 仓管 (Warehouse) | 会计 (Accountant) |
| :--- | :---: | :---: | :---: | :---: |
| **库存和价格** (Inventory) | ✅ (可编辑) | ✅ (只读) | ✅ (只读) | ✅ (只读) |
| **销售提交** (Submit Sales) | ✅ | ✅ | ✅ | ❌ |
| **销售提交记录** (Submission History) | ✅ | ✅ | ✅ | ❌ |
| **我的销售记录** (My Sales) | ✅ | ✅ | ✅ | ❌ |
| **库存调整提交** (Submit Inv Adj) | ❌ (直接编辑) | ❌ | ✅ | ❌ |
| **库存调整记录** (Inv Adj History) | ❌ | ❌ | ✅ | ❌ |
| **审批中心** (Approval Center) | ✅ | ❌ | ❌ | ❌ |
| **所有已销售记录** (All Sales) | ✅ | ❌ | ❌ | ✅ |

---

## 4. 功能模块详细设计 (Detailed Features)

### 4.1 仪表盘 (Dashboard)
*   **权限**: 所有角色登录后的首页。
*   **库存概览**: 按 WooCommerce Category 显示库存总量/总值柱状图。
*   **销售统计**: 折线图显示销售额。
    *   *时间过滤器*: 1 Week, 1 Month, 2 Months, 3 Months, 6 Months, 12 Months.
*   **数据源**: `sales_invoices` (Status='Approved') + `products` (Inventory).

### 4.2 库存产品页面 (Inventory & Prices)
*   **展示**: 表格列出 Category (Parent-Child), Description (Name), SKU, Other Config, Qty, Unit Price。
*   **同步**: 页面加载时或点击"刷新"按钮，调用 API 拉取 WooCommerce 最新数据更新 Supabase。
*   **操作**:
    *   **搜索/排序**: Client-side filtering.
    *   **管理员编辑**: 双击单元格 (Qty, Price, Other Config) 进入编辑模式。
        *   *保存逻辑*:
            1.  更新 Supabase `products` 表。
            2.  如果是 Qty 或 Price，触发 Server Action 调用 WooCommerce API 更新远程商城数据。
        *   *取消*: 恢复原值。
    *   **非管理员**: 仅查看。

### 4.3 销售提交 (Sales Invoice Submission)
*   **表单字段**:
    *   **Sales Manager Info**: 自动预填 (Name: User Profile, Email: info@..., Phone: 054...)。
    *   **Quotation No.**: 自动生成 (AFLX + YYYYMMDD + Random/Seq)。
    *   **Invoice For / Customer ID**: 下拉框选择 (来自 `customers` 表)。选择后自动填充 Address。支持输入新客户(提交时自动创建新客户记录)。
    *   **Sales Items (Table)**:
        *   *Add Row*: 选择 Product (Typing search 'Description' -> Filter inventory).
        *   *Auto-fill*: SKU, Other Config, Unit Price (from DB).
        *   *Input*: Qty, Adjustment (+/- value), Adj Note.
        *   *Calc*: Total Price.
    *   **Submission**:
        *   点击提交 -> 写入 `sales_invoices` 表，Status = 'Pending'。
    *   **Footer**: 自动附加 3 条 Remarks 条款。

### 4.4 审批中心 (Approval Center) - *管理员独有*
*   **Tabs**:
    1.  **Sales Invoice Approvals**:
        *   列表显示所有 'Pending' 的申请。
        *   **Action**: View Details, Approve, Reject (填写理由), Delete.
        *   *Approve 逻辑*:
            1.  更新 Invoice Status -> 'Approved'.
            2.  扣减 Supabase `products` 库存。
            3.  **同步**: 调用 WC API 扣减远程库存。
        *   *Reject 逻辑*: Status -> 'Pending Submit' (退回修改).
    2.  **Inventory Adjustment Approvals**:
        *   列表显示仓管的库存调整申请。
        *   *Approve 逻辑*: 更新本地及远程 WooCommerce 库存。

### 4.5 记录与历史 (History & Records)
*   **销售提交记录 (My Submissions)**:
    *   显示当前用户提交的所有单据及状态 (Draft, Pending, Rejected, Approved).
    *   **Action**:
        *   若 Rejected: 点击编辑，重新提交。
        *   若 Pending/Approved: 查看详情。
        *   **Download PDF**: 生成包含 Logo、抬头、条款的正式 PDF 文件。
*   **我的销售记录 (My Sales)**:
    *   仅显示 Status = 'Approved' 的记录。
*   **所有已销售记录 (All Sales)**:
    *   管理员/会计查看所有人的 Approved 记录。

### 4.6 库存调整 (Warehouse Only)
*   **提交**: 选择产品 -> 输入调整数量 (+/-) -> 原因。
*   **记录**: 查看申请状态。

---

## 5. 开发流程与 API 集成

### 5.1 WooCommerce 集成
需要配置 `WOOCOMMERCE_CK` (Consumer Key) 和 `WOOCOMMERCE_CS` (Consumer Secret)。
*   `GET /wp-json/wc/v3/products`: 全量/增量同步产品。
*   `PUT /wp-json/wc/v3/products/{id}`: 更新库存和价格。

### 5.2 Next.js 路由结构
```
app/
  (auth)/login/             # 登录页
  (dashboard)/
    page.js                 # Dashboard (Home)
    inventory/              # 库存列表
    sales/
      submit/               # 销售提交表单
      history/              # 提交记录
      my-sales/             # 我的销售 (Approved)
      all-sales/            # 所有销售 (Admin/Accountant)
    approval/               # 审批中心 (Admin)
    warehouse/
      adjust/               # 库存调整提交
      history/              # 调整记录
  api/
    webhooks/wc-update/     # 接收 WC 产品变更 (可选)
    sync-products/          # 手动触发同步
```

## 6. 实施路线图

1.  **环境配置**: 初始化 Next.js, 配置 Supabase Auth (Email/Pass), 设置 WooCommerce API Keys.
2.  **数据库构建**: 在 Supabase 建立上述表结构及 Row Level Security (RLS) 策略，确保通过 Role 控制访问。
3.  **同步模块开发**: 编写 Service 层，实现从 WC 拉取产品并 Upsert 到 Supabase。
4.  **库存页面开发**: 实现带权限控制的表格 (TanStack Table)，集成行内编辑与 WC 回写。
5.  **业务流程开发**:
    *   开发 Sales Invoice 表单与 PDF 生成器。
    *   开发审批流逻辑 (Approve/Reject 状态机)。
6.  **仪表盘开发**: 集成 Recharts 展示图表。
7.  **测试于部署**: 模拟各角色登录，测试闭环流程，部署至 Vercel。
