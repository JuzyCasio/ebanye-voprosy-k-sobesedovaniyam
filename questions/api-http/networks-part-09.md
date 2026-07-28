# Сети и HTTP для QA/SDET — полный конспект — часть 9

[← Оглавление](networks.md) · [← К разделу](../api-http.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/networks.md)

Темы 81-90.

## 81. Docker ports

ports:  
  - "8080:80"  

Значит:  

8080 на хосте -> 80 внутри контейнера  

### Проверка:

curl http://localhost:8080  
expose vs ports  

ports публикует порт наружу на хост.  

expose только документирует/открывает порт для других контейнеров в сети, но не публикует на хост.  

## 82. Kubernetes networking — минимум для собеса

Даже если не требуют Kubernetes, базу полезно знать.  

Pod  

Pod имеет свой IP.  

Service  

Service дает стабильный адрес для группы pod'ов.  

Типы:  

Тип	Назначение  
ClusterIP	доступ только внутри кластера  
NodePort	доступ через порт на ноде  
LoadBalancer	внешний балансировщик  
ExternalName	alias на внешний DNS  
Ingress  

Ingress управляет входящим HTTP/HTTPS трафиком.  

Client -> Ingress -> Service -> Pod  
На собесе  

В Kubernetes pod'ы могут пересоздаваться и менять IP, поэтому для стабильного доступа используют Service. Для входящего HTTP/HTTPS трафика часто используют Ingress.  

## 83. Базы данных и сеть

БД — это тоже сетевой сервис.  

### Примеры:

PostgreSQL: 5432  
MySQL: 3306  
Redis: 6379  
MongoDB: 27017  
Частые проблемы  
порт закрыт;  
БД слушает только localhost;  
firewall блокирует;  
неверный host;  
неверный user/password;  
TLS required;  
max connections exceeded;  
DNS не резолвится;  
container localhost problem.  
PostgreSQL listen_addresses  

В PostgreSQL есть настройка:  

listen_addresses  

Если стоит:  

localhost  

то подключиться можно только локально.  

Если:  

*  

то БД слушает внешние интерфейсы.  

Еще есть pg_hba.conf, который управляет доступом.  

## 84. Брокеры и сеть

Kafka  

### Частый порт:

9092  

Важная проблема Kafka:  

advertised.listeners  

Если Kafka отдает клиенту неправильный адрес брокера, клиент может подключиться сначала, но потом упасть при попытке работать с metadata.  

RabbitMQ  

Порты:  

5672 — AMQP  
15672 — management UI  
Redis  
6379  

Может работать как:  

cache;  
key-value storage;  
pub/sub;  
distributed lock.  

## 85. Как диагностировать: сервис не отвечает

Допустим, API не открывается:  

http://test-api.local:8080/health  

Порядок диагностики:  

1. Проверить DNS  
dig test-api.local  

или:  

nslookup test-api.local  
2. Проверить доступность хоста  
ping test-api.local  

Но ping может быть закрыт, это не всегда показатель.  

3. Проверить порт  
nc -vz test-api.local 8080  
4. Проверить HTTP  
curl -v http://test-api.local:8080/health  
5. Проверить, слушает ли сервис порт на сервере  
ss -tulpen | grep 8080  
6. Проверить логи  
journalctl -u service-name  

или Docker:  

docker logs container_name  
7. Проверить firewall/security group  
sudo iptables -L -n  

или правила в облаке/инфраструктуре.  

8. Проверить маршруты  
ip route  

## 86. Как диагностировать: DNS не работает

Симптом:  

Could not resolve host  
Name or service not known  

### Проверки:

cat /etc/resolv.conf  
dig domain.local  
nslookup domain.local  
resolvectl status  

Проверить /etc/hosts:  

cat /etc/hosts  

Проверить напрямую через конкретный DNS:  

dig @8.8.8.8 example.com  

## 87. Как диагностировать: порт занят

Ошибка:  

Address already in use  

Проверить:  

lsof -i :8080  

или:  

ss -tulpen | grep 8080  

Убить процесс, если это твой локальный тестовый процесс:  

kill <pid>  

Если не завершается:  

kill -9 <pid>  

### На собеседовании лучше сказать, что kill -9 — крайний вариант.

## 88. Как диагностировать: TLS ошибка

### Пример:

SSL certificate problem  
certificate verify failed  

Проверить:  

curl -v https://example.com  
openssl s_client -connect example.com:443 -servername example.com  

Возможные причины:  

истек сертификат;  
сертификат не для этого домена;  
self-signed;  
отсутствует intermediate certificate;  
клиент не доверяет CA;  
неправильное время на машине.  

## 89. Как диагностировать: 502/503/504

502  

Проверить:  

жив ли backend;  
правильно ли настроен upstream;  
не падает ли приложение;  
нет ли connection reset;  
не перепутан ли HTTP/HTTPS.  
503  

Проверить:  

есть ли доступные backend-инстансы;  
readiness;  
не перегружен ли сервис;  
не maintenance ли;  
не заблокировал ли rate limiter.  
504  

Проверить:  

долго ли отвечает backend;  
таймауты nginx/gateway;  
медленные запросы в БД;  
зависания;  
сетевые задержки.  

## 90. Логи и correlation ID

В микросервисах важно уметь связать запросы между сервисами.  

Для этого используют:  

X-Request-ID  
X-Correlation-ID  
traceparent  
Для QA  

В баг-репорте полезно указывать:  

endpoint;  
request body;  
response body;  
status code;  
headers;  
timestamp;  
request id;  
environment;  
user/test data;  
logs.  

