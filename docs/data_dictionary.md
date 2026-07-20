# Data Dictionary - IBM HR Analytics Dataset

Source: Kaggle - IBM HR Analytics Employee Attrition & Performance
Rows: 1,470 | Original columns: 35 | Columns after cleaning: 34

---

## Engineered Columns

| Column         | Type    | Logic                                              |
|---------------|---------|-----------------------------------------------------|
| AttritionFlag  | Integer | Attrition="Yes" → 1, else → 0                      |
| EducationLabel | Text    | 1=Below College, 2=College, 3=Bachelor, 4=Master, 5=Doctor |
| HireDate       | Date    | Date.AddYears(#date(2024,6,30), -[YearsAtCompany]) |

## Removed Columns

| Column        | Value in every row | Reason removed            |
|--------------|-------------------|---------------------------|
| EmployeeCount | 1                 | Always 1 — meaningless    |
| Over18        | Y                 | Always Y — meaningless    |
| StandardHours | 80                | Always 80 — meaningless   |

## Key Source Columns

| Column                  | Type    | Description                          |
|------------------------|---------|--------------------------------------|
| EmployeeNumber          | Text    | Unique employee ID (not a number)    |
| Age                     | Integer | Employee age in years                |
| Attrition               | Text    | "Yes" or "No"                        |
| Department              | Text    | Human Resources / R&D / Sales        |
| JobRole                 | Text    | 9 unique roles                       |
| MonthlyIncome           | Integer | Monthly salary in USD                |
| OverTime                | Text    | "Yes" or "No"                        |
| YearsAtCompany          | Integer | Total tenure in years                |
| YearsSinceLastPromotion | Integer | Years since last promotion           |
| JobSatisfaction         | Integer | 1–4 scale (4 = highest)              |
| EnvironmentSatisfaction | Integer | 1–4 scale                            |
| WorkLifeBalance         | Integer | 1–4 scale                            |
| RelationshipSatisfaction| Integer | 1–4 scale                            |
| PerformanceRating       | Integer | 3 or 4 only in this dataset          |
| EducationField          | Text    | 6 unique fields                      |
| Education               | Integer | 1–5 scale (see EducationLabel)       |
