### SELECT

#### 1. Recyclable and Low Fat Products
[link](https://leetcode.com/problems/recyclable-and-low-fat-products/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT product_id
FROM Products
WHERE low_fats = 'Y' AND recyclable = 'Y'
```

#### 2. Find Customer Referee
[link](https://leetcode.com/problems/find-customer-referee/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT name
FROM Customer
WHERE referee_id IS NULL OR referee_id != 2 
```

#### 3. Big Countries
[link](https://leetcode.com/problems/big-countries/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT name, population, area 
FROM World 
WHERE area >= 3000000 OR population >= 25000000
```

#### 4. Article Views I
[link](https://leetcode.com/problems/article-views-i/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT DISTINCT(author_id) id
FROM Views  
WHERE author_id = viewer_id
ORDER BY 1
```

#### 5. Invalid Tweets
[link](https://leetcode.com/problems/invalid-tweets/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT tweet_id
FROM Tweets
WHERE char_length(content) > 15
```

### BASIC JOIN

#### 6. Replace Employee ID With The Unique Identifier
[link](https://leetcode.com/problems/replace-employee-id-with-the-unique-identifier/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT eu.unique_id, e.name
FROM EmployeeUNI eu
RIGHT JOIN Employees e USING(id)
```

#### 7. Product sales analysis
[link](https://leetcode.com/problems/product-sales-analysis-i/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT p.product_name, s.year, s.price
FROM Sales s 
LEFT JOIN Product p ON p.product_id = s.product_id
```

#### 8.Customer Who Visited but Did Not Make Any Transactions
[link](https://leetcode.com/problems/customer-who-visited-but-did-not-make-any-transactions/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT  v.customer_id, 
        COUNT(customer_id) count_no_trans 
FROM Visits v
LEFT JOIN Transactions t ON v.visit_id = t.visit_id
WHERE transaction_id IS NULL
GROUP BY v.customer_id;
```
```python
import pandas as pd

def find_customers(visits: pd.DataFrame, transactions: pd.DataFrame) -> pd.DataFrame:
    df = pd.merge(visits, transactions, how='left', on='visit_id')
    df = df[df['transaction_id'].isna()] \
        .groupby(['customer_id'], as_index=False) \
        .agg(count_no_trans=('visit_id', 'nunique'))
    return df
```

#### 9. Rising Temperature
[link](https://leetcode.com/problems/rising-temperature/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT w1.id
FROM Weather w
RIGHT JOIN Weather w1 ON w1.recordDate = w.recordDate + 1
WHERE w1.temperature > w.temperature
```
```python
import pandas as pd

def rising_temperature(weather: pd.DataFrame) -> pd.DataFrame:
    # Need sort
    df = weather.sort_values('recordDate')

    # Calc diff in temp btw prev day and check if its > 0
    temp_increase = df['temperature'].diff() > 0

    # Calc diff in days and check if it == 1
    days = df['recordDate'].diff().dt.days == 1

    # Combine 
    res = df[temp_increase & days][['id']]

    return res
```

#### 10. 
[link]()
```sql

```

#### 11. 
[link]()
```sql

```

#### 12. 
[link]()
```sql

```

#### 13. 
[link]()
```sql

```

#### 14. 
[link]()
```sql

```

#### 15. 
[link]()
```sql

```

#### 16. 
[link]()
```sql

```

#### 17. 
[link]()
```sql

```

#### 18. 
[link]()
```sql

```

#### 19. 
[link]()
```sql

```

#### 20. 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```
#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```
#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```
#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```
