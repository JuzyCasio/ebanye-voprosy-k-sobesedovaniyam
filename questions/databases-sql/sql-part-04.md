# SQL — полный конспект к собеседованию — часть 4

[← Оглавление](sql.md) · [← К разделу](../databases-sql.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/sql.md)

Темы 27-35.

## 27. Чем GROUP BY отличается от оконных функций

GROUP BY уменьшает количество строк.  

SELECT user_id, SUM(total)  
FROM orders  
GROUP BY user_id;  

Было 100 заказов, стало 10 пользователей.  

Оконная функция оставляет строки.  

SELECT id,  
       user_id,  
       total,  
       SUM(total) OVER (PARTITION BY user_id) AS user_total  
FROM orders;  

Было 100 заказов, осталось 100 строк, но к каждой добавилась сумма по пользователю.  

## 28. INSERT

INSERT INTO users (name, email)  
VALUES ('Alex', 'alex@test.com');  

Несколько строк:  

INSERT INTO users (name, email)  
VALUES  
    ('Alex', 'alex@test.com'),  
    ('Ivan', 'ivan@test.com'),  
    ('Maria', 'maria@test.com');  
INSERT RETURNING в PostgreSQL  
INSERT INTO users (name, email)  
VALUES ('Alex', 'alex@test.com')  
RETURNING id;  

Очень удобно в автотестах: создали тестовые данные и сразу получили id.  

## 29. UPDATE

UPDATE users  
SET is_active = false  
WHERE id = 1;  

Несколько полей:  

UPDATE users  
SET name = 'Alexander',  
    updated_at = NOW()  
WHERE id = 1;  

### Важно:

Перед UPDATE без уверенности лучше сначала выполнить SELECT с таким же WHERE.  

SELECT *  
FROM users  
WHERE id = 1;  

Потом:  

UPDATE users  
SET is_active = false  
WHERE id = 1;  

Без WHERE обновятся все строки.  

## 30. DELETE

DELETE FROM users  
WHERE id = 1;  

Без WHERE удалит все строки:  

DELETE FROM users;  

## 31. DELETE vs TRUNCATE vs DROP

DELETE  

Удаляет строки.  

DELETE FROM users WHERE id = 1;  

Особенности:  

можно использовать WHERE;  
обычно логируется построчно;  
можно откатить в транзакции;  
триггеры могут сработать.  
TRUNCATE  

Быстро очищает всю таблицу.  

TRUNCATE TABLE users;  

Особенности:  

нельзя указать WHERE;  
обычно быстрее DELETE;  
очищает всю таблицу;  
может сбрасывать sequence/id;  
может быть ограничен foreign key.  

В PostgreSQL:  

TRUNCATE TABLE users RESTART IDENTITY;  
DROP  

Удаляет саму таблицу.  

DROP TABLE users;  

После DROP таблицы больше нет.  

## 32. CREATE TABLE

CREATE TABLE users (  
    id BIGSERIAL PRIMARY KEY,  
    name TEXT NOT NULL,  
    email TEXT UNIQUE NOT NULL,  
    age INT CHECK (age >= 0),  
    created_at TIMESTAMP DEFAULT NOW()  
);  

## 33. Типы данных

Частые типы  
INT  
BIGINT  
NUMERIC  
DECIMAL  
FLOAT  
BOOLEAN  
TEXT  
VARCHAR(n)  
DATE  
TIME  
TIMESTAMP  
UUID  
JSON  
JSONB  
VARCHAR vs TEXT  

В PostgreSQL разницы по производительности почти нет.  

name VARCHAR(255)  
description TEXT  

VARCHAR(n) ограничивает длину.  

TEXT — произвольная строка.  

NUMERIC vs FLOAT  

Для денег лучше:  

NUMERIC(10, 2)  

А не FLOAT.  

Почему?  

FLOAT хранит приблизительные значения и может давать ошибки округления.  

DATE vs TIMESTAMP  
DATE       -- только дата  
TIMESTAMP  -- дата + время  

### Пример:

created_at TIMESTAMP DEFAULT NOW()  
UUID  
id UUID PRIMARY KEY  

### Часто используется в распределённых системах, когда ID генерируется не одной БД.

## 34. Constraints — ограничения

PRIMARY KEY  

Уникальный идентификатор строки.  

id BIGSERIAL PRIMARY KEY  

Особенности:  

уникальный;  
не может быть NULL;  
обычно по нему строится индекс.  
FOREIGN KEY  

Связь с другой таблицей.  

CREATE TABLE orders (  
    id BIGSERIAL PRIMARY KEY,  
    user_id BIGINT REFERENCES users(id),  
    total NUMERIC(10, 2)  
);  
NOT NULL  
email TEXT NOT NULL  

Поле обязательно.  

UNIQUE  
email TEXT UNIQUE  

Значение должно быть уникальным.  

CHECK  
age INT CHECK (age >= 0)  

Проверяет условие.  

DEFAULT  
created_at TIMESTAMP DEFAULT NOW()  

Значение по умолчанию.  

## 35. Foreign key actions

ON DELETE CASCADE  
ON DELETE SET NULL  
ON DELETE RESTRICT  
ON DELETE CASCADE  

Если удаляем пользователя, удаляются его заказы.  

CREATE TABLE orders (  
    id BIGSERIAL PRIMARY KEY,  
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE  
);  
ON DELETE SET NULL  

Если пользователь удалён, в заказе user_id станет NULL.  

user_id BIGINT REFERENCES users(id) ON DELETE SET NULL  
ON DELETE RESTRICT  

Нельзя удалить пользователя, если на него есть ссылки.  

user_id BIGINT REFERENCES users(id) ON DELETE RESTRICT  

