# SQL — полный конспект к собеседованию — часть 3

[← Оглавление](sql.md) · [← К разделу](../databases-sql.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/sql.md)

Темы 21-26.

## 21. Подзапросы

Подзапрос в WHERE  

Найти пользователей, у которых есть заказы.  

SELECT *  
FROM users  
WHERE id IN (  
    SELECT user_id  
    FROM orders  
);  
Подзапрос в FROM  
SELECT user_id, total_sum  
FROM (  
    SELECT user_id, SUM(total) AS total_sum  
    FROM orders  
    GROUP BY user_id  
) AS user_orders  
WHERE total_sum > 1000;  
Подзапрос в SELECT  
SELECT u.id,  
       u.name,  
       (  
           SELECT COUNT(*)  
           FROM orders o  
           WHERE o.user_id = u.id  
       ) AS orders_count  
FROM users u;  

Работает, но на больших данных может быть хуже, чем JOIN + GROUP BY.  

## 22. EXISTS / NOT EXISTS

EXISTS проверяет факт существования строк.  

SELECT *  
FROM users u  
WHERE EXISTS (  
    SELECT 1  
    FROM orders o  
    WHERE o.user_id = u.id  
);  

SELECT 1 здесь значит: нам не нужны конкретные данные, нам важно только наличие строки.  

Пользователи без заказов:  

SELECT *  
FROM users u  
WHERE NOT EXISTS (  
    SELECT 1  
    FROM orders o  
    WHERE o.user_id = u.id  
);  
IN vs EXISTS  
WHERE id IN (SELECT user_id FROM orders)  

и  

WHERE EXISTS (  
    SELECT 1 FROM orders WHERE orders.user_id = users.id  
)  

### часто решают похожую задачу.

Общее правило:  

IN удобно, когда подзапрос возвращает список значений;  
EXISTS удобно, когда проверяем наличие связанной строки;  
с NULL у NOT IN могут быть неприятные сюрпризы.  
Ловушка NOT IN и NULL  
SELECT *  
FROM users  
WHERE id NOT IN (  
    SELECT user_id  
    FROM orders  
);  

Если в orders.user_id есть NULL, результат может быть неожиданным.  

Надёжнее:  

SELECT *  
FROM users u  
WHERE NOT EXISTS (  
    SELECT 1  
    FROM orders o  
    WHERE o.user_id = u.id  
);  

## 23. CTE — WITH

CTE делает запрос читаемее.  

WITH user_orders AS (  
    SELECT user_id, SUM(total) AS total_sum  
    FROM orders  
    GROUP BY user_id  
)  
SELECT *  
FROM user_orders  
WHERE total_sum > 1000;  

### Хорошо использовать, когда:

сложная логика;  
несколько промежуточных шагов;  
нужно повысить читаемость;  
один результат используется дальше.  
Несколько CTE  
WITH paid_orders AS (  
    SELECT *  
    FROM orders  
    WHERE status = 'paid'  
),  
user_totals AS (  
    SELECT user_id, SUM(total) AS total_sum  
    FROM paid_orders  
    GROUP BY user_id  
)  
SELECT *  
FROM user_totals  
WHERE total_sum > 5000;  

## 24. Рекурсивный CTE

Используется для деревьев, иерархий, категорий, оргструктур.  

WITH RECURSIVE category_tree AS (  
    SELECT id, name, parent_id  
    FROM categories  
    WHERE id = 1  

    UNION ALL  

    SELECT c.id, c.name, c.parent_id  
    FROM categories c  
    JOIN category_tree ct ON c.parent_id = ct.id  
)  
SELECT *  
FROM category_tree;  

### На собеседовании достаточно понимать идею:

Рекурсивный CTE позволяет обходить иерархические данные, например дерево категорий или сотрудников.  

## 25. UNION / UNION ALL / INTERSECT / EXCEPT

UNION  

Объединяет результаты и убирает дубликаты.  

SELECT email FROM customers  
UNION  
SELECT email FROM users;  
UNION ALL  

Объединяет результаты, не убирая дубликаты.  

SELECT email FROM customers  
UNION ALL  
SELECT email FROM users;  

UNION ALL быстрее, потому что не делает deduplication.  

INTERSECT  

Возвращает пересечение.  

SELECT email FROM customers  
INTERSECT  
SELECT email FROM users;  
EXCEPT  

Возвращает строки из первого запроса, которых нет во втором.  

SELECT email FROM users  
EXCEPT  
SELECT email FROM blocked_users;  

## 26. Оконные функции

Оконные функции позволяют считать значения по группе строк, но не схлопывать результат как GROUP BY.  

### Главная форма:

FUNCTION() OVER (  
    PARTITION BY ...  
    ORDER BY ...  
)  
ROW_NUMBER  

Нумерует строки.  

SELECT user_id,  
       total,  
       ROW_NUMBER() OVER (  
           PARTITION BY user_id  
           ORDER BY total DESC  
       ) AS rn  
FROM orders;  

Для каждого пользователя заказы будут пронумерованы по убыванию суммы.  

Найти самый дорогой заказ каждого пользователя  
WITH ranked_orders AS (  
    SELECT id,  
           user_id,  
           total,  
           ROW_NUMBER() OVER (  
               PARTITION BY user_id  
               ORDER BY total DESC  
           ) AS rn  
    FROM orders  
)  
SELECT *  
FROM ranked_orders  
WHERE rn = 1;  
RANK и DENSE_RANK  
SELECT user_id,  
       total,  
       RANK() OVER (ORDER BY total DESC) AS rank,  
       DENSE_RANK() OVER (ORDER BY total DESC) AS dense_rank  
FROM orders;  

Разница:  

total | RANK | DENSE_RANK  
------+------|-----------  
1000  | 1    | 1  
1000  | 1    | 1  
900   | 3    | 2  
800   | 4    | 3  

RANK оставляет пропуски.  

DENSE_RANK не оставляет.  

LAG / LEAD  

Получить предыдущее или следующее значение.  

SELECT id,  
       user_id,  
       total,  
       LAG(total) OVER (  
           PARTITION BY user_id  
           ORDER BY created_at  
       ) AS previous_total  
FROM orders;  

LEAD — следующее значение:  

SELECT id,  
       user_id,  
       total,  
       LEAD(total) OVER (  
           PARTITION BY user_id  
           ORDER BY created_at  
       ) AS next_total  
FROM orders;  
Сумма накопительным итогом  
SELECT id,  
       user_id,  
       created_at,  
       total,  
       SUM(total) OVER (  
           PARTITION BY user_id  
           ORDER BY created_at  
       ) AS running_total  
FROM orders;  
COUNT OVER  

Можно посчитать количество заказов пользователя, не схлопывая строки.  

SELECT id,  
       user_id,  
       total,  
       COUNT(*) OVER (  
           PARTITION BY user_id  
       ) AS user_orders_count  
FROM orders;  

