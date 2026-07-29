# SQL — быстрый повтор

[← Все шпаргалки](README.md) · [Полный материал](../materials/sql.md)

## Что такое SQL

SQL — язык для работы с реляционными базами данных: чтения, изменения и определения структуры данных и управления доступом.

## Группы команд

| Группа | Назначение | Примеры |
|---|---|---|
| DQL | чтение | `SELECT` |
| DML | изменение данных | `INSERT`, `UPDATE`, `DELETE` |
| DDL | структура БД | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| TCL | транзакции | `COMMIT`, `ROLLBACK`, `SAVEPOINT` |
| DCL | права | `GRANT`, `REVOKE` |

## Базовый SELECT

```sql
SELECT id, email
FROM users
WHERE status = 'active'
ORDER BY created_at DESC
LIMIT 20;
```

Логический порядок:

1. `FROM` и `JOIN`;
2. `WHERE`;
3. `GROUP BY`;
4. `HAVING`;
5. `SELECT`;
6. `DISTINCT`;
7. `ORDER BY`;
8. `LIMIT` и `OFFSET`.

## Фильтрация

```sql
WHERE status IN ('new', 'paid')
  AND amount BETWEEN 100 AND 500
  AND email ILIKE '%@example.com'
```

### NULL

`NULL` означает отсутствие известного значения.

```sql
WHERE deleted_at IS NULL
WHERE deleted_at IS NOT NULL
```

Нельзя использовать `= NULL`. Большинство сравнений с `NULL` дают `UNKNOWN`.

```sql
SELECT COALESCE(display_name, email)
FROM users;
```

## Агрегации

```sql
SELECT user_id,
       COUNT(*) AS orders_count,
       SUM(amount) AS total_amount,
       AVG(amount) AS average_amount
FROM orders
WHERE status = 'paid'
GROUP BY user_id
HAVING COUNT(*) >= 3;
```

- `WHERE` фильтрует строки до группировки;
- `HAVING` фильтрует группы после `GROUP BY`;
- `COUNT(*)` считает строки;
- `COUNT(column)` не считает `NULL` в колонке.

## JOIN

| JOIN | Результат |
|---|---|
| `INNER JOIN` | только совпавшие строки |
| `LEFT JOIN` | все строки слева и совпадения справа |
| `RIGHT JOIN` | все строки справа |
| `FULL JOIN` | все строки обеих таблиц |
| `CROSS JOIN` | декартово произведение |

```sql
SELECT u.id, u.email, o.id AS order_id
FROM users AS u
LEFT JOIN orders AS o
  ON o.user_id = u.id;
```

### Найти записи без связи

```sql
SELECT u.*
FROM users AS u
LEFT JOIN orders AS o
  ON o.user_id = u.id
WHERE o.id IS NULL;
```

Альтернатива:

```sql
SELECT u.*
FROM users AS u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.user_id = u.id
);
```

### Ловушка LEFT JOIN

Условие по правой таблице в `WHERE` может фактически превратить `LEFT JOIN` в `INNER JOIN`.

```sql
-- Сохранит пользователей без оплаченных заказов
LEFT JOIN orders AS o
  ON o.user_id = u.id
 AND o.status = 'paid'
```

## Подзапрос и CTE

```sql
WITH paid_orders AS (
    SELECT *
    FROM orders
    WHERE status = 'paid'
)
SELECT user_id, COUNT(*)
FROM paid_orders
GROUP BY user_id;
```

CTE улучшает читаемость и позволяет ссылаться на промежуточный результат. Рекурсивный CTE используют для деревьев и иерархий.

## Операции над результатами

- `UNION` объединяет и удаляет дубликаты;
- `UNION ALL` объединяет без удаления дубликатов и обычно быстрее;
- `INTERSECT` оставляет общие строки;
- `EXCEPT` оставляет строки первого запроса, которых нет во втором.

Количество и совместимые типы колонок должны совпадать.

## Оконные функции

Оконная функция вычисляет значение по группе строк, не объединяя их в одну строку.

```sql
SELECT user_id,
       amount,
       ROW_NUMBER() OVER (
           PARTITION BY user_id
           ORDER BY amount DESC
       ) AS position
FROM orders;
```

Частые функции:

- `ROW_NUMBER()`;
- `RANK()` и `DENSE_RANK()`;
- `LAG()` и `LEAD()`;
- `SUM() OVER (...)`;
- `COUNT() OVER (...)`.

