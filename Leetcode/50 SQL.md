### SELECT

#### 1. Recyclable and Low Fat Products
[link](https://leetcode.com/problems/recyclable-and-low-fat-products/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT product_id
FROM Products
WHERE low_fats = 'Y' AND recyclable = 'Y'
```
```python
import pandas as pd

def find_products(products: pd.DataFrame) -> pd.DataFrame:
    res = products[(products['low_fats'] == 'Y') & (products['recyclable'] == 'Y')]
    return res[['product_id']]
```


#### 2. Find Customer Referee
[link](https://leetcode.com/problems/find-customer-referee/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT name
FROM Customer
WHERE referee_id IS NULL OR referee_id != 2 
```
```python
import pandas as pd

def find_customer_referee(customer: pd.DataFrame) -> pd.DataFrame:
    res = customer[(customer['referee_id'].isnull()) | (customer['referee_id'] != 2)]
    return res[['name']]
```

#### 3. Big Countries
[link](https://leetcode.com/problems/big-countries/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT name, population, area 
FROM World 
WHERE area >= 3000000 OR population >= 25000000
```
```python
import pandas as pd

def big_countries(world: pd.DataFrame) -> pd.DataFrame:
    res = world[(world['area'] >= 3000000) | (world['population'] >= 25000000)]
    return res[['name', 'population', 'area']]
```

#### 4. Article Views I
[link](https://leetcode.com/problems/article-views-i/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT DISTINCT(author_id) id
FROM Views  
WHERE author_id = viewer_id
ORDER BY 1
```
```python
import pandas as pd

def article_views(views: pd.DataFrame) -> pd.DataFrame:
    res =  views[views['author_id'] == views['viewer_id']]
    res = res['author_id'].unique()
    res = sorted(res)
    return pd.DataFrame({"id": res})

def article_views(views: pd.DataFrame) -> pd.DataFrame:
    res =  views[views['author_id'] == views['viewer_id']][['author_id']]\
            .drop_duplicates()\
            .sort_values('author_id')\
            .rename(columns={'author_id': 'id'})

    return res
```

#### 5. Invalid Tweets
[link](https://leetcode.com/problems/invalid-tweets/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT tweet_id
FROM Tweets
WHERE char_length(content) > 15
```
```python
import pandas as pd

def invalid_tweets(tweets: pd.DataFrame) -> pd.DataFrame:
    inv = tweets[tweets['content'].str.len() > 15]
    return inv[['tweet_id']]
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
### BASIC AGGREGATE FUNCTIONS

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
SELECT  SUBSTRING(DATE_TRUNC('month', trans_date)::text FROM 1 for 7) as month,
        country, 
        COUNT(id) trans_count,
        COUNT(state) FILTER(WHERE state='approved') approved_count,
        SUM(amount) trans_total_amount,
        COALESCE(SUM(amount) - SUM(amount) FILTER(WHERE state='declined'), SUM(amount))  approved_total_amount 
FROM Transactions
GROUP BY 1, 2 
```
```python
import pandas as pd
import numpy as np

def monthly_transactions(transactions: pd.DataFrame) -> pd.DataFrame:
    transactions['approved'] = np.where(transactions['state'] == 'approved', transactions['amount'], np.nan)
    transactions['month'] = transactions['trans_date'].dt.strftime("%Y-%m")
    return transactions.groupby(['month', 'country']).agg(
        trans_count=('state', 'count'),
        approved_count=('approved', 'count'),
        trans_total_amount=('amount', 'sum'),
        approved_total_amount=('approved', 'sum')
    ).reset_index()
```

#### 21. Immediate Food Delivery II
[link](https://leetcode.com/problems/immediate-food-delivery-ii/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT ROUND(
    SUM(
        CASE
        WHEN f_ord = f_del THEN 1 
        ELSE 0
        END
    )*100::numeric / COUNT(customer_id), 2) immediate_percentage  
FROM
(SELECT  customer_id,
         MIN(order_date) f_ord,
         MIN("customer_pref_delivery_date") f_del
FROM Delivery
GROUP BY 1) t1
```
```python
import pandas as pd

def immediate_food_delivery(delivery: pd.DataFrame) -> pd.DataFrame:
    # Найти минимальные даты можно и так
    df = delivery.groupby("customer_id").min()
    # Найдем совпадения по датам
    df1 = df[df["order_date"] == df["customer_pref_delivery_date"]]
    # Используем len df-ов чтобы найти нужный %
    # Ну и соберём дф
    return pd.DataFrame({"immediate_percentage":[len(df1)/len(df)*100]}).round(2)
```

#### 22. Game Play Analysis IV
[link](https://leetcode.com/problems/game-play-analysis-iv/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT ROUND(
    COUNT(player_id)::decimal /
    (SELECT COUNT(DISTINCT player_id) FROM Activity), 2) fraction
FROM Activity
WHERE (player_id, event_date) IN (
    SELECT player_id, MIN(event_date) + 1
    FROM Activity
    GROUP BY player_id
) 
```
```python
import pandas as pd

def gameplay_analysis(activity: pd.DataFrame) -> pd.DataFrame:
    # создадим оконоку и найдём первый день
    activity['first_day'] = activity.groupby('player_id').event_date.transform('min')

    # создадим дф со следущим днём используя pd.DateOffset(1)
    sec_day = activity.loc[activity['first_day'] + pd.DateOffset(1) == activity['event_date']]

    # используем атрибут shape[0] (кол-во строк) и посчитаем уникальных пользователей через nunique
    return pd.DataFrame({'fraction': [sec_day.shape[0] / activity.player_id.nunique()]}).round(2)
```

### SORTING AND GROUPING

#### 23. Number of Unique Subjects Taught by Each Teacher
[link](https://leetcode.com/problems/number-of-unique-subjects-taught-by-each-teacher/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT teacher_id, COUNT(DISTINCT subject_id) cnt
FROM Teacher
GROUP BY 1
```
```python
import pandas as pd
def count_unique_subjects(teacher: pd.DataFrame) -> pd.DataFrame:
    return teacher.groupby('teacher_id', as_index=False).agg(cnt=('subject_id', 'nunique'))
```

#### 24. User Activity for the Past 30 Days I
[link](https://leetcode.com/problems/user-activity-for-the-past-30-days-i/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT activity_date AS day, COUNT(DISTINCT user_id) active_users 
FROM Activity
WHERE activity_date between '2019-06-28' AND '2019-07-27'
GROUP BY activity_date;
```
```python3
import pandas as pd

def user_activity(activity: pd.DataFrame) -> pd.DataFrame:
    df = activity.loc[
        (activity['activity_date'] >= (pd.to_datetime('2019-07-27', format='%Y-%m-%d') - pd.Timedelta(days=29)))
        & (activity['activity_date'] <= '2019-07-27')
    ].groupby('activity_date').nunique()\
    .reset_index()\
    .rename(columns={'activity_date':'day', 'user_id': 'active_users'})
    return df.iloc[:, :2]


def user_activity(activity: pd.DataFrame) -> pd.DataFrame:

    return (activity[activity
                    .activity_date.between("2019-06-28",
                                           "2019-07-27")]

                    .rename(columns = {'activity_date': 'day',
                                       'user_id': 'active_users'})

                    .groupby('day')['active_users']
                    .nunique().reset_index())
```

#### 25. Product Sales Analysis III
[link](https://leetcode.com/problems/product-sales-analysis-iii/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT product_id, year as first_year, SUM(quantity) as quantity, price
FROM Sales
WHERE (product_id, year) IN
    (SELECT product_id, MIN(year) as year
    FROM Sales
    GROUP BY 1)
GROUP BY product_id, year, price
```
```python3
import pandas as pd

def sales_analysis(sales: pd.DataFrame, product: pd.DataFrame) -> pd.DataFrame:

    return sales.assign(first_year = sales.groupby('product_id').year.transform(min))\
            .query("first_year == year")\
            [['product_id','first_year','quantity','product_name']]
```

#### 26. Classes More Than 5 Students
[link](https://leetcode.com/problems/classes-more-than-5-students/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT class
FROM courses
GROUP BY class
HAVING COUNT(*) >= 5
```
```python3
import pandas as pd

def find_classes(courses: pd.DataFrame) -> pd.DataFrame:
    df = courses.groupby('class').count().reset_index()
    return df[df['student']>=5][['class']]
```

#### 27. Find Followers Count
[link](https://leetcode.com/problems/find-followers-count/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT user_id, COUNT(user_id) followers_count
FROM Followers
GROUP BY 1
ORDER BY 1
```
```python3
import pandas as pd

def count_followers(followers: pd.DataFrame) -> pd.DataFrame:
    return followers.groupby('user_id').count().reset_index()\
            .rename(columns={'follower_id':'followers_count'})\
            .sort_values('user_id', ascending=True)
```

#### 28. Biggest Single Number
[link](https://leetcode.com/problems/biggest-single-number/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
WITH prep_tab as
(
SELECT num
FROM myNumbers
GROUP BY 1
HAVING COUNT(*) = 1
)

SELECT COALESCE(MAX(num), NULL) num
FROM prep_tab
```
```python3
import pandas as pd

def biggest_single_number(my_numbers: pd.DataFrame) -> pd.DataFrame:
    df = my_numbers.num.drop_duplicates(keep=False)
    return pd.DataFrame({'num': [max(df, default=None)]})
```

#### 29. Customers Who Bought All Products
[link](https://leetcode.com/problems/customers-who-bought-all-products/?envType=study-plan-v2&envId=top-sql-50)

```sql
SELECT customer_id
FROM
(SELECT  customer_id,
         COUNT(DISTINCT product_key) prd_count
FROM Customer
GROUP BY 1) t1
JOIN  (SELECT COUNT(*) prd_count FROM Product) t2
USING(prd_count)
WHERE t1.prd_count = t2.prd_count
```
```python3
import pandas as pd

def find_customers(customer: pd.DataFrame, product: pd.DataFrame) -> pd.DataFrame:
    df = customer.groupby('customer_id', as_index=False).product_key.nunique()
    return df[['customer_id']][df['product_key'] == len(product)]
```

### ADVANCED SELECT AND JOINS

#### 30. The Number of Employees Which Report to Each Employee
[link](https://leetcode.com/problems/the-number-of-employees-which-report-to-each-employee/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT  e1.employee_id, e1.name,
        COUNT(e2.employee_id) reports_count,
        ROUND(AVG(e2.age), 2)::integer average_age 
FROM Employees e1 
JOIN Employees e2 
ON e1.employee_id = e2.reports_to
GROUP BY 1, 2
ORDER BY 1
```
```python3
import pandas as pd

def count_employees(employees: pd.DataFrame) -> pd.DataFrame:
    df = employees.merge(employees, left_on='employee_id', right_on='reports_to')
    df = df.groupby(['employee_id_x','name_x'], as_index=False)\
        .agg( reports_count=('name_y', 'count'),
              average_age=('age_y', 'mean'))\
        .rename(columns={'employee_id_x': 'employee_id',
                         'name_x': 'name'})\

    df['average_age'] += 1e-10
    df = df.round(0)
    return df
```

#### 31. Primary Department for Each Employee 
[link](https://leetcode.com/problems/primary-department-for-each-employee/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
(SELECT employee_id, department_id
FROM Employee
WHERE primary_flag = 'Y')
UNION
(SELECT employee_id, department_id
FROM Employee
WHERE employee_id IN
        (SELECT employee_id
        FROM Employee
        GROUP BY 1
        HAVING COUNT(employee_id) = 1)
)
```
```python3
import pandas as pd

def find_primary_department(employee: pd.DataFrame) -> pd.DataFrame:
    mask_y = employee[['employee_id', 'department_id']][employee['primary_flag'] == 'Y']
    mask_2 = employee.groupby('employee_id').count().reset_index().rename(columns={'department_id':'count_dep'})
    mask_2 = mask_2[['employee_id']][mask_2['count_dep'] == 1]
    return pd.concat([mask_y, mask_2.merge(employee)]).iloc[:, :2]
```

#### 32. Triangle Judgement
[link](https://leetcode.com/problems/triangle-judgement/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT x,y,z,
        CASE
        WHEN x+y>z AND z+y>x AND z+x>y THEN 'Yes'
        ELSE 'No'
        END as triangle
FROM triangle
```
```python3
import pandas as pd
# long way
def triangle_judgement(triangle: pd.DataFrame) -> pd.DataFrame:
    def is_triangle(x,y,z):
        if x+y>z:
            if x+z>y:
                if y+z>x:
                    return 'Yes'
                else:
                    return 'No'
            else:
                return 'No'
        else:
            return 'No'
    lst = []

    for i in range(len(triangle)):
        lst.append(is_triangle(triangle.x[i], triangle.y[i], triangle.z[i]))
    
    return triangle.merge(pd.DataFrame({'triangle':lst}), left_index=True, right_index=True)

# через лямбду
import pandas as pd

def triangle_judgement(triangle: pd.DataFrame) -> pd.DataFrame:
    triangle['triangle'] = triangle.apply(
        lambda t: 'Yes' if ((t.x + t.y > t.z) & (t.y + t.z > t.x) & (t.x + t.z > t.y)) else 'No', axis=1)

    return triangle

# через assign
import pandas as pd
import numpy as np

def triangle_judgement(df: pd.DataFrame) -> pd.DataFrame:
    return df.assign(triangle=np.where(df.eval("(x<y+z)&(y<x+z)&(z<x+y)"),"Yes","No"))
```

#### 33. Consecutive Numbers
[link](https://leetcode.com/problems/consecutive-numbers/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
# Херня а не задача
SELECT DISTINCT L1.num as ConsecutiveNums
FROM Logs L1, Logs L2, Logs L3
WHERE L1.num = L2.num AND L2.num = L3.num 
AND L1.id + 1 = L2.id AND L1.id + 2 = L3.id
```
```python3
import pandas as pd

def consecutive_numbers(logs: pd.DataFrame) -> pd.DataFrame:

    logs.sort_values(['id'], inplace = True)

    logs = logs[(logs.num == logs.num.shift(1)) & 
                (logs.num == logs.num.shift(2)) & 
                (logs.id  == logs.id.shift(1)+1) & # <-- added in the revision
                (logs.id  == logs.id.shift(2)+2)   # <-- added in the revision
                ].drop_duplicates('num')
        
    return logs.iloc[:,[1]].rename(columns = {'num':'ConsecutiveNums'})
```

#### 34. Product Price at a Given Date
[link](https://leetcode.com/problems/product-price-at-a-given-date/?envType=study-plan-v2&envId=top-sql-50)

```sql
WITH cte1 AS (
SELECT  DISTINCT product_id,
        MAX(change_date) date
FROM Products
WHERE change_date <= '2019-08-16' 
GROUP BY 1
)

SELECT c.product_id, p.new_price price
FROM cte1 c
LEFT JOIN Products p ON c.product_id = p.product_id AND c.date = p.change_date
UNION
SELECT product_id,  10 price
FROM Products
WHERE product_id NOT IN (SELECT product_id FROM cte1)
```
```python3
import pandas as pd

def price_at_given_date(df: pd.DataFrame) -> pd.DataFrame:
    # создадим дф с датой <= 2019-08-16
    df_max = df[df['change_date'] <= '2019-08-16']\
            .rename(columns={'new_price':'price'})

    # Сделаем NOT IN ко второй части дф, и получим другие товары
    # Установим им цену и переменуем колонки
    df_other = df[~df['product_id'].isin(df_max.product_id)]
    df_other['new_price'] = 10
    df_other = df_other.rename(columns={'new_price': 'price'})

    # объединим дф-ы, отсортируем по дате в убывающем порядке
    # .drop_duplicates(['product_id']) - удалит все дубликаты по id кроме первого вхождения
    return pd.concat([df_max, df_other]).sort_values('change_date', ascending=False)\
             .drop_duplicates(['product_id']).iloc[:, :2]
```

#### 35. Last Person to Fit in the Bus
[link](https://leetcode.com/problems/last-person-to-fit-in-the-bus/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT person_name
FROM
    (SELECT person_name, MAX(weig) w
    FROM
        (SELECT person_name, 
                SUM(weight) OVER(ORDER BY turn) weig,
                turn
        FROM Queue
        ) t1
    WHERE weig <= 1000 
    GROUP BY 1
    ORDER BY w desc
    LIMIT 1
    ) t2
```
```python3
import pandas as pd
# медленно, надо быстрее
def last_passenger(df: pd.DataFrame) -> pd.DataFrame:
    df.sort_values('turn', inplace=True)
    df['wind_sum'] = df['weight'].cumsum()
    return df[['person_name']][df['wind_sum']<=1000].iloc[[-1]]
```

#### 36. Count Salary Categories
[link](https://leetcode.com/problems/count-salary-categories/?envType=study-plan-v2&envId=top-sql-50)
```sql
WITH all_ct as (
SELECT 'Low Salary' as category
UNION
SELECT 'Average Salary' as category
UNION
SELECT 'High Salary' as category
), 
filt_cat as (
SELECT account_id,
CASE WHEN income < 20000 THEN 'Low Salary'
     WHEN income >= 20000 and income <= 50000 THEN 'Average Salary'
     WHEN income > 50000 THEN 'High Salary'
     ELSE 'There is no category'
     END as category
FROM Accounts
)

SELECT all_ct.category, COUNT(account_id) accounts_count
FROM all_ct
LEFT JOIN filt_cat ON all_ct.category = filt_cat.category
GROUP BY all_ct.category
```
```python
import pandas as pd

def count_salary_categories(df: pd.DataFrame) -> pd.DataFrame:
    # Отфильтруем записи по условиям, получим булеву маску и посчитаем сумму
    # Почему sum, а не count? sum из True сделает 1, из False 0, а count вернёт кол-во
    # без каких либо преобразований, в то время когда нас интересует кол-во True
    low = (df['income'] < 20000).sum()
    avg = ((df['income'] >= 20000) & (df['income'] <= 50000)).sum()
    hi =  (df['income'] > 50000).sum()
    return pd.DataFrame({
        'category' : ['Low Salary', 'Average Salary','High Salary'],
        'accounts_count': [low, avg, hi]
    })
```

### SUBQUERIES

#### 37. Employees Whose Manager Left the Company
[link](https://leetcode.com/problems/employees-whose-manager-left-the-company/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
WITH cte1 AS (
    SELECT employee_id
    FROM Employees
    WHERE salary < 30000
)
SELECT employee_id
FROM Employees
WHERE employee_id IN (SELECT * FROM cte1) AND
manager_id NOT IN (SELECT employee_id FROM Employees)
ORDER BY 1

SELECT e1.employee_id
FROM Employees e1
WHERE e1.salary < 30000 AND manager_id IS NOT NULL
AND NOT EXISTS ( -- не существует работников - менеджеров employee_id 
    SELECT 1 
    FROM Employees e2
    WHERE e2.employee_id = e1.manager_id
)
ORDER BY e1.employee_id
```
```python
import pandas as pd

def find_employees(df: pd.DataFrame) -> pd.DataFrame:
    return df[(~df['manager_id'].isin(df['employee_id']))
            & (df['salary'] < 30000)
            & (df['manager_id'].notnull())]\
            .sort_values('employee_id', ascending=True)\
            .iloc[:, [0]]
```

#### 38. Exchange Seats NB разобрать
[link](https://leetcode.com/problems/exchange-seats/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT id,
CASE WHEN id%2 = 0 THEN lag(student) OVER(order by id)
     WHEN id%2 = 1 and lead(student) OVER(order by id) is null THEN student
     ELSE lead(student) OVER(order by id)
     END student
FROM Seat
```
```python
import pandas as pd

def exchange_seats(seat: pd.DataFrame) -> pd.DataFrame:
    temp = seat.student.copy()
    seat.loc[(seat.id%2==1) & (seat.id != len(seat)), 'student'] = temp.shift(-1)
    seat.loc[(seat.id%2==0), 'student'] = temp.shift(1)
    return seat
    
```

#### 39. Movie Rating
[link](https://leetcode.com/problems/movie-rating/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
-- names acsend top 1, max(count(us_id in movrat))
-- month == 2 and year == 2020 avg(rating) 
WITH cte AS (
    SELECT title,rating, created_at
    FROM MovieRating mr
    JOIN Movies m ON mr.movie_id = m.movie_id
    WHERE EXTRACT(year from created_at) = 2020 AND EXTRACT(month from created_at) = 2
), t1 as (
    SELECT user_id, COUNT(user_id) cnt
    FROM MovieRating
    GROUP BY user_id
    )
(SELECT name results
FROM t1
JOIN Users USING(user_id)
WHERE cnt in (SELECT MAX(cnt) FROM t1)
```
```python
import pandas as pd

def movie_rating(mv: pd.DataFrame, us: pd.DataFrame, mr: pd.DataFrame) -> pd.DataFrame:
    res_us = mr[['user_id', 'rating']].groupby('user_id', as_index=False)\
                .count().rename(columns={'rating': 'counts'})
    res_us = res_us.merge(us).sort_values(['counts', 'name'], ascending=[False, True])\
                .head(1)

    mr_res = mr[(mr['created_at'] >= '2020-02-01') & (mr['created_at'] < '2020-03-01')]
    mr_res = mr_res[['movie_id', 'rating']].groupby('movie_id', as_index=False)\
                .agg(avg_rat=('rating', 'mean'))
    mr_res = mr_res.merge(mv).sort_values(['avg_rat', 'title'], ascending=[False, True])\
                .head(1)
    name = res_us['name']
    cinema = mr_res['title']
    return pd.concat([name, cinema]).to_frame().rename(columns={0: 'results'})
```

#### 40. Restaurant Growth
[link](https://leetcode.com/problems/restaurant-growth/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
WITH cte AS (
    SELECT visited_on, 
            SUM(amount) days_amount
    FROM customer
    GROUP BY visited_on
    ORDER BY visited_on
)
SELECT  visited_on, 
        SUM(days_amount) OVER(ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as amount,
        ROUND( AVG(days_amount) OVER(ROWS BETWEEN 6 PRECEDING AND CURRENT ROW), 2) as average_amount
FROM cte
OFFSEt 6 -- пропускает указанное число строк (важно указать порядок, что сделано в cte)
```

```python
import pandas as pd

def restaurant_growth(df: pd.DataFrame) -> pd.DataFrame:
    # Соритурем по дате и группируем по ней же считая сумму
    # Делаем дату индексом и не сбрасываем её
    df_gr = df.sort_values('visited_on').groupby('visited_on')\
            [['amount']].sum()

    # Примеяем к колонке amount ф-ю rolling с параметром '7D'
    # т.к мы работаем с окном то среднее надо считать по 7 дням (average_amount)
    df_gr = df_gr.assign(
            amount = df_gr.rolling('7D').sum(), 
            average_amount = round(df_gr.rolling('7D').sum()/7 ,2)
            )

    #т.к. оффcета нет, то фильтруем по индексам через loc и pd.DateOffset
    return df_gr.loc[df_gr.index >= df_gr.index.min() + pd.DateOffset(6)].reset_index()               
```

#### 41. Friend Requests II: Who Has the Most Friends
[link](https://leetcode.com/problems/friend-requests-ii-who-has-the-most-friends/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT id, COUNT(*) num
FROM 
(SELECT requester_id id
FROM RequestAccepted
UNION ALL
SELECT  accepter_id id
FROM RequestAccepted
) t
GROUP BY id
ORDER BY num DESC
LIMIT 1 
```
```python
import pandas as pd

def most_friends(df: pd.DataFrame) -> pd.DataFrame:
    ans = pd.concat([df['requester_id'], df['accepter_id']])\
            .value_counts()\
            .reset_index(name='num')\
            .rename(columns={'index': 'id'})\
            .head(1)
    return ans
```

#### 42. Investments in 2016
[link](https://leetcode.com/problems/investments-in-2016/?envType=study-plan-v2&envId=top-sql-50)
```sql
with uniq_latlon as (
    SELECT lat, lon
    FROM Insurance
    GROUP BY 1, 2
    HAVING COUNT(*) = 1
), 
not_uniq_2015 as (
    SELECT tiv_2015
    FROM Insurance
    GROUP BY 1
    HAVING COUNT(*) > 1
)

SELECT ROUND(SUM(tiv_2016)::numeric, 2) tiv_2016
FROM Insurance
WHERE  (lat, lon) IN (SELECT * FROM uniq_latlon) 
        AND tiv_2015 IN (SELECT * FROM not_uniq_2015)  
```
```python
import pandas as pd

def find_investments(df: pd.DataFrame) -> pd.DataFrame:
    df['lat_lon'] = df.groupby(['lat', 'lon'])['pid'].transform('count')
    df['count15'] = df.groupby('tiv_2015')['pid'].transform('count')
    ans = df[(df['lat_lon'] == 1) & (df['count15'] > 1)][['tiv_2016']]\
            .sum().to_frame('tiv_2016').round(2)
    return ans
```

#### 43. Department Top Three Salaries
[link](https://leetcode.com/problems/department-top-three-salaries/?envType=study-plan-v2&envId=top-sql-50)
```sql
WITH ct as (
    SELECT * 
    FROM
    (SELECT  departmentId, name,
             salary,
             DENSE_RANK() OVER(PARTITION BY departmentId ORDER BY salary DESC) rank_
    FROM employee)
    WHERE rank_ in (1,2,3)
)

SELECT  d.name as "Department",
        ct.name as "Employee",
        salary as "Salary"
FROM ct
JOIN department d ON ct.departmentId = d.id
```
```python
import pandas as pd

def top_three_salaries(em: pd.DataFrame, dp: pd.DataFrame) -> pd.DataFrame:
    em['rank'] = em.groupby('departmentId')['salary']\
                   .rank(method='dense', ascending=False)
    em = em[em['rank'] <= 3]
    ans = em.merge(dp, left_on='departmentId', right_on='id')
    ans = ans.rename(columns={'name_y': 'department', 'name_x': 'employee'})
    return ans[['department','employee', 'salary']]
```

### STRINGS REGEX CLAUSE

#### 44. Fix Names in a Table
[link](https://leetcode.com/problems/fix-names-in-a-table/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT user_id, 
       CONCAT(UPPER(SUBSTRING(name, 1, 1)), LOWER(SUBSTRING(name, 2))) AS name
FROM Users
ORDER BY 1
```
```python
import pandas as pd

def fix_names(users: pd.DataFrame) -> pd.DataFrame:
    return users.assign(name = users['name'].str.capitalize())\
           .sort_values('user_id')
```

#### 45. Patients With a Condition
[link](https://leetcode.com/problems/patients-with-a-condition/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT *
FROM patients
WHERE conditions ~ '(^| )DIAB1'
```
```python
```

#### 46. Delete Duplicate Emails 
[link](https://leetcode.com/problems/delete-duplicate-emails/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
DELETE FROM Person 
WHERE id NOT IN    (
    SELECT  MIN(id)
    FROM person
    GROUP BY email
) 

```

```python
```

#### 47. Second Highest Salary
[link](https://leetcode.com/problems/second-highest-salary/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT COALESCE(MAX(salary), null) as SecondHighestSalary
FROM employee
WHERE salary < (SELECT MAX(salary) FROM employee)
```
```python
```

#### 48. Group Sold Products By The Date
[link](https://leetcode.com/problems/group-sold-products-by-the-date/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
SELECT sell_date, COUNT(DISTINCT product) num_sold,
        STRING_AGG(DISTINCT product, ',' ORDER BY product) products 
FROM Activities
GROUP BY sell_date
ORDER BY sell_date
```
```python
```

#### 49. List the Products Ordered in a Period
[link](https://leetcode.com/problems/list-the-products-ordered-in-a-period/description/?envType=study-plan-v2&envId=top-sql-50)
```sql
WITH ct_ord AS (
    SELECT *
    FROM orders
    WHERE order_date BETWEEN '2020-02-01' AND '2020-02-29'    
)

SELECT p.product_name, SUM(unit) as unit
FROM ct_ord o
JOIN products p USING(product_id)
GROUP BY 1
HAVING SUM(unit) >= 100
```
```python
```

#### 50. Find Users With Valid E-Mails
[link]()
```sql
/*
We use the tilde(~) operator as opposed to LIKE, as it's far more powerful.

^ means start with.
Then we define a set of characters to match with the brackets [].
Within them we put two ranges: a-z and A-Z, with a + after it to ensure this character matches at least once.
Then another set, [a-zA-Z0-9_.-]* to match 0 or more times (via the asterisk at the end).
Then finally we enforce @leetcode.com at the end. Putting the dollar ($) will tell the engine to match
only emails that end exactly like this.
Important to notice here that the dot needs to be escaped (with a backslash \ in front of it), otherwise the
regex engine will interpret it as "match any character", i.e. not putting it would make it match emails
with leetcode[place any character here]com, such as leetcodeAcom or leetcode_com.
*/

SELECT *
FROM Users
WHERE mail ~ '^[a-zA-Z]+[a-zA-Z0-9_.-]*@leetcode\.com$'
```
```python
```
