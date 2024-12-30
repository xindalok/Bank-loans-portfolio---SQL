### Total Loan Applications
Calculate total loan applications, track MTD and MoM changes.

<br>

**Total Applications:** **38,576**
	
```sql
SELECT COUNT(DISTINCT id) AS total_applications 
FROM financial_loan fl;
```

**Total Applications MoM changes:** <br>
Calculate monthly totals and Month-over-Month (MoM) changes using a subquery and window functions.

<details>
<summary style="color: lightblue;">▶▶Show code </summary>
	
```sql
SELECT 
	month_num,
	month,
	total_applications,
	(total_applications - LAG(total_applications) OVER (ORDER BY month_num))::NUMERIC AS mom_change,
	ROUND((total_applications - LAG(total_applications) OVER (ORDER BY month_num))::NUMERIC / (LAG(total_applications) OVER (ORDER BY month_num)),4) AS mom_pct_change
FROM (
SELECT 
	EXTRACT(MONTH from issue_date) AS month_num,
    	TO_CHAR(issue_date, 'Mon') AS month,
    	COUNT(DISTINCT id) AS total_applications
FROM financial_loan fl
GROUP BY EXTRACT(MONTH from issue_date), TO_CHAR(issue_date, 'Mon')
ORDER BY month_num
) AS ordered_month
ORDER BY month_num
```

**Subquery (ordered_month):**

1. Extracts the numerical month (EXTRACT(MONTH FROM issue_date) AS month_num) and abbreviated month name (TO_CHAR(issue_date, 'Mon') AS month) from the issue_date column.
2. Groups by and orders the data by month_num to ensure chronological ordering.
3. Counts distinct loan application IDs for each month (COUNT(DISTINCT id) AS total_applications).

**Outer Query:**

1. Calculates the absolute MoM change using the LAG window function:

``` sql
(total_applications - LAG(total_applications) OVER (ORDER BY month_num))::NUMERIC AS mom_change
```
The LAG function retrieves the previous month's total_applications value for each row, and the difference is cast to NUMERIC.

2. Calculates the percentage MoM change:

``` sql
ROUND((total_applications - LAG(total_applications) OVER (ORDER BY month_num))::NUMERIC / (LAG(total_applications) OVER (ORDER BY month_num)), 4) AS mom_pct_change
```
The difference in applications is divided by the previous month's total and rounded to four decimal places. The LAG function ensures this calculation references the correct preceding row.

**Ordering:** <br>
The final output is sorted by month_num in ascending order to maintain chronological order.
</details>

<img src="images/MoM_changes.png" width="500" height="300" />

---------------------------------------------

### Total Funded Amount
Calculate total funded amount, track MTD and MoM changes.

**Total funded amount:** **$435,757,075**
```sql
SELECT 
	SUM(loan_amount) 
FROM financial_loan fl ;
```
**Total Applications MoM changes:** <br>

``` sql
SELECT 
	month_num,
	month,
	total_monthly_loan,
	total_monthly_loan - LAG(total_monthly_loan) OVER(ORDER BY month_num) AS mom_change,
	ROUND((total_monthly_loan - (LAG(total_monthly_loan) OVER(ORDER BY month_num)))::NUMERIC / (LAG(total_monthly_loan) OVER(ORDER BY month_num)),4)  AS mom_pct_change
FROM (
SELECT 
	EXTRACT(MONTH from issue_date) AS month_num,
	TO_CHAR(issue_date,'Mon') AS month,
	SUM(loan_amount) AS total_monthly_loan
FROM financial_loan fl 
GROUP BY EXTRACT(MONTH from issue_date),TO_CHAR(issue_date,'Mon')
ORDER BY EXTRACT(MONTH from issue_date),TO_CHAR(issue_date,'Mon')
) AS loan_month
```
