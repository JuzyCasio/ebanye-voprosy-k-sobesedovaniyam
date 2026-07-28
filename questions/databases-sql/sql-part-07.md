# SQL — полный конспект к собеседованию — часть 7

[← Оглавление](sql.md) · [← К разделу](../databases-sql.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/sql.md)

Темы 46-49.

## 46. Stored Procedures / Functions

В БД можно хранить функции и процедуры.  

### Пример идеи:

CREATE FUNCTION get_user_orders_count(user_id_param BIGINT)  
RETURNS INT AS $$  
BEGIN  
    RETURN (  
        SELECT COUNT(*)  
        FROM orders  
        WHERE user_id = user_id_param  
    );  
END;  
$$ LANGUAGE plpgsql;  

### На собеседовании достаточно:

Хранимые процедуры и функции позволяют переносить часть бизнес-логики в БД, но это может усложнять тестирование, версионирование и поддержку.  

## 47. Триггеры

Триггер — автоматическое действие при событии:  

INSERT;  
UPDATE;  
DELETE.  

### Пример использования:

обновить updated_at;  
записать аудит;  
проверить сложное правило.  

### Минусы:

неочевидная логика;  
сложнее дебажить;  
можно получить неожиданные сайд-эффекты.  

## 48. SQL Injection

SQL-инъекция — уязвимость, когда пользовательский ввод напрямую вставляется в SQL.  

### Плохо:

query = f"SELECT * FROM users WHERE email = '{email}'"  

Если пользователь введёт:  

' OR '1' = '1  

запрос может сломаться или вернуть лишние данные.  

Правильно использовать параметризованные запросы:  

cursor.execute(  
    "SELECT * FROM users WHERE email = %s",  
    (email,)  
)  

### На собеседовании:

Данные пользователя нельзя конкатенировать в SQL. Нужно использовать параметры/плейсхолдеры ORM или драйвера.  

## 49. Полезные функции

Работа со строками  
LOWER(name)  
UPPER(name)  
LENGTH(name)  
TRIM(name)  
SUBSTRING(name FROM 1 FOR 3)  
CONCAT(first_name, ' ', last_name)  

### Примеры:

SELECT LOWER(email)  
FROM users;  
SELECT TRIM(name)  
FROM users;  
Работа с датами  

PostgreSQL:  

NOW()  
CURRENT_DATE  
CURRENT_TIMESTAMP  
DATE_TRUNC('day', created_at)  
created_at + INTERVAL '1 day'  
created_at - INTERVAL '1 hour'  

### Пример группировки по дням:

SELECT DATE_TRUNC('day', created_at) AS day,  
       COUNT(*) AS orders_count  
FROM orders  
GROUP BY day  
ORDER BY day;  
CAST  
SELECT CAST('123' AS INT);  

Или PostgreSQL-стиль:  

SELECT '123'::INT;  

