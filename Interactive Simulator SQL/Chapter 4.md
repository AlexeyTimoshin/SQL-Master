## 4.1
```sql
-- 1
SELECT  beg_range, end_range,
        ROUND(AVG(price),2) as "Средняя_цена",
        ROUND(SUM(price*amount),2) as "Стоимость",
        COUNT(*) as "Количество"
FROM book b
JOIN stat s ON price BETWEEN beg_range AND end_range
GROUP BY 1, 2
ORDER BY 5 DESC

-- 2
SELECT * FROM book
ORDER BY length(title) ASC

-- 3
DELETE FROM book
WHERE price LIKE '%.99';

DELETE FROM supply
WHERE price LIKE '%.99';

-- 4
SELECT  author, title, price, amount,
        CASE WHEN price > 600 THEN ROUND(price*0.2, 2)
        ELSE "-"
        END AS sale_20,
        CASE WHEN price > 600 THEN ROUND(price - price*0.2, 2)
        ELSE "-"
        END AS price_sale
FROM book
ORDER BY 1, 2

-- 5
SELECT author, SUM(price*amount) as Стоимость
FROM book
GROUP BY author
HAVING MAX(price) > (SELECT AVG(price) FROM book)
ORDER BY 2 DESC

-- 6
SELECT author Автор,
       title Название_книги,
       amount Количество,
       price Розничная_цена,
       CASE 
       WHEN amount >= 10 THEN 15
       ELSE 0
       END as Скидка,
       CASE 
       WHEN amount >= 10 THEN ROUND(price - price*0.15, 2)
       ELSE price
       END as Оптовая_цена
FROM book
ORDER BY 1, 2

--7
SELECT  author, COUNT(author) Количество_произведений,
        MIN(price) Минимальная_цена,
        SUM(amount) Число_книг
FROM book
GROUP BY author
HAVING COUNT(author) > 1 AND max(price) > 500 and MIN(amount) > 1
ORDER BY 1
```

## 4.2
```sql
-- 1
SELECT CASE
        WHEN author IS NOT NULL THEN "Донцова Дарья"
        END as author,
        CONCAT("Евлампия Романова и ", title) title,
        ROUND(price*1.42, 2) price
FROM book
ORDER BY 3 DESC

-- 2
WITH ct AS (
SELECT name_genre, SUM(bb.amount) sum
FROM genre g
JOIN book b USING(genre_id)
JOIN buy_book bb USING(book_id)
GROUP BY name_genre)

SELECT name_genre, sum as Количество
FROM ct
WHERE sum IN (SELECT MIN(sum) FROM ct)

-- 3


-- 4
SELECT  author, title, 
        CASE
        WHEN price < 500 THEN "низкая"
        WHEN price > 500 AND price < 700 THEN "средняя"
        ELSE "высокая"
        END as price_category, 
        price * amount cost
FROM book
WHERE author NOT LIKE "Есенин%" AND title NOT LIKE "Белая Гвардия"
ORDER BY cost DESC, title

-- 5
SELECT  title, author, amount,
        (SELECT MAX(amount * price) FROM book) - price*amount as Разница_с_макс_стоимостью

FROM book
WHERE amount % 2 = 1
ORDER BY 4 DESC

-- 6
SELECT  title Наименование,
        price Цена,
        CASE
        WHEN amount <= 5 THEN 500
        ELSE "Бесплатно"
        END AS "Стоимость_доставки"
FROM book
WHERE price > 600
ORDER BY 2 DESC

-- 7
SELECT  author, title, amount, price,
        CASE 
        WHEN amount >= 5 THEN "50%"
        WHEN amount < 5 AND price >= 700 THEN "20%"
        ELSE "10%"
        END as "Скидка",
        CASE
        WHEN amount >= 5 THEN ROUND(price*0.5, 2)
        WHEN  amount < 5 AND price >= 700 THEN ROUND(price - price*0.2, 2)
        ELSE ROUND(price - price*0.1, 2)
        END AS "Цена_со_скидкой"
FROM book

-- 8
WITH price_sum AS (
    SELECT title, SUM(price*amount) sum_price
    FROM book
    GROUP BY 1
)

SELECT  author, title, amount, price real_price,
        CASE 
        WHEN p_s.sum_price >= 5000 THEN ROUND(price*1.2, 2)
        ELSE ROUND(price*0.8, 2)
        END as new_price,
        CASE
        WHEN price < 500 THEN 99.99
        WHEN amount < 5 THEN 149.99
        ELSE 0
        END delivery_price
FROM book
JOIN price_sum p_s USING(title)
WHERE author IN ("Булгаков М.А.", "Есенин С.А.") AND amount >= 3 AND amount <= 14
ORDER BY 1, title DESC, delivery_price DESC
```

## 4.3
```sql
-- 1
SELECT  author, title, 
        FLOOR(price) as "Рубли",
        FLOOR((price - FLOOR(price))*100) as "Копейки"
FROM book
ORDER BY 4 DESC

-- 2
SELECT  CONCAT("Графоман и ", author) as Автор, 
        CONCAT(title, ". Краткое содержание.") as Название,
        IF(price*0.4 < 250,  price*0.4, 250) as Цена,
        IF(amount <= 3, "высокий", IF(amount <= 10, "средний", "низкий")) as Спрос,
        IF(amount <=2, "очень мало", IF(amount <= 14, "в наличии", "много")) as Наличие
FROM book
ORDER BY 3, FIELD(Спрос, 'высокий', 'средний', 'низкий'), 2

-- 3
WITH all_tab AS
(
SELECT name_client, bk.book_id, bu.buy_id, bb.amount, bk.price
FROM client c
JOIN buy bu USING(client_id)
JOIN buy_book bb USING(buy_id)
JOIN book bk USING(book_id)
)

SELECT  name_client, 
        SUM(amount*price) "Общая_сумма_заказов",
        COUNT(DISTINCT buy_id) "Заказов_всего",
        SUM(amount) "Книг_всего"
FROM all_tab
GROUP BY 1
HAVING SUM(amount*price) > ( SELECT AVG(sum_) FROM(SELECT SUM(price*amount) sum_ FROM all_tab GROUP BY name_client) tab)
ORDER BY 1

-- 4
SELECT  author, title, price, amount, 
        ROUND((price*amount / SUM(price*amount) OVER()) * 100, 2) income_percent
FROM book
ORDER BY income_percent DESC

-- 5
SELECT  name_author, name_genre,
        COUNT(book_id) Количество
FROM author a
CROSS JOIN genre g
LEFT JOIN book b ON a.author_id = b.author_id and g.genre_id = b.genre_id
GROUP BY 1, 2
ORDER BY 1, 3 DESC, 2

-- 6
SELECT author Автор, title Название_книги, price Цена, 
       IF(price <= 600, "ручка", IF(price <= 700, "детская раскраска", "гороскоп")) Подарок
FROM book
WHERE price >= 500
ORDER BY 1, 2

-- 7
SELECT  author Автор, 
        MIN(amount) Наименьшее_кол_во, 
        MAX(amount) Наибольшее_кол_во 
FROM book
GROUP BY author
HAVING SUM(amount) <= 10

-- 8

```

## 4.4
```sql
```
