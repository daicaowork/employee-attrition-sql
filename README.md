# 📊 Employee Attrition Analysis using SQL
## 📝 Project Introduction

This project explores employee attrition trends using SQL queries on an HR dataset.
The main goal is to uncover insights that help HR departments improve retention strategies, optimize compensation policies, and identify risk factors for employee turnover.
Dataset: Includes employee demographics, department, job role, salary, tenure, promotions, marital status, distance from home, and more.
Tools Used: SQL Server (queries and analysis).
SQL Techniques Applied:
Aggregate Functions (COUNT, SUM, AVG, ROUND)
Conditional Logic (CASE WHEN)
Window Functions (ROW_NUMBER, RANK, SUM() OVER, AVG() OVER)
Common Table Expressions (CTEs)
Subqueries and Cumulative Analysis

## 📂 Business Questions & SQL Queries


---

### 1. Attrition Rate by Department
Calculate attrition rate across departments.  

```sql
-- 1. Attrition Rate by Department
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
``` 

---
### 2. Attrition Rate by Gender within Each Department
```sql
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

``` 

### 3. Attrition Rate by Age Band
```sql
SELECT
    CF_age_band,
    COUNT(*) AS total_employees,
    SUM(is_attrition) AS total_attrition,
    ROUND(100.0 * SUM(is_attrition) / COUNT(*), 2) AS attrition_rate_percent
FROM (
    SELECT *,
           CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END AS is_attrition
    FROM hr_attrition
) AS attrition_flag
GROUP BY CF_age_band;

``` 
### 4. Attrition Rate by Marital Status
```sql
SELECT
    Marital_Status,
    COUNT(*) AS total_employees,
    SUM(is_attrition) AS total_attrition,
    ROUND(100.0 * SUM(is_attrition) / COUNT(*), 2) AS attrition_rate_percent
FROM (
    SELECT *,
           CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END AS is_attrition
    FROM hr_attrition
) AS attrition_flag
GROUP BY Marital_Status;
```

### 5. Top 3 Job Roles with Highest Attrition Rates
```sql
SELECT TOP 3
    Job_Role,
    COUNT(*) AS total_employees,
    SUM(is_attrition) AS total_attrition,
    ROUND(100.0 * SUM(is_attrition) / COUNT(*), 2) AS attrition_rate_percent
FROM (
    SELECT *,
           CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END AS is_attrition
    FROM hr_attrition
) AS attrition_flag
GROUP BY Job_Role
ORDER BY attrition_rate_percent DESC;
```

### 6. Average Monthly Income: Ex-Employees vs Current Employees
```sql
SELECT 
    employee_status,
    AVG(Monthly_Income) AS avg_monthly_income
FROM (
    SELECT *,
           CASE WHEN Attrition = 'Yes' THEN 'Ex-Employees'
                ELSE 'Current Employees'
           END AS employee_status
    FROM hr_attrition
) AS income_flag
GROUP BY employee_status;
```

### 7. Attrition Rate by Percent Salary Hike
```sql
SELECT  
    Percent_Salary_Hike,
    COUNT(*) AS total_employees,
    SUM(is_attrition) AS total_attrition,
    ROUND(100.0 * SUM(is_attrition) / COUNT(*), 2) AS attrition_rate_percent
FROM (
    SELECT *,
           CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END AS is_attrition
    FROM hr_attrition
) AS attrition_flag
GROUP BY Percent_Salary_Hike
ORDER BY attrition_rate_percent DESC;
```

### 8. Highest Paid Employee in Each Department
```sql
SELECT *
FROM (
    SELECT
        Employee_Number,
        Department,
        Monthly_Income,
        ROW_NUMBER() OVER (PARTITION BY Department ORDER BY Monthly_Income DESC) AS salary_rank
    FROM hr_attrition
) AS ranked_salaries
WHERE salary_rank = 1;

```
### 9. Top 5 Longest Commutes among Current Employees
```sql
SELECT TOP 5
    Employee_Number,
    Distance_From_Home
FROM hr_attrition
WHERE Attrition = 'No'
ORDER BY Distance_From_Home DESC;

```
### 10. Cumulative Attrition Trend by Years Since Last Promotion
```sql
WITH cte_attrition_flag AS (
    SELECT *,
           CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END AS is_attrition
    FROM hr_attrition
),
agg_promotion AS (
    SELECT  
        Years_Since_Last_Promotion,
        COUNT(*) AS total_employees,
        SUM(is_attrition) AS total_attrition,
        ROUND(100.0 * SUM(is_attrition) / COUNT(*), 2) AS attrition_rate_percent
    FROM cte_attrition_flag
    GROUP BY Years_Since_Last_Promotion
)
SELECT
    Years_Since_Last_Promotion,
    total_employees,
    total_attrition,
    attrition_rate_percent,
    SUM(total_attrition) OVER (ORDER BY Years_Since_Last_Promotion) AS cumulative_attrition,
    SUM(total_employees) OVER (ORDER BY Years_Since_Last_Promotion) AS cumulative_employees,
    ROUND(100.0 * SUM(total_attrition) OVER (ORDER BY Years_Since_Last_Promotion) 
              / SUM(total_employees) OVER (ORDER BY Years_Since_Last_Promotion), 2) AS cumulative_attrition_rate_percent
FROM agg_promotion
ORDER BY Years_Since_Last_Promotion;

```
### 11. Employees Above Department Average Tenure with Attrition Rate
```sql
WITH cte_avg_tenure AS (
    SELECT *,
           CAST(AVG(Years_At_Company) OVER (PARTITION BY Department) AS DECIMAL(10,2)) AS avg_years_in_dept,
           CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END AS is_attrition
    FROM hr_attrition
),
cte_above_avg_tenure AS (
    SELECT
        h.Employee_Number,
        h.Department,
        h.Age,
        h.Years_At_Company,
        a.avg_years_in_dept,
        a.is_attrition
    FROM hr_attrition AS h
    JOIN cte_avg_tenure AS a
        ON h.Employee_Number = a.Employee_Number
    WHERE h.Years_At_Company > a.avg_years_in_dept
)
SELECT 
    Employee_Number,
    Department,
    Age,
    Years_At_Company,
    avg_years_in_dept,
    CASE WHEN is_attrition = 0 THEN 'Current Employee' ELSE 'Ex-Employee' END AS employee_status,
    SUM(is_attrition) OVER (PARTITION BY Department) AS total_attrition_in_dept,
    COUNT(Employee_Number) OVER (PARTITION BY Department) AS total_employees_in_dept,
    ROUND(
        100.0 * SUM(is_attrition) OVER (PARTITION BY Department) 
              / COUNT(Employee_Number) OVER (PARTITION BY Department),
        2
    ) AS attrition_rate_percent
FROM cte_above_avg_tenure
ORDER BY Department, Years_At_Company DESC;
```
