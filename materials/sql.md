# SQL — полный конспект

[← Все полные материалы](README.md) · [Короткая шпаргалка](../cheatsheets/sql.md)

Полный материал из присланного конспекта. Сохранены подробные объяснения, примеры, практические сценарии и вопросы для собеседования.

> **Как пользоваться конспектом**
>
> Выбери тему в навигации, прочитай объяснение и затем проговори выделенный короткий ответ своими словами. Код и команды оформлены отдельными блоками, чтобы их можно было быстро найти и скопировать.

## Навигация по разделу

- [Основы SELECT и агрегация](#основы-select-и-агрегация) — вопросы 1–15
- [JOIN, подзапросы и аналитика](#join-подзапросы-и-аналитика) — вопросы 16–27
- [Изменение данных и схема](#изменение-данных-и-схема) — вопросы 28–35
- [Производительность, транзакции и архитектура БД](#производительность-транзакции-и-архитектура-бд) — вопросы 36–48
- [Практика SQL для QA/SDET](#практика-sql-для-qasdet) — вопросы 49–56
- [Собеседование и самопроверка](#собеседование-и-самопроверка) — вопросы 57–64

## Основы SELECT и агрегация

### 1. Что такое SQL

SQL — язык для работы с реляционными базами данных.

**Он используется для**

- получения данных;
- добавления, изменения и удаления данных;
- создания таблиц, индексов, ограничений;
- управления транзакциями;
- настройки прав доступа.

#### Основные СУБД:

- PostgreSQL;
- MySQL;
- Oracle;
- MS SQL Server;
- SQLite;
- ClickHouse — чаще аналитическая БД, SQL-подобный язык.

### 2. Основные группы SQL-команд

**DQL — запросы данных**

```sql
SELECT * FROM users;
```

Используется для чтения данных.

**DML — изменение данных**

```sql
INSERT INTO users (name, email) VALUES ('Alex', 'alex@test.com');

UPDATE users
SET name = 'Alexander'
WHERE id = 1;

DELETE FROM users
WHERE id = 1;
```

DML работает с содержимым таблиц.

**DDL — структура базы**

```bash
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE
);

ALTER TABLE users ADD COLUMN age INT;

DROP TABLE users;
```

DDL меняет структуру БД.

**TCL — транзакции**

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

**Или откат**

```sql
ROLLBACK;
```

**DCL — права**

```sql
GRANT SELECT ON users TO readonly_user;

REVOKE SELECT ON users FROM readonly_user;
```

### 3. Базовый SELECT

```sql
SELECT id, name, email
FROM users;
```

**Получить все поля**

```sql
SELECT *
FROM users;
```

> **Короткий ответ для собеседования**
>
> В реальном коде SELECT * лучше избегать, потому что он тянет лишние данные, ломает явный контракт и может ухудшить производительность.

### 4. Логический порядок выполнения SELECT

#### Важно: SQL пишется в одном порядке, а выполняется логически в другом.

**Пишем**

```sql
SELECT name, COUNT(*)
FROM users
WHERE is_active = true
GROUP BY name
HAVING COUNT(*) > 1
ORDER BY name
LIMIT 10;
```

**Логически выполняется так**

```sql
FROM
JOIN
WHERE
GROUP BY
HAVING
SELECT
```

**DISTINCT**

```sql
ORDER BY
LIMIT / OFFSET
```

Это часто спрашивают.

### 5. WHERE — фильтрация строк

```sql
SELECT *
FROM users
WHERE age >= 18;
```

**Основные операторы**
=       -- равно
!=      -- не равно
<>      -- не равно
>       -- больше
<       -- меньше
>=      -- больше или равно
<=      -- меньше или равно

#### Пример:

```sql
SELECT *
FROM orders
WHERE total > 1000;
```

AND / OR / NOT

```sql
SELECT *
FROM users
WHERE age >= 18 AND is_active = true;
SELECT *
FROM users
WHERE city = 'Moscow' OR city = 'Saint Petersburg';
SELECT *
FROM users
WHERE NOT is_active;
```

#### Важно помнить про скобки:

```sql
SELECT *
FROM users
WHERE age >= 18
```

  AND (city = 'Moscow' OR city = 'Saint Petersburg');

### 6. IN

```sql
SELECT *
FROM users
WHERE city IN ('Moscow', 'Saint Petersburg', 'Kazan');
```

**То же самое, что**

```sql
WHERE city = 'Moscow'
   OR city = 'Saint Petersburg'
   OR city = 'Kazan'
```

### 7. BETWEEN

```sql
SELECT *
FROM orders
WHERE total BETWEEN 1000 AND 5000;
```

#### Важно:

BETWEEN 1000 AND 5000

**включает обе границы**

total >= 1000 AND total <= 5000

Для дат часто безопаснее писать так:

```sql
SELECT *
FROM orders
WHERE created_at >= '2026-07-01'
```

  AND created_at <  '2026-08-01';

#### Почему лучше так?

**Потому что если написать**

```sql
WHERE created_at BETWEEN '2026-07-01' AND '2026-07-31'
```

можно случайно потерять записи за 2026-07-31 12:30:00, если тип поля содержит время.

### 8. LIKE / ILIKE

```sql
SELECT *
FROM users
WHERE email LIKE '%@gmail.com';
```

**Шаблоны**

%  -- любое количество символов
_  -- один символ

#### Примеры:

-- начинается с Alex

```sql
WHERE name LIKE 'Alex%'
```

-- заканчивается на gmail.com

```sql
WHERE email LIKE '%gmail.com'
```

-- содержит test

```sql
WHERE email LIKE '%test%'
```

В PostgreSQL есть ILIKE — поиск без учёта регистра:

```sql
SELECT *
FROM users
WHERE name ILIKE 'alex%';
```

### 9. NULL

NULL — это отсутствие значения, не ноль и не пустая строка.

**Неправильно**

```sql
WHERE deleted_at = NULL
```

**Правильно**

```sql
WHERE deleted_at IS NULL
```

**И наоборот**

```sql
WHERE deleted_at IS NOT NULL
```

#### Важно:

```sql
NULL = NULL
```

не возвращает true.

Потому что NULL означает неизвестное значение.

**COALESCE**

Возвращает первое не-NULL значение.

```sql
SELECT COALESCE(phone, email, 'no contact')
FROM users;
```

#### Пример:

```sql
SELECT id, COALESCE(discount, 0) AS discount
FROM orders;
```

Если discount равен NULL, вернётся 0.

**NULLIF**

```sql
SELECT NULLIF(status, 'unknown')
FROM orders;
```

Если status = 'unknown', вернётся NULL.

#### Частый пример — защита от деления на ноль:

```sql
SELECT revenue / NULLIF(users_count, 0)
FROM stats;
```

### 10. DISTINCT

Убирает дубликаты.

```sql
SELECT DISTINCT city
FROM users;
```

**По нескольким колонкам**

```sql
SELECT DISTINCT city, age
FROM users;
```

Тут уникальность считается по паре city + age.

### 11. ORDER BY

```sql
SELECT *
FROM users
ORDER BY created_at DESC;
```

**Сортировка**

**ASC   -- по возрастанию**
DESC  -- по убыванию

**Несколько условий**

```sql
SELECT *
FROM users
ORDER BY city ASC, age DESC;
```

Сначала сортировка по городу, внутри города — по возрасту.

### 12. LIMIT / OFFSET

```sql
SELECT *
FROM users
ORDER BY id
LIMIT 10;
```

Пропустить первые 10:

```sql
SELECT *
FROM users
ORDER BY id
LIMIT 10 OFFSET 10;
```

Используется для пагинации.

Но для больших таблиц OFFSET может быть дорогим, потому что БД всё равно должна пройти пропускаемые строки.

Лучше для больших данных использовать keyset pagination:

```sql
SELECT *
FROM users
WHERE id > 1000
ORDER BY id
LIMIT 10;
```

### 13. Агрегатные функции

```sql
COUNT()
SUM()
AVG()
MIN()
MAX()
```

#### Пример:

```sql
SELECT COUNT(*)
FROM users;
```

**Сумма заказов**

```sql
SELECT SUM(total)
FROM orders;
```

**Средний чек**

```sql
SELECT AVG(total)
FROM orders;
```

**Минимальный и максимальный заказ**

```sql
SELECT MIN(total), MAX(total)
FROM orders;
COUNT(*), COUNT(column), COUNT(DISTINCT column)
SELECT COUNT(*)
FROM users;
```

Считает все строки.

```sql
SELECT COUNT(email)
FROM users;
```

Считает строки, где email IS NOT NULL.

```sql
SELECT COUNT(DISTINCT city)
FROM users;
```

Считает количество уникальных городов.

### 14. GROUP BY

Группировка данных.

**Например, есть таблица**

**orders**

```bash
id | user_id | total
```

---+---------+------
1  | 1       | 100
2  | 1       | 300
3  | 2       | 500

**Запрос**

```sql
SELECT user_id, SUM(total)
FROM orders
GROUP BY user_id;
```

**Результат**

**user_id | sum**
--------+-----
1       | 400
2       | 500
Частый вопрос

#### Почему нельзя так?

```sql
SELECT user_id, id, SUM(total)
FROM orders
GROUP BY user_id;
```

Потому что для одного user_id может быть много разных id.

БД не понимает, какой именно id показать.

**Правильно**

```sql
SELECT user_id, SUM(total)
FROM orders
GROUP BY user_id;
```

Или надо добавить id в группировку:

```sql
SELECT user_id, id, SUM(total)
FROM orders
GROUP BY user_id, id;
```

Но это уже другая логика.

### 15. HAVING

```sql
WHERE фильтрует строки до группировки.

HAVING фильтрует группы после группировки.
```

#### Пример: найти пользователей, у которых больше 3 заказов.

```sql
SELECT user_id, COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 3;
```

**Неправильно**

```sql
SELECT user_id, COUNT(*)
FROM orders
WHERE COUNT(*) > 3
GROUP BY user_id;
```

Так нельзя, потому что WHERE выполняется до агрегации.

## JOIN, подзапросы и аналитика

### 16. JOIN

`JOIN` объединяет данные из разных таблиц.

**Допустим**

**Таблица `users`:**

| id | name |
|---:|---|
| 1 | Alex |
| 2 | Ivan |
| 3 | Maria |

**Таблица `orders`:**

| id | user_id | total |
|---:|---:|---:|
| 1 | 1 | 100 |
| 2 | 1 | 200 |
| 3 | 2 | 500 |

#### `INNER JOIN`

Возвращает только совпадающие записи.

```sql
SELECT users.name, orders.total
FROM users
INNER JOIN orders ON users.id = orders.user_id;
```

**Результат**

| name | total |
|---|---:|
| Alex | 100 |
| Alex | 200 |
| Ivan | 500 |

Maria не попадёт, потому что у неё нет заказов.

#### `LEFT JOIN`

Возвращает все строки из левой таблицы и совпадения из правой.

```sql
SELECT users.name, orders.total
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```

**Результат**

| name | total |
|---|---:|
| Alex | 100 |
| Alex | 200 |
| Ivan | 500 |
| Maria | `NULL` |

#### `RIGHT JOIN`

Возвращает все строки из правой таблицы и совпадения из левой.

```sql
SELECT users.name, orders.total
FROM users
RIGHT JOIN orders ON users.id = orders.user_id;
```

На практике чаще используют LEFT JOIN, потому что его проще читать.

#### `FULL OUTER JOIN`

Возвращает все строки из обеих таблиц.

```sql
SELECT users.name, orders.total
FROM users
FULL OUTER JOIN orders ON users.id = orders.user_id;
```

Полезно для сверок данных.

**Например**

- есть пользователь без заказа;
- есть заказ с некорректным `user_id`.

#### `CROSS JOIN`

Декартово произведение.

```sql
SELECT *
FROM colors
CROSS JOIN sizes;
```

Если в colors 3 строки, а в sizes 4 строки, результат будет 12 строк.

#### `SELF JOIN`

Таблица джойнится сама с собой.

#### Пример: сотрудники и их руководители.

```sql
SELECT e.name AS employee,
       m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### 17. Важная ловушка с LEFT JOIN

Допустим, нужно найти всех пользователей и их оплаченные заказы.

#### Плохо:

```sql
SELECT u.id, u.name, o.id AS order_id
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.status = 'paid';
```

Проблема: WHERE o.status = 'paid' убьёт строки, где заказа нет. В итоге LEFT JOIN фактически превратится в INNER JOIN.

**Правильно**

```sql
SELECT u.id, u.name, o.id AS order_id
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
   AND o.status = 'paid';
```

### 18. Найти записи без связи

Например, пользователи без заказов.

```sql
SELECT u.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

**Альтернатива через NOT EXISTS**

```sql
SELECT u.*
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

> **Короткий ответ для собеседования**
>
> Для поиска отсутствующих связей часто использую LEFT JOIN ... IS NULL или NOT EXISTS. На больших данных NOT EXISTS часто читается лучше и может быть эффективнее в зависимости от плана запроса.

### 19. Алиасы

```sql
SELECT u.id, u.name
FROM users AS u;
```

**Можно без AS**

```sql
SELECT u.id, u.name
FROM users u;
```

Алиасы особенно важны при JOIN.

### 20. CASE WHEN

Условная логика в SQL.

```sql
SELECT id,
       total,
       CASE
           WHEN total >= 10000 THEN 'big'
           WHEN total >= 1000 THEN 'medium'
           ELSE 'small'
       END AS order_size
FROM orders;
```

#### Пример для тестирования:

```sql
SELECT id,
       status,
       CASE
           WHEN status = 'paid' THEN true
           ELSE false
       END AS is_paid
FROM orders;
```

### 21. Подзапросы

**Подзапрос в WHERE**

Найти пользователей, у которых есть заказы.

```sql
SELECT *
FROM users
WHERE id IN (
    SELECT user_id
    FROM orders
);
```

**Подзапрос в FROM**

```sql
SELECT user_id, total_sum
FROM (
    SELECT user_id, SUM(total) AS total_sum
    FROM orders
    GROUP BY user_id
) AS user_orders
WHERE total_sum > 1000;
```

**Подзапрос в SELECT**

```sql
SELECT u.id,
       u.name,
       (
           SELECT COUNT(*)
           FROM orders o
           WHERE o.user_id = u.id
       ) AS orders_count
FROM users u;
```

Работает, но на больших данных может быть хуже, чем JOIN + GROUP BY.

### 22. EXISTS / NOT EXISTS

EXISTS проверяет факт существования строк.

```sql
SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);

SELECT 1 здесь значит: нам не нужны конкретные данные, нам важно только наличие строки.
```

**Пользователи без заказов**

```sql
SELECT *
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

**IN vs EXISTS**

```sql
WHERE id IN (SELECT user_id FROM orders)
```

**и**

```sql
WHERE EXISTS (
    SELECT 1 FROM orders WHERE orders.user_id = users.id
)
```

#### часто решают похожую задачу.

**Общее правило**

IN удобно, когда подзапрос возвращает список значений;
EXISTS удобно, когда проверяем наличие связанной строки;
с NULL у NOT IN могут быть неприятные сюрпризы.
Ловушка NOT IN и NULL

```sql
SELECT *
FROM users
WHERE id NOT IN (
    SELECT user_id
    FROM orders
);
```

Если в orders.user_id есть NULL, результат может быть неожиданным.

**Надёжнее**

```sql
SELECT *
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

### 23. CTE — WITH

CTE делает запрос читаемее.

```sql
WITH user_orders AS (
    SELECT user_id, SUM(total) AS total_sum
    FROM orders
    GROUP BY user_id
)
SELECT *
FROM user_orders
WHERE total_sum > 1000;
```

#### Хорошо использовать, когда:

- сложная логика;
- несколько промежуточных шагов;
- нужно повысить читаемость;
- один результат используется дальше.
Несколько CTE

```sql
WITH paid_orders AS (
    SELECT *
    FROM orders
    WHERE status = 'paid'
),
```

**user_totals AS (**

```sql
    SELECT user_id, SUM(total) AS total_sum
    FROM paid_orders
    GROUP BY user_id
)
SELECT *
FROM user_totals
WHERE total_sum > 5000;
```

### 24. Рекурсивный CTE

Используется для деревьев, иерархий, категорий, оргструктур.

```sql
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
```

> **Короткий ответ для собеседования**
>
> Рекурсивный CTE позволяет обходить иерархические данные, например дерево категорий или сотрудников.

### 25. UNION / UNION ALL / INTERSECT / EXCEPT

```sql
UNION
```

Объединяет результаты и убирает дубликаты.

```sql
SELECT email FROM customers
UNION
SELECT email FROM users;
UNION ALL
```

Объединяет результаты, не убирая дубликаты.

```sql
SELECT email FROM customers
UNION ALL
SELECT email FROM users;

UNION ALL быстрее, потому что не делает deduplication.

INTERSECT
```

Возвращает пересечение.

```sql
SELECT email FROM customers
INTERSECT
SELECT email FROM users;
EXCEPT
```

Возвращает строки из первого запроса, которых нет во втором.

```sql
SELECT email FROM users
EXCEPT
SELECT email FROM blocked_users;
```

### 26. Оконные функции

Оконные функции позволяют считать значения по группе строк, но не схлопывать результат как GROUP BY.

#### Главная форма:

**FUNCTION() OVER (**

```sql
    PARTITION BY ...
    ORDER BY ...
)
```

**ROW_NUMBER**

Нумерует строки.

```sql
SELECT user_id,
       total,
       ROW_NUMBER() OVER (
           PARTITION BY user_id
           ORDER BY total DESC
       ) AS rn
FROM orders;
```

Для каждого пользователя заказы будут пронумерованы по убыванию суммы.

Найти самый дорогой заказ каждого пользователя

```sql
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
```

**RANK и DENSE_RANK**

```sql
SELECT user_id,
       total,
       RANK() OVER (ORDER BY total DESC) AS rank,
       DENSE_RANK() OVER (ORDER BY total DESC) AS dense_rank
FROM orders;
```

**Разница**

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

```sql
SELECT id,
       user_id,
       total,
       LAG(total) OVER (
           PARTITION BY user_id
           ORDER BY created_at
       ) AS previous_total
FROM orders;
```

**LEAD — следующее значение**

```sql
SELECT id,
       user_id,
       total,
       LEAD(total) OVER (
           PARTITION BY user_id
           ORDER BY created_at
       ) AS next_total
FROM orders;
```

**Сумма накопительным итогом**

```sql
SELECT id,
       user_id,
       created_at,
       total,
       SUM(total) OVER (
           PARTITION BY user_id
           ORDER BY created_at
       ) AS running_total
FROM orders;
```

**COUNT OVER**

Можно посчитать количество заказов пользователя, не схлопывая строки.

```sql
SELECT id,
       user_id,
       total,
       COUNT(*) OVER (
           PARTITION BY user_id
       ) AS user_orders_count
FROM orders;
```

### 27. Чем GROUP BY отличается от оконных функций

```sql
GROUP BY уменьшает количество строк.

SELECT user_id, SUM(total)
FROM orders
GROUP BY user_id;
```

Было 100 заказов, стало 10 пользователей.

Оконная функция оставляет строки.

```sql
SELECT id,
       user_id,
       total,
       SUM(total) OVER (PARTITION BY user_id) AS user_total
FROM orders;
```

Было 100 заказов, осталось 100 строк, но к каждой добавилась сумма по пользователю.

## Изменение данных и схема

### 28. INSERT

```sql
INSERT INTO users (name, email)
VALUES ('Alex', 'alex@test.com');
```

**Несколько строк**

```sql
INSERT INTO users (name, email)
VALUES
    ('Alex', 'alex@test.com'),
    ('Ivan', 'ivan@test.com'),
    ('Maria', 'maria@test.com');
INSERT RETURNING в PostgreSQL
INSERT INTO users (name, email)
VALUES ('Alex', 'alex@test.com')
```

RETURNING id;

Очень удобно в автотестах: создали тестовые данные и сразу получили id.

### 29. UPDATE

```sql
UPDATE users
SET is_active = false
WHERE id = 1;
```

**Несколько полей**

```sql
UPDATE users
SET name = 'Alexander',
    updated_at = NOW()
WHERE id = 1;
```

#### Важно:

Перед UPDATE без уверенности лучше сначала выполнить SELECT с таким же WHERE.

```sql
SELECT *
FROM users
WHERE id = 1;
```

**Потом**

```sql
UPDATE users
SET is_active = false
WHERE id = 1;
```

Без WHERE обновятся все строки.

### 30. DELETE

```sql
DELETE FROM users
WHERE id = 1;
```

Без WHERE удалит все строки:

```sql
DELETE FROM users;
```

### 31. DELETE vs TRUNCATE vs DROP

```sql
DELETE
```

Удаляет строки.

```sql
DELETE FROM users WHERE id = 1;
```

**Особенности**

- можно использовать WHERE;
- обычно логируется построчно;
- можно откатить в транзакции;
- триггеры могут сработать.

```sql
TRUNCATE
```

Быстро очищает всю таблицу.

```sql
TRUNCATE TABLE users;
```

**Особенности**

- нельзя указать WHERE;
- обычно быстрее DELETE;
- очищает всю таблицу;
- может сбрасывать sequence/id;
- может быть ограничен foreign key.

**В PostgreSQL**

```sql
TRUNCATE TABLE users RESTART IDENTITY;
DROP
```

Удаляет саму таблицу.

```sql
DROP TABLE users;
```

После DROP таблицы больше нет.

### 32. CREATE TABLE

```bash
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    age INT CHECK (age >= 0),
    created_at TIMESTAMP DEFAULT NOW()
);
```

### 33. Типы данных

**Частые типы**
INT
BIGINT
NUMERIC
DECIMAL
FLOAT
BOOLEAN
TEXT

```sql
VARCHAR(n)
```

**DATE**
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

**NUMERIC vs FLOAT**

**Для денег лучше**

```sql
NUMERIC(10, 2)
```

А не FLOAT.

Почему?

FLOAT хранит приблизительные значения и может давать ошибки округления.

**DATE vs TIMESTAMP**
DATE       -- только дата
TIMESTAMP  -- дата + время

#### Пример:

created_at TIMESTAMP DEFAULT NOW()
UUID

```bash
id UUID PRIMARY KEY
```

#### Часто используется в распределённых системах, когда ID генерируется не одной БД.

### 34. Constraints — ограничения

**PRIMARY KEY**

Уникальный идентификатор строки.

**id BIGSERIAL PRIMARY KEY**

**Особенности**

уникальный;
не может быть NULL;
обычно по нему строится индекс.
FOREIGN KEY

Связь с другой таблицей.

```bash
CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    total NUMERIC(10, 2)
);
```

**NOT NULL**
email TEXT NOT NULL

Поле обязательно.

**UNIQUE**
email TEXT UNIQUE

Значение должно быть уникальным.

**CHECK**
age INT CHECK (age >= 0)

Проверяет условие.

**DEFAULT**
created_at TIMESTAMP DEFAULT NOW()

Значение по умолчанию.

### 35. Foreign key actions

**ON DELETE CASCADE**
ON DELETE SET NULL
ON DELETE RESTRICT
ON DELETE CASCADE

Если удаляем пользователя, удаляются его заказы.

```bash
CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE
);
```

**ON DELETE SET NULL**

Если пользователь удалён, в заказе user_id станет NULL.

user_id BIGINT REFERENCES users(id) ON DELETE SET NULL
ON DELETE RESTRICT

Нельзя удалить пользователя, если на него есть ссылки.

user_id BIGINT REFERENCES users(id) ON DELETE RESTRICT

## Производительность, транзакции и архитектура БД

### 36. Индексы

Индекс — структура данных, которая ускоряет поиск, сортировку и JOIN.

#### Пример:

```sql
CREATE INDEX idx_users_email ON users(email);
```

**Когда индекс полезен**

```sql
SELECT *
FROM users
WHERE email = 'alex@test.com';
```

Если по email есть индекс, БД может быстро найти строку.

Когда индекс может не помочь

```sql
SELECT *
FROM users
WHERE LOWER(email) = 'alex@test.com';
```

Обычный индекс по email может не использоваться, потому что применена функция.

**Можно создать функциональный индекс**

```sql
CREATE INDEX idx_users_lower_email ON users(LOWER(email));
```

**Индекс замедляет запись**

Индексы ускоряют чтение, но замедляют:

```sql
INSERT;
UPDATE;
DELETE.
```

Потому что БД должна обновлять не только таблицу, но и индексы.

> **Короткий ответ для собеседования**
>
> Индекс — это компромисс между скоростью чтения и стоимостью записи/хранения.

**Уникальный индекс**

```sql
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);
```

Похож на UNIQUE constraint.

**Составной индекс**

```sql
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

