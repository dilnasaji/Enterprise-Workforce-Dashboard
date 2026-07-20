# Project Summary

## Phase 1 — Data Engineering (Power Query)
**Problem:** Raw 35-column flat CSV with no dates, no structure, constant columns, and text-based attrition field.

**Actions taken:**
- Removed 3 constant columns: EmployeeCount, Over18, StandardHours
- Changed EmployeeNumber data type: Numeric → Text (it is an ID, not a number)
- Engineered AttritionFlag: "Yes" → 1, "No" → 0 (enables all DAX math)
- Engineered EducationLabel: decoded 1–5 scale to readable text
- Engineered HireDate: Date.AddYears(#date(2024,6,30), -[YearsAtCompany])
- Created DimCalendar via DAX: 2007–2024, 12 date attributes
- Marked DimCalendar as official Date Table
- Query renamed to FactEmployeeMetrics

**Key insight:** HireDate did not exist in the source. Derived from YearsAtCompany to enable all time intelligence measures.

---

## Phase 2 — Star Schema (Model View)
**5 tables. 4 relationships. All 1-to-many. Single cross-filter direction.**

| Dimension Table   | PK Column      | Rows |
|-------------------|---------------|------|
| DimCalendar       | Date           | 6,575|
| DimDepartment     | Department     | 3    |
| DimJobRole        | JobRole        | 9    |
| DimEducationField | EducationField | 6    |

- Dimension tables created via Reference (not Duplicate)
- All join columns hidden from Report View
- Single cross-filter direction on all 4 relationships
- Auto Date/Time disabled (File → Options → Data Load)

---

## Phase 3 — DAX Measures
20 measures in _Measures table across 5 display folders.
See dax-measures.md for full documentation.

**Patterns used:**
- VAR/RETURN — Revenue Projection, Attrition Cost Estimate
- SWITCH(TRUE()) — Attrition Risk Score (multi-condition risk classifier)
- AVERAGEX — Satisfaction Index (row-level iterator)
- CALCULATE + filter — Overtime Headcount, High Performer Count
- SELECTEDVALUE — What-If parameter integration
- DATESYTD, SAMEPERIODLASTYEAR — Time intelligence measures

**Critical fix:** SWITCH thresholds recalibrated from assumed to empirical ranges after initial model returned static "Low Risk" output
regardless of filter context.

---

## Phase 4 — Dashboard (3 Pages + RLS)

### Page 1 — Financial Risk View (Eg taken: Manulife lens)
KPIs: Total Annual Cost · Cost Per Employee · Attrition Cost Estimate · Attrition Rate %
Charts: Income by Job Role (Bar) · Cost Trend by Year (Line) · 
Attrition by Dept (Donut) · YoY Cost KPI visual · Department slicer

### Page 2 — Tech Workforce View (Eg taken: CGI lens)
KPIs: Total Employees · Overtime Headcount · Utilization Rate · High Performer Count
Charts: Overtime by Dept (Stacked bar) · Education Breakdown (Donut) ·
Performance Distribution (Column) · Income vs Tenure (Scatter) ·
Education Field tile slicer

### Page 3 — Strategic Advisory View (Eg Taken: Deloitte lens)
KPIs: Attrition Risk Score · Satisfaction Index · Avg Tenure · Active Employees
Charts: Satisfaction by Dept (Bar) · Attrition vs Tenure by Role (Line + Column) ·
Dept × Job Level heatmap (matrix, green→red conditional formatting) ·
Overtime Rate (gauge) · Job Role dropdown slicer

### Row-Level Security — 3 Roles
| Role           | Filter                           |
|----------------|----------------------------------|
| Manulife_View  | Department = "Sales"             |
| CGI_View       | Department = "Research & Development" |
| Deloitte_View  | Department = "Human Resources"   |

---

## Phase 5 — Optimisations (8 of 9 upgrades)

| # | Upgrade                  | Technique                              |
|---|--------------------------|----------------------------------------|
| 1 | Dynamic report titles    | SELECTEDVALUE + ISFILTERED DAX measure |
| 2 | Page navigation buttons  | Insert → Buttons → Page Navigator     |
| 3 | What-If scenario slider  | Numeric range parameter + SELECTEDVALUE|
| 4 | Tooltip pages            | Hidden page 320×240, assigned to chart |
| 5 | Drillthrough page        | Drillthrough well + Back button        |
| 6 | Field Parameters         | Dynamic axis switcher, 4 measures      |
| 7 | Dynamic Top N            | RANKX + numeric parameter + filter     |
| 8 | Auto date/time disabled  | File → Options → Data Load             |