### Последняя запись в каждой группе

```sql
WITH ranked AS (
    SELECT o.*,
           ROW_NUMBER() OVER (
               PARTITION BY user_id
               ORDER BY created_at DESC
           ) AS rn
    FROM orders AS o
)
SELECT *
FROM ranked
WHERE rn = 1;
```

## Изменение данных

```sql
INSERT INTO users (email, status)
VALUES ('user@example.com', 'active')
RETURNING id;
```

```sql
UPDATE users
SET status = 'blocked'
WHERE id = 42;
```

```sql
DELETE FROM users
WHERE id = 42;
```

Перед `UPDATE` или `DELETE` полезно сначала выполнить `SELECT` с тем же `WHERE`.

## DELETE, TRUNCATE и DROP

- `DELETE` удаляет выбранные строки и поддерживает `WHERE`;
- `TRUNCATE` быстро очищает таблицу целиком;
- `DROP` удаляет сам объект БД.

Поведение блокировок, транзакций и sequence зависит от СУБД, поэтому его нужно уточнять.

## Ограничения

- `PRIMARY KEY` — уникальный идентификатор строки;
- `FOREIGN KEY` — ссылка на другую таблицу;
- `UNIQUE` — запрет дубликатов;
- `NOT NULL` — значение обязательно;
- `CHECK` — проверка условия;
- `DEFAULT` — значение по умолчанию.

## Индексы

Индекс ускоряет поиск и сортировку, но:

- занимает место;
- замедляет `INSERT`, `UPDATE` и `DELETE`;
- не гарантирует использование;
- порядок колонок в составном индексе важен.

Индексы полезны для колонок из `WHERE`, `JOIN` и `ORDER BY`, если запрос достаточно избирателен.

```sql
CREATE INDEX idx_orders_user_created
ON orders (user_id, created_at DESC);
```

## EXPLAIN

```sql
EXPLAIN
SELECT *
FROM orders
WHERE user_id = 42;
```

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE user_id = 42;
```

`EXPLAIN` показывает план. `EXPLAIN ANALYZE` реально выполняет запрос, поэтому с изменяющими запросами нужна осторожность.

Смотреть:

- `Seq Scan` и `Index Scan`;
- estimated и actual rows;
- стоимость;
- сортировки;
- loops;
- время выполнения.

## Транзакции и ACID

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

- Atomicity — всё или ничего;
- Consistency — ограничения данных сохраняются;
- Isolation — параллельные транзакции не должны недопустимо мешать друг другу;
- Durability — зафиксированные изменения переживают сбой.

## Уровни изоляции

| Уровень | Основная идея |
|---|---|
| Read Uncommitted | допускает самые слабые гарантии |
| Read Committed | не читает незакоммиченные данные |
| Repeatable Read | повторное чтение видит согласованный снимок |
| Serializable | результат как при последовательном выполнении |

Чем сильнее изоляция, тем больше возможны ожидания, конфликты и повторы транзакций. Точное поведение зависит от СУБД.

## Блокировки и MVCC

- блокировки защищают строки или объекты от конфликтующих операций;
- deadlock возникает, когда транзакции ждут друг друга;
- СУБД обнаруживает deadlock и отменяет одну транзакцию;
- MVCC хранит версии строк, чтобы чтение меньше блокировало запись.

## SQL Injection

Нельзя собирать SQL из пользовательского ввода:

```python
# Плохо
query = f"SELECT * FROM users WHERE email = '{email}'"
```

Нужны параметры драйвера:

```python
cursor.execute(
    "SELECT * FROM users WHERE email = %s",
    (email,),
)
```

## Что проверяет AQA/SDET

- данные записались корректно;
- ограничения и связи работают;
- транзакция откатывается при ошибке;
- миграция применима и обратима согласно стратегии проекта;
- старые данные совместимы с новой схемой;
- запросы не создают дубликаты;
- время и timezone записываются ожидаемо;
- параллельные операции не нарушают инварианты;
- чувствительные данные не попадают в логи.

Нельзя запускать тесты записи на production и нельзя строить тесты, зависящие от порядка строк без `ORDER BY`.

## Официальная документация

- [PostgreSQL SELECT](https://www.postgresql.org/docs/current/sql-select.html)
- [PostgreSQL Window Functions](https://www.postgresql.org/docs/current/tutorial-window.html)