#### Хорошо работает для:

```sql
WHERE user_id = 1
```

**и**

```sql
WHERE user_id = 1 AND status = 'paid'
```

Но может плохо работать для:

```sql
WHERE status = 'paid'
```

Потому что status — второй столбец в индексе.

**Правило**

В составном индексе важен порядок колонок.

#### Частичный индекс

**PostgreSQL**

```sql
CREATE INDEX idx_active_users_email
```

ON users(email)

```sql
WHERE is_active = true;
```

Полезно, если часто ищем только активных пользователей.

**Типы индексов**

**В PostgreSQL часто встречаются**

B-tree  -- основной индекс по умолчанию
Hash    -- equality-поиск
GIN     -- массивы, JSONB, полнотекстовый поиск
GiST    -- геоданные, специфичные структуры
BRIN    -- большие таблицы, где данные физически упорядочены

На обычном собеседовании достаточно уверенно знать B-tree.

### 37. EXPLAIN

```sql
EXPLAIN показывает план выполнения запроса.

EXPLAIN
SELECT *
FROM users
WHERE email = 'alex@test.com';
```

**В PostgreSQL**

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'alex@test.com';

EXPLAIN ANALYZE реально выполняет запрос и показывает фактическое время.
```

**Осторожно с**

```sql
EXPLAIN ANALYZE DELETE ...
EXPLAIN ANALYZE UPDATE ...
```

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

**Index Scan**

Использование индекса.

#### Хорошо, когда выбирается небольшая часть таблицы.

**Nested Loop**

Вложенный цикл.

Может быть нормально для маленьких выборок, но плохо для больших.

**Hash Join**

БД строит хеш-таблицу по одной таблице и джойнится с другой.

#### Часто хорошо для больших таблиц.

**Merge Join**

Обе выборки сортируются и соединяются.

Полезно, если данные уже отсортированы или есть подходящие индексы.

### 38. Транзакции

Транзакция — группа операций, которая выполняется как единое целое.

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

**Если ошибка**

```sql
ROLLBACK;
```

### 39. ACID

**Atomicity — атомарность**

Либо выполняется всё, либо ничего.

#### Пример: перевод денег.

Нельзя списать деньги с одного счёта и не зачислить на другой.

**Consistency — согласованность**

База переходит из одного корректного состояния в другое.

Например, constraint не должен нарушаться.

**Isolation — изолированность**

Параллельные транзакции не должны некорректно влиять друг на друга.

**Durability — долговечность**

После COMMIT данные не должны потеряться даже при сбое.

### 40. Уровни изоляции транзакций

#### Основные:

**READ UNCOMMITTED**
READ COMMITTED
REPEATABLE READ
SERIALIZABLE

В PostgreSQL фактически READ UNCOMMITTED работает как READ COMMITTED.

**Dirty read**

Транзакция читает незакоммиченные данные другой транзакции.

#### Пример:

Транзакция A изменила баланс, но не сделала COMMIT.
Транзакция B прочитала это изменение.
Транзакция A сделала ROLLBACK.
Транзакция B прочитала данные, которых как бы никогда не было.
Non-repeatable read

В рамках одной транзакции один и тот же запрос возвращает разные данные.

Транзакция A читает пользователя.
Транзакция B обновляет пользователя и делает COMMIT.
Транзакция A снова читает пользователя и видит другое значение.
Phantom read

В рамках одной транзакции повторный запрос возвращает новый набор строк.

Транзакция A ищет все заказы total > 1000.
Транзакция B добавляет новый такой заказ и делает COMMIT.
Транзакция A повторяет запрос и видит новую строку.
Таблица уровней изоляции
| Уровень | Dirty read | Non-repeatable read | Phantom read |
|---|---|---|---|
| READ UNCOMMITTED | возможно | возможно | возможно |
| READ COMMITTED | нет | возможно | возможно |
| REPEATABLE READ | нет | нет | зависит от СУБД |
| SERIALIZABLE | нет | нет | нет |

### 41. Блокировки

Блокировки нужны для конкурентного доступа.

#### Пример:

```sql
SELECT *
FROM accounts
WHERE id = 1
```

FOR UPDATE;

FOR UPDATE блокирует выбранные строки для изменения другими транзакциями.

**Deadlock**

Deadlock — взаимная блокировка.

#### Пример:

Транзакция A заблокировала строку 1.
Транзакция B заблокировала строку 2.
Транзакция A хочет строку 2.
Транзакция B хочет строку 1.
Обе ждут друг друга.

БД обычно обнаруживает deadlock и отменяет одну транзакцию.

### 42. MVCC

MVCC — Multi-Version Concurrency Control.

#### Идея:

БД хранит несколько версий строк, чтобы читающие транзакции не блокировали пишущие и наоборот.

В PostgreSQL MVCC — важная часть работы транзакций.

> **Короткий ответ для собеседования**
>
> За счёт MVCC одна транзакция может видеть свой согласованный снимок данных, пока другая уже изменила строки.

### 43. Нормализация

Нормализация — способ проектирования таблиц, чтобы уменьшить дублирование и избежать аномалий данных.

1NF — первая нормальная форма
В ячейке одно значение.
Нет списков внутри одного поля.

#### Плохо:

**user_id | phones**
--------+---------------------
1       | 123, 456, 789

**Лучше**

**user_phones**

**user_id | phone**
--------+------
1       | 123
1       | 456
1       | 789
2NF — вторая нормальная форма
Таблица в 1NF.
Неключевые поля зависят от всего составного ключа, а не от его части.
3NF — третья нормальная форма
Таблица во 2NF.
Неключевые поля не зависят от других неключевых полей.

#### Пример плохой таблицы:

**orders**

order_id | user_id | user_name | user_email

user_name и user_email зависят от user_id, а не от order_id.

**Лучше**

**users**
orders
Денормализация

Денормализация — осознанное добавление дублирования ради производительности.

Например, хранить orders.user_email, чтобы не делать JOIN при аналитике.

> **Короткий ответ для собеседования**
>
> Нормализация уменьшает дублирование и повышает целостность данных, денормализация может ускорять чтение, но усложняет поддержку консистентности.

### 44. Связи между таблицами

**One-to-one**

Один пользователь — один профиль.

**users**
profiles
profiles.user_id UNIQUE REFERENCES users(id)
One-to-many

Один пользователь — много заказов.

**users**
orders
orders.user_id REFERENCES users(id)
Many-to-many

Пользователи и роли.

Один пользователь может иметь много ролей.

Одна роль может быть у многих пользователей.

**Нужна промежуточная таблица**

```sql
CREATE TABLE user_roles (
    user_id BIGINT REFERENCES users(id),
    role_id BIGINT REFERENCES roles(id),
    PRIMARY KEY (user_id, role_id)
);
```

### 45. Views — представления

View — сохранённый SQL-запрос.

```sql
CREATE VIEW active_users AS
SELECT *
FROM users
WHERE is_active = true;
```

**Использование**

```sql
SELECT *
FROM active_users;
```

**Materialized view**

Материализованное представление хранит результат физически.

```sql
CREATE MATERIALIZED VIEW user_order_stats AS
SELECT user_id, COUNT(*) AS orders_count, SUM(total) AS total_sum
FROM orders
GROUP BY user_id;
```

**Обновление**

REFRESH MATERIALIZED VIEW user_order_stats;

Обычный VIEW каждый раз выполняет запрос.

MATERIALIZED VIEW хранит результат, но его надо обновлять.

### 46. Stored Procedures / Functions

В БД можно хранить функции и процедуры.

#### Пример идеи:

```sql
CREATE FUNCTION get_user_orders_count(user_id_param BIGINT)
```

**RETURNS INT AS $$**

```sql
BEGIN
    RETURN (
        SELECT COUNT(*)
        FROM orders
        WHERE user_id = user_id_param
    );
