# 🍕 Case Study #2 Pizza Runner

## Solution - B. Runner and Customer Experience

### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```
WITH weeks AS
  (SELECT generate_series (DATE '2021-01-01'
                        	,(SELECT MAX(registration_date) 
                          	FROM pizza_runner.runners)
                        	,'1 week') AS week_start_date)
SELECT week_start_date
		,COUNT(*) AS runner_count
FROM pizza_runner.runners AS r
LEFT JOIN weeks AS w
  ON r.registration_date >= w.week_start_date
  AND r.registration_date < (w.week_start_date + '1 week')
GROUP BY week_start_date
ORDER BY week_start_date
```

#### Explanation:

