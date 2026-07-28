# Брокеры сообщений — полный конспект — часть 1

[← Оглавление](message-brokers.md) · [← К разделу](../api-http.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/message-brokers.md)

Темы 1-3.

## 1. Что такое брокер сообщений

Брокер сообщений — это промежуточный сервис между отправителем и получателем сообщений.  

Вместо того чтобы один сервис напрямую вызывал другой, он отправляет сообщение в брокер, а другой сервис забирает его позже.  

### Пример:

Сервис заказов  --->  Брокер  --->  Сервис оплаты  
Producer              Broker       Consumer  
Зачем нужен брокер  

Брокер помогает:  

Развязать сервисы  
producer не обязан знать, кто именно обработает сообщение;  
consumer может быть временно недоступен.  
Сгладить нагрузку  
если сообщений пришло много, они могут накопиться в очереди;  
consumers обработают их постепенно.  
Повысить отказоустойчивость  
если сервис-получатель упал, сообщения не теряются сразу.  
Организовать асинхронную обработку  
пользователь сделал заказ;  
заказ создан сразу;  
письмо, списание бонусов, уведомления выполняются позже.  
Масштабировать обработку  
можно добавить несколько consumers.  

## 2. Основные термины

Message  

Message — сообщение, которое передаётся через брокер.  

Обычно содержит:  

{  
  "event_id": "123",  
  "event_type": "order_created",  
  "created_at": "2026-07-08T10:00:00Z",  
  "payload": {  
    "order_id": 777,  
    "user_id": 42,  
    "amount": 1500  
  }  
}  

В сообщении часто есть:  

id — уникальный идентификатор;  
type — тип события;  
payload — полезная нагрузка;  
timestamp — время создания;  
headers — метаданные;  
correlation_id / trace_id — для трассировки.  
Producer  

Producer — сервис, который отправляет сообщение.  

### Пример:

Order Service создал заказ и отправил событие order_created  
Consumer  

Consumer — сервис, который получает и обрабатывает сообщение.  

### Пример:

Notification Service получил order_created и отправил email  
Broker  

Broker — сервер или кластер, который принимает, хранит и отдаёт сообщения.  

### Примеры брокеров:

Kafka;  
RabbitMQ;  
NATS;  
Redis Streams;  
ActiveMQ;  
Pulsar;  
SQS.  
Queue  

Queue — очередь сообщений.  

Обычно сообщение из очереди получает один consumer.  

Queue:  
[message1] [message2] [message3]  

Consumer A забрал message1  
Consumer B забрал message2  

Очередь часто используется, когда задачу должен обработать один исполнитель.  

### Пример:

Очередь задач на генерацию PDF  
Topic  

Topic — логический канал сообщений.  

Чаще используется в Kafka/NATS.  

Topic: orders.created  

В topic producer публикует сообщения, а consumers читают их.  

Exchange  

В RabbitMQ producer обычно отправляет сообщение не напрямую в очередь, а в exchange.  

Exchange решает, в какую очередь отправить сообщение.  

Типы exchange:  

Тип	Как работает  
direct	отправляет по точному routing key  
fanout	рассылает во все связанные очереди  
topic	маршрутизация по шаблону  
headers	маршрутизация по headers  

### Пример:

Producer -> Exchange -> Queue -> Consumer  
Routing key  

Routing key — ключ маршрутизации в RabbitMQ.  

Например:  

orders.created  
orders.paid  
orders.cancelled  

Exchange может по нему понять, в какую очередь положить сообщение.  

Partition  

В Kafka topic делится на partitions.  

Topic orders  

Partition 0: msg1 msg4 msg7  
Partition 1: msg2 msg5 msg8  
Partition 2: msg3 msg6 msg9  

Partition нужна для:  

параллельной обработки;  
масштабирования;  
хранения порядка сообщений внутри partition.  

### Важно:

В Kafka порядок гарантируется только внутри одной partition, а не во всём topic.  

Offset  

Offset — номер сообщения внутри partition в Kafka.  

Partition 0:  
offset 0 -> msg A  
offset 1 -> msg B  
offset 2 -> msg C  

Consumer хранит offset, чтобы понимать, какие сообщения уже обработал.  

Consumer group  

Consumer group — группа consumers, которые совместно читают topic.  

В Kafka:  

Topic orders имеет 3 partition  

Consumer Group: payment-service  

Consumer 1 читает partition 0  
Consumer 2 читает partition 1  
Consumer 3 читает partition 2  

### Важное правило Kafka:

В рамках одной consumer group одну partition в один момент времени читает только один consumer.  

Если consumers больше, чем partitions, лишние consumers будут простаивать.  

## 3. Очередь vs Publish/Subscribe

Queue / Point-to-Point  

Сообщение обрабатывается одним consumer.  

Producer -> Queue -> Consumer 1  
                  -> Consumer 2  
                  -> Consumer 3  

Каждое сообщение заберёт только один consumer.  

### Пример:

Очередь задач на обработку изображений  
Pub/Sub  

Одно событие получают несколько независимых подписчиков.  

Order Created Event  

-> Billing Service  
-> Notification Service  
-> Analytics Service  
-> Delivery Service  

### Пример:

Создан заказ.  
Один сервис отправляет email.  
Второй обновляет аналитику.  
Третий резервирует товар.  