```

END;
$$ LANGUAGE plpgsql;

> **Короткий ответ для собеседования**
>
> Хранимые процедуры и функции позволяют переносить часть бизнес-логики в БД, но это может усложнять тестирование, версионирование и поддержку.

### 47. Триггеры

Триггер — автоматическое действие при событии:

```sql
INSERT;
UPDATE;
DELETE.
```

#### Пример использования:

обновить updated_at;
записать аудит;
проверить сложное правило.

#### Минусы:

неочевидная логика;
сложнее дебажить;
можно получить неожиданные сайд-эффекты.

### 48. SQL Injection

SQL-инъекция — уязвимость, когда пользовательский ввод напрямую вставляется в SQL.

#### Плохо:

```sql
query = f"SELECT * FROM users WHERE email = '{email}'"
```

**Если пользователь введёт**

' OR '1' = '1

запрос может сломаться или вернуть лишние данные.

**Правильно использовать параметризованные запросы**

**cursor.execute(**

```sql
    "SELECT * FROM users WHERE email = %s",
    (email,)
)
```

> **Короткий ответ для собеседования**
>
> Данные пользователя нельзя конкатенировать в SQL. Нужно использовать параметры/плейсхолдеры ORM или драйвера.

## Практика SQL для QA/SDET

### 49. Полезные функции

**Работа со строками**

```sql
LOWER(name)
UPPER(name)
LENGTH(name)
TRIM(name)
SUBSTRING(name FROM 1 FOR 3)
CONCAT(first_name, ' ', last_name)
```

#### Примеры:

```sql
SELECT LOWER(email)
FROM users;
SELECT TRIM(name)
FROM users;
```

**Работа с датами**

**PostgreSQL**

```sql
NOW()
```

**CURRENT_DATE**
CURRENT_TIMESTAMP

```sql
DATE_TRUNC('day', created_at)
```

created_at + INTERVAL '1 day'
created_at - INTERVAL '1 hour'

#### Пример группировки по дням:

```sql
SELECT DATE_TRUNC('day', created_at) AS day,
       COUNT(*) AS orders_count
