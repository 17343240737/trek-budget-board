# TREK 旅行预算看板

> 基于 [TREK](https://gitcode.com/GitHub_Trending/nomad22/TREK)（AGPL-3.0）的增量开发，新增「预算看板」功能。

## 体验地址

**https://rjdfeq2h-3000.devbridge-s2.hwtunnel.com/**

> 通过华为云 DevBridge 隧道暴露的公网地址，可直接访问运行中的 TREK 实例。

## 功能说明

在行程详情页的「费用」页签右侧新增「看板」页签（英文环境显示 "Board"），提供四块可视化内容：

| 区块 | 说明 | 技术 |
|------|------|------|
| **分类支出对比** | 按类别汇总支出，显示金额、笔数、占比，纯 CSS 进度条 | CSS `width` + `transition` |
| **分类占比饼图** | 环形图 + 右侧图例（分类名 / 百分比 / 金额） | CSS `conic-gradient` |
| **每日支出趋势** | 按日期分柱的柱状图，峰值柱高亮，顶部显示总计/日均/峰值日 | 纯 SVG `<rect>` + `<text>` |
| **成员分摊与结算** | 每人已付/应摊/余额 + 结算流向（A owes €X to B） | 现有 `budgetApi` 接口 |

**无新数据表、无新后端接口。** 所有数据来自现有 budget 域的 Zustand store + 两个已有 API 端点（`perPersonSummary` / `settlement`）。图表全部用纯 SVG/CSS 实现，未引入任何图表库。

## 改动文件清单

| 类型 | 路径 | 说明 |
|------|------|------|
| 修改 | `client/src/constants/tripTabs.ts` | 新增 `budgetboard` 页签 ID + 标签映射 |
| 修改 | `client/src/pages/tripPlanner/useTripPlanner.ts` | TRIP_TABS 数组新增条目（`enabledAddons.budget` 门控，LayoutDashboard 图标）+ loadBudgetItems 触发 |
| 修改 | `client/src/pages/TripPlannerPage.tsx` | 懒加载 import + 渲染块 |
| **新增** | `client/src/components/Budget/BudgetBoard.tsx` | 主组件 + 4 个子区块组件（455 行） |
| 修改 | `shared/src/i18n/en/budget.ts` | +24 个 `budgetBoard.*` 英文 key |
| 修改 | `shared/src/i18n/zh/budget.ts` | +24 个 `budgetBoard.*` 中文 key |
| 修改 | `shared/src/i18n/en/trip.ts` | `trip.tabs.budgetboard` = "Board" |
| 修改 | `shared/src/i18n/zh/trip.ts` | `trip.tabs.budgetboard` = "看板" |

## 仓库结构

```
trek-budget-board/
├── README.md                          # 本文件
├── budget-board.patch                 # 相对 TREK 上游的完整 git diff（含新文件）
├── client/
│   └── src/
│       ├── components/Budget/
│       │   └── BudgetBoard.tsx        # 新增：看板主组件
│       ├── constants/
│       │   └── tripTabs.ts            # 修改：页签注册
│       └── pages/
│           ├── TripPlannerPage.tsx     # 修改：渲染入口
│           └── tripPlanner/
│               └── useTripPlanner.ts  # 修改：TRIP_TABS + 数据加载
└── shared/
    └── src/i18n/
        ├── en/
        │   ├── budget.ts              # 修改：+24 i18n keys
        │   └── trip.ts                # 修改：tab 标签
        └── zh/
            ├── budget.ts              # 修改：+24 i18n keys
            └── trip.ts                # 修改：tab 标签
```

## 部署步骤

### 前置条件

- Node.js 18+
- npm workspaces（monorepo 结构：shared → server → client）

### 1. 克隆 TREK 上游

```bash
git clone https://gitcode.com/GitHub_Trending/nomad22/TREK.git
cd TREK
```

### 2. 应用本仓库的改动

**方式 A：用 patch 文件**

```bash
patch -p1 < /path/to/budget-board.patch
```

**方式 B：手动覆盖文件**

将本仓库 `client/` 和 `shared/` 下的文件覆盖到 TREK 对应路径。

### 3. 安装依赖并构建

```bash
npm install
npm run build --workspace=shared
npm run build --workspace=client
```

### 4. 配置并启动服务

```bash
# 编辑 server/.env，设置数据库、加密密钥等
# 确保 ALLOWED_ORIGINS 包含你的访问域名
npm run start --workspace=server
```

### 5. 验证

1. 打开浏览器访问服务地址
2. 登录后打开任意行程
3. 在页签栏中找到「看板」（英文 "Board"）页签
4. 点击查看四块可视化内容

## i18n 说明

- `zh`（中文）和 `en`（英文）已补齐所有 24 个 `budgetBoard.*` key
- 其他语言通过 `t()` 函数的回退机制自动使用英文：`strings[key] ?? en[key] ?? key`
- 不会出现裸键名

## 技术栈

- **前端**：React 19 + Vite 8 + TypeScript 6 + PWA
- **状态管理**：Zustand（tripStore with slices）
- **后端**：NestJS（现有 budget/costs 模块）
- **图表**：纯 SVG/CSS（conic-gradient pie, SVG bars, CSS progress bars）
- **i18n**：自建翻译系统，per-locale aggregation

## 许可证

本项目基于 TREK（AGPL-3.0）进行增量开发，继承 AGPL-3.0 许可证。

## 体验数据

测试行程「杭州西湖三日游」包含以下支出数据：

| 分类 | 金额 (EUR) |
|------|-----------|
| 住宿 | 800 |
| 餐饮 | 260 |
| 观光 | 120 |
| 门票 | 60 |
| 购物 | 40 |
| **总计** | **1,280** |
