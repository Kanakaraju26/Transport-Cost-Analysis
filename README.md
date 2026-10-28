# 🚛 Transportation Cost Optimization & Delivery Performance Analysis

An interactive, 3-page Power BI dashboard engineered to analyze transportation expenditure, unit cost efficiency, carrier reliability, and route performance for a logistics business case.

## 📌 Executive Summary

* **Total Spend Analyzed:** ₹1.15B

* **Shipment Volume:** 15.31K unique shipments

* **On-Time Delivery SLA:** 86.98%

* **Primary Cost Driver:** Air mode absorbs **51%** of total transport expenditure (₹592M).

## 🎯 Business Objectives

This project resolves three core business challenges faced by supply chain managers:

1. **Expenditure Concentration:** Where is transportation capital deployed across modes, cost components, and carriers?

2. **Cost Efficiency & Unit Economics:** Which carriers and shipping lanes exhibit high total spend vs. rate-level inefficiencies?

3. **Delivery Reliability:** Which carriers, modes, and routes deviate most from promised delivery schedules?

## 📊 Dashboard Architecture & Structure

The dashboard is structured into three dedicated, highly focused reporting pages:

### Page 1 — Executive Overview

Designed for executive decision-makers to track macro KPIs, seasonal cost trends, mode allocation, non-freight cost breakdowns, and carrier-level expenditure.

### Page 2 — Cost & Carrier Analysis

Focuses on unit economics, cost per distance/weight, top route spending, and distance-band cost distribution across transport modes.

### Page 3 — Delivery Performance

Measures operational SLAs, delivery duration compliance (Actual vs. Promised Days), carrier delay rankings, and quarterly reliability trends.

## 🛠️ Data Transformation & Modeling

### Data Model Overview

The model uses a star-schema-inspired design linking a central `Shipments` fact table with a dedicated `Calendar` table and a normalized `Cost Breakdown` table.

```
                                  +-----------------------+
                                  |       Shipments       |
                                  +-----------------------+
                                  | Shipment ID (PK)      |
                                  | Date (FK)             |
                                  | Origin / Destination  |
                                  | Mode / Carrier        |
                                  | Distance / Weight     |
                                  | Total Cost            |
                                  | OnTime (Boolean)      |
                                  +-----------------------+
                                              | 1
                                              |
                                              | *
                                  +-----------------------+
                                  |    Cost Breakdown     |
                                  +-----------------------+
                                  | Shipment ID (FK)      |
                                  | Cost Component        |
                                  | Cost Amount           |
                                  +-----------------------+

```

### Power Query Unpivot Logic

To compare non-freight cost drivers without creating complex DAX measure arrays, six separate cost columns were referenced and unpivoted:

```
let
    Source = Shipments,
    SelectCostCols = Table.SelectColumns(Source, {"Shipment ID", "Freight Cost", "Fuel Cost", "Handling Cost", "Warehousing Cost", "LastMile Cost", "Other Cost"}),
    UnpivotedCosts = Table.UnpivotOtherColumns(SelectCostCols, {"Shipment ID"}, "Cost Component", "Cost Amount")
in
    UnpivotedCosts

```

## 📐 Key DAX Calculations

### 1. Financial & Volume Measures

```
Total Transportation Cost = 
SUM(Shipments[Total Cost])

```

```
Total Freight Cost = 
SUM(Shipments[Freight Cost])

```

```
Total Shipments = 
DISTINCTCOUNT(Shipments[Shipment ID])

```

```
Average Cost per Shipment = 
DIVIDE(
    [Total Transportation Cost],
    [Total Shipments],
    0
)

```

### 2. Efficiency & Unit Cost DAX

```
Cost per KM = 
DIVIDE(
    [Total Transportation Cost],
    SUM(Shipments[Distance (km)]),
    BLANK()
)

```

```
Cost per KG = 
DIVIDE(
    [Total Transportation Cost],
    SUM(Shipments[Weight (kg)]),
    BLANK()
)

```

### 3. Delivery SLA & Delay DAX

```
On-Time Delivery % = 
DIVIDE(
    CALCULATE([Total Shipments], Shipments[OnTime] = 1),
    [Total Shipments],
    0
)

```

```
Average Delay Days = 
AVERAGEX(
    Shipments,
    MAX(0, Shipments[Actual Days] - Shipments[Promised Days])
)

```

### 4. Distance Band & Non-Circular Sorting

```
Distance Band = 
SWITCH(
    TRUE(),
    ISBLANK(Shipments[Distance (km)]), BLANK(),
    Shipments[Distance (km)] <= 500, "0–500",
    Shipments[Distance (km)] <= 1000, "501–1000",
    Shipments[Distance (km)] <= 1500, "1001–1500",
    Shipments[Distance (km)] <= 2000, "1501–2000",
    "2001+"
)

```

```
Distance Band Sort = 
SWITCH(
    TRUE(),
    ISBLANK(Shipments[Distance (km)]), BLANK(),
    Shipments[Distance (km)] <= 500, 1,
    Shipments[Distance (km)] <= 1000, 2,
    Shipments[Distance (km)] <= 1500, 3,
    Shipments[Distance (km)] <= 2000, 4,
    5
)

```

## 💡 Key Business Insights

1. **Air Transport Capital Concentration:** Air mode accounts for **51% (₹592M)** of total costs. While Air delivers high SLA compliance (**91.85%**), its average cost per shipment (**₹242K**) is 4.5x higher than Road (**₹53K**).

2. **Secondary Overhead Isolation:** Outside raw freight, **Fuel Cost (₹109M)** and **Handling Cost (₹70M)** represent the top secondary operational expenses.

3. **Route Spending Concentration:** Directional shipping lanes originating from **Chennai** (Chennai → Hyderabad at ₹26M, Chennai → Pune at ₹25M) absorb the largest route-level budget.

4. **SLA Bottlenecks:** **Ocean Line Two** generates the highest average delay (**5.5 days**), making it a key candidate for contract renegotiation or route re-alignment.

## 💻 Tech Stack & Tools Used

* **Business Intelligence Tool:** Microsoft Power BI Desktop

* **Data Transformation:** Power Query (M Language)

* **Data Modeling & Analytics:** DAX (Data Analysis Expressions)

* **Theme & UI:** Custom High-Contrast Dark Navy / Neon Cyan / Orange Layout (1280x720 Canvas)