FROM orders
GROUP BY day
ORDER BY day;
```

**CAST**

```sql
SELECT CAST('123' AS INT);
```

**Или PostgreSQL-стиль**

```sql
SELECT '123'::INT;
```

### 50. Типовые задачи на собеседовании

Ниже набор задач, которые часто дают QA Automation / Backend / Data-ish кандидатам.

Задача 1. Найти дубликаты email

```sql
SELECT email, COUNT(*) AS cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

Задача 2. Найти пользователей без заказов

```sql
SELECT u.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

**Или**

```sql
SELECT u.*
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

Задача 3. Посчитать количество заказов по каждому пользователю

```sql
SELECT u.id,
       u.name,
       COUNT(o.id) AS orders_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;
```

#### Почему COUNT(o.id), а не COUNT(*)?

Потому что при LEFT JOIN пользователь без заказов всё равно даст одну строку, и COUNT(*) вернёт 1.

COUNT(o.id) вернёт 0.

Задача 4. Найти пользователей с количеством заказов больше 3

```sql
SELECT u.id,
       u.name,
       COUNT(o.id) AS orders_count
FROM users u
JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name
HAVING COUNT(o.id) > 3;
```

Задача 5. Найти последний заказ каждого пользователя

**Вариант через оконную функцию**

```sql
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
```

