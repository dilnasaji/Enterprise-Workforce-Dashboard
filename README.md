# Enterprise Workforce Intelligence Dashboard
### Power BI Portfolio Project - Targeting Financial Services . IT Consulting . Management Advisory

## Project Overview 
A fully governed, enterprise-grade **Workforce Analytics Dashboard** built entirely in Power BI Desktop. This was a self-directed skill-building project to understand what real Business Analysis work looks like in three major industry verticals, before stepping into those environments professionally.

The project transforms a raw 35-column IBM HR flat file into a structured, multi-stakeholder analytical system with three firm-targeted report pages, 20 production-ready DAX measures, Row-Level Security, and 8 advanced UX optimisations.

| Lens | Target Firm Profile | Key Questions Answered |
|---|---|---|
| 📘 Financial Risk View | Financial Services (e.g. Manulife) | What is attrition costing us? Where is our payroll exposure? |
| 📗 Tech Workforce View | IT Consulting (e.g. CGI) | Where are we over capacity? Who is at burnout risk? |
| 📙 Strategic Advisory View | Management Consulting (e.g. Deloitte) | Which departments are at critical retention risk? |

---

## Dataset 
| Attribute | Detail |
|---|---|
| **Source** | [IBM HR Analytics Employee Attrition & Performance -Kaggle](https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset) |
| **Rows** | 1470 employees |
| **Raw Columns** | 35 |
| **Columns After Cleaning** | 32 (3 constant columns removed) |
| **Null Values** | 0-100% data quality score |
| **Date Coverage** | Synthetic HireDate engineered from YearsAtCompany - 2007 to 2024 |

---

## Repository Structure 
enterprise-workforce-intelligence-powerbi/
│
├── README.md                              ← You are here
├── EnterpriseWorkforceIntelligence.pbix   ← Main Power BI file
│
├── docs/
│   ├── dax-measures.md                   ← All 20 DAX measures documented
│   ├── project-summary.md                ← Full 5-phase project write-up
│   ├── data-dictionary.md                ← What every column means
│   └── screenshots/
│       ├── star-schema.png               ← Model View screenshot
│       ├── financial-risk-view.png       ← Page 1 screenshot
│       ├── tech-workforce-view.png       ← Page 2 screenshot
│       └── strategic-advisory-view.png  ← Page 3 screenshot
│
└── data/
    └── WA_HR_Dataset.csv                 ← Raw IBM source dataset

---

