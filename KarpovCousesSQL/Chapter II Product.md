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

### 8.

```sql
```
