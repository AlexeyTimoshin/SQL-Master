### **Максимум и минимум строк при разных JOIN**
    
Минимум и максимум будут зависеть от данных(совпадающие значения).  
  
  |**t1-N, t2 - M**|**MIN**|**MAX**|
  |---|---|---|
  |**INNER JOIN**|0|N*M|
  |**LEFT JOIN**|N|N * M|
  |**RIGHT JOIN**|M|N * M|
  |**FULL JOIN**|MAX(M, N)|N * M|
  |**CROSS JOIN**|N * M|N * M|

### FULL OUTER JOIN
<details> <summary> Код таблицы </summary>

```sql
CREATE Table a (id int, val varchar);
INSERT INTO a (id, val) VALUES 
(1, 'A'),
(2, 'B'),
(3, 'C'), 
(4, 'D'),
(5, 'E');
CREATE Table B (id int, val varchar);
INSERT INTO b (id, val) VALUES 
(1, 'Aa'),
(2, 'Bb'),
(3, 'Cc'), 
(4, 'Dd'),
(5, 'Ee');



CREATE Table a (id int, val varchar);
INSERT INTO a (id, val) VALUES 
(1, 'A'),
(1, 'B'),
(1, 'C');
CREATE Table B (id int, val varchar);
INSERT INTO b (id, val) VALUES 
(1, 'Aa'),
(1, 'Bb'),
(1, 'Cc');

SELECT *
FROM a 
FULL OUTER JOIN b ON a.id = b.i
```
id|val|id|val|
---|---|---|---|
1| 'A'| 1 | 'Aa' |
1| 'A'| 1 | 'Bb' |
1| 'A'| 1 | 'Cc' |
1| 'B'| 1 | 'Aa' |
1| 'B'| 1 | 'Bb' |
1| 'B'| 1 | 'Cc' |
1| 'C'| 1 | 'Aa' |
1| 'C'| 1 | 'Bb' |
1| 'C'| 1 | 'Cc' |

</details>

```sql
-- Это работает
SELECT *
FROM a 
FULL OUTER JOIN b ON a.id = b.id
-- Каждому значению сопоставится другое значение 

-- А это нет
SELECT *
FROM a 
FULL OUTER JOIN b ON a.id <> b.id
--Error FULL JOIN is only supported with merge-joinable or hash-joinable join conditions

Сделаем Truncate table b и положим другие данные  

Truncate table b;
INSERT INTO b (id, val) VALUES 
(6, 'Aa'),
(7, 'Bb'),
(8, 'Cc'), 
(9, 'Dd'),
(10, 'Ee');

SELECT *
FROM a 
FULL OUTER JOIN b ON a.id = b.id
-- Совпадающих айди нет - получим по итогу A+B строк которые заполнены NULL 

Truncate table b;
INSERT INTO b (id, val) VALUES 
(1, 'Aa'),
(1, 'Bb'),
(2, 'Cc'), 
(3, NULL),
(10, 'Ee');

-- Теперь id = 1 из таблицы А будет соответствовать 2 значения из B

Транкейтим А, добавляем другие данные
Truncate table a;
INSERT INTO a (id, val) VALUES
(1, 'A'),
(1, 'B'),
(1, 'C'), 
(4, 'D'),
(NULL, 'E');

SELECT *
FROM a 
FULL OUTER JOIN b ON a.id = b.id
-- И получаем Cross JOIN по одинаковым id
-- При условии одинаковых id в обоих таблицах мы получаем M*N строк
```

### INNER JOIN

```sql
CREATE Table a (id int, val varchar);
CREATE Table B (id int, val varchar);
INSERT INTO a (id, val) VALUES 
(1, 'A'),
(1, 'B'),
(1, 'C'), 
(1, 'D'),
(1, 'E');
INSERT INTO b (id, val) VALUES 
(1, 'Aa'),
(1, 'Bb'),
(1, 'Cc'), 
(1, 'Dd'),
(1, 'Ee');

SELECT *
FROM a 
JOIN b on a.id = b.id
-- Итого получаем Cross join

SELECT *
FROM a 
JOIN b on a.id != b.id
-- А тут будет 0
```

### LEFT OUTER JOIN

```sql
Используем предыдующие таблицы
SELECT *
FROM a 
LEFT JOIN b on a.id = b.id
-- Получим Cross join
```




