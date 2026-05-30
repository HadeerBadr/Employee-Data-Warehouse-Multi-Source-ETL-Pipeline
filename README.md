# 🏢 Employee Data Warehouse — Multi-Source ETL Pipeline

![Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen)
![Layer](https://img.shields.io/badge/Layer-STG%20%7C%20DWH-blue)
![Schema](https://img.shields.io/badge/Schema-Star%20Schema-orange)
![Sources](https://img.shields.io/badge/Sources-CSV%20%7C%20JSON%20%7C%20XML-purple)

> **A production-grade Data Warehouse solution** that consolidates employee data from multiple heterogeneous sources into a clean, analytics-ready star schema — with full staging (STG) and data warehouse (DWH) layers.

---

## 📌 Why This Project Matters

Every company — regardless of size or industry — faces the same critical challenge:

> *"We have data everywhere, but we can't use it."*

HR data lives in spreadsheets. Payroll data is exported as XML from legacy ERP systems. Employee records come from APIs as JSON. Each source has its own format, its own naming conventions, its own data quality issues.

**This project solves that.** It demonstrates a complete, real-world ETL pipeline that any organization can adopt to:

- 🔍 **Unify** scattered employee data into a single source of truth
- 📊 **Enable** workforce analytics, salary benchmarking, and headcount reporting
- 🛡️ **Enforce** data quality standards before data ever reaches the warehouse
- ⚡ **Accelerate** decision-making by giving leadership clean, trusted data

---

## 🗂️ Data Sources

This project ingests data from **three real-world source formats** — simulating the reality of most enterprise environments:

| Source | Format | Records | Key Challenges |
|--------|--------|---------|----------------|
| HR System Export | **CSV** | 1,000 rows | `countyr` typo in column name, `salary` stored as string with `$`, dates in `DD.MM.YYYY` |
| Employee API | **JSON** | 1,000 rows | Different schema (`full_name` vs `first_name`/`last_name`), dates in `MM/DD/YYYY`, gender as `M`/`F` |
| Legacy ERP Export | **XML** | 1,000 rows | Schema defined by XSD with weak typing (all fields as `xs:string`), `age` column contains birth dates |

> **The hardest part of real data engineering isn't writing the pipeline — it's handling the mess that comes out of real systems. This project documents and solves every one of these issues.**

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                   SOURCE LAYER                      │
│   [ CSV ]        [ JSON ]        [ XML + XSD ]      │
└────────────┬──────────┬──────────────┬──────────────┘
             │          │              │
             ▼          ▼              ▼
┌─────────────────────────────────────────────────────┐
│              STAGING LAYER (STG)                    │
│  • Raw data loaded as-is                            │
│  • Schema validation                                │
│  • Source tracking metadata                         │
│  • No transformations applied                       │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│           DATA WAREHOUSE LAYER (DWH)                │
│              ⭐ Star Schema                          │
│                                                     │
│   DimEmployee   DimDep   DimlLoc   DIMGender        │
│        └──────────┴──────────┴─────────┘            │
│                       │                             │
│               [ Fact Table ]                        │
│           Fact_id, Emp_id, LOC_ID,                  │
│           DEP_ID, Gender_ID, Date_Key,              │
│           Age, Salary                               │
└─────────────────────────────────────────────────────┘
```

---

## 📐 Data Model — Star Schema

### ⚡ Fact Table: `FactEmployeeSalary`

| Column | Data Type | Allow Nulls | Description |
|--------|-----------|-------------|-------------|
| `Fact_id` | `int` | ❌ | Primary key (surrogate) |
| `Emp_id` | `int` | ✅ | FK → DimEmployee |
| `LOC_ID` | `int` | ✅ | FK → DimlLoc |
| `DEP_ID` | `int` | ✅ | FK → DimDep |
| `Gender_ID` | `int` | ✅ | FK → DIMGender |
| `Date_Key` | `int` | ✅ | FK → DimDate |
| `Age` | `int` | ✅ | Age at snapshot |
| `Salary` | `decimal(18,2)` | ✅ | Monthly salary |

### 📦 Dimension Tables

#### `DimEmployee`
| Column | Type | Notes |
|--------|------|-------|
| `Emp_Id` | `int` PK | Surrogate key |
| `Frist_Name` | `varchar` | ⚠️ Typo preserved from source — consider `First_Name` |
| `Last_Name` | `varchar` | |
| `Email` | `varchar` | |

#### `DimDep` — Department Dimension
| Column | Type | Notes |
|--------|------|-------|
| `DEP_ID` | `int` PK | |
| `DEP_Name` | `varchar` | e.g. Engineering, Sales, HR |

#### `DimlLoc` — Location / Country Dimension
| Column | Type | Notes |
|--------|------|-------|
| `LOC_ID` | `int` PK | |
| `Country_Name` | `varchar` | Standardized from all 3 sources |

#### `DIMGender` — Gender Dimension
| Column | Type | Notes |
|--------|------|-------|
| `Gender_ID` | `int` PK | |
| `Gender` | `varchar` | Standardized: Male / Female |

---

## 🧹 Data Quality Issues Resolved

This project identified and resolved **10+ data quality issues** across all three sources:

| # | Issue | Source | Fix Applied |
|---|-------|--------|-------------|
| 1 | Column `countyr` (typo) | All 3 | Renamed to `country` in STG |
| 2 | `salary` stored as string with `$` | All 3 | Stripped `$`, cast to `decimal(18,2)` |
| 3 | `age` column contains birth dates, not age | All 3 | Renamed to `birth_date`, computed age |
| 4 | 3 different date formats across sources | CSV/JSON/XML | Normalized to `YYYY-MM-DD` |
| 5 | `gender` encoded as `M`/`F` in one source | CSV | Standardized to `Male`/`Female` |
| 6 | `first_name`+`last_name` vs `full_name` | CSV vs JSON | Unified to separate columns |
| 7 | XSD schema defines all fields as `xs:string` | XML | Root cause of issues 1, 2, 3 |
| 8 | `minOccurs="0"` on all XSD fields | XML | No required fields enforced |
| 9 | Suspicious test data names (e.g. "Stinky", "Pooh") | All 3 | Flagged for business review |
| 10 | Data entry error — `@` inside last name field | CSV row 288 | Quarantined in STG |

---

## 🏭 Why Every Company Needs This

This is not an academic exercise. This is the foundation of every data-driven organization:

### 📈 Business Value
- **HR Analytics**: Who are our highest-paid employees? Which departments are growing?
- **Workforce Planning**: Headcount by country, department, and gender
- **Compensation Benchmarking**: Salary distribution across roles and geographies
- **Compliance Reporting**: Clean, auditable employee records

### 🏢 Industry Relevance
| Industry | Use Case |
|----------|----------|
| Banking & Finance | Regulatory headcount and compensation reporting |
| Retail | Store staff analytics across hundreds of locations |
| Healthcare | Staff allocation and workforce optimization |
| Technology | Engineering team growth and attrition tracking |
| Manufacturing | Shift staffing and labor cost analysis |

### ⚙️ Engineering Value
- Demonstrates **separation of concerns** between STG and DWH layers
- Shows how to handle **schema drift** across heterogeneous sources
- Implements a **star schema** optimized for BI tools (Power BI, Tableau, Looker)
- Provides a **reusable pattern** for any similar HR/payroll data problem

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-----------|
| Source Formats | CSV, JSON, XML |
| Schema Validation | XSD |
| Staging Layer | SQL Server / your preferred RDBMS |
| Data Warehouse | SQL Server with Star Schema |
| Data Modeling | Star Schema (Kimball methodology) |
| Version Control | Git / GitHub |

---
## 🚀 Getting Started

### Prerequisites
- SQL Server (localhost\SQLEXPRESS)
- Visual Studio / SSDT (for `.dtsx` SSIS packages)
- SQL Server Management Studio (SSMS)

### 1. Clone the repository
```bash
git clone https://github.com/HadeerBadr/Employee-Data-Warehouse-Multi-Source-ETL-Pipeline.git
cd Employee-Data-Warehouse-Multi-Source-ETL-Pipeline
```

### 2. Set up the source data
The following source files are already included in the repo:
- `MOCK_DATA.json` — Employee API data (JSON)
- `MOCK_DATA.txt` / `MOCK_DATA.xlsx` / `MOCK_DATA_Excel.txt` — HR system exports
- `dataset.xml` + `dataset.xsd` — Legacy ERP export with schema

### 3. Open the SSIS project
1. Open `EmpDatawarehouse.dtproj` in **Visual Studio**
2. Restore the connection manager using `localhost_SQLEXPRESS.DWH.conmgr`
3. Set your SQL Server instance to `localhost\SQLEXPRESS`

### 4. Run the Staging layer
Execute `STG.dtsx` first — this loads raw data from all 3 sources into the staging database:

### 5. Run the Data Warehouse layer
After staging completes, execute `DWH.dtsx` — this transforms and loads data into the star schema:

### 6. Verify in SSMS
Open SSMS and connect to `localhost\SQLEXPRESS` → database `EmpDatawarehouse`

Check the tables:
```sql
SELECT * FROM DimEmployee;
SELECT * FROM DimDep;
SELECT * FROM DimlLoc;
SELECT * FROM DIMGender;
SELECT * FROM FactEmployeeSalary;
```
```

---

## 📊 Sample Queries

```sql
-- Total salary spend by department
SELECT d.DEP_Name, SUM(f.Salary) AS total_salary
FROM FactEmployeeSalary f
JOIN DimDep d ON f.DEP_ID = d.DEP_ID
GROUP BY d.DEP_Name
ORDER BY total_salary DESC;

-- Headcount by country
SELECT l.Country_Name, COUNT(*) AS headcount
FROM FactEmployeeSalary f
JOIN DimlLoc l ON f.LOC_ID = l.LOC_ID
GROUP BY l.Country_Name
ORDER BY headcount DESC;

-- Average salary by gender
SELECT g.Gender, AVG(f.Salary) AS avg_salary
FROM FactEmployeeSalary f
JOIN DIMGender g ON f.Gender_ID = g.Gender_ID
GROUP BY g.Gender;
```

---

## 🔮 Roadmap

- [ ] Add `DimDate` table with full calendar attributes
- [ ] Add SCD Type 2 to `DimEmployee` for history tracking
- [ ] Build automated data quality checks in STG layer
- [ ] Fix `Frist_Name` typo in `DimEmployee` → `First_Name`
- [ ] Add `birth_date` column to `DimEmployee` (moved from fact)
- [ ] Integrate with Power BI for dashboard layer

---

## 👤 Author
Hadeer Badr Hassan

Built as a complete end-to-end data engineering project covering data profiling, staging, and warehouse design.

**Connect on LinkedIn** | **Star this repo ⭐ if it was useful**

---

## 📄 License

MIT License — feel free to use this as a template for your own data warehouse projects.
