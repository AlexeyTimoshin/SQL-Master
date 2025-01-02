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
# Time: O(n) , Space: O(n) 
import pandas as pd

def students_and_examinations(students: pd.DataFrame, subjects: pd.DataFrame, examinations: pd.DataFrame) -> pd.DataFrame:
    
    df = students.merge(subjects, how='cross')
    examinations.rename(columns={'subject_name':'subject_name2'},inplace=True)
    df_result=df.merge(examinations, how='left', left_on=['student_id','subject_name'], right_on=['student_id','subject_name2'])

    return df_result.groupby(['student_id','student_name','subject_name'], dropna=False)['subject_name2'].count().reset_index(name='attended_exams')
```

#### 13. 
[link]()
```sql

```

```python
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
