# DAX Measures

All measures stored in _Measures table, organised into 5 display folders.

---

## 1. Core Workforce

**Total Employees**
```dax
Total Employees = COUNTROWS(FactEmployeeMetrics)
```
Counts every row in the fact table. One row per employee.

**Total Attrition**
```dax
Total Attrition = SUM(FactEmployeeMetrics[AttritionFlag])
```
Sums the 0/1 flag engineered in Power Query. Attrition=Yes(1), No(0).

**Attrition Rate %**
```dax
Attrition Rate % = DIVIDE([Total Attrition], [Total Employees], 0)
```
DIVIDE used instead of / to safely return 0 on divide-by-zero.

**Active Employees**
```dax
Active Employees = [Total Employees] - [Total Attrition]
```

---

## 2. Financial Cost

**Total Monthly Cost**
```dax
Total Monthly Cost = SUM(FactEmployeeMetrics[MonthlyIncome])
```

**Total Annual Cost**
```dax
Total Annual Cost = [Total Monthly Cost] * 12
```

**Avg Monthly Income**
```dax
Avg Monthly Income = AVERAGE(FactEmployeeMetrics[MonthlyIncome])
```

**Cost Per Employee**
```dax
Cost Per Employee = DIVIDE([Total Annual Cost], [Total Employees], 0)
```

**Revenue Projection**
```dax
Revenue Projection =
VAR AnnualCost        = [Total Annual Cost]
VAR RevenueMultiplier = 3.5
RETURN AnnualCost * RevenueMultiplier
```
Models potential revenue using a 3.5x salary-to-revenue multiplier.

---

## 3. Utilization

**Overtime Headcount**
```dax
Overtime Headcount =
CALCULATE([Total Employees], FactEmployeeMetrics[OverTime] = "Yes")
```

**Overtime Rate %**
```dax
Overtime Rate % = DIVIDE([Overtime Headcount], [Total Employees], 0)
```

**Workforce Utilization Rate**
```dax
Workforce Utilization Rate =
VAR BaseCapacity   = [Total Employees]
VAR OvertimeStrain = [Overtime Headcount] * 0.25
VAR AdjustedCap    = BaseCapacity - OvertimeStrain
RETURN DIVIDE(AdjustedCap, BaseCapacity, 0)
```
Above 90% = burnout risk. Below 60% = underutilized.

---

## 4. Risk & Satisfaction

**Attrition Risk Score**
```dax
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
```
Note: Thresholds were calibrated against actual data distributions,
not assumed ranges. Initial thresholds caused static output.

**Satisfaction Index**
```dax
Satisfaction Index =
AVERAGEX(FactEmployeeMetrics,
    (FactEmployeeMetrics[JobSatisfaction] +
     FactEmployeeMetrics[EnvironmentSatisfaction] +
     FactEmployeeMetrics[WorkLifeBalance] +
     FactEmployeeMetrics[RelationshipSatisfaction]) / 4
)
```
Composite score using AVERAGEX iterator. Score out of 4. Healthy ≥ 3.0.

**Attrition Cost Estimate**
```dax
Attrition Cost Estimate =
VAR AvgSalary         = [Avg Monthly Income]
VAR LostHeadcount     = [Total Attrition]
VAR ReplacementMonths = SELECTEDVALUE('Replacement Cost Months'[Replacement Cost Months], 6)
RETURN LostHeadcount * AvgSalary * ReplacementMonths
```
Connected to What-If parameter slider (1–12 months).

---

## 5. Time Intelligence

**YTD Annual Cost**
```dax
YTD Annual Cost = CALCULATE([Total Annual Cost], DATESYTD(DimCalendar[Date]))
```

**LY Annual Cost**
```dax
LY Annual Cost = CALCULATE([Total Annual Cost], SAMEPERIODLASTYEAR(DimCalendar[Date]))
```

**YoY Cost Growth %**
```dax
YoY Cost Growth % = DIVIDE([Total Annual Cost] - [LY Annual Cost], [LY Annual Cost], 0)
```

**High Performer Count**
```dax
High Performer Count = CALCULATE([Total Employees], FactEmployeeMetrics[PerformanceRating] = 4)
```

**Avg Tenure**
```dax
Avg Tenure = ROUND(AVERAGE(FactEmployeeMetrics[YearsAtCompany]), 1)
```
