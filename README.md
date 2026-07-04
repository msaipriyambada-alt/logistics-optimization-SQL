# logistics-optimization-SQL
A SQL-based analytics project analyzing delivery performance across 300 orders. Using window functions and CTEs, this project identifies delayed routes, bottleneck warehouses, and underperforming delivery agents to improve on-time delivery rates and logistics efficiency.
# 🚚 Logistics Optimization for Delivery Routes (SQL)

An end-to-end SQL analytics project analyzing delivery performance across a 5-table logistics dataset (orders, routes, warehouses, delivery agents, and shipment tracking) to identify delay patterns, bottleneck warehouses, and underperforming routes/agents — helping logistics operations improve on-time delivery.

## 📋 Quick Overview

| | |
|---|---|
| **Dataset** | 5 relational tables — orders, routes, warehouses, delivery agents, shipment tracking |
| **Tools** | SQL (MySQL) |
| **Techniques** | Window Functions (`RANK`, `ROW_NUMBER`), CTEs, Multi-table JOINs, Aggregations |
| **Project Type** | Operations / Logistics Analytics |

## 🎯 Business Problem

How can the logistics team identify **where** delivery delays are happening and **why**, so they can prioritize fixes across routes, warehouses, and agents instead of reacting to complaints one at a time?

## 🗂️ Dataset Overview

| Table | Description |
|---|---|
| `orders` | Order-level data — placement date, delivery date, delay days |
| `routes` | Route metadata connecting warehouses to delivery zones |
| `warehouses` | Warehouse location and processing time data |
| `delivery_agents` | Agent-level delivery speed and assignment data |
| `shipment_tracking` | Checkpoint-level tracking logs per order |

## 🧹 Data Cleaning

- Removed duplicate order records
- Standardized inconsistent date formats across tables
- Handled missing/null delay reasons

## 🔍 SQL Analysis — Key Questions Answered

| # | Business Question | Technique Used | Insight |
|---|---|---|---|
| 1 | Which orders are most delayed within each warehouse? | `RANK() OVER (PARTITION BY warehouse)` | Surfaced worst-delay orders per warehouse for targeted review |
| 2 | Which warehouses are "bottlenecks" (processing time above average)? | CTE comparing per-warehouse avg vs. global avg | W001, W004, W009 flagged as bottlenecks |
| 3 | Which delivery agents have the best/worst on-time rate within their route? | `RANK() OVER (PARTITION BY route)` | Bottom 5 agents averaged 56 km/hr — below fleet average |
| 4 | What is each order's last tracking checkpoint before delay? | `ROW_NUMBER()` on tracking logs | Sorting delays were the most common last-checkpoint issue |
| 5 | What are the most common delay reasons overall? | `GROUP BY` + `COUNT` | Sorting delays, weather, and traffic were the top 3 reasons |

## 📊 Key Findings

- **Overall on-time delivery rate: ~56%** — a clear improvement target
- Routes **R004, R005, R013** are consistently the most delayed
- Warehouses **W001, W004, W009** show processing times above the global average
- The bottom 5 delivery agents deliver ~56 km/hr on average, noticeably slower than top performers
- **Sorting delays, weather, and traffic** account for the majority of delay incidents

## 💡 Recommendations

**Immediate:**
- Reassign top-performing agents to the most delayed routes (R004, R005, R013)

**Short-term:**
- Audit sorting processes at bottleneck warehouses (W001, W004, W009) to identify process fixes

**Long-term:**
- Build a live on-time-delivery dashboard so operations can monitor these KPIs continuously instead of via one-off analysis

## 📁 Repo Contents

- `queries.sql` — all SQL queries used in this analysis
- SELECT
       Order_ID,
       Order_Date,
       Actual_Delivery_Date,
       DATEDIFF(Actual_Delivery_Date, Order_Date) AS Delay_Days
       FROM orders;
- SELECT     Route_ID,    ROUND(AVG(DATEDIFF(Actual_Delivery_Date, Order_Date))) AS Avg_Delay_DaysFROM ordersGROUP BY Route_IDLIMIT 10;
<img width="2111" height="70" alt="image" src="https://github.com/user-attachments/assets/e0287b44-a089-41ab-913c-923b6aeb8aec" />

- `screenshots/` — query output screenshots

## 🔮 Next Steps

- Add indexing strategy for scaling this analysis to millions of rows
- Build a companion Power BI dashboard for non-technical stakeholders

## 📬 Contact

**Saipriyambada Mohapatra**
📧 msaipriyambada@gmail.com
🔗 [LinkedIn](https://www.linkedin.com/in/saipriyambada-mohapatra-297065383)