Задача 6. Найти максимальный заказ каждого пользователя

```sql
SELECT user_id, MAX(total) AS max_total
FROM orders
GROUP BY user_id;
```

Если нужны все поля заказа:

```sql
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
```

Задача 7. Найти второй максимальный заказ

**Через DENSE_RANK**

```sql
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
```

Если нужен второй максимум по каждому пользователю:

```sql
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
```

Задача 8. Посчитать выручку по дням

```sql
SELECT DATE_TRUNC('day', created_at) AS day,
       SUM(total) AS revenue
FROM orders
WHERE status = 'paid'
GROUP BY day
ORDER BY day;
```

Задача 9. Найти пользователей, которые сделали заказ в июле 2026

```sql
SELECT DISTINCT u.*
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.created_at >= '2026-07-01'
```

  AND o.created_at <  '2026-08-01';
Задача 10. Найти заказы без существующего пользователя

Если нет foreign key или данные грязные:

```sql
SELECT o.*
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;
```

Задача 11. Найти пользователей, у которых нет оплаченных заказов

```sql
SELECT u.*
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
      AND o.status = 'paid'
);
```

Задача 12. Найти топ-3 пользователя по сумме заказов

```sql
SELECT u.id,
       u.name,
       SUM(o.total) AS total_sum
FROM users u
JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name
ORDER BY total_sum DESC
LIMIT 3;
```

