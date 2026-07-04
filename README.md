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
- SELECT     Order_ID,    Warehouse_ID,    DATEDIFF(Actual_Delivery_Date, Order_Date) AS Delay_Days,    RANK() OVER (        PARTITION BY Warehouse_ID        ORDER BY DATEDIFF(Actual_Delivery_Date, Order_Date) DESC    ) AS Delay_RankFROM orders;
<img width="3422" height="70" alt="image" src="https://github.com/user-attachments/assets/d22c887c-dabb-4616-a956-daf412131ccf" />
SELECT     Route_ID,    Distance_KM / Average_Travel_Time_Min AS Efficiency_RatioFROM routesORDER BY Efficiency_Ratio ASCLIMIT 3;
<img width="1968" height="70" alt="image" src="https://github.com/user-attachments/assets/6f1c73d1-4ae4-44a9-afd3-faf787567469" />
SELECT     Route_ID,    COUNT(*) AS Total Orders,    SUM(CASE         WHEN DATEDIFF(Actual_Delivery_Date, Order_Date) > 0         THEN 1 ELSE 0 END) AS Delayed Orders,    ROUND(        SUM(CASE             WHEN DATEDIFF(Actual_Delivery_Date, Order_Date) > 0             THEN 1 ELSE 0 END) * 100.0 / COUNT(*),     2) AS Delay_PercentageFROM ordersGROUP BY Route_IDHAVING Delay Percentage > 20;
<img width="4713" height="62" alt="image" src="https://github.com/user-attachments/assets/ab6e7dcb-bbec-45f4-9d60-841095834f43" />
SELECT     Route_ID,    COUNT(*) AS Total Orders,    SUM(CASE         WHEN DATEDIFF(Actual_Delivery_Date, Order_Date) > 0         THEN 1 ELSE 0 END) AS Delayed Orders,    ROUND(        SUM(CASE             WHEN DATEDIFF(Actual_Delivery_Date, Order_Date) > 0             THEN 1 ELSE 0 END) * 100.0 / COUNT(*),     2) AS Delay_PercentageFROM ordersGROUP BY Route_IDHAVING Delay Percentage > 20;
<img width="4713" height="62" alt="image" src="https://github.com/user-attachments/assets/afc20656-9a6f-4b10-9669-0f3a19901e93" />
WITH warehouse processing AS (SELECT o.Warehouse_ID,	AVG(DATEDIFF(o.Actual_Delivery_Date, o.Order_Date)) AS Avg_Processing_Time    FROM orders o    GROUP BY o.Warehouse_ID),global_avg AS (SELECT AVG(DATEDIFF(Actual_Delivery_Date, Order_Date)) AS Global_Avg_Time    FROM orders)SELECT wp.Warehouse_ID,    wp.Avg_Processing_Time,    g.Global_Avg_TimeFROM warehouse_processing wpCROSS JOIN global_avg gWHERE wp.Avg_Processing_Time > g.Global_Avg
SELECT Warehouse_ID,COUNT(Order_ID) AS Total_Orders,    SUM(CASE WHEN Actual_Delivery_Date <= Expected_Delivery_Date     THEN 1 ELSE 0 END ) AS On_Time_Orders,ROUND(SUM(CASE 	WHEN Actual_Delivery_Date <= Expected_Delivery_Date THEN 1 ELSE 0         END) * 100.0 / COUNT(Order_ID), 2) AS On_Time_Percentage,RANK() OVER (ORDER BY         SUM(CASE             WHEN Actual_Delivery_Date <= Expected_Delivery_Date THEN 1             ELSE 0         END) * 100.0 / COUNT(Order_ID) DESC) AS Warehouse_RankFROM ordersGROUP BY Warehouse_ID;
<img width="7632" height="67" alt="image" src="https://github.com/user-attachments/assets/d64c11b5-26dd-4671-b1bc-130ad7c352d5" />
SELECT Route_ID,    Agent_ID,    On_Time_Percentage,RANK() OVER (PARTITION BY Route_IDORDER BY On_Time_Percentage DESC) AS Rank_In_RouteFROM deliveryagents;
<img width="2490" height="70" alt="image" src="https://github.com/user-attachments/assets/b2a0b989-4538-4514-aabc-e0c97e7617f9" />
SELECT     Agent_ID,    Route_ID,    On_Time_PercentageFROM deliveryagentsWHERE On_Time_Percentage < 80ORDER BY On_Time_Percentage ASC;
<img width="2153" height="70" alt="image" src="https://github.com/user-attachments/assets/05358813-b91e-46f2-bd22-7facb2a07e89" />
WITH ranked_agents AS (SELECT         Agent_ID,        Avg_Speed_KM_HR,        On_Time_Percentage,RANK() OVER (ORDER BY On_Time_Percentage DESC) AS top_rank,RANK() OVER (ORDER BY On_Time_Percentage ASC) AS bottom_rankFROM deliveryagents)SELECT     'Top 5 Agents' AS Category,    AVG(Avg_Speed_KM_HR) AS Avg_SpeedFROM ranked_agentsWHERE top_rank <= 5UNION ALLSELECT     'Bottom 5 Agents',    AVG(Avg_Speed_KM_HR)FROM ranked_agentsWHERE bottom_rank <= 5;
<img width="7027" height="70" alt="image" src="https://github.com/user-attachments/assets/42687e77-91c3-43a0-bb86-ef83885d2fcb" />
WITH ranked_agents AS (SELECT         Agent_ID,        Avg_Speed_KM_HR,        On_Time_Percentage,RANK() OVER (ORDER BY On_Time_Percentage DESC) AS top_rank,RANK() OVER (ORDER BY On_Time_Percentage ASC) AS bottom_rankFROM deliveryagents)SELECT     'Top 5 Agents' AS Category,    AVG(Avg_Speed_KM_HR) AS Avg_SpeedFROM ranked_agentsWHERE top_rank <= 5UNION ALLSELECT     'Bottom 5 Agents',    AVG(Avg_Speed_KM_HR)FROM ranked_agentsWHERE bottom_rank <= 5;
<img width="7027" height="70" alt="image" src="https://github.com/user-attachments/assets/79b27b60-5088-4fa5-9461-dbf0334f6f5a" />
SELECT     Delay_Reason,    COUNT(*) AS FrequencyFROM `shipment tracking`WHERE Delay_Reason <> 'None’ GROUP BY Delay_ReasonORDER BY Frequency DESC;
<img width="2409" height="70" alt="image" src="https://github.com/user-attachments/assets/057acb41-c4e8-4352-8535-657760aba263" />
SELECT     Order_ID,    COUNT(*) AS Delay_CountFROM `shipment tracking`WHERE Delay_Reason <> 'None’ GROUP BY Order_IDHAVING COUNT(*) > 2;
<img width="2211" height="70" alt="image" src="https://github.com/user-attachments/assets/e396922a-86ca-42b3-9922-973d2349b603" />
SELECT     r.Start_Location,COUNT(o.Order_ID) AS Total_Orders,ROUND(AVG(r.Traffic_Delay_Min), 2) AS Avg_Traffic_Delay_MinFROM orders oJOIN routes r ON o.Route_ID = r.Route_IDGROUP BY r.Start_Location;
<img width="3035" height="70" alt="image" src="https://github.com/user-attachments/assets/93691a24-55c0-43eb-920b-c2d3aaf3fa46" />
SELECT COUNT(DISTINCT Order_ID) AS Total_Orders, COUNT(DISTINCT CASE         WHEN Delay_Reason = 'None' THEN Order_ID     END) AS On_Time_Orders,ROUND(COUNT(DISTINCT CASE WHEN Delay_Reason = 'None' THEN Order_ID END) * 100.0         / COUNT(DISTINCT Order_ID), 2) AS On_Time_PercentageFROM `shipment tracking`;
<img width="4877" height="70" alt="image" src="https://github.com/user-attachments/assets/c57eea8c-f059-40c3-ad64-4908398e414c" />
SELECT     o.Route_ID,COUNT(o.Order_ID) AS Total_Orders,ROUND(AVG(r.Traffic_Delay_Min), 2) AS Avg_Traffic_Delay_MinFROM orders oJOIN routes r     ON o.Route_ID = r.Route_IDGROUP BY o.Route_ID;
<img width="2924" height="70" alt="image" src="https://github.com/user-attachments/assets/635ffbc8-f9cb-48ad-b3fb-b0d8764e258e" />

- `screenshots/` — query output screenshots

## 🔮 Next Steps

- Add indexing strategy for scaling this analysis to millions of rows
- Build a companion Power BI dashboard for non-technical stakeholders

## 📬 Contact

**Saipriyambada Mohapatra**
📧 msaipriyambada@gmail.com
🔗 [LinkedIn](https://www.linkedin.com/in/saipriyambada-mohapatra-297065383)
