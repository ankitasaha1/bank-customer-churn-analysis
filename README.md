# Bank Customer Churn Analysis

##  Project Overview
This end-to-end data analysis project explores customer churn behavior 
at a bank using a dataset of 10,000 customers across France, Germany, 
and Spain. The goal is to identify key factors driving customer churn 
and present actionable insights through an interactive Tableau dashboard.

---

##  Live Dashboard
[View on Tableau Public]https://public.tableau.com/shared/YW4QG7NK7?:display_count=n&:origin=viz_share_link

---

##  Dataset
- **Source:** Kaggle — Bank Customer Churn Prediction
- **Rows:** 10,000 customers
- **Columns:** 14 features
- **File:** Churn_Modelling.csv

### Columns Used:
| Column | Description |
|--------|-------------|
| CreditScore | Customer credit score (300–850) |
| Geography | Country (France, Germany, Spain) |
| Gender | Male / Female |
| Age | Customer age |
| Tenure | Years with bank (0–10) |
| Balance | Account balance |
| NumOfProducts | Number of bank products |
| HasCrCard | Has credit card (0/1) |
| IsActiveMember | Active member (0/1) |
| EstimatedSalary | Estimated salary |
| Exited | Churned (1) or Retained (0) |

---

##  Tools Used
- **DB Browser for SQLite** — Data cleaning & EDA
- **Tableau Public** — Dashboard & visualization
- **GitHub** — Version control & portfolio

---

##  Data Cleaning (SQL)

### 1. Row Count Validation
```sql
SELECT COUNT(*) FROM Churn_Modelling;
-- Result: 10,000 rows confirmed
```

### 2. Duplicate Check
```sql
SELECT CustomerID, COUNT(*) 
FROM Churn_Modelling 
GROUP BY CustomerID 
HAVING COUNT(*) > 1;
-- Result: No duplicates found
```

### 3. Null Value Check
```sql
SELECT 
  SUM(CASE WHEN CreditScore IS NULL THEN 1 ELSE 0 END) AS null_credit,
  SUM(CASE WHEN Age IS NULL THEN 1 ELSE 0 END) AS null_age,
  SUM(CASE WHEN Balance IS NULL THEN 1 ELSE 0 END) AS null_balance,
  SUM(CASE WHEN Geography IS NULL THEN 1 ELSE 0 END) AS null_geo,
  SUM(CASE WHEN Gender IS NULL THEN 1 ELSE 0 END) AS null_gender
FROM Churn_Modelling;
-- Result: All zeros — no null values
```

### 4. Range Validation
```sql
SELECT 
  MIN(CreditScore), MAX(CreditScore),
  MIN(Age), MAX(Age),
  MIN(Balance), MAX(Balance),
  MIN(Tenure), MAX(Tenure)
FROM Churn_Modelling;
-- CreditScore: 350–850 
-- Age: 18–92 
-- Balance: 0–250,898 
-- Tenure: 0–10 
```

### 5. Categorical Validation
```sql
SELECT Geography, COUNT(*) FROM Churn_Modelling GROUP BY Geography;
-- France: 5014 | Germany: 2509 | Spain: 2477 

SELECT Gender, COUNT(*) FROM Churn_Modelling GROUP BY Gender;
-- Female: 4543 | Male: 5457
```

### Cleaning Summary
| Check | Result |
|-------|--------|
| Total Rows | 10,000  |
| Duplicates | 0  |
| Null Values | 0  |
| Invalid Ranges | None  |
| Category Errors | None  |

---

##  Exploratory Data Analysis (SQL)

### Overall Churn Rate
```sql
SELECT Exited, COUNT(*) as Count,
ROUND(COUNT(*) * 100.0 / 10000, 2) AS Percentage
FROM Churn_Modelling
GROUP BY Exited;
```
| Status | Count | Percentage |
|--------|-------|------------|
| Retained | 7,963 | 79.63% |
| Churned | 2,037 | 20.37% |

### Churn by Geography
```sql
SELECT Geography, COUNT(*) as Total,
SUM(Exited) as Churned,
ROUND(SUM(Exited) * 100.0 / COUNT(*), 2) AS Churn_Rate
FROM Churn_Modelling
GROUP BY Geography;
```
| Geography | Total | Churned | Churn Rate |
|-----------|-------|---------|------------|
| France | 5,014 | 810 | 16.15% |
| Germany | 2,509 | 814 | 32.44% |
| Spain | 2,477 | 413 | 16.67% |

