# SQL — полный конспект к собеседованию — часть 8

[← Оглавление](sql.md) · [← К разделу](../databases-sql.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/sql.md)

Темы 50.

## 50. Типовые задачи на собеседовании

Ниже набор задач, которые часто дают QA Automation / Backend / Data-ish кандидатам.  

Задача 1. Найти дубликаты email  
SELECT email, COUNT(*) AS cnt  
FROM users  
GROUP BY email  
HAVING COUNT(*) > 1;  
Задача 2. Найти пользователей без заказов  
SELECT u.*  
FROM users u  
LEFT JOIN orders o ON u.id = o.user_id  
WHERE o.id IS NULL;  

Или:  

SELECT u.*  
FROM users u  
WHERE NOT EXISTS (  
    SELECT 1  
    FROM orders o  
    WHERE o.user_id = u.id  
);  
Задача 3. Посчитать количество заказов по каждому пользователю  
SELECT u.id,  
       u.name,  
       COUNT(o.id) AS orders_count  
FROM users u  
LEFT JOIN orders o ON u.id = o.user_id  
GROUP BY u.id, u.name;  

### Почему COUNT(o.id), а не COUNT(*)?

Потому что при LEFT JOIN пользователь без заказов всё равно даст одну строку, и COUNT(*) вернёт 1.  

COUNT(o.id) вернёт 0.  

Задача 4. Найти пользователей с количеством заказов больше 3  
SELECT u.id,  
       u.name,  
       COUNT(o.id) AS orders_count  
FROM users u  
JOIN orders o ON u.id = o.user_id  
GROUP BY u.id, u.name  
HAVING COUNT(o.id) > 3;  
Задача 5. Найти последний заказ каждого пользователя  

Вариант через оконную функцию:  

WITH ranked_orders AS (  
    SELECT o.*,  
           ROW_NUMBER() OVER (  
               PARTITION BY user_id  
               ORDER BY created_at DESC  
           ) AS rn  
    FROM orders o  
)  
SELECT *  
FROM ranked_orders  
WHERE rn = 1;  
Задача 6. Найти максимальный заказ каждого пользователя  
SELECT user_id, MAX(total) AS max_total  
FROM orders  
GROUP BY user_id;  

Если нужны все поля заказа:  

WITH ranked_orders AS (  
    SELECT o.*,  
           ROW_NUMBER() OVER (  
               PARTITION BY user_id  
               ORDER BY total DESC  
           ) AS rn  
    FROM orders o  
)  
SELECT *  
FROM ranked_orders  
WHERE rn = 1;  
Задача 7. Найти второй максимальный заказ  

Через DENSE_RANK:  

WITH ranked_orders AS (  
    SELECT o.*,  
           DENSE_RANK() OVER (  
               ORDER BY total DESC  
           ) AS rnk  
    FROM orders o  
)  
SELECT *  
FROM ranked_orders  
WHERE rnk = 2;  

Если нужен второй максимум по каждому пользователю:  

WITH ranked_orders AS (  
    SELECT o.*,  
           DENSE_RANK() OVER (  
               PARTITION BY user_id  
               ORDER BY total DESC  
           ) AS rnk  
    FROM orders o  
)  
SELECT *  
FROM ranked_orders  
WHERE rnk = 2;  
Задача 8. Посчитать выручку по дням  
SELECT DATE_TRUNC('day', created_at) AS day,  
       SUM(total) AS revenue  
FROM orders  
WHERE status = 'paid'  
GROUP BY day  
ORDER BY day;  
Задача 9. Найти пользователей, которые сделали заказ в июле 2026  
SELECT DISTINCT u.*  
FROM users u  
JOIN orders o ON u.id = o.user_id  
WHERE o.created_at >= '2026-07-01'  
  AND o.created_at <  '2026-08-01';  
Задача 10. Найти заказы без существующего пользователя  

Если нет foreign key или данные грязные:  

SELECT o.*  
FROM orders o  
LEFT JOIN users u ON o.user_id = u.id  
WHERE u.id IS NULL;  
Задача 11. Найти пользователей, у которых нет оплаченных заказов  
SELECT u.*  
FROM users u  
WHERE NOT EXISTS (  
    SELECT 1  
    FROM orders o  
    WHERE o.user_id = u.id  
      AND o.status = 'paid'  
);  
Задача 12. Найти топ-3 пользователя по сумме заказов  
SELECT u.id,  
       u.name,  
       SUM(o.total) AS total_sum  
FROM users u  
JOIN orders o ON u.id = o.user_id  
GROUP BY u.id, u.name  
ORDER BY total_sum DESC  
LIMIT 3;  
Задача 13. Найти топ-3 заказа каждого пользователя  
WITH ranked_orders AS (  
    SELECT o.*,  
           ROW_NUMBER() OVER (  
               PARTITION BY user_id  
               ORDER BY total DESC  
           ) AS rn  
    FROM orders o  
)  
SELECT *  
FROM ranked_orders  
WHERE rn <= 3;  
Задача 14. Найти пользователей с одинаковыми email  
SELECT email, COUNT(*)  
FROM users  
GROUP BY email  
HAVING COUNT(*) > 1;  

Получить сами строки:  

SELECT *  
FROM users  
WHERE email IN (  
    SELECT email  
    FROM users  
    GROUP BY email  
    HAVING COUNT(*) > 1  
);  
Задача 15. Удалить дубликаты, оставив самую раннюю запись  

PostgreSQL:  

WITH duplicates AS (  
    SELECT id,  
           ROW_NUMBER() OVER (  
               PARTITION BY email  
               ORDER BY created_at ASC  
           ) AS rn  
    FROM users  
)  
DELETE FROM users  
WHERE id IN (  
    SELECT id  
    FROM duplicates  
    WHERE rn > 1  
);  

Перед удалением лучше сначала сделать SELECT:  

WITH duplicates AS (  
    SELECT id,  
           email,  
           ROW_NUMBER() OVER (  
               PARTITION BY email  
               ORDER BY created_at ASC  
           ) AS rn  
    FROM users  
)  
SELECT *  
FROM duplicates  
WHERE rn > 1;  

