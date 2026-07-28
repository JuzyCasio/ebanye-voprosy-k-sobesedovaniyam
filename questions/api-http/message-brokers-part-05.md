# Брокеры сообщений — полный конспект — часть 5

[← Оглавление](message-brokers.md) · [← К разделу](../api-http.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/message-brokers.md)

Темы 17.

## 17. Kafka — главное к собеседованию

Kafka — распределённая платформа для потоковой обработки событий.  

### Основные сущности:

Producer -> Topic -> Partition -> Consumer Group -> Consumer  
Kafka topic  

Topic — канал сообщений.  

### Пример:

orders  
payments  
users  
notifications  
Kafka partition  

Topic делится на partitions.  

Зачем:  

параллелизм;  
масштабирование;  
распределённое хранение;  
порядок внутри partition.  
Kafka offset  

Offset — позиция сообщения в partition.  

Consumer запоминает, до какого offset он дочитал.  

Kafka consumer group  

Consumer group позволяет нескольким consumers читать topic параллельно.  

Правило:  

Одна partition в рамках одной group назначается только одному consumer-у.  

### Пример:

Topic orders: 3 partitions  

Consumer group A:  
Consumer 1 -> partition 0  
Consumer 2 -> partition 1  
Consumer 3 -> partition 2  

Если добавить четвёртого consumer-а, он будет простаивать, потому что partitions только 3.  

Kafka key  

Key нужен, чтобы управлять попаданием сообщения в partition.  

key = order_id  

Все сообщения с одним key обычно попадают в одну partition.  

Это полезно для порядка событий по одной сущности.  

Kafka replication  

Kafka хранит partitions с репликацией.  

Partition 0:  
leader replica  
follower replica  
follower replica  

Producer и consumer работают с leader replica.  

Followers копируют данные.  

Если leader падает, Kafka выбирает нового leader из replicas.  

Kafka ISR  

ISR — in-sync replicas.  

Это реплики, которые успевают синхронизироваться с leader.  

Если replica отстала, она может быть исключена из ISR.  

Kafka acks  

Producer может ждать разные уровни подтверждения.  

acks=0  

Producer не ждёт подтверждения.  

Быстро, но ненадёжно.  

acks=1  

Producer ждёт подтверждение от leader.  

Если leader подтвердил, producer считает сообщение записанным.  

acks=all  

Producer ждёт подтверждения от всех ISR.  

Надёжнее, но медленнее.  

Kafka retries  

Producer может повторять отправку при ошибках.  

Но при retry возможны дубли, если не настроить идемпотентность.  

Idempotent producer  

Idempotent producer помогает избежать дублей при повторной отправке producer-ом.  

Но это не отменяет необходимость идемпотентной бизнес-логики на стороне consumer-а.  

Kafka rebalance  

Rebalance — перераспределение partitions между consumers.  

Происходит, когда:  

consumer добавился;  
consumer упал;  
изменилось количество partitions;  
consumer слишком долго не отправлял heartbeat.  

Во время rebalance обработка может временно останавливаться.  

Kafka commit offset  

Consumer должен фиксировать offset.  

Варианты:  

Auto commit  

Offset коммитится автоматически.  

Проще, но есть риск потерь или дублей.  

Manual commit  

Consumer сам решает, когда коммитить offset.  

Надёжнее.  

Правильный подход:  

1. Получил сообщение  
2. Обработал  
3. Сохранил результат  
4. Закоммитил offset  

Если закоммитить offset до обработки, можно потерять сообщение.  

