# 货物仓储与销售管理系统 - 开发计划书 (Development Plan)

基于 `Goods_Storage_Sales_System.canvas` 流程图生成的详细系统开发方案。

## 1. 项目概述 (Project Overview)
本系统旨在建立一个闭环的货物进销存管理平台，实现从销售前端到仓储后端以及财务核算的无缝连接。核心目标是提高库存周转率、确保账实相符，并为管理层提供实时数据决策支持。

## 2. 系统角色 (User Roles)
根据流程图定义，系统支持以下角色权限：
*   **销售人员 (Salesperson)**: 负责客户对接、询价、订单创建及售后。
*   **仓管员 (Warehouse Keeper)**: 负责出入库操作、库存盘点、单据执行。
*   **会计 (Accountant)**: 负责应收应付审核、发票开具、财务报表。
*   **总经理 (GM)**: 负责大额采购审批、经营数据监控。

---

## 3. 核心功能模块 (Core Modules)

### 3.1 市场与销售模块 (Market & Sales)
*   **客户咨询列表 (Inquiry Management)**: 记录客户初步需求与联系记录。
*   **报价系统 (Quotation Engine)**:
    *   *功能*: 关联实时库存数据，生成报价单。
    *   *逻辑*: 报价前强制检查库存可用性 (Check Availability)。
*   **销售订单 (Sales Order - SO)**:
    *   *流程*: 创建订单 -> 提交财务审核 -> 锁定库存。
*   **售后反馈 (Feedback)**: 关联历史订单，记录退换货或评价。

### 3.2 仓储与库存模块 (Warehouse & Inventory)
*   **库存中心 (Inventory DB)**:
    *   *核心数据*: SKU信息、库位分布、成本价/零售价动态管理。
    *   *功能*: 支持库存流水查询 (Stock Ledger)。
*   **出库管理 (Outbound)**:
    *   *触发*: 仅财务审核通过的 SO 可生成出库单。
    *   *动作*: 拣货 (Pick) -> 打包 (Pack) -> 发货 (Ship) -> 自动扣减库存。
*   **入库管理 (Inbound)**:
    *   *触发*: 采购单 (PO) 到货。
    *   *动作*: 质检 (QC) -> 上架 (Shelving) -> 自动增加库存。
*   **智能预警 (Inventory Alert)**:
    *   *逻辑*: 当 `Current Stock < Safe Stock` 时，自动生成采购申请 (PR)。

### 3.3 财务与管理模块 (Finance & Management)
*   **应收管理 (Account Receivable)**:
    *   *审核*: 确认客户款项项到账后，释放销售订单给仓库。
    *   *单据*: 销售发票管理。
*   **应付管理 (Account Payable)**:
    *   *流程*: 收到采购单 -> 审核 -> 付款给供应商。
*   **审批流 (Approval Workflow)**:
    *   *规则*: 采购申请 -> 总经理审批 (Approve) -> 生成正式采购单 (PO)。
*   **财务月报 (Financial Report)**:
    *   自动计算销售总额、毛利分析、库存周转成本。

### 3.4 数据驾驶舱 (Data Dashboard)
*   **销售看板**: 今日/本月销量趋势、订单完成率。
*   **资产看板**: 实时库存总值、呆滞库存预警。
*   **热销排行**: Top Selling Products 榜单。

---

## 4. 推荐技术架构 (Tech Stack)

### 前端 (Frontend)
*   **框架**: React 或 Vue 3 (搭配 Ant Design 或 Element Plus).
*   **特点**: 侧重于表格操作效率与 Dashboard 图表展示 (使用 ECharts/Recharts).

### 后端 (Backend)
*   **服务**: Python (Django/FastAPI) 或 Node.js (NestJS).
*   **API**: RESTful API 供前后端分离调用。

### 数据库 (Database)
*   **关系型数据库**: PostgreSQL (推荐) 或 MySQL.
    *   *Schema 设计*: 需严格设计 `Products`, `Orders`, `InventoryLogs`, `Transactions` 表的关联。
*   **缓存**: Redis (用于缓存实时库存查询，提高报价响应速度)。

---

## 5. 数据库设计概要 (Schema Draft)

```sql
-- 产品表
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    sku VARCHAR(50) UNIQUE,
    name VARCHAR(100),
    cost_price DECIMAL(10,2),
    sale_price DECIMAL(10,2),
    safe_stock_level INT
);

-- 库存表
CREATE TABLE inventory (
    product_id INT REFERENCES products(id),
    quantity INT,
    location_code VARCHAR(20)
);

-- 销售订单
CREATE TABLE sales_orders (
    id SERIAL PRIMARY KEY,
    order_no VARCHAR(50),
    status VARCHAR(20), -- 'PENDING_FINANCE', 'PAID', 'SHIPPED', 'COMPLETED'
    total_amount DECIMAL(10,2),
    created_at TIMESTAMP
);

-- 采购单
CREATE TABLE purchase_orders (
    id SERIAL PRIMARY KEY,
    status VARCHAR(20), -- 'NEW', 'APPROVED_GM', 'PAID', 'RECEIVED'
    total_cost DECIMAL(10,2)
);
```

## 6. 开发实施路线图 (Roadmap)

*   **Phase 1: 基础建设 (Base)**
    *   搭建数据库，实现 SKU 管理与基础库存增删改查。
    *   完成用户权限系统 (RBAC)。
*   **Phase 2: 进销存核心 (Core Logic)**
    *   实现“采购入库”与“销售出库”的核心闭环。
    *   打通库存自动扣减逻辑。
*   **Phase 3: 财务与审批 (Finance & Flow)**
    *   加入财务审核节点，实现“不见款不发货”的业务控制。
    *   实现经理审批流。
*   **Phase 4: 数据可视化 (Dashboard)**
    *   开发管理驾驶舱，接入实时统计数据。