Задача 13. Найти топ-3 заказа каждого пользователя

```sql
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
```

Задача 14. Найти пользователей с одинаковыми email

```sql
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

**Получить сами строки**

```sql
SELECT *
FROM users
WHERE email IN (
    SELECT email
    FROM users
    GROUP BY email
    HAVING COUNT(*) > 1
);
```

Задача 15. Удалить дубликаты, оставив самую раннюю запись

**PostgreSQL**

```sql
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
```

Перед удалением лучше сначала сделать SELECT:

```sql
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
```

### 51. SQL для QA / SDET

Для QA SQL нужен не только чтобы писать запросы, но и чтобы проверять состояние системы.

Что QA обычно проверяет через БД
- создалась ли запись после API-запроса;
- корректно ли обновился статус;
- появилась ли запись в связанной таблице;
- не создались ли дубликаты;
- корректно ли записались даты;
- правильно ли обработались nullable-поля;
- очистка тестовых данных;
- подготовка тестовых данных;
- проверка миграций;
- проверка прав доступа;
- проверка консистентности после интеграций.
Пример: тестируем создание пользователя через API

**После запроса**

```sql
POST /users
```

**Проверяем в БД**

```sql
SELECT id, email, name, created_at
FROM users
WHERE email = 'test_user@example.com';
```

**Проверяем**

- запись есть;
- email корректный;
- name корректный;
- created_at заполнен;
- статус дефолтный;
- пароль не хранится в открытом виде.
Проверить, что пароль не хранится plain text

```sql
SELECT password_hash
FROM users
WHERE email = 'test_user@example.com';
```

**Ожидание**

поле не равно исходному паролю;
значение похоже на hash;
поле не NULL.
Проверить создание заказа

```sql
SELECT *
FROM orders
WHERE external_id = 'test-order-123';
```

**Проверить позиции заказа**

```sql
SELECT *
FROM order_items
WHERE order_id = 123;
```

Проверить, что не создались дубликаты

```sql
SELECT external_id, COUNT(*)
FROM orders
WHERE external_id = 'test-order-123'
GROUP BY external_id
HAVING COUNT(*) > 1;
```

Если запрос вернул строки — есть дубликат.

**Очистка тестовых данных**

```sql
DELETE FROM order_items
WHERE order_id IN (
    SELECT id
    FROM orders
    WHERE external_id LIKE 'autotest-%'
);

