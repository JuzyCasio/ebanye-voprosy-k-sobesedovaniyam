# Брокеры сообщений — полный конспект — часть 6

[← Оглавление](message-brokers.md) · [← К разделу](../api-http.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/message-brokers.md)

Темы 18-22.

## 18. RabbitMQ — главное к собеседованию

RabbitMQ — брокер сообщений, часто используется для очередей задач и маршрутизации.  

### Основные сущности:

Producer -> Exchange -> Queue -> Consumer  
Exchange  

Producer отправляет сообщение в exchange.  

Exchange решает, в какие очереди положить сообщение.  

Queue  

Очередь хранит сообщения до обработки consumer-ом.  

Binding  

Binding связывает exchange и queue.  

Exchange orders  
  binding key orders.created -> Queue created_orders  
Routing key  

Ключ маршрутизации.  

Например:  

orders.created  
orders.paid  
orders.cancelled  
Direct exchange  

Сообщение попадает в очередь, если routing key совпал точно.  

routing key = orders.created  
Fanout exchange  

Сообщение рассылается во все очереди, связанные с exchange.  

Используется для broadcast.  

Topic exchange  

Маршрутизация по шаблонам.  

Например:  

orders.*  
orders.created  
orders.#  

### Пример:

orders.*  -> orders.created, orders.paid  
orders.#  -> orders.created.eu.spb  
Headers exchange  

Маршрутизация по headers, а не по routing key.  

Используется реже.  

Durable queue  

Durable queue переживает перезапуск RabbitMQ.  

Но важно:  

Durable queue сама по себе не гарантирует сохранность сообщения. Сообщение тоже должно быть persistent.  

Persistent message  

Persistent message сохраняется на диск.  

Если брокер перезапустится, такое сообщение может быть восстановлено.  

Prefetch  

Prefetch — сколько сообщений RabbitMQ может выдать consumer-у без ack.  

Например:  

prefetch = 10  

Consumer может получить 10 сообщений, но пока не подтвердит их, новые сверх лимита не получит.  

Это помогает:  

не перегружать consumer;  
равномерно распределять сообщения;  
контролировать память.  
RabbitMQ ack  

Consumer подтверждает обработку через ack.  

Если consumer умер без ack, RabbitMQ может вернуть сообщение в очередь.  

## 19. NATS — что знать кратко

NATS — лёгкий и быстрый брокер/система messaging.  

### Часто используется для:

microservices;  
pub/sub;  
request/reply;  
lightweight messaging.  

Базовый NATS — быстрый, но проще по гарантиям.  

Для хранения и более надёжной доставки используется JetStream.  

NATS особенности  
низкая latency;  
простой pub/sub;  
request/reply из коробки;  
хорошо подходит для микросервисного общения;  
JetStream добавляет persistence, replay, durable consumers.  

## 20. Redis как брокер

Redis может использоваться как брокер через:  

Redis Pub/Sub;  
Redis Streams;  
Celery broker;  
lists.  

Но Redis Pub/Sub сам по себе не очень надёжный:  

Если subscriber был отключён, сообщение может быть потеряно.  

Redis Streams надёжнее, потому что сообщения хранятся в stream и могут читаться consumer groups.  

## 21. Kafka vs RabbitMQ

Очень частый вопрос.  

Критерий	Kafka	RabbitMQ  
Модель	distributed log	message queue  
Хранение	сообщения хранятся по retention	обычно удаляются после ack  
Масштабирование	partitions	queues/consumers  
Порядок	внутри partition	внутри queue, но может нарушаться при параллелизме  
Replay	удобно перечитывать события	не основной сценарий  
Use case	event streaming, analytics, event sourcing	task queues, routing, commands  
Routing	проще, через topics/keys	мощная маршрутизация через exchanges  
Consumer state	offset	ack состояния очереди  
Подходит для	больших потоков событий	задач, workflow, RPC-like patterns  
Как ответить на собеседовании  

Kafka лучше подходит для потоков событий, когда важны высокая пропускная способность, хранение истории, replay и обработка большими объёмами. RabbitMQ чаще используют как классический брокер очередей задач, когда важна гибкая маршрутизация, ack/nack, retry, DLQ и распределение задач между consumers.  

## 22. Kafka vs RabbitMQ на примерах

Когда выбрать Kafka  

### Подходит:

поток событий заказов;  
аналитика;  
логи;  
аудит;  
event sourcing;  
интеграция многих consumers;  
возможность перечитать историю;  
high throughput.  

### Пример:

Все события пользователей отправляются в Kafka.  
Их читают аналитика, антифрод, рекомендации, BI.  
Когда выбрать RabbitMQ  

### Подходит:

фоновые задачи;  
отправка email;  
генерация отчётов;  
обработка файлов;  
команды между сервисами;  
сложная маршрутизация.  

### Пример:

Пользователь загрузил документ.  
Задача попала в очередь.  
Worker забрал документ и обработал его.  

