### 1. Для каждого дня, представленного в таблицах user_actions и courier_actions, рассчитайте следующие показатели:
Число новых пользователей.  
Число новых курьеров.  
Общее число пользователей на текущий день.  
Общее число курьеров на текущий день.  

```sql
SELECT  dcc.min_date as date,
        count_users as new_users,
        count_courier as new_couriers, 
        SUM(count_users)  OVER(ORDER BY dcc.min_date)::int as total_users,
        SUM(count_courier) OVER(ORDER BY dcc.min_date)::int as total_couriers  
FROM
(SELECT min_date, COUNT(courier_id) count_courier
FROM
    (SELECT courier_id, 
            min(time::date) min_date 
    FROM courier_actions
    GROUP BY courier_id)  courier_min_date
GROUP BY min_date)  dcc
JOIN
(SELECT min_date, COUNT(user_id) count_users
FROM 
    (
    SELECT  user_id,
            min(time::date) as min_date
    FROM user_actions
    GROUP BY user_id
    ) min_date_user
GROUP BY min_date) dcu
ON dcc.min_date = dcu.min_date
```

### 2. Дополните запрос из предыдущего задания и теперь для каждого дня, представленного в таблицах user_actions и courier_actions, дополнительно рассчитайте следующие показатели:
Прирост числа новых пользователей.  
Прирост числа новых курьеров.  
Прирост общего числа пользователей.  
Прирост общего числа курьеров.  

```sql

SELECT  date, new_users, new_couriers, total_users, total_couriers,

        ROUND((new_users - lag(new_users, 1) OVER(ORDER BY date))::decimal / 
        lag(new_users, 1) OVER(ORDER BY date) * 100 , 2) as new_users_change,
        
        ROUND((new_couriers - lag(new_couriers, 1) OVER(ORDER BY date))::decimal / 
        lag(new_couriers, 1) OVER(ORDER BY date) * 100, 2) new_couriers_change,
        
        ROUND((total_users - lag(total_users, 1) OVER(ORDER BY date))::decimal /
        lag(total_users, 1) OVER(ORDER BY date) * 100, 2) total_users_growth,
        
        ROUND((total_couriers - lag(total_couriers, 1) OVER(ORDER BY date))::decimal /
        lag(total_couriers, 1) OVER(ORDER BY date) * 100, 2) total_couriers_growth
FROM
(SELECT dcc.min_date as date,
        count_users as new_users,
        count_courier as new_couriers, 
        SUM(count_users)  OVER(ORDER BY dcc.min_date)::int as total_users,
        SUM(count_courier) OVER(ORDER BY dcc.min_date)::int as total_couriers  
FROM
(SELECT min_date, COUNT(courier_id) count_courier
FROM
    (SELECT courier_id, 
            min(time::date) min_date 
    FROM courier_actions
    GROUP BY courier_id)  courier_min_date
GROUP BY min_date)  dcc
JOIN
(SELECT min_date, COUNT(user_id) count_users
FROM 
    (
    SELECT  user_id,
            min(time::date) as min_date
    FROM user_actions
    GROUP BY user_id
    ) min_date_user
GROUP BY min_date) dcu
ON dcc.min_date = dcu.min_date ) fin_tab
```

### 3. Для каждого дня, представленного в таблицах user_actions и courier_actions, рассчитайте следующие показатели:
Число платящих пользователей.  
Число активных курьеров.  
Долю платящих пользователей в общем числе пользователей на текущий день.  
Долю активных курьеров в общем числе курьеров на текущий день.  

