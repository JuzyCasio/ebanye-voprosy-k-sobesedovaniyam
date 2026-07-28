# SQL — полный конспект к собеседованию — часть 10

[← Оглавление](sql.md) · [← К разделу](../databases-sql.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/sql.md)

Темы 53-55.

## 53. Частые вопросы на собеседовании

Чем WHERE отличается от HAVING?  

WHERE фильтрует строки до группировки.  

HAVING фильтрует группы после группировки.  

### Пример:

SELECT user_id, COUNT(*)  
FROM orders  
WHERE status = 'paid'  
GROUP BY user_id  
HAVING COUNT(*) > 5;  

Здесь:  

WHERE status = 'paid' оставляет только оплаченные заказы;  
GROUP BY user_id группирует по пользователю;  
HAVING COUNT(*) > 5 оставляет пользователей с количеством заказов больше 5.  
Чем INNER JOIN отличается от LEFT JOIN?  

INNER JOIN возвращает только совпавшие строки.  

LEFT JOIN возвращает все строки из левой таблицы, даже если справа совпадений нет.  

Чем DELETE отличается от TRUNCATE?  

DELETE удаляет строки, может использовать WHERE.  

TRUNCATE быстро очищает всю таблицу, без WHERE.  

Чем DROP отличается от DELETE?  

DELETE удаляет данные из таблицы.  

DROP удаляет саму таблицу.  

Чем UNION отличается от UNION ALL?  

UNION убирает дубликаты.  

UNION ALL не убирает дубликаты и обычно быстрее.  

### Что такое индекс?

Индекс — структура данных для ускорения поиска.  

Но он:  

занимает место;  
замедляет вставку/обновление/удаление;  
должен создаваться под конкретные запросы.  
Что такое первичный ключ?  

PRIMARY KEY — уникальный идентификатор строки.  

Он:  

уникален;  
не может быть NULL;  
часто используется для связей между таблицами.  
Что такое внешний ключ?  

FOREIGN KEY — ссылка на запись в другой таблице.  

Он помогает поддерживать ссылочную целостность.  

### Что такое транзакция?

Транзакция — набор операций, который выполняется целиком или не выполняется вообще.  

### Пример: перевод денег между счетами.

Что такое ACID?  
Atomicity — атомарность;  
Consistency — согласованность;  
Isolation — изолированность;  
Durability — долговечность.  
Что такое нормализация?  

Процесс проектирования структуры БД, чтобы уменьшить дублирование и повысить целостность данных.  

### Что такое денормализация?

Осознанное добавление избыточности ради ускорения чтения.  

### Что такое оконные функции?

Функции, которые считают значения по группе строк, но не схлопывают результат, в отличие от GROUP BY.  

### Пример:

ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC)  

## 54. PostgreSQL-specific полезности

SERIAL / BIGSERIAL  
id SERIAL PRIMARY KEY  
id BIGSERIAL PRIMARY KEY  

Автоинкремент.  

В новых версиях PostgreSQL часто рекомендуют стандартный вариант:  

id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY  
RETURNING  
INSERT INTO users (name, email)  
VALUES ('Alex', 'alex@test.com')  
RETURNING id;  

Можно использовать и с UPDATE, DELETE.  

UPDATE users  
SET is_active = false  
WHERE id = 1  
RETURNING *;  
UPSERT — ON CONFLICT  
INSERT INTO users (email, name)  
VALUES ('alex@test.com', 'Alex')  
ON CONFLICT (email)  
DO UPDATE SET name = EXCLUDED.name;  

Если email уже есть — обновит имя.  

Если нет — создаст запись.  

JSONB  
CREATE TABLE events (  
    id BIGSERIAL PRIMARY KEY,  
    payload JSONB  
);  

Запрос:  

SELECT *  
FROM events  
WHERE payload->>'type' = 'user_created';  

-> возвращает JSON.  

->> возвращает текст.  

## 55. Практические паттерны запросов

Проверка существования записи  
SELECT EXISTS (  
    SELECT 1  
    FROM users  
    WHERE email = 'alex@test.com'  
);  
Безопасная проверка перед удалением  

Сначала:  

SELECT *  
FROM users  
WHERE email LIKE 'autotest-%';  

Потом:  

DELETE FROM users  
WHERE email LIKE 'autotest-%';  
Найти битые связи  
SELECT o.*  
FROM orders o  
LEFT JOIN users u ON o.user_id = u.id  
WHERE u.id IS NULL;  
Найти записи с пустыми важными полями  
SELECT *  
FROM users  
WHERE email IS NULL  
   OR name IS NULL  
   OR name = '';  
Проверить уникальность  
SELECT email, COUNT(*)  
FROM users  
GROUP BY email  
HAVING COUNT(*) > 1;  
Проверить распределение статусов  
SELECT status, COUNT(*)  
FROM orders  
GROUP BY status  
ORDER BY COUNT(*) DESC;  
Найти долгие незавершённые операции  
SELECT *  
FROM operations  
WHERE status = 'processing'  
  AND created_at < NOW() - INTERVAL '1 hour';  

