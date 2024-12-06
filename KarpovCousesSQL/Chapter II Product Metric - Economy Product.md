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
Показатели revenue и count_orders из предыдущей задачи считаем накопительно.  
Так же необходимо рассчитать накопительно платящих пользователей и всех пользователей.  
Считаем всех новых пользователей по дням - платящих и нет.  
И расчет по формулам из предыдущей задачи.  

```sql
WITH canc_orders AS (
SELECT order_id
FROM user_actions
WHERE action = 'cancel_order'
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
),
total_orders as (
SELECT  time::date as date,
        COUNT(DISTINCT order_id) total_orders
FROM user_actions
WHERE order_id not IN (SELECT * FROM canc_orders)
GROUP BY time::date
), 
new_pay_users as (
SELECT  date, 
        COUNT(DISTINCT user_id) new_paying_users
FROM
    (SELECT  time::date as date,
             user_id,
             min(time::date) OVER(PARTITION by user_id) as start_time
    FROM user_actions
    WHERE order_id not IN (SELECT * FROM canc_orders)
    GROUP BY time::date, user_id
    ) prep_user
WHERE date = start_time
GROUP BY date
), 
new_total_users as (
SELECT  date,
        COUNT(DISTINCT user_id) new_total_users
FROM
    (SELECT  time::date as date,
             user_id,
             min(time::date) OVER(PARTITION by user_id) as start_time
    FROM user_actions
    GROUP BY time::date, user_id
    ) prep_all_users
WHERE date = start_time
GROUP BY date 
) 

-- Running APRU  - cum revenue / New_total_users
-- Running ARPPU - cum revenue / new_paying_users
-- Running AOV   - cum revenue / total_orders

SELECT  revenue.date,
        ROUND(SUM(revenue) OVER(ORDER BY date)::decimal / SUM(new_total_users) OVER(ORDER BY date), 2) as running_arpu,
        ROUND(SUM(revenue) OVER(ORDER BY date)::decimal/ SUM(new_paying_users) OVER(ORDER BY date), 2) running_arppu,
        ROUND(SUM(revenue) OVER(ORDER BY date)::decimal/ SUM(total_orders) OVER(ORDER BY date), 2) running_aov
FROM revenue
JOIN new_total_users USING(date)
JOIN new_pay_users USING(date)
JOIN total_orders USING(date)
```

### 4. Для каждого дня недели в таблицах orders и user_actions рассчитайте следующие показатели:
  Выручку на пользователя (ARPU).  
  Выручку на платящего пользователя (ARPPU).  
  Выручку на заказ (AOV).  

```sql
WITH canc_orders AS (
SELECT order_id
FROM user_actions
WHERE action = 'cancel_order' 
and time between '2022-08-26' and '2022-09-09'
), 
pay_users as (
SELECT  date_part('isodow', time) weekday_number,
        to_char(time, 'day') as weekday,
        COUNT(DISTINCT user_id) count_un_user
FROM user_actions
WHERE order_id not IN (SELECT * FROM canc_orders)
and time between '2022-08-26' and '2022-09-09'
GROUP BY 1, 2
), 
total_orders as (
SELECT  date_part('isodow', time) weekday_number,
        to_char(time, 'day') as weekday,
        COUNT(DISTINCT order_id) total_orders
FROM user_actions
WHERE order_id not IN (SELECT * FROM canc_orders)
and time between '2022-08-26' and '2022-09-09'
GROUP BY 1, 2
), 
total_users as (
SELECT  date_part('isodow', time) weekday_number,
        to_char(time, 'day') as weekday,
        COUNT(DISTINCT user_id) total_users
FROM user_actions
WHERE time between '2022-08-26' and '2022-09-09'
GROUP BY 1, 2
), 
revenue as (
SELECT weekday_number, weekday, sum(price) as revenue
FROM
    (SELECT  date_part('isodow', creation_time) weekday_number,
             to_char(creation_time, 'Day') as weekday,
             order_id,
             UNNEST(product_ids) as product_id
    FROM orders
    WHERE order_id NOT IN (SELECT * FROM canc_orders)
        and creation_time between '2022-08-26' and '2022-09-09'
    ) product
LEFT JOIN products USING(product_id)
GROUP BY 1, 2
)

-- APRU  - revenue / total_users
-- ARPPU - revenue / paying_users
-- AOV   - revenue / total_orders

SELECT  revenue.weekday_number,
        revenue.weekday,
        ROUND(revenue::decimal / total_users, 2) as arpu,
        ROUND(revenue::decimal / count_un_user, 2) arppu,
        ROUND(revenue::decimal / total_orders, 2) aov
FROM revenue
JOIN total_users USING(weekday_number)
JOIN pay_users USING(weekday_number)
JOIN total_orders USING(weekday_number)
```

