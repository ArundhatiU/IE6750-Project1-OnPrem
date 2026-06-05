# IE6750-Project1-OnPrem
# Operational Analytics for Multi-Location Fitness Centers

A centralized data warehouse and analytics platform built for a multi-location fitness 
chain, enabling cross-location visibility into facility utilization, member engagement, 
instructor performance, and equipment usage.

---

## Team

Arundhati Ubhad, Muthu Chellappa, Senthilkumaran Ramanathan

---

## Problem Statement

Fitness centers generate large volumes of operational data across member check-ins, 
class enrollments, equipment usage, and staff scheduling — but this data lives in 
disconnected systems. Without integrated analytics, decision-makers cannot optimize 
facility usage, allocate staff efficiently, or evaluate instructor effectiveness.

---

## Objective

Design and implement an end-to-end data warehouse that integrates operational data 
from multiple fitness center locations to support analytics on:

- Facility and equipment utilization
- Class enrollment and attendance patterns
- Instructor performance and member feedback
- Staff scheduling and labor efficiency
- Membership trends and retention

---

## Tech Stack

| Layer | Tool |
|---|---|
| Database | PostgreSQL |
| ETL | Talend Open Studio |
| Data Warehouse | PostgreSQL (Star Schema) |
| BI & Dashboards | Tableau |
| Data Generation | Python (Faker library) |
| Version Control | GitHub |

---

## Architecture Overview

```plaintext
Synthetic ODS Data (PostgreSQL)
        │
        ▼
   Talend ETL Jobs
   ├── Dimension Loads (SCD2)     ← Member, Staff, Facility, Equipment, Date, Time
   ├── Fact Table Loads           ← Class Enrollment, Equipment Usage
   └── Membership History (SCD4)  ← Full membership tier change tracking
        │
        ▼
   Star Schema Data Warehouse
        │
        ▼
   Tableau Dashboards
   ├── Equipment Operations
   ├── Class Engagement
   └── Staff & Instructor Performance
```

---

## Data Model

### Fact Tables

| Fact Table | Key Measures |
|---|---|
| `Fact_Class_Enrollment` | enrollment count, attendance, no-shows, cancellations, instructor rating, payment amount |
| `Fact_Equipment_Usage` | usage duration, session count, peak hour flag, intensity level |

### Dimension Tables

`Dim_Member` · `Dim_Staff` · `Dim_Facility` · `Dim_FacilityArea` · `Dim_Equipment` · 
`Dim_Class` · `Dim_Geography` · `Dim_Date` · `Dim_Time` · `Dim_MembershipTypes`

### Hierarchies

```plaintext
Geography  : Country → State → City → Facility → Facility Area
Date       : Year → Quarter → Month → Week → Day
Time       : Hour → Minute (+ Morning / Afternoon / Evening blocks)
Equipment  : Category → Type → Item
Class      : Class Type → Schedule → Instance → Enrollment
```

---

## ETL Pipeline (Talend)

- **Base Hierarchy Control Job** — parallel loading of foundational dimensions
  (Staff, ClassType, Geography, Time, Date, Facilities) using `tParallelize`
- **Member Dimension Load** — file-based ingestion with SCD logic, lookup validation,
  and error logging
- **Membership History (SCD Type 4)** — tracks full tier transition history
  (e.g. Basic → Premium) with before/after snapshots for auditability
- **Fact_Equipment_Usage Load** — sequential dimension lookups to resolve surrogate
  keys before inserting usage records
- **Fact_Class_Enrollment Load** — multi-dimension join with unmatched key logging
  and intermediate staging files

---

## Dataset

- **Type:** Synthetic, generated via Python (Faker library)
- **Scale:** MDM reference tables + 5,000–10,000 ODS transactional records
- **Scope:** Member access logs, class bookings, equipment usage, staff schedules,
  payroll, instructor feedback

---

## Tableau Dashboards

### Dashboard 1 — Equipment Operations
- Usage by equipment category (bar chart)
- Drill-down from category → type → individual machine
- Demographics filter: Cardio vs Strength by gender
- KPI: Utilization Score

**Key Insight:** Treadmills drive 3× the volume of Ellipticals within Cardio —
informing next quarter's purchasing decisions.

### Dashboard 2 — Class Engagement
- Dual-axis chart: Enrollment vs Actual Attendance (reveals no-show gap)
- Cancellation trend with time drill-down: Year → Quarter → Month
- Peak class times heatmap

**Key Insight:** Booking numbers masked a no-show problem only visible after
tracking actual attendance separately.

### Dashboard 3 — Staff & Instructor Performance
- Average instructor rating by staff member
- Identifies top performers and coaching needs

---

## Key Business Questions Answered

- Which equipment categories and machines have the highest utilization?
- What is the attendance vs no-show rate by class type and day of week?
- Which instructors have the highest ratings and lowest no-show rates?
- What are peak facility usage hours across locations?
- How do membership tiers change over time across the member base?

---

## How to Run

### Prerequisites
- PostgreSQL installed and running
- Talend Open Studio for Data Integration
- Tableau Desktop or Tableau Public

### Steps

```bash
# Clone the repo
git clone https://github.com/mchellappa/IE6750-Project1-OnPrem.git
cd IE6750-Project1-OnPrem

# Set up ODS schema
psql -U <user> -d <database> -f ProjectSetup/ods_schema.sql

# Generate synthetic data
python data_generation/generate_data.py

# Load data warehouse
# Open Talend and run: BaseHierarchyControlJob → MemberDimensionLoad → FactLoads

# Open Tableau workbooks from /dashboards
```

> Detailed setup steps available at:
> [Implementation Roadmap](https://github.com/mchellappa/IE6750-Project1-OnPrem/blob/main/ProjectSetup/Implementation_Roadmap.md)

---

## Limitations & Future Improvements

- Dataset is synthetic; not connected to a live operational system
- Real-time streaming (Kafka + Spark) would replace daily batch ETL in production
- Predictive analytics for capacity planning and churn detection are planned extensions
- Role-based access control and row-level security not yet implemented in Tableau

---

## Learnings

- End-to-end data warehouse design from ODS → dimensional model → BI layer
- SCD Type 2 and Type 4 implementation in Talend for slowly changing dimensions
- Star schema design with additive, semi-additive, and derived measures
- Tableau drill-down, drill-through, and cross-dashboard navigation
- Synthetic data generation strategies for realistic transactional patterns
