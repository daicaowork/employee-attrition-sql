# 📊 Employee Attrition Analysis using SQL

## 📂 Business Questions & SQL Queries

---

### 1. Attrition Rate by Department
Calculate attrition rate across departments.  

```sql
SELECT
    Department,
    COUNT(*) AS total_employees,
    SUM(is_attrition) AS total_attrition,
    ROUND(100.0 * SUM(is_attrition) / COUNT(*), 2) AS attrition_rate_percent
FROM (
    SELECT *,
           CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END AS is_attrition
    FROM hr_attrition
) AS attrition_flag
GROUP BY Department;

### 2. Attrition Rate by Gender within Each Department

Compare attrition rate between genders across departments.

SELECT
    Department,
    Gender,
    COUNT(*) AS total_employees,
    SUM(is_attrition) AS total_attrition,
    ROUND(100.0 * SUM(is_attrition) / COUNT(*), 2) AS attrition_rate_percent
FROM (
    SELECT *,
           CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END AS is_attrition
    FROM hr_attrition
) AS attrition_flag
GROUP BY Department, Gender;