### 5. Для каждого дня в таблицах orders и user_actions рассчитайте следующие показатели:
  Выручку, полученную в этот день.  
  Выручку с заказов новых пользователей, полученную в этот день.  
  Долю выручки с заказов новых пользователей в общей выручке, полученной за этот день.  
  Долю выручки с заказов остальных пользователей в общей выручке, полученной за этот день.  

```sql
WITH canc_orders AS (
SELECT order_id
FROM user_actions
WHERE action = 'cancel_order'
), 
revenue as (
SELECT  date, 
        order_id,
        sum(price) as revenue
FROM
    (SELECT  creation_time::date as date,
             order_id,
             UNNEST(product_ids) as product_id
    FROM orders
    WHERE order_id NOT IN (SELECT * FROM canc_orders)
    ) product
LEFT JOIN products USING(product_id)
GROUP BY date, order_id
), new_users as (
SELECT  date,
        user_id,
        order_id
FROM
    (SELECT  time::date as date,
             user_id,
             order_id,
             min(time::date) OVER(PARTITION by user_id) as start_time
    FROM user_actions
    GROUP BY time::date, user_id, order_id
    ) prep_all_users
WHERE order_id NOT IN (SELECT * FROM canc_orders) and
date = start_time
) 

SELECt rev_tab.date, 
        rev_tab.revenue,
        nu.new_users_rev new_users_revenue, -- new_users_revenue_share, 
        ROUND(nu.new_users_rev::decimal / rev_tab.revenue * 100, 2) new_users_revenue_share,
        ROUND((rev_tab.revenue - nu.new_users_rev)::decimal / rev_tab.revenue * 100  , 2) old_users_revenue_share
FROM
(SELECT  date, 
         SUM(revenue) revenue 
FROM revenue 
GROUP BY date) rev_tab
JOIN 
(
SELECT  new_users.date,
        sum(revenue) new_users_rev
FROM   new_users 
JOIN   revenue USING(order_id)
GROUP BY new_users.date
) nu ON rev_tab.date = nu.date
ORDER BY date
```

### 6. Для каждого товара, представленного в таблице products, за весь период времени в таблице orders рассчитайте следующие показатели:
  Суммарную выручку, полученную от продажи этого товара за весь период.  
  Долю выручки от продажи этого товара в общей выручке, полученной за весь период.  

```sql
with canc_orders as
(SELECT order_id
FROM   user_actions
WHERE  action = 'cancel_order')
, revenue as
(SELECT name product_name,
        sum(price) as revenue
FROM   (SELECT creation_time::date as date,
               order_id,
               unnest(product_ids) as product_id
        FROM   orders
        WHERE  order_id not in (SELECT * FROM   canc_orders)) product
LEFT JOIN products using(product_id)
GROUP BY name)

SELECT product_name,
       sum(revenue) revenue,
       sum(share_in_rev) share_in_revenue
FROM   (SELECT case
                when share_in_rev >= 0.5 then product_name
                else 'ДРУГОЕ' end as product_name,
               revenue,
               share_in_rev -- share_in_revenue
        FROM   (SELECT product_name,
                       revenue,
                       round(revenue::decimal / sum(revenue) OVER() * 100, 2) as share_in_rev
                FROM   revenue) tab) final
GROUP BY 1
ORDER BY 2 desc
```

### 7. Для каждого дня в таблицах orders и courier_actions рассчитайте следующие показатели:

Выручку, полученную в этот день.  
Затраты, образовавшиеся в этот день.  
Сумму НДС с продажи товаров в этот день.  
Валовую прибыль в этот день (выручка за вычетом затрат и НДС).  
Суммарную выручку на текущий день.  
Суммарные затраты на текущий день.  
Суммарный НДС на текущий день.  
Суммарную валовую прибыль на текущий день.  
Долю валовой прибыли в выручке за этот день (долю п.4 в п.1).  
Долю суммарной валовой прибыли в суммарной выручке на текущий день (долю п.8 в п.5).  

