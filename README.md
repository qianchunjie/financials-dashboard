# Power BI Financials 交互式仪表盘

> 一个**单文件、零依赖**的交互式 BI 仪表盘，数据来自 Power BI 内置的「financials」示例数据集，用于面试展示数据 / BI 能力。

## 访问方式

| 方式 | 地址 / 操作 |
|---|---|
| 🌐 在线演示（任何人可访问） | https://qianchunjie.github.io/financials-dashboard/ |
| 📦 源码仓库 | https://github.com/qianchunjie/financials-dashboard |
| 💻 本地打开 | 双击 `financials-dashboard.html`（无需联网、无需安装任何东西） |

## 项目简介

- **数据**：Power BI Desktop 内置「financials」销售 / 财务示例，**700 行明细**、16 个月（2013-09 ~ 2014-12）、5 个维度、5 个指标。
- **形态**：一个自包含的 HTML 文件，数据直接嵌入文件内，浏览器端用 JavaScript 现场聚合渲染，**没有后端、没有数据库、没有外部依赖**。
- **交互**：
  - 跨图联动（点任意柱 / 扇区 → 全局图表 + KPI + 表格联动过滤）
  - 时间序列**刷选**（拖拽选择区间）
  - 筛选**面包屑** + 一键重置
  - KPI 实时重算
  - 明细表搜索 / 排序 / 分页
  - 月度汇总表

## 关键指标（与 Power BI 逐项核对一致）

| 指标 | 数值 |
|---|---|
| 销售额 Sales | $118,726,350.26 |
| 利润 Profit | $16,893,702.26 |
| 销量 Units Sold | 1,125,806 |
| 成本 COGS | $101,832,648 |
| 折扣 Discounts | $9,205,248.24 |
| 毛利 Gross（Sales + Discounts） | $127,931,598.50 |

> 注：利润 = 销售额 − 成本；36 行销量为 0.5 步进的分数值，已按原始值保留，避免舍入误差。

## 数据是怎么来的（架构）

```
Power BI 的 .pbix 文件（内存模型 msmdsrv）
        │
        │  ← Power BI Modeling MCP Server：Claude 通过它连上 Power BI
        │     的内存模型，用 DAX 查询导出 financials 表 700 行明细
        ▼
  financials_compact.json   （700 行数据，压缩编码后的纯文本）
        │
        ▼
  build_dashboard.mjs 把数据注入模板 → financials-dashboard.html
        │
        ▼
  push 到 GitHub → GitHub Pages 静态托管 → 任何人可访问的链接
```

- **MCP 的角色**：只在「构建仪表盘」时用一次，是「Claude 读取 Power BI 数据」的桥。Power BI 的数据存在它自己的内存模型里，无法直接打开；通过 MCP 服务器才能连进去跑 DAX、把数据导出来。**网页运行时完全用不到 MCP。**
- **为什么 GitHub 上「看不到那么多数据」**：700 行明细不是单独摆着的表格，而是被压缩编码成数字下标，作为 `const DATA = {...}` 写死在 `index.html` 里（这也是文件有 73KB 的原因）。网页打开后由浏览器现场算出所有 KPI 和图表。

## 如果 Power BI 数据变了，如何更新

网页是**静态快照**，Power BI 里改动数据**不会自动**反映到网页。要更新需重跑一遍流水线：

1. 打开 Power BI Desktop 中的 financials 模型（保证本地 Analysis Services 进程在运行）
2. `mcp_extract.mjs` → 通过 MCP 重新导出明细数据
3. `prepare_data.mjs` → 生成 `financials_compact.json`
4. `build_dashboard.mjs` → 生成新的 `financials-dashboard.html`
5. 重新 push 到 GitHub（或直接在 GitHub 网页上传新文件）

> 构建脚本在 `d:\神经计算科学1\powerbi-dashboard-build\`。

## 面试要点速记

- **数据层**：直接从 Power BI 模型用 DAX 导出，而非手工整理 Excel。
- **架构层**：单文件 + 客户端聚合，跨图联动、时间刷选、KPI 重算都是纯前端实现。
- **正确性**：所有总额与 Power BI 逐项核对一致（含 0.5 分数销量的边界情况）。
- **部署**：GitHub Pages 免费静态托管，任何人点链接即可访问。