DELETE FROM orders
WHERE external_id LIKE 'autotest-%';
```

#### Важно удалять в правильном порядке:

сначала дочерние записи;
потом родительские.

Если настроен ON DELETE CASCADE, можно удалить родителя.

### 52. Миграции БД

Миграция — изменение схемы БД:

- создать таблицу;
- добавить колонку;
- изменить тип;
- добавить индекс;
- добавить constraint;
- заполнить данные;
- удалить старое поле.
Что проверять QA при миграции
миграция накатывается на пустую БД;
миграция накатывается на БД с существующими данными;

- rollback работает, если предусмотрен;
- данные не теряются;
- новые constraints не ломают старые данные;
- индексы созданы;
- дефолты работают;
- приложение стартует после миграции;
- старые API работают;
- новые API работают;
- нет сильной деградации по времени.
Пример проверки новой колонки

**Была добавлена колонка**

```sql
ALTER TABLE users ADD COLUMN is_active BOOLEAN DEFAULT true;
```

#### Проверки:

```sql
SELECT COUNT(*)
FROM users
WHERE is_active IS NULL;
```

Ожидаем 0, если поле должно быть заполнено.

```sql
SELECT COUNT(*)
FROM users
WHERE is_active = false;
```

Проверяем бизнес-логику, если старые пользователи должны быть активны.

### 53. Частые вопросы на собеседовании

Чем WHERE отличается от HAVING?

```sql
WHERE фильтрует строки до группировки.

HAVING фильтрует группы после группировки.
```

#### Пример:

```sql
SELECT user_id, COUNT(*)
FROM orders
WHERE status = 'paid'
GROUP BY user_id
HAVING COUNT(*) > 5;
```

**Здесь**

```sql
WHERE status = 'paid' оставляет только оплаченные заказы;
GROUP BY user_id группирует по пользователю;
HAVING COUNT(*) > 5 оставляет пользователей с количеством заказов больше 5.
```

Чем INNER JOIN отличается от LEFT JOIN?

```sql
INNER JOIN возвращает только совпавшие строки.

LEFT JOIN возвращает все строки из левой таблицы, даже если справа совпадений нет.
```

Чем DELETE отличается от TRUNCATE?

```sql
DELETE удаляет строки, может использовать WHERE.

TRUNCATE быстро очищает всю таблицу, без WHERE.
```

Чем DROP отличается от DELETE?

```sql
DELETE удаляет данные из таблицы.

DROP удаляет саму таблицу.
```

Чем UNION отличается от UNION ALL?

```sql
UNION убирает дубликаты.

UNION ALL не убирает дубликаты и обычно быстрее.
```

#### Что такое индекс?

Индекс — структура данных для ускорения поиска.

**Но он**

занимает место;
замедляет вставку/обновление/удаление;
должен создаваться под конкретные запросы.
Что такое первичный ключ?

PRIMARY KEY — уникальный идентификатор строки.

**Он**

уникален;
не может быть NULL;
часто используется для связей между таблицами.
Что такое внешний ключ?

FOREIGN KEY — ссылка на запись в другой таблице.

Он помогает поддерживать ссылочную целостность.

#### Что такое транзакция?

Транзакция — набор операций, который выполняется целиком или не выполняется вообще.

#### Пример: перевод денег между счетами.

Что такое ACID?
- Atomicity — атомарность;
- Consistency — согласованность;
- Isolation — изолированность;
- Durability — долговечность.
Что такое нормализация?

Процесс проектирования структуры БД, чтобы уменьшить дублирование и повысить целостность данных.

#### Что такое денормализация?

Осознанное добавление избыточности ради ускорения чтения.

#### Что такое оконные функции?

Функции, которые считают значения по группе строк, но не схлопывают результат, в отличие от GROUP BY.

#### Пример:

```sql
ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC)
```

### 54. PostgreSQL-specific полезности

SERIAL / BIGSERIAL

```bash
id SERIAL PRIMARY KEY
id BIGSERIAL PRIMARY KEY
```

Автоинкремент.

В новых версиях PostgreSQL часто рекомендуют стандартный вариант:

```bash
id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
```

RETURNING

```sql
INSERT INTO users (name, email)
VALUES ('Alex', 'alex@test.com')
```

RETURNING id;

Можно использовать и с UPDATE, DELETE.

```sql
UPDATE users
SET is_active = false
WHERE id = 1
```

RETURNING *;
UPSERT — ON CONFLICT

```sql
INSERT INTO users (email, name)
VALUES ('alex@test.com', 'Alex')
```

ON CONFLICT (email)

```sql
DO UPDATE SET name = EXCLUDED.name;
```

Если email уже есть — обновит имя.

Если нет — создаст запись.

**JSONB**

```bash
CREATE TABLE events (
    id BIGSERIAL PRIMARY KEY,
    payload JSONB
);
```

**Запрос**

```sql
SELECT *
FROM events
WHERE payload->>'type' = 'user_created';
```

-> возвращает JSON.

->> возвращает текст.

### 55. Практические паттерны запросов

**Проверка существования записи**

```sql
SELECT EXISTS (
    SELECT 1
    FROM users
    WHERE email = 'alex@test.com'
);
```

**Безопасная проверка перед удалением**

**Сначала**

```sql
SELECT *
FROM users
WHERE email LIKE 'autotest-%';
```

**Потом**

```sql
DELETE FROM users
WHERE email LIKE 'autotest-%';
```

**Найти битые связи**

```sql
SELECT o.*
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;
```

Найти записи с пустыми важными полями

```sql
SELECT *
FROM users
WHERE email IS NULL
```

**OR name IS NULL**

```sql
   OR name = '';
```

**Проверить уникальность**

```sql
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

**Проверить распределение статусов**

```sql
SELECT status, COUNT(*)
FROM orders
GROUP BY status
ORDER BY COUNT(*) DESC;
```

**Найти долгие незавершённые операции**

```sql
SELECT *
FROM operations
WHERE status = 'processing'
```

  AND created_at < NOW() - INTERVAL '1 hour';

### 56. Частые ошибки новичков

Ошибка 1. Использовать = NULL

**Неправильно**

```sql
WHERE field = NULL
```

**Правильно**

```sql
WHERE field IS NULL
```

Ошибка 2. Портить LEFT JOIN через WHERE

#### Плохо:

```sql
SELECT *
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.status = 'paid';
```

**Лучше**

```sql
SELECT *
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
   AND o.status = 'paid';
```

Ошибка 3. Забыть WHERE в UPDATE/DELETE

**Опасно**

```sql
DELETE FROM users;
UPDATE users
SET is_active = false;
```

Ошибка 4. Использовать COUNT(*) после LEFT JOIN

```sql
SELECT u.id, COUNT(*)
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;
```

Пользователь без заказов получит 1.

**Правильно**