```sql
SELECT  count_.date, 
        paying_users, 
        active_couriers, 
        ROUND(paying_users::decimal / SUM(count_users) OVER(ORDER BY count_.date) * 100, 2)  paying_users_share,
        ROUND(active_couriers::decimal / SUM(count_courier) OVER(ORDER BY count_.date) * 100, 2) active_couriers_share
FROM
(  SELECT dcc.min_date as date, count_courier, count_users
   FROM
   (
   SELECT min_date, 
          COUNT(courier_id) count_courier
    FROM
        (
        SELECT courier_id, 
               min(time::date) min_date 
        FROM courier_actions
        GROUP BY courier_id
        )  courier_min_date
    GROUP BY min_date
    )  dcc
JOIN
   (
   SELECT min_date, 
          COUNT(user_id) count_users
    FROM 
        (
        SELECT  user_id,
                min(time::date) as min_date
        FROM user_actions
        GROUP BY user_id
        ) min_date_user
    GROUP BY min_date
    ) dcu
ON dcc.min_date = dcu.min_date 
) count_
JOIN
(   SELECT a_c.date, active_couriers, paying_users
    FROM
       (
       SELECT  time::date as date,
               COUNT(DISTINCT courier_id) active_couriers
       FROM courier_actions 
       WHERE order_id IN (
                       SELECT order_id 
                       FROM courier_actions 
                       WHERE  action = 'deliver_order'
                          )
       GROUP BY time::date
      ) a_c
    JOIN
       (
       SELECT  time::date as date,
               COUNT(DISTINCT user_id) paying_users
       FROM user_actions
       WHERE order_id NOT IN ( 
                        SELECT order_id 
                        FROM user_actions 
                        WHERE action = 'cancel_order'
                            ) 
       GROUP BY time::date
       ) p_u 
    ON a_c.date = p_u.date 
) activCour_payUsers
ON count_.date = activCour_payUsers.date
```

### 4. Для каждого дня, представленного в таблице user_actions, рассчитайте следующие показатели:
Долю пользователей, сделавших в этот день всего один заказ, в общем количестве платящих пользователей.  
Долю пользователей, сделавших в этот день несколько заказов, в общем количестве платящих пользователей.  

```sql
SELECT date,
       round(count(count_orders) filter(WHERE count_orders = 1)::decimal /
        total_day_orders * 100, 2) single_order_users_share,

       round(count(count_orders) filter(WHERE count_orders > 1)::decimal /
        total_day_orders * 100, 2) several_orders_users_share

FROM   (SELECT time::date as date,
               count(user_id) count_orders,
               count(user_id) OVER(PARTITION BY time::date) total_day_orders
        FROM   user_actions
        WHERE  order_id not in (SELECT order_id
                                FROM   user_actions
                                WHERE  action = 'cancel_order')
        GROUP BY time::date, user_id) tab
GROUP BY date, total_day_orders
ORDER BY date
```

### 5. Для каждого дня, представленного в таблице user_actions, рассчитайте следующие показатели:
Общее число заказов.  
Число первых заказов (заказов, сделанных пользователями впервые).  
Число заказов новых пользователей (заказов, сделанных пользователями в тот же день, когда они впервые воспользовались сервисом).  
Долю первых заказов в общем числе заказов (долю п.2 в п.1).  
Долю заказов новых пользователей в общем числе заказов (долю п.3 в п.1).  

