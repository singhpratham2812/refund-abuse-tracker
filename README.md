# E-Commerce Refund Abuse Tracker

![Excel](https://img.shields.io/badge/Tool-Microsoft%20Excel-217346?style=flat&logo=microsoftexcel&logoColor=white)
![Dataset](https://img.shields.io/badge/Dataset-UCI%20Online%20Retail%20II-0C447C?style=flat)
![Domain](https://img.shields.io/badge/Domain-Risk%20%26%20Fraud%20Analytics-E24B4A?style=flat)
![Status](https://img.shields.io/badge/Status-Completed-639922?style=flat)

---

## Overview

Return abuse is a significant financial risk in e-commerce. Customers exploit liberal refund policies through excessive returns, multi-account manipulation, and low-value product exploitation — resulting in measurable GMV loss that often goes undetected without systematic monitoring.

This project builds a **risk intelligence layer** on top of raw transactional data to automatically flag high-risk customers, quantify refund loss patterns, and deliver an actionable Excel dashboard for operations and risk teams.

> **Real-world context:** This project mirrors the kind of refund abuse monitoring I designed and implemented as part of the Risk team at Noon (a leading Middle East e-commerce platform), rebuilt here on a public dataset for portfolio purposes.

---

## Problem Statement

Given a dataset of 541,909 e-commerce transactions, can we:
1. Identify customers who are abusing the return policy?
2. Quantify the financial impact of refund abuse over time?
3. Classify customers into risk tiers to enable targeted interventions?
4. Surface the most abused products and peak abuse periods?

---

## Dataset

| Attribute | Detail |
|-----------|--------|
| Name | UCI Online Retail II |
| Source | [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/502/online+retail+ii) / [Kaggle](https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci) |
| Rows | 541,909 transactions |
| Period | December 2009 – December 2011 |
| Geography | UK-based online retailer |
| License | Public / Open |

### Key columns used

| Column | Description |
|--------|-------------|
| `Invoice` | Transaction ID — prefix `C` indicates a cancellation/return |
| `Quantity` | Units sold — negative values indicate returned items |
| `Price` | Unit price in GBP |
| `Customer ID` | Unique customer identifier (note: ~130K rows have no ID — guest checkouts excluded from customer-level analysis) |
| `InvoiceDate` | Transaction timestamp |
| `Description` | Product name |
| `Country` | Customer location |

---

## Methodology

### Step 1 — Flag return transactions
Identified returns using two signals present in the raw data:
- Invoice numbers prefixed with `"C"` (cancellation)
- Negative `Quantity` values

```excel
=IF(LEFT([@Invoice],1)="C","Yes","No")
```

### Step 2 — Engineer risk features
Added 8 calculated columns to create a customer-level risk profile:

| Column | Formula | Purpose |
|--------|---------|---------|
| `Is_Return` | `=IF(LEFT([@Invoice],1)="C","Yes","No")` | Return flag |
| `Order_Value` | `=[@Quantity]*[@Price]` | Transaction value |
| `Month` | `=TEXT([@InvoiceDate],"MMM-YYYY")` | For trend analysis |
| `Customer_Total_Orders` | `=COUNTIF(...)` | Order frequency per customer |
| `Customer_Total_Returns` | `=COUNTIFS(...)` | Return frequency per customer |
| `Return_Rate_%` | `=Returns/Orders` | Abuse intensity signal |
| `Risk_Score` | Nested IF on return rate | High / Medium / Low tier |
| `Repeat_Offender` | `=IF(Returns>=3,"Yes","No")` | Binary abuse flag |

### Step 3 — Risk scoring model
Applied threshold-based classification mirroring real-world fraud policy design:

| Tier | Threshold | Action |
|------|-----------|--------|
| **High Risk** | Return rate > 50% | Flag for manual review, restrict returns |
| **Medium Risk** | Return rate 25–50% | Monitor closely, soft restrictions |
| **Low Risk** | Return rate < 25% | No action required |

### Step 4 — Aggregate with Pivot Tables
Built 4 Pivot Tables to power the dashboard:
- Monthly refund loss trend
- Top 10 highest return-rate customers (watch list)
- Most returned products
- Risk score distribution

### Step 5 — Dashboard
Designed an operational Excel dashboard with KPI cards, 4 charts, and interactive slicers for Risk Score and Month filtering.

---

## Key Findings

| KPI | Value |
|-----|-------|
| Total GMV | £2,908,178 |
| Total Refund Loss | £143,445 |
| Total Returns | 3,134 |
| High-Risk Customers | 445 |
| Avg Return Rate (per customer) | 18.4% |

**Insight 1 — Pareto concentration:** The top 2% of customers account for approximately 38% of total refund value. A small, identifiable cohort drives disproportionate loss.

**Insight 2 — Seasonal spike:** Return abuse peaks in November–December, aligning with promotional sale periods. Tightening return windows during promotional events would reduce peak exposure.

**Insight 3 — Product signal:** The top 10 most-returned products are predominantly low-value items (under £5). This pattern is more consistent with policy exploitation than genuine quality complaints.

**Data quality note:** ~130,000 rows had no Customer ID (guest checkouts and some cancellation records). These were excluded from customer-level KPIs using COUNTIFS-based deduplication. Total transaction metrics (GMV, refund loss, return count) include all rows.

---

## Dashboard

> <img width="1303" height="524" alt="image" src="https://github.com/user-attachments/assets/2c816686-7db5-459d-8a87-c14f7712b275" />
*

The dashboard contains:
- **5 KPI cards** — Total GMV, Refund Loss, Total Returns, High-Risk Customers, Avg Return Rate
- **Line chart** — Monthly refund loss trend
- **Bar chart** — Top 10 highest return-rate customers
- **Bar chart** — Most returned products
- **Donut chart** — Risk score distribution (High / Medium / Low)

---

## Technical Notes

**Performance optimization:** Working on the full 541K row dataset with COUNTIFS across the entire column caused significant lag. A subsample of 50,000 rows was used for the customer-level helper columns — a standard practice when working with large datasets in Excel. The KPI metrics referencing total transaction counts use the full dataset.

**Deduplication approach:** KPIs for unique customer counts use `SUMPRODUCT` with `COUNTIFS`-based deduplication rather than the simpler `COUNTIF` division method — the latter fails silently when blank Customer IDs are present, returning 0 instead of an error.

```excel
=SUMPRODUCT((Table1[Risk_Score]="High")*(1/COUNTIFS(Table1[Customer ID],Table1[Customer ID],Table1[Risk_Score],"High")))
```

---

## Tools Used

- **Microsoft Excel** — Data processing, feature engineering, Pivot Tables, dashboard
- **Formulas** — COUNTIFS, SUMPRODUCT, AGGREGATE, IF, TEXT, LEFT
- **Features** — Pivot Tables, Slicers, Conditional Formatting, Dynamic Charts

---

## Learnings

- Handling 540K+ rows in Excel with AGGREGATE for performance-safe calculations
- Identifying silent formula failures caused by blank foreign keys (Customer IDs)
- Designing risk thresholds that mirror real fraud policy logic, not just statistical quartiles
- Building dashboards for operational use (actionable, filterable) vs. analytical use (exploratory)
- Deduplicating at customer level to avoid row-count inflation in KPI metrics

---

## Roadmap

This project is part of a broader portfolio series building toward end-to-end fraud analytics capability:

| # | Project | Tools | Status |
|---|---------|-------|--------|
| 1 | E-Commerce Refund Abuse Tracker | Excel | ✅ Complete |
| 2 | Support Team KPI Dashboard | Excel | 🔄 In progress |
| 3 | Fraud Detection SQL Case Study | SQL | 📅 Planned |
| 4 | Edtech Cohort | Power BI | 📅 Planned |
| 5 | ATO Pattern Detection | Python + SQL | 📅 Planned |

---

## Author

**Pratham Singh**

- Email: pratham.singh2800@gmail.com
- LinkedIn: *https://www.linkedin.com/in/prathamsingh9996/*
- Dataset source: [UCI Online Retail II](https://archive.ics.uci.edu/dataset/502/online+retail+ii)
