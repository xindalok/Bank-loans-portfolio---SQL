**1. Original Dataset Overview**


![](images/info.png)

**2. Data Cleaning Process**

```sql
SELECT 
	issue_date
FROM financial_loan fl 
WHERE issue_date !~ '^\d{2}-\d{2}-\d{4}$';


```