```sql
SELECT u.id, COUNT(o.id)
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;
```

Ошибка 5. Неправильная фильтрация по датам

**Потенциально плохо**

```sql
WHERE created_at BETWEEN '2026-07-01' AND '2026-07-31'
```

**Лучше**

```sql
WHERE created_at >= '2026-07-01'
```

  AND created_at <  '2026-08-01'
Ошибка 6. Считать, что индекс всегда ускоряет

Индекс помогает не всегда.

Если запрос возвращает большую часть таблицы, БД может выбрать Seq Scan.

Ошибка 7. Путать WHERE и HAVING

#### Плохо:

```sql
WHERE COUNT(*) > 5
```

**Правильно**

```sql
HAVING COUNT(*) > 5
```

## Собеседование и самопроверка

### 57. Мини-шпаргалка по синтаксису

```sql
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
```

### 58. Как отвечать на собеседовании

Если спрашивают: «Как оптимизировать медленный запрос?»

> **Короткий ответ для собеседования**
>
> Сначала посмотрю план выполнения через EXPLAIN или EXPLAIN ANALYZE. Проверю, используются ли индексы, нет ли полного сканирования большой таблицы, дорогих сортировок или неудачных JOIN. Потом посмотрю условия фильтрации, порядок JOIN, объём данных, селективность, наличие подходящих индексов. Также проверю, не используются ли функции поверх индексируемых колонок и не тянем ли лишние поля через SELECT *.

Если спрашивают: «Как проверить данные после API-запроса?»

> **Короткий ответ для собеседования**
>
> Я бы отправил API-запрос, проверил HTTP-ответ, а затем сходил в БД и проверил фактическое состояние: создана ли запись, корректны ли поля, есть ли связанные записи, не появились ли дубликаты, правильно ли выставлены статусы и timestamps. После теста удалил бы тестовые данные или использовал изолированную тестовую транзакцию/фикстуры.

Если спрашивают: «Что важнее — проверять через API или через БД?»

> **Короткий ответ для собеседования**
>
> Основную бизнес-проверку лучше делать через публичный контракт системы — API. БД я использую дополнительно: для подготовки данных, проверки сайд-эффектов, диагностики, сложных интеграционных сценариев и проверки консистентности. Но тесты не должны чрезмерно завязываться на внутреннюю структуру БД, если она не является частью контракта.

### 59. Что обязательно повторить перед интервью

**Самый важный минимум**

- `SELECT`, `WHERE`, `ORDER BY`, `LIMIT`, `DISTINCT`;
- `JOIN`: `INNER`, `LEFT`, `FULL`;
- `GROUP BY`, `HAVING`;
- `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`;
- `NULL`, `IS NULL`, `COALESCE`;
- подзапросы, `EXISTS`, `NOT EXISTS`, CTE через `WITH`;
- оконные функции: `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LAG`, `LEAD`;
- индексы, транзакции и ACID;
- `DELETE` против `TRUNCATE` и `DROP`;
- первичные и внешние ключи;
- типовые задачи: дубликаты, последняя запись в группе, записи без связи и топ-N.

### 60. Самые частые live-задачи

#### 1. Дубликаты

```sql
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

#### 2. Последняя запись в группе

```sql
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
```

#### 3. Записи без связи

```sql
SELECT u.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

#### 4. Агрегация по группе

```sql
SELECT user_id, SUM(total)
FROM orders
GROUP BY user_id;
```

#### 5. Топ-N по группе

```sql
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
```

### 61. Короткие формулировки для ответа голосом

**JOIN:** нужен, чтобы объединять строки из нескольких таблиц по условию связи.

**LEFT JOIN:** возвращает все строки из левой таблицы и найденные совпадения из правой. Если совпадений нет, справа будут `NULL`.

**GROUP BY:** группирует строки по указанным колонкам, после чего можно применять агрегатные функции.

**HAVING:** фильтрует уже сгруппированные данные, в отличие от `WHERE`, который фильтрует строки до группировки.

**Индекс**

Индекс ускоряет поиск и сортировку, но замедляет запись и занимает место.

**Транзакция**

Транзакция позволяет выполнить несколько операций атомарно: либо все изменения сохраняются, либо все откатываются.

**Оконная функция**

Оконная функция считает значение по группе строк, но не уменьшает количество строк в результате.

**Нормализация**

Нормализация уменьшает дублирование данных и помогает поддерживать целостность.

**EXPLAIN:** показывает, как база планирует выполнить запрос: будет ли использовать индекс, какой `JOIN` выберет, потребуется ли сортировка или полный проход по таблице.

### 62. Мини-набор запросов, который надо уметь писать с закрытыми глазами

```sql
SELECT *
FROM users
WHERE email = 'test@example.com';
SELECT status, COUNT(*)
FROM orders
GROUP BY status;
SELECT user_id, SUM(total)
FROM orders
GROUP BY user_id
HAVING SUM(total) > 10000;
SELECT u.name, o.total
FROM users u
JOIN orders o ON u.id = o.user_id;
SELECT u.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
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
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'test@example.com';
```

### 63. Что особенно важно для Senior QA Automation

Для твоего профиля лучше сделать акцент не только на синтаксисе, а на практическом применении SQL:

**Нужно уверенно уметь**
- проверить данные после API-запроса;
- подготовить тестовые данные;
- почистить тестовые данные;
- проверить связи между таблицами;
- найти дубликаты;
- проверить статусы бизнес-операций;
- проверить миграции;
- понимать индексы на базовом уровне;
- читать простой EXPLAIN;
- понимать транзакции;
- не ломать данные опасными UPDATE/DELETE;
- понимать, почему LEFT JOIN может внезапно превратиться в INNER JOIN;
- писать запросы с GROUP BY, HAVING, оконными функциями.

### 64. Быстрая самопроверка

Если можешь без подсказки написать эти запросы — для большинства QA/SDET собеседований по SQL база уже нормальная:

Найти пользователя по email.
Найти пользователей без заказов.
Посчитать заказы по пользователю.
Найти пользователей с количеством заказов больше 5.
Найти дубликаты email.
Найти последний заказ каждого пользователя.
Найти топ-3 заказа каждого пользователя.
Посчитать выручку по дням.
Найти заказы без пользователя.
Объяснить WHERE vs HAVING.
Объяснить INNER JOIN vs LEFT JOIN.
Объяснить DELETE vs TRUNCATE vs DROP.
Объяснить, зачем индекс.
Объяснить транзакцию и ACID.
Объяснить, что делает EXPLAIN.

Следующий логичный шаг — прорешать 30–50 SQL-задач в формате собеседования: сначала простые JOIN/GROUP BY, потом оконные функции и задачи на поиск ошибок в запросах.
