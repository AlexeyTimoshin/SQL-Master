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

### 11 Дана таблица, что вернут следующие запросы?

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
