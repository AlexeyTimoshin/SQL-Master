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
users_count AS (
SELECT  time::date as date,
        COUNT(DISTINCT user_id) cnt_unique_user    
FROM user_actions
GROUP BY time::date
)



SELECT  date, 
        ROUND(revenue::decimal / count_orders, 2) as aov,
        ROUND(revenue::decimal / cnt_unique_user, 2) arpu
FROM
(SELECT  date,
        sum(price) as revenue,
        count(distinct order_id) count_orders
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
GROUP BY date) rev_day
JOIN
(SELECT * FROM users_count) t USING(date)

```

### 3.

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
