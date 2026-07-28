# SQL — быстрый повтор — часть 2

[← Оглавление](sql.md) · [← Все шпаргалки](README.md) · [Подробный раздел SQL](../questions/databases-sql.md)

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

