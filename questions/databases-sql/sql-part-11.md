# SQL — полный конспект к собеседованию — часть 11

[← Оглавление](sql.md) · [← К разделу](../databases-sql.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/sql.md)

Темы 56-60.

## 56. Частые ошибки новичков

Ошибка 1. Использовать = NULL  

Неправильно:  

WHERE field = NULL  

Правильно:  

WHERE field IS NULL  
Ошибка 2. Портить LEFT JOIN через WHERE  

### Плохо:

SELECT *  
FROM users u  
LEFT JOIN orders o ON u.id = o.user_id  
WHERE o.status = 'paid';  

Лучше:  

SELECT *  
FROM users u  
LEFT JOIN orders o  
    ON u.id = o.user_id  
   AND o.status = 'paid';  
Ошибка 3. Забыть WHERE в UPDATE/DELETE  

Опасно:  

DELETE FROM users;  
UPDATE users  
SET is_active = false;  
Ошибка 4. Использовать COUNT(*) после LEFT JOIN  
SELECT u.id, COUNT(*)  
FROM users u  
LEFT JOIN orders o ON u.id = o.user_id  
GROUP BY u.id;  

Пользователь без заказов получит 1.  

Правильно:  

SELECT u.id, COUNT(o.id)  
FROM users u  
LEFT JOIN orders o ON u.id = o.user_id  
GROUP BY u.id;  
Ошибка 5. Неправильная фильтрация по датам  

Потенциально плохо:  

WHERE created_at BETWEEN '2026-07-01' AND '2026-07-31'  

Лучше:  

WHERE created_at >= '2026-07-01'  
  AND created_at <  '2026-08-01'  
Ошибка 6. Считать, что индекс всегда ускоряет  

Индекс помогает не всегда.  

Если запрос возвращает большую часть таблицы, БД может выбрать Seq Scan.  

Ошибка 7. Путать WHERE и HAVING  

### Плохо:

WHERE COUNT(*) > 5  

Правильно:  

HAVING COUNT(*) > 5  

## 57. Мини-шпаргалка по синтаксису

SELECT column1, column2  
FROM table_name  
WHERE condition  
GROUP BY column1  
HAVING aggregate_condition  
ORDER BY column1 DESC  
LIMIT 10 OFFSET 20;  
SELECT u.name, o.total  
FROM users u  
JOIN orders o ON u.id = o.user_id;  
SELECT user_id, COUNT(*)  
FROM orders  
GROUP BY user_id  
HAVING COUNT(*) > 3;  
WITH ranked AS (  
    SELECT *,  
           ROW_NUMBER() OVER (  
               PARTITION BY user_id  
               ORDER BY created_at DESC  
           ) AS rn  
    FROM orders  
)  
SELECT *  
FROM ranked  
WHERE rn = 1;  

## 58. Как отвечать на собеседовании

Если спрашивают: «Как оптимизировать медленный запрос?»  

### Хороший ответ:

Сначала посмотрю план выполнения через EXPLAIN или EXPLAIN ANALYZE. Проверю, используются ли индексы, нет ли полного сканирования большой таблицы, дорогих сортировок или неудачных JOIN. Потом посмотрю условия фильтрации, порядок JOIN, объём данных, селективность, наличие подходящих индексов. Также проверю, не используются ли функции поверх индексируемых колонок и не тянем ли лишние поля через SELECT *.  

Если спрашивают: «Как проверить данные после API-запроса?»  

### Хороший ответ:

Я бы отправил API-запрос, проверил HTTP-ответ, а затем сходил в БД и проверил фактическое состояние: создана ли запись, корректны ли поля, есть ли связанные записи, не появились ли дубликаты, правильно ли выставлены статусы и timestamps. После теста удалил бы тестовые данные или использовал изолированную тестовую транзакцию/фикстуры.  

Если спрашивают: «Что важнее — проверять через API или через БД?»  

### Хороший ответ:

Основную бизнес-проверку лучше делать через публичный контракт системы — API. БД я использую дополнительно: для подготовки данных, проверки сайд-эффектов, диагностики, сложных интеграционных сценариев и проверки консистентности. Но тесты не должны чрезмерно завязываться на внутреннюю структуру БД, если она не является частью контракта.  

## 59. Что обязательно повторить перед интервью

Самый важный минимум:  

SELECT, WHERE, ORDER BY, LIMIT.  
JOIN: INNER, LEFT, FULL.  
GROUP BY, HAVING.  
COUNT, SUM, AVG, MIN, MAX.  
NULL, IS NULL, COALESCE.  
DISTINCT.  
Подзапросы.  
EXISTS, NOT EXISTS.  
CTE через WITH.  
Оконные функции: ROW_NUMBER, RANK, DENSE_RANK, LAG, LEAD.  
Индексы.  
Транзакции и ACID.  
DELETE vs TRUNCATE vs DROP.  
Первичные и внешние ключи.  
Типовые задачи: дубликаты, последний заказ, пользователи без заказов, топ-N.  

## 60. Самые частые live-задачи

1. Дубликаты  
SELECT email, COUNT(*)  
FROM users  
GROUP BY email  
HAVING COUNT(*) > 1;  
2. Последняя запись в группе  
WITH ranked AS (  
    SELECT *,  
           ROW_NUMBER() OVER (  
               PARTITION BY user_id  
               ORDER BY created_at DESC  
           ) AS rn  
    FROM orders  
)  
SELECT *  
FROM ranked  
WHERE rn = 1;  
3. Записи без связи  
SELECT u.*  
FROM users u  
LEFT JOIN orders o ON u.id = o.user_id  
WHERE o.id IS NULL;  
4. Агрегация по группе  
SELECT user_id, SUM(total)  
FROM orders  
GROUP BY user_id;  
5. Топ-N по группе  
WITH ranked AS (  
    SELECT *,  
           ROW_NUMBER() OVER (  
               PARTITION BY user_id  
               ORDER BY total DESC  
           ) AS rn  
    FROM orders  
)  
SELECT *  
FROM ranked  
WHERE rn <= 3;  

