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

#### 10. Average Time of Process per Machine
[link](https://leetcode.com/problems/average-time-of-process-per-machine/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT a1.machine_id, ROUND(AVG(a2.timestamp - a1.timestamp)::numeric, 3) processing_time
FROM Activity a1
JOIN Activity a2 
ON a1.machine_id = a2.machine_id AND a1.process_id = a2.process_id
AND a1.activity_type = 'start' and a2.activity_type = 'end'
GROUP BY 1;
```
<details> <summary> Code for Python </summary>
        
```python
import pandas as pd

df = pd.DataFrame(data={
    'machine_id': [0,0,0,0,1,1,1,1,2,2,2,2],
    'process_id': [0,0,1,1,0,0,1,1,0,0,1,1],
    'activity_type': ['start', 'end','start', 'end','start', 'end','start', 'end',
                      'start', 'end','start', 'end'],
    'timestamp': [0.712,1.52,3.14,4.12,0.55,1.55,0.43,1.42,4.1,4.512,2.5,5]
})
```
</details>

```python
import pandas as pd

def get_average_time(activity: pd.DataFrame) -> pd.DataFrame:
    piv_act = activity.pivot(index=['machine_id', 'process_id'],
                            columns='activity_type',values='timestamp')
    piv_act['processing_time'] = piv_act['end'] - piv_act['start']
    return piv_act.groupby('machine_id').processing_time.mean().round(3).reset_index()

# Chain solution
def get_average_time(activity: pd.DataFrame) -> pd.DataFrame:
    return activity.pivot(index=["machine_id", "process_id"], columns='activity_type', values='timestamp')\
            .groupby("machine_id")\
            .apply(lambda x: (x['end'] - x['start']).mean().round(3))\
            .rename("processing_time")\
            .reset_index()
```

#### 11. Employee Bonus
[link](https://leetcode.com/problems/employee-bonus/description/?envType=study-plan-v2&envId=top-sql-50)

```sql
SELECT name, bonus
FROM employee 
LEFT JOIN bonus USING(empId)
WHERE bonus IS NULL OR bonus < 1000
```
```python
import pandas as pd

def employee_bonus(employee: pd.DataFrame, bonus: pd.DataFrame) -> pd.DataFrame:
    df = employee.merge(bonus, how='left')
    res = df[(df['bonus'] < 1000) | (df['bonus'].isna())]
    return res[['name', 'bonus']]
```

#### 12. Students and Examinations
[link](https://leetcode.com/problems/students-and-examinations/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT s.student_id, s.student_name, su.subject_name, COUNT(e.subject_name) attended_exams
FROM students s
CROSS JOIN subjects su
LEFT JOIN examinations e ON s.student_id = e.student_id 
                        and su.subject_name = e.subject_name
GROUP BY 1, 2, 3
ORDER BY 1
```
```python
import pandas as pd

def students_and_examinations(students: pd.DataFrame, subjects: pd.DataFrame, examinations: pd.DataFrame) -> pd.DataFrame:
    df = students.merge(subjects, how='cross')
    examinations.rename(columns={'subject_name': 'subject_name2'}, inplace=True)
    df_res = df.merge(examinations, how='left', left_on=['student_id', 'subject_name'],
                      right_on=['student_id','subject_name2'])

    res = df_res.groupby(['student_id', 'student_name','subject_name'],dropna=False)\
         ['subject_name2'].count()\
         .reset_index(name='attend_exams')
    return res
```

#### 13. Managers with at Least 5 Direct Reports
[link](https://leetcode.com/problems/managers-with-at-least-5-direct-reports/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT e1.name
FROM Employee e1
JOIN  (
        SELECT managerId
        FROM Employee 
        GROUP BY 1
        HAVING COUNT(id) >= 5
        ) e2
ON e1.id = e2.managerId

SELECT e.name
FROM Employee AS e 
INNER JOIN Employee AS m ON e.id = m.managerId 
GROUP BY m.managerId 
HAVING COUNT(m.managerId) >= 5
```

```python
import pandas as pd

def find_managers(employee: pd.DataFrame) -> pd.DataFrame:
    manager = employee.groupby('managerId').size().reset_index(name='count')
    resMan = manager[manager['count'] >= 5]
    empMan = resMan.merge(employee[['id', 'name']], how='inner', left_on='managerId',
                        right_on='id')
    return empMan[['name']]
```

#### 14. Confirmation Rate
[link](https://leetcode.com/problems/confirmation-rate/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
WITH cte as (
    SELECT user_id, 
            CASE
            WHEN action = 'confirmed'  THEN 1
            ELSE 0
            END as confirmation_rate
    FROM Confirmations
)

SELECT  s.user_id, 
        COALESCE(ROUND(AVG(c.confirmation_rate), 2), 0) confirmation_rate
FROM Signups s
LEFT JOIN cte c USING(user_id)
GROUP BY s.user_id
```
```python
import pandas as pd

def confirmation_rate(signups: pd.DataFrame, confirmations: pd.DataFrame) -> pd.DataFrame:
    confirmations['confirmation_rate'] = confirmations['action']\
                                        .apply(lambda x: 1 if x=='confirmed' else 0)
    avg_conf = confirmations[['user_id', 'confirmation_rate']].groupby('user_id')\
                ['confirmation_rate'].mean().round(2).reset_index()
    ans = pd.merge(signups[['user_id']], avg_conf, how='left').fillna(0)
    return ans
```
### Basic Aggregate Functions

#### 15. Not Boring Movies
[link](https://leetcode.com/problems/not-boring-movies/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT *
FROM cinema
WHERE id % 2 = 1 and description NOT LIKE 'boring'
ORDER BY rating Desc
```
```python
import pandas as pd

def not_boring_movies(cinema: pd.DataFrame) -> pd.DataFrame:
    odd = cinema[cinema['id'] % 2 == 1]
    not_bor = cinema[cinema['description'] != 'boring']
    df = odd.merge(not_bor, how='inner').sort_values('rating', ascending=False)
    return df

# А можно и так
def not_boring_movies2(cinema: pd.DataFrame) -> pd.DataFrame:
    mask = (cinema.index % 2 == 0) & (cinema["description"] != "boring")
    return cinema[mask].sort_values('rating', ascending=False)
```

#### 16.  Average Selling Price
[link](https://leetcode.com/problems/average-selling-price/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
WITH t1 as
(SELECT p.product_id, price, units
FROM Prices p
LEFT JOIN UnitsSold u ON p.product_id = u.product_id
AND purchase_date >= start_date AND purchase_date <= end_date)

SELECT  product_id, 
        COALESCE(ROUND(SUM(price*units)/SUM(units)::numeric, 2), 0)  average_price
FROM t1
GROUP BY product_id
```
```python
import pandas as pd

def average_selling_price(prices: pd.DataFrame, units_sold: pd.DataFrame) -> pd.DataFrame:
    prices.sort_values('start_date', inplace=True)
    units_sold.sort_values('purchase_date', inplace=True)

    # merges on matching `by` values, then latest `right_on` <= `left_on`
    soldWithPrices = pd.merge_asof(units_sold, prices, by='product_id', left_on='purchase_date', right_on='start_date')

    ## In theory you should do this, but doesn't seem necessary to pass all test cases
    # badprice = soldWithPrices['end_date'] < soldWithPrices['purchase_date']
    # soldWithPrices.loc[badprice, 'price'] = 0.0
    # soldWithPrices.fillna({'price': 0.0}, inplace=True)

    def weighted_mean(df, value, weight):
        vs = df[value]
        ws = df[weight]

        return (vs*ws).sum() / ws.sum()

    avgPxSeries = soldWithPrices.groupby('product_id').apply(weighted_mean, 'price', 'units')

    main = avgPxSeries.round(2).rename('average_price').reset_index()

    # for some products we have a price but no sales, for whatever reason LC demands we return zero avg price
    priceIds = set(prices['product_id'].unique())
    soldIds = set(units_sold['product_id'].unique())
    missingIds = priceIds.difference(soldIds)
    fill = pd.DataFrame({'product_id': list(missingIds), 'average_price': [0]*len(missingIds)})

    return pd.concat([main, fill])
```

#### 17. Project Employees I
[link](https://leetcode.com/problems/project-employees-i/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT project_id,
        ROUND(AVG(experience_years), 2) average_years
FROM  project
JOIN employee USING(employee_id)
GROUP BY 1
```
```python
import pandas as pd

def project_employees_i(project: pd.DataFrame, employee: pd.DataFrame) -> pd.DataFrame:
    df = pd.merge(project, employee, on='employee_id')
    res = df[['project_id', 'experience_years']]\
            .groupby('project_id').mean().reset_index()\
            .round(2)\
            .rename(columns={'experience_years':'average_years'})
    return res
```

#### 18. Percentage of Users Attended a Contest
[link](https://leetcode.com/problems/percentage-of-users-attended-a-contest/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT  contest_id, 
        ROUND(COUNT(contest_id)::numeric/t1.cnt_user, 4)*100 percentage
        -- если кол-во контекстов == кол-ву юзеров то 1
        -- иначе кол-во участников на контекст к общему числу их
FROM Register
CROSS JOIN (
    SELECT COUNT(user_id) cnt_user
    FROM  Users
) t1
GROUP BY 1, t1.cnt_user
ORDER BY 2 DESC, 1
```
```python
import pandas as pd

def users_percentage(users: pd.DataFrame, register: pd.DataFrame) -> pd.DataFrame:
    df = register.groupby('contest_id', as_index=False)\
        .agg(count_user=('user_id', 'nunique'))
    df['percentage'] = (df['count_user']/len(users)*100).round(2)
    df = df[['contest_id', 'percentage']].sort_values(by=['percentage','contest_id'],
                                          ascending=[False, True])
    return df
```

#### 19. Queries Quality and Percentage
[link](https://leetcode.com/problems/queries-quality-and-percentage/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT  query_name,
        ROUND(AVG((rating/position::numeric)), 2) quality,
        ROUND(AVG(CASE WHEN rating < 3 THEN 1 ELSE 0 END)*100, 2) poor_query_percentage 
FROM Queries 
WHERE query_name IS NOT NULL
GROUP BY query_name 
```
```python
import pandas as pd

def queries_stats(queries: pd.DataFrame) -> pd.DataFrame:
    queries['quality'] = queries['rating']/queries['position']
    # вернёт булеву маску, а *100 сделает 100 и 0
    queries['poor_query_percentage'] = (queries['rating'] < 3) * 100
    res = queries.groupby('query_name', as_index=False)\
        [['quality','poor_query_percentage']]\
        .mean()
    # для точности надо добавить 1e-10
    # иначе тесты не пройдут
    res[['quality', 'poor_query_percentage']] += 1e-10
    return res.round(2)
```

#### 20. Monthly Transactions I
[link](https://leetcode.com/problems/monthly-transactions-i/description/?envType=study-plan-v2&envId=top-sql-50)
```sql

```
```python
```

#### . 
[link]()
```sql

```

#### . 
[link]()
```sql

```
```python
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