### Churn by Gender
```sql
SELECT Gender, COUNT(*) as Total,
SUM(Exited) as Churned,
ROUND(SUM(Exited) * 100.0 / COUNT(*), 2) AS Churn_Rate
FROM Churn_Modelling
GROUP BY Gender;
```
| Gender | Total | Churned | Churn Rate |
|--------|-------|---------|------------|
| Female | 4,543 | 1,139 | 25.07% |
| Male | 5,457 | 898 | 16.46% |

### Churn by Age Group
```sql
SELECT 
  CASE 
    WHEN Age < 30 THEN 'Under 30'
    WHEN Age BETWEEN 30 AND 45 THEN '30-45'
    WHEN Age BETWEEN 46 AND 60 THEN '46-60'
    ELSE 'Above 60'
  END AS Age_Group,
  COUNT(*) as Total,
  SUM(Exited) as Churned,
  ROUND(SUM(Exited) * 100.0 / COUNT(*), 2) AS Churn_Rate
FROM Churn_Modelling
GROUP BY Age_Group;
```
| Age Group | Total | Churned | Churn Rate |
|-----------|-------|---------|------------|
| Under 30 | 1,641 | 124 | 7.56% |
| 30-45 | 6,248 | 956 | 15.30% |
| 46-60 | 1,647 | 842 | 51.12% |
| Above 60 | 464 | 115 | 24.78% |

### Churn by Number of Products
```sql
SELECT NumOfProducts, COUNT(*) as Total,
SUM(Exited) as Churned,
ROUND(SUM(Exited) * 100.0 / COUNT(*), 2) AS Churn_Rate
FROM Churn_Modelling
GROUP BY NumOfProducts;
```
| Products | Total | Churned | Churn Rate |
|----------|-------|---------|------------|
| 1 | 5,084 | 1,409 | 27.71% |
| 2 | 4,590 | 348 | 7.58%  |
| 3 | 266 | 220 | 82.71% |
| 4 | 60 | 60 | 100.00% |

### Churned vs Retained — Avg Profile
```sql
SELECT Exited,
ROUND(AVG(Balance), 2) AS Avg_Balance,
ROUND(AVG(CreditScore), 2) AS Avg_CreditScore,
ROUND(AVG(Age), 2) AS Avg_Age
FROM Churn_Modelling
GROUP BY Exited;
```
| Status | Avg Balance | Avg CreditScore | Avg Age |
|--------|-------------|-----------------|---------|
| Retained | 72,745 | 651 | 37 |
| Churned | 91,108 | 645 | 45 |

---

## Key Business Insights

1. **20.37% Overall Churn Rate** — 1 in 5 customers left the bank

2. **Germany Churn Crisis** — Germany has a 32.44% churn rate, double that of France (16.15%) and Spain (16.67%). Urgent retention strategies needed for German customers.

3. **Female Customers Churn More** — Females churn at 25.07% vs 16.46% for males. The bank should investigate gender-specific dissatisfaction factors.

4. **Middle-Aged Customers at High Risk** — The 46–60 age group has a staggering 51.12% churn rate. More than 1 in 2 customers in this segment leave the bank.

5. **Product Overload Drives Churn** — Customers with 3 products churn at 82.71% and those with 4 products churn at 100%. This suggests product bundling beyond 2 is counterproductive.

6. **Inactive Members Churn More** — Inactive members churn at 26.85% vs 14.27% for active members. Re-engagement campaigns could significantly reduce churn.

7. **Churned Customers Have Higher Balances** — Avg balance of churned customers (91,108) is higher than retained (72,745), suggesting high-value customers are being lost.

---

##  Recommendations

-  Launch targeted retention campaigns in **Germany**
-  Investigate and address **female customer** dissatisfaction
-  Create special loyalty programs for **46–60 age group**
-  Review and simplify **product bundling strategy**
- Re-engage **inactive members** through personalized outreach
   Prioritize retention of **high-balance customers**

---

## Dashboard Preview
![Dashboard]<img width="1203" height="558" alt="BANK CHURN- 2" src="https://github.com/user-attachments/assets/7840d8e3-b929-46ac-b435-25c48bbcbe6a" />
<img width="1200" height="551" alt="BANK CHURN-3" src="https://github.com/user-attachments/assets/1e4d9b24-9544-4cca-987f-0b8f37c7603e" />
<img width="1204" height="536" alt="bank churn-main" src="https://github.com/user-attachments/assets/09210f18-6db1-473b-8144-20a3079754bd" />





---

## 🗂️ Project Structure
