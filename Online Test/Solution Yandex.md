### 1. За что отвечает ORDER BY?

Сортировка данных

### 2. Какой оператор в стандартном SQL позволяет построить гистрограмму?

Нет такого оператора в стандартном SQL

### 3. В каком порядке выполняются SELECT, FROM и GROUP BY

FROM - GROUP BY - SELECT

### 4. В чём разница между SQL и MYSQL

SQL - язык программирования (WTF, Yandex?) MYSQL - СУБД

### 5. Чем можно заменить RIGHT OUTER JOIN?

1. RIGTH JOIN
2. LEFT OUTER JOIN и поменять местами таблицы

### 6. Какой результат покажет выполнение операторов SELECT COUNT(*)?

Число строк таблицы указанной во FROM + количество NULL  

### 7. Отличие UNION от UNION ALL

UNION удалит дубликаты, UNION ALL оставит их

### 8. Какие данные отберёт запрос при фильтрации WHERE total <> NULL

Получится пустая таблица (почему?)

### 9. Для чего нужен оператор WITH

Позволяет дать блоку подзапроса псевдоним, на который можно ссылаться в других местах запроса

### 10. Что делает оператор DISTINCT(revenue)

Отбирает только уникальные значения из поля revenue

### 11. Дана таблица, что вернут следующие запросы?

ID|LAST_NAME|DEPARTMENTS|ORDERS|
---|---|---|---|
1|Thomson|Electronics| 12|
2|Garcia|Food|16|
3|Moore|Food|16|
4|Moore|Electronics|16|
5|Hall|Food|16|
6|Moore|Electronics|16|
7|Garcia|Electronics|12|
8|Thomson|Food|16|
9|Hall|Electronics|16|
10|Garcia|Electronics|16|

<details> 
<summary> КОД ДЛЯ БД </summary>

```sql
CREATE TABLE NEW_ORDERS
  (ID INTEGER PRIMARY KEY,
  LAST_NAME VARCHAR(100),
  DEPARTMENTS VARCHAR(100),
  ORDERS INTEGER);

INSERT INTO NEW_ORDERS (ID, LAST_NAME, DEPARTMENTS, ORDERS) VALUES
(1, 'Thomson', 'Electronics', 12),
(2, 'Garcia', 'Food', 16),
(3, 'Moore', 'Food', 16),
(4, 'Moore', 'Electronics', 16),
(5, 'Hall', 'Food', 16),
(6, 'Moore', 'Electronics', 16),
(7, 'Garcia', 'Electronics', 12),
(8, 'Thomson', 'Food', 16),
(9, 'Hall', 'Electronics', 16),
(10, 'Garcia', 'Electronics', 16);
```
</details>

```sql
SELECT LAST_NAME
FROM NEW_ORDERS
GROUP BY ORDERS
-- вернёт ошибку, ибо нет группировки по LAST_NAME
```
### 12. Какое количество строк вернёт запрос?
```sql
SELECT COUNT(DEPARTMENTS)
FROM NEW_ORDERS
GROUP BY ORDERS, LAST_NAME
--- 6 
```
<details> 
<summary> Результат запроса (добавлено поле LAST_NAME для наглядности) </summary>

  COUNT(DEPARTMENTS)|LAST_NAME
  ---|---|
  2| Garcia|
  2| Hall|
  1| Thomson|
  1| Garcia|
  3| Moore|
  1| Thomson|
  
</details>

### 13. Строка с каким ID будет на первом месте в результате следующего запроса?

```sql
SELECT *
FROM NEW_ORDERS
ORDER BY ORDERS DESC, LAST_NAME DESC

-- 8
```

### 14. Даны 2 таблицы

LAST_NAMES
last_name|department
---|---|
Thomson|2|
Garcia|3|
Hall|4|
Moore|4|

DEP
departments|orders
---|---|
2|12
3|17
4|4
5|2

<details> 
<summary> КОД ДЛЯ ТАБЛИЦ </summary>

```sql
CREATE TABLE LAST_NAMES (last_name VARCHAR(100), department INTEGER);
CREATE TABLE DEP (department INTEGER, orders INTEGER);
INSERT INTO LAST_NAMES (last_name, department) VALUES
('Thomson', 2),
('Garcia', 3),
('Hall', 4),
('Moore', 4);
INSERT INTO DEP (department, orders) VALUES
(2, 12),
(3, 17),
(4, 4),
(5, 2);
```  
</details>

Сколько строк вернёт запрос?
```sql
SELECT *
FROM DEP
JOIN LAST_NAMES LN ON dep.department = ln.department

-- 4
```
### 15. Сколько строк вернёт запрос?
```sql
SELECT ln.department,
      COUNT(orders)
FROM dep
JOIN LAST_NAMES LN on dep.department = ln.department
WHERE ln.last_name IN ('Moore', 'Hall')
GROUP BY ln.department
LIMIT 3

-- 1
```
 ln.department | COUNT(orders)
 ---|---|
 4|2

 ### 16. В чём разница между LEFT JOIN и LEFT OUTER JOIN?

 Разницы нет, это 2 варината одного и того же оператора.
