# Loan Disbursal Department from CI Capital

This provides an overview of the data analysis conducted on CI Capital's dataset, which includes demographic data, behavioral data, and macroeconomic data.

## Dataset Description

### 1. Demographic Data (At Account Level)
- Account Number
- Type of Loan
- Gender
- City
- Age
- Education Level
- Income
- Marital Status

### 2. Behavioral Data (At Account Level)
- Account Number
- Type of Loan
- Credit Score
- Percentage of Credit Utilization
- Credit Limit
- Number of Times Defaulted on Loans

### 3. Macroeconomic Data (At Monthly Level)
- Real GDP
- Retail Sales
- Unemployment Rate
- Consumer Price Index (CPI)

-------------------------------------------------------------------------------------------------------------------------------------
#### Question 1: What are the different types of loans given by CI Capital? 
#### How many distinct loan types are given by CI Capital?

```sql
SELECT DISTINCT loan_type 
FROM CI_customer;
```
#### Question 2: Find out the number of loans for each loan type? 
#### How many Auto loans for two-wheelers have been given?

```sql
SELECT loan_type, Count(*) AS Number_of_loans
FROM CI_customer
GROUP BY loan_type;
```
#### Question 3: Find out the customers who are less than 30 years old and have taken loans? 
#### What is the age of account_no CI11?

```sql
-- List of Customers with Age Less Than 30
SELECT *
FROM CI_customer
WHERE age < 30;
```
#### Question 4: How many loans have been given where credit score is less than 580 by different loan types? 
#### What is the minimum credit score for the Housing Loan?

```sql
SELECT loan_type, Count(account_no) AS count_of_loans
FROM CI_loan
WHERE credit_score < 580
GROUP BY loan_type;
```
```sql
SELECT *
FROM ci_loan
WHERE credit_score < 580
AND loan_type = "hl"
ORDER BY credit_score;
```

#### Question 5: Find out the average income of customers who have credit scores more than 700 and have been defaulted?
#### What is the average annual income of the customers who have defaulted?

```sql
SELECT b.if_default, Avg(annual_income) AS Average_annual_income
FROM CI_customer a
INNER JOIN CI_loan b
ON a.account_no = b.account_no
WHERE b.credit_score > 700
GROUP BY b.if_default;
```
#### Question 6: What is the average credit score by different marital status?
#### What is the average credit score for widower?

```sql
SELECT a.marital_status, Avg(b.credit_score) AS Average_credit_Score
FROM CI_customer a
INNER JOIN CI_loan b
ON a.account_no = b.account_no
GROUP BY a.marital_status;
```
#### Question 7: How many customers have more than or equal to 5 defaults by different education levels? 
#### How many customers who are doing Masters education have been defaulted?

```sql
SELECT a.education_level, Sum(b.if_default) as default_count
FROM CI_customer a
INNER JOIN CI_loan b
ON a.account_no = b.account_no
GROUP BY a.education_level
HAVING default_count >= 5;
```
#### Question 8: Create a report that shows the relationship between the number of loans granted for each month and respective unemployment rate. It should be sorted by unemployment rate, from lowest to highest. 
    Note: The CI_economics table has data from 2018 to 2020.
    Report Should contain the following Columns in the same exact sequence:
    Report_Month, Real_GDP_in_Lakh_Crore, unemp_rate, Count of Loans

#### What is the unemployment rate of the country in Feb 2019?

```sql
SELECT a.report_month, a.real_gdp_in_lakh_crore, a.unemp_rate, Count(b.account_no) AS count_of_loans
FROM CI_economics a
LEFT JOIN CI_loan b
ON Year(a.report_month) = Year(b.start_date)
AND Month(a.report_month) = Month(b.start_date)
GROUP BY a.report_month, a.real_gdp_in_lakh_crore, a.unemp_rate
ORDER BY a.unemp_rate ASC;
```