```sql
With not_cancel_ord AS (
SELECT  time,
        user_id,
        order_id
FROM user_actions
WHERE order_id NOT IN (
                SELECT order_id
                FROM user_actions
                WHERE action = 'cancel_order'
                      )
)


SELECT  total_orders.date,
        orders,
        first_orders,
        new_users_orders,
        ROUND(first_orders::decimal / orders * 100, 2) as first_orders_share,
        ROUND(new_users_orders::decimal / orders * 100, 2)  as new_users_orders_share
FROM
(
SELECT time::date as date, COUNT(order_id) orders
FROM not_cancel_ord
GROUP BY time::date
) total_orders -- общее число заказов
JOIN
(
SELECT  date, 
        COUNT(user_id) first_orders
FROM
(SELECT MIN(time::date) as date,
        user_id
FROM not_cancel_ord
GROUP BY user_id) min_time_for_orders
GROUP BY date 
) t_first_orders_count  USING(date)
JOIN
(
SELECT  date, 
        COUNT(order_id) new_users_orders
FROM
(
SELECT  time::date date, 
        user_id,
        order_id,
        MIN(time::date) OVER(partition by user_id) min_time
FROM user_actions -- т.к. нам нужны все даты первых отменённых заказов, которые мы не должны учитывать
GROUP BY time::date, user_id, order_id
) prep_tab
WHERE date = min_time and order_id NOT IN (
                SELECT order_id
                FROM user_actions
                WHERE action = 'cancel_order'
                      )
GROUP BY date) new_users USING(date)

---------------------------------------------------------------------
-- предложенный вариант решения
SELECT date,
       orders,
       first_orders,
       new_users_orders::int,
       round(100 * first_orders::decimal / orders, 2) as first_orders_share,
       round(100 * new_users_orders::decimal / orders, 2) as new_users_orders_share
FROM   (SELECT creation_time::date as date,
               count(distinct order_id) as orders
        FROM   orders
        WHERE  order_id not in (SELECT order_id
                                FROM   user_actions
                                WHERE  action = 'cancel_order')
           and order_id in (SELECT order_id
                         FROM   courier_actions
                         WHERE  action = 'deliver_order')
        GROUP BY date) t5
    LEFT JOIN (SELECT first_order_date as date,
                      count(user_id) as first_orders
               FROM   (SELECT user_id,
                              min(time::date) as first_order_date
                       FROM   user_actions
                       WHERE  order_id not in (SELECT order_id
                                               FROM   user_actions
                                               WHERE  action = 'cancel_order')
                       GROUP BY user_id) t4
               GROUP BY first_order_date) t7 using (date)
    LEFT JOIN (SELECT start_date as date,
                      sum(orders) as new_users_orders
               FROM   (SELECT t1.user_id,
                              t1.start_date,
                              coalesce(t2.orders, 0) as orders
                       FROM   (SELECT user_id,
                                      min(time::date) as start_date
                               FROM   user_actions
                               GROUP BY user_id) t1
                           LEFT JOIN (SELECT user_id,
                                             time::date as date,
                                             count(distinct order_id) as orders
                                      FROM   user_actions
                                      WHERE  order_id not in (SELECT order_id
                                                              FROM   user_actions
                                                              WHERE  action = 'cancel_order')
                                      GROUP BY user_id, date) t2
                               ON t1.user_id = t2.user_id and
                                  t1.start_date = t2.date) t3
               GROUP BY start_date) t6 using (date)
ORDER BY date
```

### 6. На основе данных в таблицах user_actions, courier_actions и orders для каждого дня рассчитайте следующие показатели:
    Число платящих пользователей на одного активного курьера.  
    Число заказов на одного активного курьера.  

```sql
With not_cancel_order AS (
SELECT  time,
        user_id,
        order_id
FROM user_actions
WHERE order_id NOT IN (
                SELECT order_id
                FROM user_actions
                WHERE action = 'cancel_order'
                      )
)

SELECT  fin_tab.date,
        ROUND(count_users::decimal / count_couriers, 2) users_per_courier,
        ROUND(count_orders::decimal/ count_couriers, 2) orders_per_courier
FROM (
(
SELECT  time::date as date,
        COUNT(DISTINCT user_id) count_users,
        COUNT(order_id) count_orders
FROM not_cancel_order
GROUP BY time::date) count_us_cour
JOIN
(
SELECT  time::date as date,
        count(distinct courier_id) count_couriers
FROM courier_actions
WHERE order_id IN (SELECT order_id FROM not_cancel_order)
GROUP BY time::date
) count_courier USING(date)
) fin_tab
```

### 7. Давайте рассчитаем ещё один полезный показатель, характеризующий качество работы курьеров.
На основе данных в таблице courier_actions для каждого дня рассчитайте, за сколько минут в среднем  
курьеры доставляли свои заказы.  
```sql
```

### 8.

```sql
```
