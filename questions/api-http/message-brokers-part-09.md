# Брокеры сообщений — полный конспект — часть 9

[← Оглавление](message-brokers.md) · [← К разделу](../api-http.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/message-brokers.md)

Темы 31-36.

## 31. Saga pattern

Saga используется для распределённых бизнес-процессов.  

Например заказ:  

1. Создать заказ  
2. Списать оплату  
3. Зарезервировать товар  
4. Создать доставку  

Если шаг 3 упал, нужно откатить предыдущие шаги.  

Но в микросервисах нет одной общей транзакции.  

Поэтому используют saga.  

Choreography  

Сервисы реагируют на события друг друга.  

OrderCreated -> Payment Service  
PaymentCompleted -> Inventory Service  
ProductReserved -> Delivery Service  

### Плюсы:

слабая связность.  

### Минусы:

сложнее понять весь процесс;  
логика размазана по сервисам.  
Orchestration  

Есть центральный orchestrator.  

Order Orchestrator:  
  create order  
  call payment  
  call inventory  
  call delivery  

### Плюсы:

проще видеть процесс.  

### Минусы:

orchestrator становится центральной точкой логики.  

## 32. Request/reply через брокер

Иногда через брокер делают запрос-ответ.  

Например:  

Service A отправляет request message  
Service B отвечает в reply queue  

Для связи используют:  

correlation_id;  
reply_to.  

### Пример:

{  
  "correlation_id": "abc-123",  
  "reply_to": "service-a.reply",  
  "payload": {  
    "user_id": 42  
  }  
}  

Но если нужен простой быстрый запрос-ответ, часто проще использовать HTTP/gRPC.  

## 33. Мониторинг брокеров

### Что мониторить:

Для Kafka  
consumer lag;  
throughput;  
producer error rate;  
consumer error rate;  
under-replicated partitions;  
offline partitions;  
ISR shrink/expand;  
disk usage;  
request latency;  
rebalance count;  
broker availability.  
Для RabbitMQ  
queue length;  
ready messages;  
unacked messages;  
consumers count;  
publish rate;  
deliver rate;  
ack rate;  
redelivered messages;  
DLQ size;  
memory usage;  
disk usage;  
connection count;  
channel count.  
Для consumers  
время обработки сообщения;  
количество успешных обработок;  
количество ошибок;  
retry count;  
DLQ count;  
latency от создания события до обработки;  
idempotency duplicate count.  

## 34. Логирование и трассировка

В сообщениях полезно иметь:  

correlation_id;  
trace_id;  
event_id;  
causation_id.  
event_id  

Идентификатор конкретного события.  

correlation_id  

Объединяет цепочку действий.  

Например:  

Пользователь нажал "Оплатить"  
API создал request  
Payment отправил event  
Notification отправил email  

У всех этих действий может быть один correlation_id.  

causation_id  

Показывает, какое событие стало причиной текущего.  

## 35. Безопасность

### Что важно:

TLS между сервисами и брокером;  
аутентификация;  
авторизация на topic/queue;  
нельзя всем давать права на всё;  
не логировать чувствительные данные;  
шифровать секреты;  
ограничивать размер сообщений;  
маскировать персональные данные;  
контролировать доступ к DLQ, потому что там могут быть реальные payload-ы.  

## 36. Размер сообщения

Большие сообщения через брокер — плохая практика.  

### Плохо:

Отправлять PDF на 50 MB прямо в Kafka/RabbitMQ  

Лучше:  

Сохранить файл в S3/MinIO/файловое хранилище  
В брокер отправить ссылку/id файла  

### Пример:

{  
  "document_id": "doc-123",  
  "storage_url": "s3://bucket/doc-123.pdf"  
}  

