# Брокеры сообщений — быстрый повтор — часть 3

[← Оглавление](message-brokers.md) · [← Все шпаргалки](README.md) · [Подробный раздел](../questions/api-http/message-brokers.md)

## Частые ошибки тестов

- `sleep()` вместо ожидания;
- чтение чужого сообщения;
- подтверждение сообщения до проверки;
- отсутствие проверки дубликатов;
- бесконечный retry;
- один consumer group для параллельных тестов;
- проверка только факта публикации без бизнес-эффекта;
- очистка queue, которой пользуются другие тесты.

## Официальная документация

- [Apache Kafka](https://kafka.apache.org/documentation/)
- [RabbitMQ queues and acknowledgements](https://www.rabbitmq.com/docs/queues)
