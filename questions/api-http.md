# API и HTTP

HTTP, REST, контрактное тестирование, авторизация, очереди сообщений и тестирование интеграций.

[← На главную](../README.md)

## Материалы

1. [API-автоматизация на Python](../materials/api-testing.md) — `requests`, API-клиенты, контракты, авторизация, данные, retry, polling и практические задания.
2. [Сети и HTTP для QA/SDET](../materials/networks.md) — цельный конспект из 135 тем.
3. [Брокеры сообщений](../materials/message-brokers.md) — цельный конспект из 43 тем.

Короткий повтор:

- [API-автоматизация](../cheatsheets/api-testing.md);
- [Сети и HTTP](../cheatsheets/networks.md);
- [Брокеры сообщений](../cheatsheets/message-brokers.md).

### API-автоматизация

- устройство request и response;
- библиотека `requests`, `Session`, timeout и исключения;
- `params`, `json`, `data`, headers и cookies;
- API client layer и структура pytest-проекта;
- positive, negative и CRUD-сценарии;
- авторизация, роли и проверка чужих ресурсов;
- JSON Schema, Pydantic, OpenAPI и контрактные тесты;
- пагинация, файлы, rate limit и retry;
- polling, eventual consistency, webhooks и моки;
- изоляция, параллельность, cleanup и диагностика.

### Сети и HTTP

- OSI и TCP/IP;
- IP, порты, DNS, TCP и UDP;
- HTTP, HTTPS, TLS и сертификаты;
- cookies, авторизация, прокси и балансировка;
- Docker- и Kubernetes-сети;
- таймауты, ретраи и сетевые сбои;
- команды диагностики и практические интервью-сценарии.

### Брокеры сообщений

- очереди и publish/subscribe;
- Kafka, RabbitMQ, NATS и Redis;
- delivery semantics, ack, retry и DLQ;
- идемпотентность, порядок и eventual consistency;
- тестирование producers, consumers и схем сообщений;
- Testcontainers, моки и диагностика асинхронных тестов.

## Как изучать

1. Если HTTP пока непонятен, начать с [сетевого конспекта](../materials/networks.md).
2. Затем пройти [API-автоматизацию](../materials/api-testing.md) и написать примеры самостоятельно.
3. После этого изучить [брокеры сообщений](../materials/message-brokers.md).
4. Перед собеседованием повторить короткие шпаргалки.
