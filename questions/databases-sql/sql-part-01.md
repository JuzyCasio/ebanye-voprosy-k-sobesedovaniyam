# SQL — полный конспект к собеседованию — часть 1

[← Оглавление](sql.md) · [← К разделу](../databases-sql.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/sql.md)

Темы 1-12.

## 1. Что такое SQL

SQL — язык для работы с реляционными базами данных.  

Он используется для:  

получения данных;  
добавления, изменения и удаления данных;  
создания таблиц, индексов, ограничений;  
управления транзакциями;  
настройки прав доступа.  

### Основные СУБД:

PostgreSQL;  
MySQL;  
Oracle;  
MS SQL Server;  
SQLite;  
ClickHouse — чаще аналитическая БД, SQL-подобный язык.  

## 2. Основные группы SQL-команд

DQL — запросы данных  
SELECT * FROM users;  

Используется для чтения данных.  

DML — изменение данных  
INSERT INTO users (name, email) VALUES ('Alex', 'alex@test.com');  

UPDATE users  
SET name = 'Alexander'  
WHERE id = 1;  

DELETE FROM users  
WHERE id = 1;  

DML работает с содержимым таблиц.  

DDL — структура базы  
CREATE TABLE users (  
    id SERIAL PRIMARY KEY,  
    name TEXT NOT NULL,  
    email TEXT UNIQUE  
);  

ALTER TABLE users ADD COLUMN age INT;  

DROP TABLE users;  

DDL меняет структуру БД.  

TCL — транзакции  
BEGIN;  

UPDATE accounts SET balance = balance - 100 WHERE id = 1;  
UPDATE accounts SET balance = balance + 100 WHERE id = 2;  

COMMIT;  

Или откат:  

ROLLBACK;  
DCL — права  
GRANT SELECT ON users TO readonly_user;  

REVOKE SELECT ON users FROM readonly_user;  

## 3. Базовый SELECT

SELECT id, name, email  
FROM users;  

Получить все поля:  

SELECT *  
FROM users;  

### На собеседовании лучше говорить:

В реальном коде SELECT * лучше избегать, потому что он тянет лишние данные, ломает явный контракт и может ухудшить производительность.  

## 4. Логический порядок выполнения SELECT

### Важно: SQL пишется в одном порядке, а выполняется логически в другом.

Пишем:  

SELECT name, COUNT(*)  
FROM users  
WHERE is_active = true  
GROUP BY name  
HAVING COUNT(*) > 1  
ORDER BY name  
LIMIT 10;  

Логически выполняется так:  

FROM  
JOIN  
WHERE  
GROUP BY  
HAVING  
SELECT  
DISTINCT  
ORDER BY  
LIMIT / OFFSET  

Это часто спрашивают.  

## 5. WHERE — фильтрация строк

SELECT *  
FROM users  
WHERE age >= 18;  
Основные операторы  
=       -- равно  
!=      -- не равно  
<>      -- не равно  
>       -- больше  
<       -- меньше  
>=      -- больше или равно  
<=      -- меньше или равно  

### Пример:

SELECT *  
FROM orders  
WHERE total > 1000;  
AND / OR / NOT  
SELECT *  
FROM users  
WHERE age >= 18 AND is_active = true;  
SELECT *  
FROM users  
WHERE city = 'Moscow' OR city = 'Saint Petersburg';  
SELECT *  
FROM users  
WHERE NOT is_active;  

### Важно помнить про скобки:

SELECT *  
FROM users  
WHERE age >= 18  
  AND (city = 'Moscow' OR city = 'Saint Petersburg');  

## 6. IN

SELECT *  
FROM users  
WHERE city IN ('Moscow', 'Saint Petersburg', 'Kazan');  

То же самое, что:  

WHERE city = 'Moscow'  
   OR city = 'Saint Petersburg'  
   OR city = 'Kazan'  

## 7. BETWEEN

SELECT *  
FROM orders  
WHERE total BETWEEN 1000 AND 5000;  

### Важно:

BETWEEN 1000 AND 5000  

включает обе границы:  

total >= 1000 AND total <= 5000  

Для дат часто безопаснее писать так:  

SELECT *  
FROM orders  
WHERE created_at >= '2026-07-01'  
  AND created_at <  '2026-08-01';  

### Почему лучше так?

Потому что если написать:  

WHERE created_at BETWEEN '2026-07-01' AND '2026-07-31'  

можно случайно потерять записи за 2026-07-31 12:30:00, если тип поля содержит время.  

## 8. LIKE / ILIKE

SELECT *  
FROM users  
WHERE email LIKE '%@gmail.com';  

Шаблоны:  

%  -- любое количество символов  
_  -- один символ  

### Примеры:

-- начинается с Alex  
WHERE name LIKE 'Alex%'  

-- заканчивается на gmail.com  
WHERE email LIKE '%gmail.com'  

-- содержит test  
WHERE email LIKE '%test%'  

В PostgreSQL есть ILIKE — поиск без учёта регистра:  

SELECT *  
FROM users  
WHERE name ILIKE 'alex%';  

## 9. NULL

NULL — это отсутствие значения, не ноль и не пустая строка.  

Неправильно:  

WHERE deleted_at = NULL  

Правильно:  

WHERE deleted_at IS NULL  

И наоборот:  

WHERE deleted_at IS NOT NULL  

### Важно:

NULL = NULL  

не возвращает true.  

Потому что NULL означает неизвестное значение.  

COALESCE  

Возвращает первое не-NULL значение.  

SELECT COALESCE(phone, email, 'no contact')  
FROM users;  

### Пример:

SELECT id, COALESCE(discount, 0) AS discount  
FROM orders;  

Если discount равен NULL, вернётся 0.  

NULLIF  
SELECT NULLIF(status, 'unknown')  
FROM orders;  

Если status = 'unknown', вернётся NULL.  

### Частый пример — защита от деления на ноль:

SELECT revenue / NULLIF(users_count, 0)  
FROM stats;  

## 10. DISTINCT

Убирает дубликаты.  

SELECT DISTINCT city  
FROM users;  

По нескольким колонкам:  

SELECT DISTINCT city, age  
FROM users;  

Тут уникальность считается по паре city + age.  

## 11. ORDER BY

SELECT *  
FROM users  
ORDER BY created_at DESC;  

Сортировка:  

ASC   -- по возрастанию  
DESC  -- по убыванию  

Несколько условий:  

SELECT *  
FROM users  
ORDER BY city ASC, age DESC;  

Сначала сортировка по городу, внутри города — по возрасту.  

## 12. LIMIT / OFFSET

SELECT *  
FROM users  
ORDER BY id  
LIMIT 10;  

Пропустить первые 10:  

SELECT *  
FROM users  
ORDER BY id  
LIMIT 10 OFFSET 10;  

Используется для пагинации.  

Но для больших таблиц OFFSET может быть дорогим, потому что БД всё равно должна пройти пропускаемые строки.  

Лучше для больших данных использовать keyset pagination:  

SELECT *  
FROM users  
WHERE id > 1000  
ORDER BY id  
LIMIT 10;  

