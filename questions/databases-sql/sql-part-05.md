# SQL — полный конспект к собеседованию — часть 5

[← Оглавление](sql.md) · [← К разделу](../databases-sql.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/sql.md)

Темы 36-39.

## 36. Индексы

Индекс — структура данных, которая ускоряет поиск, сортировку и JOIN.  

### Пример:

CREATE INDEX idx_users_email ON users(email);  
Когда индекс полезен  
SELECT *  
FROM users  
WHERE email = 'alex@test.com';  

Если по email есть индекс, БД может быстро найти строку.  

Когда индекс может не помочь  
SELECT *  
FROM users  
WHERE LOWER(email) = 'alex@test.com';  

Обычный индекс по email может не использоваться, потому что применена функция.  

Можно создать функциональный индекс:  

CREATE INDEX idx_users_lower_email ON users(LOWER(email));  
Индекс замедляет запись  

Индексы ускоряют чтение, но замедляют:  

INSERT;  
UPDATE;  
DELETE.  

Потому что БД должна обновлять не только таблицу, но и индексы.  

### На собеседовании:

Индекс — это компромисс между скоростью чтения и стоимостью записи/хранения.  

Уникальный индекс  
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);  

Похож на UNIQUE constraint.  

Составной индекс  
CREATE INDEX idx_orders_user_status ON orders(user_id, status);  

### Хорошо работает для:

WHERE user_id = 1  

и  

WHERE user_id = 1 AND status = 'paid'  

Но может плохо работать для:  

WHERE status = 'paid'  

Потому что status — второй столбец в индексе.  

Правило:  

В составном индексе важен порядок колонок.  

### Частичный индекс

PostgreSQL:  

CREATE INDEX idx_active_users_email  
ON users(email)  
WHERE is_active = true;  

Полезно, если часто ищем только активных пользователей.  

Типы индексов  

В PostgreSQL часто встречаются:  

B-tree  -- основной индекс по умолчанию  
Hash    -- equality-поиск  
GIN     -- массивы, JSONB, полнотекстовый поиск  
GiST    -- геоданные, специфичные структуры  
BRIN    -- большие таблицы, где данные физически упорядочены  

На обычном собеседовании достаточно уверенно знать B-tree.  

## 37. EXPLAIN

EXPLAIN показывает план выполнения запроса.  

EXPLAIN  
SELECT *  
FROM users  
WHERE email = 'alex@test.com';  

В PostgreSQL:  

EXPLAIN ANALYZE  
SELECT *  
FROM users  
WHERE email = 'alex@test.com';  

EXPLAIN ANALYZE реально выполняет запрос и показывает фактическое время.  

Осторожно с:  

EXPLAIN ANALYZE DELETE ...  
EXPLAIN ANALYZE UPDATE ...  

Они реально выполнят изменение.  

Что можно увидеть в плане  
Seq Scan  
Index Scan  
Bitmap Index Scan  
Nested Loop  
Hash Join  
Merge Join  
Sort  
Aggregate  
Seq Scan  

Полный проход по таблице.  

Это не всегда плохо.  

Если таблица маленькая или нужно прочитать большую часть строк, Seq Scan нормален.  

Index Scan  

Использование индекса.  

### Хорошо, когда выбирается небольшая часть таблицы.

Nested Loop  

Вложенный цикл.  

Может быть нормально для маленьких выборок, но плохо для больших.  

Hash Join  

БД строит хеш-таблицу по одной таблице и джойнится с другой.  

### Часто хорошо для больших таблиц.

Merge Join  

Обе выборки сортируются и соединяются.  

Полезно, если данные уже отсортированы или есть подходящие индексы.  

## 38. Транзакции

Транзакция — группа операций, которая выполняется как единое целое.  

BEGIN;  

UPDATE accounts  
SET balance = balance - 100  
WHERE id = 1;  

UPDATE accounts  
SET balance = balance + 100  
WHERE id = 2;  

COMMIT;  

Если ошибка:  

ROLLBACK;  

## 39. ACID

Atomicity — атомарность  

Либо выполняется всё, либо ничего.  

### Пример: перевод денег.

Нельзя списать деньги с одного счёта и не зачислить на другой.  

Consistency — согласованность  

База переходит из одного корректного состояния в другое.  

Например, constraint не должен нарушаться.  

Isolation — изолированность  

Параллельные транзакции не должны некорректно влиять друг на друга.  

Durability — долговечность  

После COMMIT данные не должны потеряться даже при сбое.  

