# Pizza Place Sales Analytical Dashboard

An interactive, multi-page **Power BI Analytics Dashboard** powered by a **PostgreSQL** relational semantic layer. The project models raw operational retail data (orders, line items, menu items, and ingredient categories) into aggregated analytical views in SQL before visualizing financial performance, peak transaction frequencies, and granular sales trends.

---

## Executive Summary

* **Business Focus:** Revenue performance tracking, hourly visit frequency, menu performance, and transactional detail exploration.
* **Architecture Pattern:** Database-First Semantic Layer (PostgreSQL SQL Modeling $\rightarrow$ Power BI Visualization).
* **Data Source:** [Maven Analytics - Pizza Place Dataset](https://www.mavenanalytics.io/) (1 year of transaction data).
* **Core Stack:** PostgreSQL, SQL (DDL/DML, Window Functions, Aggregations), Power BI (Data Visualization, Multi-Page Navigation, Interactive Filtering).

---

## System Architecture & Data Pipeline

Rather than performing heavy computations and relationship mapping dynamically inside Power BI using DAX, the data is pre-aggregated, cleaned, and structured directly within **PostgreSQL**. Pushing data transformations to the SQL engine reduces report rendering latency and creates a clean, decoupled reporting layer.

```
flowchart TD
    subgraph Storage ["1. Relational Staging Layer (PostgreSQL)"]
        A[(raw_orders)] 
        B[(raw_order_details)]
        C[(raw_pizzas)]
        D[(raw_pizza_types)]
    end

    subgraph Transformation ["2. SQL Modeling & Semantic Layer"]
        A & B & C & D -->|Joins, Grouping & Window Functions| E[Analytical Views & SQL Datasets]
        E --> F[Calculated Measures & Aggregations<br/>Total Revenue, AOV, Peak Hours]
    end

    subgraph Visualization ["3. Power BI Interactive Dashboard"]
        F -->|Direct Import| G[Power BI Reporting Engine]
        G --> H[Page 1: About & Context]
        G --> I[Page 2: Sales Overview]
        G --> J[Page 3: Deep-Dive Analysis]
        G --> K[Page 4: Granular Transaction Table]
    end
```

---

## Data Model & Relational Schema

The underlying database maps 4 primary operational tables into a cohesive relational structure:

```
+-------------------+        +----------------------+        +-------------------+
|      ORDERS       |        |    ORDER_DETAILS     |        |      PIZZAS       |
+-------------------+        +----------------------+        +-------------------+
| order_id (PK)     |----<   | order_details_id(PK) |   >----| pizza_id (PK)     |
| date              |        | order_id (FK)        |        | pizza_type_id(FK) |
| time              |        | pizza_id (FK)        |        | size              |
+-------------------+        | quantity             |        | price             |
                             +----------------------+        +-------------------+
                                                                       |
                                                                       v
                                                             +-------------------+
                                                             |    PIZZA_TYPES    |
                                                             +-------------------+
                                                             | pizza_type_id(PK) |
                                                             | name              |
                                                             | category          |
                                                             | ingredients       |
                                                             +-------------------+
```

---

## SQL Transformations & Business Logic

Core metrics were engineered directly in SQL before importing datasets into Power BI:

### 1. Total Revenue Calculation
```sql
SELECT 
    SUM(od.quantity * p.price) AS total_revenue
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id;
```

### 2. Hourly Sales & Visit Frequency Analysis
```sql
SELECT 
    EXTRACT(HOUR FROM o.time) AS sales_hour,
    COUNT(DISTINCT o.order_id) AS total_orders,
    SUM(od.quantity * p.price) AS hourly_revenue
FROM orders o
JOIN order_details od ON o.order_id = od.order_id
JOIN pizzas p ON od.pizza_id = p.pizza_id
GROUP BY EXTRACT(HOUR FROM o.time)
ORDER BY sales_hour ASC;
```

---

## Dashboard Structure & Features

The Power BI report is organized into four purpose-built pages:

* **Page 1: About:** Concise overview detailing project objectives, data scope, and business context.
* **Page 2: Sales Overview:** High-level executive summary displaying Total Revenue, Average Order Value (AOV), top-performing menu categories, and revenue distribution by month.
* **Page 3: Analysis:** In-depth operational performance tracking peak customer visit hours, volume of units sold per pizza category, and seasonal trends.
* **Page 4: Transaction Table:** Granular line-item search view equipped with multi-parameter slicers (Category, Date Range, Product Type) for ad-hoc auditing.

---

## Technical Trade-Offs & Key Decisions

* **SQL Aggregation vs. DAX Computation:** Pushing business logic into PostgreSQL views drastically reduced Power BI file size and query overhead, ensuring smooth visual transitions across pages.
* **Static Ingredients vs. Unnested String Analysis:** Kept ingredient lists consolidated within `pizza_types` to preserve menu display accuracy while utilizing categorical metrics for product popularity.