## Technical Architecture 
### Star Schema - 5 Table Model
                    ┌─────────────────┐
                    │  DimCalendar    │
                    │  Date (PK)      │
                    │  Year           │
                    │  Quarter        │
                    │  MonthName      │
                    │  FiscalYear     │
                    └────────┬────────┘
                             │ 1
                             │
     ┌─────────────┐    *    │    1    ┌──────────────────┐
     │DimDepartment├─────────┤─────────┤  DimJobRole      │
     │Department(PK│         │         │  JobRole (PK)    │
     └─────────────┘         │         └──────────────────┘
                             │
                    ┌────────┴──────────────┐
                    │  FactEmployeeMetrics  │  ← 1470 rows
                    │  EmployeeNumber       │
                    │  MonthlyIncome        │
                    │  AttritionFlag        │
                    │  HireDate (FK)        │
                    │  Department (FK)      │
                    │  JobRole (FK)         │
                    │  EducationField (FK)  │
                    │  + 25 other columns   │
                    └────────┬──────────────┘
                             │
                             │ *
                             │ 1
                    ┌────────┴────────────┐
                    │  DimEducationField  │
                    │  EducationField(PK) │
                    └─────────────────────┘
**Relationship rules applied across all 4 relationships:**
- Cardinality: One-to-Many (1:*)
- Cross-filter direction: Single (dimension → fact only)
- All join columns hidden from Report View

---

## Phase Breakdown 
### Phase 1. Data Engineering (Power Query)

**Problem:** A raw 35-column flat CSV with no dates, constant columns, text-based attrition, and no analytical structure.

**Actions taken:**
- Removed 3 constant columns: `EmployeeCount` (always 1), `Over18` (always Y), `StandardHours` (always 80)
- Changed `EmployeeNumber` data type: Numeric → Text (it is an ID, not a number to sum)
- Engineered `AttritionFlag`: Attrition = "Yes" → 1, else → 0 (required for all DAX math)
- Engineered `EducationLabel`: decoded numeric scale 1-5 into readable text labels
- Engineered `HireDate`: `Date.AddYears(#date(2024, 6, 30), -[YearsAtCompany])` - synthetic date derived from tenure, enabling time intelligence where no date column existed
- Created `DimCalendar` table via DAX: 2007-2024 range, 12 date attributes including FiscalYear
- Marked `DimCalendar` as official Date Table
- Renamed query to `FactEmployeeMetrics`

**Key decision:** HireDate did not exist in the source data. Deriving it from tenure using a fixed reference date is a real-world pattern used when source systems only store tenure rather than timestamps.

---

### Phase 2. Star Schema (Model View)
**5 tables. 4 relationships. All 1-to-many. Single cross-filter direction.**

| Dimension Table | Primary Key | Rows |
|---|---|---|
| DimCalendar | Date | 6575 |
| DimDepartment | Department | 3 |
| DimJobRole | JobRole | 9 |
| DimEducationField | EducationField | 6 |

**Key decisions:**
- Dimension tables created (not Duplicate) — stays linked to source, updates on refresh
- **Single cross-filter direction** on all relationships - prevents ambiguous filter paths, enterprise standard
- All foreign key join columns **hidden from Report View** in both fact and dimension tables
- **Auto Date/Time disabled** - removes Power BI's hidden auto-generated date tables, reduces model bloat.

---

### Phase 3. Advanced DAX (20 Measures)

All measures stored in a dedicated `_Measures` table (underscore prefix pins it to the top of the Data pane alphabetically), organised into 5 display folders.

| Folder | Measures |
|---|---|
| 1. Core Workforce | Total Employees, Total Attrition, Attrition Rate %, Active Employees |
| 2. Financial Cost | Total Monthly Cost, Total Annual Cost, Avg Monthly Income, Cost Per Employee, Revenue Projection |
| 3. Utilization | Overtime Headcount, Overtime Rate %, Workforce Utilization Rate |
| 4. Risk & Satisfaction | Attrition Risk Score, Satisfaction Index, Attrition Cost Estimate |
| 5. Time Intelligence | YTD Annual Cost, LY Annual Cost, YoY Cost Growth %, High Performer Count, Avg Tenure |

**Highlight measures:**

**dax**
-- Dynamic risk classifier - thresholds calibrated against actual data distributions
Attrition Risk Score =
VAR AvgSatisfaction = AVERAGE(FactEmployeeMetrics[JobSatisfaction])
VAR AvgWorkLife     = AVERAGE(FactEmployeeMetrics[WorkLifeBalance])
VAR OvertimePct     = [Overtime Rate %]
VAR PromotionGap    = AVERAGE(FactEmployeeMetrics[YearsSinceLastPromotion])
RETURN
SWITCH(TRUE(),
    AvgSatisfaction < 2.5 && OvertimePct > 0.25, "🔴 Critical Risk",
    AvgSatisfaction < 2.5 || OvertimePct > 0.25, "🟠 High Risk",
    AvgWorkLife     < 2.8 || PromotionGap > 2.0, "🟡 Medium Risk",
    "🟢 Low Risk"
)

 **Critical build note:** Initial thresholds (e.g., JobSatisfaction <= 2.0) caused the model to return static "Low Risk" output regardless of filter context - because the actual dataset average is 2.73. Thresholds were recalibrated against real column distributions before the model became functional. This is the most important lesson from the entire project.

-- Composite satisfaction score using AVERAGEX row-level iterator
Satisfaction Index =
AVERAGEX(FactEmployeeMetrics,
    (FactEmployeeMetrics[JobSatisfaction]         +
     FactEmployeeMetrics[EnvironmentSatisfaction] +
     FactEmployeeMetrics[WorkLifeBalance]         +
     FactEmployeeMetrics[RelationshipSatisfaction]) / 4
)

-- Live cost model connected to What-If parameter slider
Attrition Cost Estimate =
VAR AvgSalary         = [Avg Monthly Income]
VAR LostHeadcount     = [Total Attrition]
VAR ReplacementMonths = SELECTEDVALUE('Replacement Cost Months'[Replacement Cost Months], 6)
RETURN LostHeadcount * AvgSalary * ReplacementMonths

See [`docs/dax-measures.md`](docs/dax-measures.md) for all 20 measures with full documentation.

### Phase 4. Executive Dashboard(3 Pages + RLS)
#### Page 1 - Financial Risk View 
*Designed for Financial Services audience - focuses on cost exposure and attrition liability*

- **KPI Cards:** Total Annual Cost . Cost Per Employee . Attrition Cost Estimate  Attrition Rate %
- **Charts:** Monthly Income by Job Role (clustered bar) . Annual Cost Trend by Year (line) . Attrition by Department (donut) . YoY Cost Growth (KPI visual)
- **Slicer:** Department - Dropdown style

#### Page 2 - Tech Workforce View
*Designed for IT Consulting audience - focuses on capacity, utilization, and skills*

- **KPI Cards:** Total Employees . Overtime Headcount . Workforce Utilization Rate . High Performer Count
- **Charts:** Overtime by Department (stacked bar) . Education Field Breakdown (donut) . Performance Rating Distribution (column) . Income vs Tenure by Role (scatter)
- **Slicer:** Education Field - Tile style

#### Page 3 - Strategic Advisory View
*Designed for a Management Consulting audience - focuses on risk intelligence and workforce health*

- **KPI Cards:** Attrition Risk Score . Satisfaction Index . Avg Tenure . Active Employees
- **Charts:** Satisfaction by Department (clustered bar) . Attrition vs Tenure by Role (line & clustered column) . Department × Job Level Risk Heatmap (matrix - green to red conditional formatting) . Overtime Rate (gauge)
- **Slicer:** Job Role - Dropdown, multi-select

#### Row-Level Security - 3 Roles

| Role | Filter Applied | Simulates |
|---|---|---|
| `Manulife_View` | Department = "Sales" | Financial services stakeholder |
| `CGI_View` | Department = "Research & Development" | IT consulting stakeholder |
| `Deloitte_View` | Department = "Human Resources" | Management consulting stakeholder |

To test: **Modeling tab → View as → select a role** - the entire report filters to that department only.

---

### Phase 5 - Enterprise Optimisations (8 Upgrades)

| # | Upgrade | Technique |
|---|---|---|
| 1 | Dynamic report titles | `SELECTEDVALUE` + `ISFILTERED` DAX measure bound to text box via fx |
| 2 | Page navigation buttons | Insert → Buttons → Navigator → Page navigator |
| 3 | What-If scenario slider | Numeric range parameter (1–12 months) → `SELECTEDVALUE` in measure |
| 4 | Tooltip pages | Hidden report page (320×240) assigned to chart tooltip property |
| 5 | Drillthrough page | Drillthrough field well → Department detail page → Back button |
| 6 | Field Parameters | Dynamic axis switcher — 4 measures selectable via tile slicer |
| 7 | Dynamic Top N | `RANKX` + numeric parameter + visual-level filter set to = 1 |
| 8 | Auto date/time disabled | File → Options → Current File → Data Load → untick Auto date/time |

---

## Key Findings from the Dataset

| Metric | Value | Signal |
|---|---|---|
| Attrition Rate | 16.1% (237 employees) | Above healthy benchmark of 10-12% |
| Overtime Rate | 28% (416 employees) | Over a quarter of workforce overloaded |
| Workforce Utilization | 93% | Approaching burnout risk threshold (>90%) |
| Satisfaction Index | 2.72 / 4.0 | Below the 3.0 healthy threshold |
| Avg Tenure | ~7.0 years | Moderate - masks early-exit patterns within individual roles |
| Attrition Cost Estimate | $Multi-million | Based on 6-month replacement assumption (adjustable via slider) |

---

## Technical Decisions & Reasoning

| Decision | Why |
|---|---|
| `Reference` not `Duplicate` for dimension tables | Reference tables stay linked to source - changes propagate on refresh |
| Single cross-filter direction on all relationships | Prevents ambiguous filter paths - enterprise BI standard |
| `DIVIDE()` instead of `/` in all ratio measures | Returns 0 instead of error when denominator is zero |
| `VAR` pattern in all multi-step measures | Improves readability, prevents repeated calculation, production DAX standard |
| `_Measures` table with underscore prefix | Alphabetical pinning to top of Data pane — governance best practice |
| Disabled Auto Date/Time | Removes Power BI's hidden auto-generated date tables — reduces model bloat |
| Synthetic `HireDate` from tenure | No real date existed — this is a real-world pattern for tenure-only source systems |
| Threshold calibration against real distributions | Initial assumed thresholds caused static model output — recalibrated after profiling |

---

## The Critical Lesson

**Build your assumptions last, not first. Validate thresholds against actual data distributions before writing a single line of logic around them.**

The SWITCH-based Attrition Risk Scorer initially returned identical output irrespective of which department was selected. The thresholds looked logical on paper - but were set below a level. After running column distribution analysis and recalibrating all four thresholds against actual values, the scorer became genuinely dynamic and analytically meaningful.

This is not just a Power BI lesson. It is the core BA principle applied to every scoring model, risk matrix, and KPI threshold.

---

## About

Built as a self-directed Business Analyst portfolio project to build real analytical skills and understand what enterprise BI work looks like in Financial Services, IT Consulting, and Management Advisory.
LinkedIn: https://www.linkedin.com/in/dilna-saji/

---

*Dataset: IBM HR Analytics - publicly available on Kaggle for educational use. All analysis performed on anonymised synthetic data.*

