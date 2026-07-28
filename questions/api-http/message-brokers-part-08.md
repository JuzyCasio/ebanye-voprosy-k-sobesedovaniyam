# Брокеры сообщений — полный конспект — часть 8

[← Оглавление](message-brokers.md) · [← К разделу](../api-http.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/message-brokers.md)

Темы 26-30.

## 26. Как тестировать асинхронность в автотестах

### Главная ошибка — сразу проверять результат.

### Плохо:

send_message()  
assert get_status() == "DONE"  

Потому что consumer может ещё не успеть обработать сообщение.  

Лучше:  

send_message()  
wait_until(lambda: get_status() == "DONE", timeout=10)  
Пример wait_until на Python  
from collections.abc import Callable  
from time import sleep, monotonic  
from typing import TypeVar  

T = TypeVar("T")  

def wait_until(  
    condition: Callable[[], T],  
    timeout: float = 10.0,  
    interval: float = 0.5,  
) -> T:  
    deadline = monotonic() + timeout  
    last_result: T | None = None  

    while monotonic() < deadline:  
        last_result = condition()  
        if last_result:  
            return last_result  
        sleep(interval)  

    raise TimeoutError(f"Condition was not met within {timeout} seconds")  

Использование:  

result = wait_until(  
    lambda: order_client.get_order(order_id)["status"] == "PAID",  
    timeout=15,  
    interval=1,  
)  

## 27. Как писать автотесты с брокером

Есть несколько подходов.  

1. Интеграционный тест с реальным брокером  

Например, поднимаем Kafka/RabbitMQ в Docker.  

### Плюсы:

максимально близко к реальности;  
проверяется реальная интеграция.  

### Минусы:

тесты медленнее;  
сложнее поддерживать;  
нужна очистка данных.  
2. Testcontainers  

Можно поднимать брокер на время тестов.  

### Примерно:

pytest запускает Kafka container  
тест публикует сообщение  
consumer обрабатывает  
тест проверяет результат  
container удаляется  

### Плюсы:

изоляция;  
реальный брокер;  
удобно для CI.  

### Минусы:

требует Docker;  
медленнее unit-тестов.  
3. Mock/Fake broker  

Используется для unit-тестов.  

### Плюсы:

быстро;  
просто;  
можно проверить бизнес-логику consumer-а.  

### Минусы:

не проверяется настоящая интеграция с брокером;  
можно не поймать проблемы routing/ack/serialization.  
4. Contract testing  

Проверяем, что producer и consumer договорились о формате сообщения.  

Например:  

{  
  "event_id": "string",  
  "event_type": "order_created",  
  "payload": {  
    "order_id": "integer",  
    "amount": "number"  
  }  
}  

Проверяем:  

обязательные поля;  
типы;  
версии схем;  
совместимость изменений.  

## 28. Что важно в тестовых данных

Для сообщений нужны:  

уникальный event_id;  
уникальный correlation_id;  
понятный event_type;  
валидный payload;  
версия схемы;  
timestamp;  
business key, например order_id.  

### Пример хорошего сообщения:

{  
  "event_id": "evt-1001",  
  "event_type": "order_created",  
  "schema_version": 1,  
  "correlation_id": "corr-777",  
  "created_at": "2026-07-08T10:00:00Z",  
  "payload": {  
    "order_id": 777,  
    "user_id": 42,  
    "amount": 1500  
  }  
}  

## 29. Schema evolution

Со временем формат сообщений меняется.  

Например, было:  

{  
  "order_id": 777,  
  "amount": 1500  
}  

Стало:  

{  
  "order_id": 777,  
  "amount": 1500,  
  "currency": "RUB"  
}  
Backward compatibility  

Новая версия consumer-а должна уметь читать старые сообщения.  

Forward compatibility  

Старый consumer должен не падать, если появилось новое поле.  

### Что нельзя делать без осторожности

Опасные изменения:  

удалить обязательное поле;  
изменить тип поля;  
переименовать поле;  
поменять семантику поля;  
начать отправлять null, если consumer не готов.  
Хороший ответ  

При изменении схемы сообщений нужно думать о совместимости. Безопасно добавлять необязательные поля. Опасно удалять поля или менять их тип. Для контроля можно использовать schema registry, версионирование сообщений и contract tests.  

## 30. Transactional outbox

Очень важный паттерн.  

### Проблема:

Сервис создал заказ в БД.  
Потом должен отправить событие в брокер.  
Но между записью в БД и отправкой события сервис упал.  

Получили:  

Заказ в БД есть.  
События order_created нет.  
Другие сервисы не узнают о заказе.  
Решение: Outbox  

В одной транзакции сохраняем:  

бизнес-данные;  
событие в outbox-таблицу.  
BEGIN  
  INSERT INTO orders ...  
  INSERT INTO outbox_events ...  
COMMIT  

Потом отдельный процесс читает outbox и отправляет события в брокер.  

Пример  
orders table:  
id | status  

outbox_events table:  
event_id | event_type | payload | status  
Зачем это нужно  

Outbox помогает не потерять событие между БД и брокером.  

### Хороший ответ:

Transactional outbox нужен, чтобы атомарно сохранить изменение бизнес-сущности и событие, которое потом будет отправлено в брокер. Так мы избегаем ситуации, когда данные в БД уже изменились, а событие не было опубликовано из-за падения сервиса.  