```sql
WITH canc_orders AS (
SELECT order_id
FROM user_actions
WHERE action = 'cancel_order' 
), prep_ord_rev as (
SELECT date, order_id, product_id, name, price   
FROM
    (SELECT  creation_time::date as date,
             order_id,
             UNNEST(product_ids) as product_id
    FROM orders
    WHERE order_id NOT IN (SELECT * FROM canc_orders)
    ) product
LEFT JOIN products USING(product_id)
), revenue as (
SELECT date, sum(price) as revenue
FROM prep_ord_rev
GROUP BY date
), 
prep_costs as (
SELECT  date,
        
        CASE -- траты на сборку в один день, доставка в другой.
            WHEN DATE_PART('month', date) = 8 THEN 120000 + deliv_ord*150 + accp_ord*140
            WHEN DATE_PART('month', date) = 9 THEN 150000 + deliv_ord*150 + accp_ord*115
            else 0
            END as costs 
          
FROM    
    (
    SELECT  time::date as date,
            COUNT(order_id) FILTER (WHERE action = 'accept_order') accp_ord,
            COUNT(order_id) FILTER (WHERE action = 'deliver_order') deliv_ord
    FROM courier_actions
    WHERE order_id NOT IN (SELECT * FROM canc_orders)
    GROUP BY 1) t
), bonusFor5ord AS (
SELECT  date, 
        CASE 
        WHEN DATE_PART('month', date) = 8 THEN COUNT(cnt_bonus)*400
        WHEN DATE_PART('month', date) = 9 THEN COUNT(cnt_bonus)*500
        END as bonus_costs,
        COUNT(cnt_bonus) total_bonus
FROM
    (SELECT  time::date as date,
            COUNT(courier_id) cnt_bonus
    FROM courier_actions
    WHERE order_id NOT IN (SELECT * FROM canc_orders)
    and action = 'deliver_order'
    GROUP BY 1, courier_id
    HAVING COUNT(courier_id) >= 5) tab
    GROUP BY date
), nds as (
SELECT date, SUM(nds) tax
FROM
(SELECT  date, 
        order_id,
        CASE 
        WHEN name IN ('сахар', 'сухарики', 'сушки', 'семечки', 
                'масло льняное', 'виноград', 'масло оливковое', 
                'арбуз', 'батон', 'йогурт', 'сливки', 'гречка', 
                'овсянка', 'макароны', 'баранина', 'апельсины', 
                'бублики', 'хлеб', 'горох', 'сметана', 'рыба копченая', 
                'мука', 'шпроты', 'сосиски', 'свинина', 'рис', 
                'масло кунжутное', 'сгущенка', 'ананас', 'говядина', 
                'соль', 'рыба вяленая', 'масло подсолнечное', 'яблоки', 
                'груши', 'лепешка', 'молоко', 'курица', 'лаваш', 'вафли', 'мандарины') 
        THEN ROUND(price::decimal / 110 * 10, 2)
        ELSE ROUND(price::decimal / 120 * 20, 2)
        end as nds
FROM prep_ord_rev) nds_
GROUP BY date
)


SELECT date, revenue, costs::decimal, tax, 
       revenue - (costs + tax) as gross_profit,
       SUM(revenue) OVER(ORDER BY date) total_revenue,  -- ,  gross_profit_ratio, total_gross_profit_ratio
       SUM(costs) OVER(ORDER BY date) total_costs, 
       SUM(tax) OVER(ORDER BY date) total_tax,
       SUM(revenue-(costs + tax)) OVER(ORDER BY date) total_gross_profit,
       ROUND((revenue-(costs + tax))::decimal / revenue * 100, 2) gross_profit_ratio,
       ROUND(SUM((revenue-(costs + tax))::decimal) OVER(ORDER BY date) 
       / SUM(revenue) OVER(ORDER BY date) * 100, 2)  total_gross_profit_ratio

FROM
    (
    SELECT date, costs + COALESCE(bonus_costs, 0) as costs, tax, revenue
    FROM prep_costs 
    LEFT JOIN bonusFor5ord USING(date)
    LEFT JOIN nds USING(date)
    LEFT JOIN revenue USING(date)
    ) final_tab
```
