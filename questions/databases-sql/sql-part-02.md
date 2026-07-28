# SQL — полный конспект к собеседованию — часть 2

[← Оглавление](sql.md) · [← К разделу](../databases-sql.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/sql.md)

Темы 13-20.

## 13. Агрегатные функции

COUNT()  
SUM()  
AVG()  
MIN()  
MAX()  

### Пример:

SELECT COUNT(*)  
FROM users;  

Сумма заказов:  

SELECT SUM(total)  
FROM orders;  

Средний чек:  

SELECT AVG(total)  
FROM orders;  

Минимальный и максимальный заказ:  

SELECT MIN(total), MAX(total)  
FROM orders;  
COUNT(*), COUNT(column), COUNT(DISTINCT column)  
SELECT COUNT(*)  
FROM users;  

Считает все строки.  

SELECT COUNT(email)  
FROM users;  

Считает строки, где email IS NOT NULL.  

SELECT COUNT(DISTINCT city)  
FROM users;  

Считает количество уникальных городов.  

## 14. GROUP BY

Группировка данных.  

Например, есть таблица:  

orders  

id | user_id | total  
---+---------+------  
1  | 1       | 100  
2  | 1       | 300  
3  | 2       | 500  

Запрос:  

SELECT user_id, SUM(total)  
FROM orders  
GROUP BY user_id;  

Результат:  

user_id | sum  
--------+-----  
1       | 400  
2       | 500  
Частый вопрос  

### Почему нельзя так?

SELECT user_id, id, SUM(total)  
FROM orders  
GROUP BY user_id;  

Потому что для одного user_id может быть много разных id.  

БД не понимает, какой именно id показать.  

Правильно:  

SELECT user_id, SUM(total)  
FROM orders  
GROUP BY user_id;  

Или надо добавить id в группировку:  

SELECT user_id, id, SUM(total)  
FROM orders  
GROUP BY user_id, id;  

Но это уже другая логика.  

## 15. HAVING

WHERE фильтрует строки до группировки.  

HAVING фильтрует группы после группировки.  

### Пример: найти пользователей, у которых больше 3 заказов.

SELECT user_id, COUNT(*) AS orders_count  
FROM orders  
GROUP BY user_id  
HAVING COUNT(*) > 3;  

Неправильно:  

SELECT user_id, COUNT(*)  
FROM orders  
WHERE COUNT(*) > 3  
GROUP BY user_id;  

Так нельзя, потому что WHERE выполняется до агрегации.  

## 16. JOIN

JOIN объединяет данные из разных таблиц.  

Допустим:  

users  

id | name  
---+------  
1  | Alex  
2  | Ivan  
3  | Maria  
orders  

id | user_id | total  
---+---------+------  
1  | 1       | 100  
2  | 1       | 200  
3  | 2       | 500  
INNER JOIN  

Возвращает только совпадающие записи.  

SELECT users.name, orders.total  
FROM users  
INNER JOIN orders ON users.id = orders.user_id;  

Результат:  

Alex | 100  
Alex | 200  
Ivan | 500  

Maria не попадёт, потому что у неё нет заказов.  

LEFT JOIN  

Возвращает все строки из левой таблицы и совпадения из правой.  

SELECT users.name, orders.total  
FROM users  
LEFT JOIN orders ON users.id = orders.user_id;  

Результат:  

Alex  | 100  
Alex  | 200  
Ivan  | 500  
Maria | NULL  
RIGHT JOIN  

Возвращает все строки из правой таблицы и совпадения из левой.  

SELECT users.name, orders.total  
FROM users  
RIGHT JOIN orders ON users.id = orders.user_id;  

На практике чаще используют LEFT JOIN, потому что его проще читать.  

FULL OUTER JOIN  

Возвращает все строки из обеих таблиц.  

SELECT users.name, orders.total  
FROM users  
FULL OUTER JOIN orders ON users.id = orders.user_id;  

Полезно для сверок данных.  

Например:  

есть пользователь без заказа;  
есть заказ с битым user_id.  
CROSS JOIN  

Декартово произведение.  

SELECT *  
FROM colors  
CROSS JOIN sizes;  

Если в colors 3 строки, а в sizes 4 строки, результат будет 12 строк.  

SELF JOIN  

Таблица джойнится сама с собой.  

### Пример: сотрудники и их руководители.

SELECT e.name AS employee,  
       m.name AS manager  
FROM employees e  
LEFT JOIN employees m ON e.manager_id = m.id;  

## 17. Важная ловушка с LEFT JOIN

Допустим, нужно найти всех пользователей и их оплаченные заказы.  

### Плохо:

SELECT u.id, u.name, o.id AS order_id  
FROM users u  
LEFT JOIN orders o ON u.id = o.user_id  
WHERE o.status = 'paid';  

Проблема: WHERE o.status = 'paid' убьёт строки, где заказа нет. В итоге LEFT JOIN фактически превратится в INNER JOIN.  

Правильно:  

SELECT u.id, u.name, o.id AS order_id  
FROM users u  
LEFT JOIN orders o  
    ON u.id = o.user_id  
   AND o.status = 'paid';  

## 18. Найти записи без связи

Например, пользователи без заказов.  

SELECT u.*  
FROM users u  
LEFT JOIN orders o ON u.id = o.user_id  
WHERE o.id IS NULL;  

Альтернатива через NOT EXISTS:  

SELECT u.*  
FROM users u  
WHERE NOT EXISTS (  
    SELECT 1  
    FROM orders o  
    WHERE o.user_id = u.id  
);  

### На собеседовании можно сказать:

Для поиска отсутствующих связей часто использую LEFT JOIN ... IS NULL или NOT EXISTS. На больших данных NOT EXISTS часто читается лучше и может быть эффективнее в зависимости от плана запроса.  

## 19. Алиасы

SELECT u.id, u.name  
FROM users AS u;  

Можно без AS:  

SELECT u.id, u.name  
FROM users u;  

Алиасы особенно важны при JOIN.  

## 20. CASE WHEN

Условная логика в SQL.  

SELECT id,  
       total,  
       CASE  
           WHEN total >= 10000 THEN 'big'  
           WHEN total >= 1000 THEN 'medium'  
           ELSE 'small'  
       END AS order_size  
FROM orders;  

### Пример для тестирования:

SELECT id,  
       status,  
       CASE  
           WHEN status = 'paid' THEN true  
           ELSE false  
       END AS is_paid  
FROM orders;  

