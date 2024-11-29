### 1. Для каждого дня в таблице orders рассчитайте следующие показатели:
  Выручку, полученную в этот день.  
  Суммарную выручку на текущий день.  
  Прирост выручки, полученной в этот день, относительно значения выручки за предыдущий день.  

```sql
WITH canc_orders AS (
SELECT order_id
FROM user_actions
WHERE action = 'cancel_order'
)

SELECT date,
        revenue,
        ROUND((revenue - lag(revenue) OVER())::decimal / lag(revenue) OVER() * 100 , 2) revenue_change,
        SUM(revenue) OVER(ORDER BY date) total_revenue
FROM
(
SELECT  date,
        sum(price) as revenue
FROM
(
    (SELECT creation_time::date as date,
           order_id,
           UNNEST(product_ids) product_id
    FROM orders
    WHERE order_id NOT IN (SELECT * FROM canc_orders)
    ) t_products
JOIN products USING(product_id) 
) revenue_tab
GROUP BY date 
) fin_tab
```

### 2.  Остановимся на следующих метриках:
 1. ARPU (Average Revenue Per User) — средняя выручка на одного пользователя за определённый период.  
 2. ARPPU (Average Revenue Per Paying User) — средняя выручка на одного платящего  
    пользователя за определённый период.  
 3. AOV (Average Order Value) — средний чек, или отношение выручки за определённый период к общему  
    количеству заказов за это же время.  

```sql
WITH canc_orders AS (
SELECT order_id
FROM user_actions
WHERE action = 'cancel_order'
), 
pay_users as (
SELECT  time::date as date,
        COUNT(DISTINCT user_id) count_un_user
FROM user_actions
WHERE order_id not IN (SELECT * FROM canc_orders)
GROUP BY time::date
), 
total_orders as (
SELECT  time::date as date,
        COUNT(DISTINCT order_id) total_orders
FROM user_actions
WHERE order_id not IN (SELECT * FROM canc_orders)
GROUP BY time::date
), 
total_users as (
SELECT  time::date as date,
        COUNT(DISTINCT user_id) total_users
FROM user_actions
GROUP BY time::date
), 
revenue as (
SELECT date, sum(price) as revenue
FROM
    (SELECT  creation_time::date as date,
             order_id,
             UNNEST(product_ids) as product_id
    FROM orders
    WHERE order_id NOT IN (SELECT * FROM canc_orders)
    ) product
LEFT JOIN products USING(product_id)
GROUP BY date
)

-- APRU  - revenue / total_users
-- ARPPU - revenue / paying_users
-- AOV   - revenue / total_orders
-- date, arpu, arppu, aov

SELECT  revenue.date,
        ROUND(revenue::decimal / total_users, 2) as arpu,
        ROUND(revenue::decimal / count_un_user, 2) arppu,
        ROUND(revenue::decimal / total_orders, 2) aov
FROM revenue
JOIN total_users USING(date)
JOIN pay_users USING(date)
JOIN total_orders USING(date)
```

### 3. По таблицам orders и user_actions для каждого дня рассчитайте следующие показатели:
  Накопленную выручку на пользователя (Running ARPU).  
  Накопленную выручку на платящего пользователя (Running ARPPU).  
  Накопленную выручку с заказа, или средний чек (Running AOV).  
Пояснение:  
показатели revenue и orders из предыдущей задачи делаешь накопительными,  
для подсчета users можешь опираться на задачу 5 (1 урок)  
показатель first_orders - делаешь накопительным,  
для подсчета paying_users см. задачу 1 (урок 1) на показатель  
new_users - делаешь накопительным.  
И расчет по формулам из предыд. задачи.  
```sql

```

### 4.

```sql
```

### 5.

```sql
```

### 6.

```sql
```

### 7.

```sql
```
