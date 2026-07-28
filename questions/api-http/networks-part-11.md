# Сети и HTTP для QA/SDET — полный конспект — часть 11

[← Оглавление](networks.md) · [← К разделу](../api-http.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/networks.md)

Темы 101-116.

## 101. DHCP

DHCP автоматически выдает устройствам сетевые настройки:  

IP;  
маску;  
gateway;  
DNS.  

Без DHCP пришлось бы прописывать всё вручную.  

## 102. SMTP, IMAP, POP3

Для почты:  

Протокол	Назначение  
SMTP	отправка почты  
IMAP	чтение почты с синхронизацией  
POP3	скачивание почты  

Порты:  

SMTP: 25, 465, 587  
IMAP: 143, 993  
POP3: 110, 995  

## 103. FTP/SFTP/SCP

FTP  

Старый протокол передачи файлов.  

Порт:  

21  
SFTP  

Передача файлов поверх SSH.  

Порт:  

22  
SCP  

Копирование файлов поверх SSH.  

### Пример:

scp file.txt user@host:/tmp/  

## 104. SSH

SSH — безопасное удаленное подключение.  

Порт:  

22  

Подключение:  

ssh user@host  

С нестандартным портом:  

ssh -p 2222 user@host  

Выполнить команду удаленно:  

ssh user@host "ls -la /tmp"  

## 105. Безопасность: базовые вещи

Для QA важно знать:  

HTTP без TLS небезопасен;  
токены нельзя логировать;  
пароли нельзя передавать в URL;  
чувствительные данные не должны быть в GET query params;  
cookie с сессией должна иметь Secure/HttpOnly;  
не должно быть лишних открытых портов;  
админки не должны быть доступны наружу;  
stack trace не должен уходить клиенту.  

## 106. Basic Auth, Bearer Token, API Key

Basic Auth  
Authorization: Basic base64(username:password)  

Base64 — это не шифрование.  

Использовать только поверх HTTPS.  

Bearer Token  
Authorization: Bearer <token>  

### Часто используется с JWT/OAuth.

API Key  
X-API-Key: key  

Или:  

Authorization: ApiKey key  

## 107. OAuth2 — очень кратко

OAuth2 — протокол авторизации.  

### Основные сущности:

Resource Owner — пользователь;  
Client — приложение;  
Authorization Server — выдает токены;  
Resource Server — API, которое принимает токены.  
Access token  

### Короткоживущий токен для доступа к API.

Refresh token  

Долгоживущий токен для получения нового access token.  

## 108. OpenID Connect

OIDC — надстройка над OAuth2 для аутентификации.  

OAuth2 отвечает больше за авторизацию.  
OIDC добавляет информацию о пользователе и id_token.  

## 109. Типовая архитектура веб-приложения

Browser  
   |  
   | HTTPS  
   v  
Nginx / Load Balancer  
   |  
   v  
Backend API  
   |  
   +--> PostgreSQL  
   +--> Redis  
   +--> Kafka/RabbitMQ  
   +--> External API  
Где могут быть проблемы  
Место	Проблемы  
Browser	CORS, cache, cookies  
DNS	домен не резолвится  
TLS	сертификат  
Nginx	502/504, route  
Backend	500, timeout  
DB	connection pool, slow query  
Redis	unavailable  
Broker	lag, connection  
External API	timeout, bad response  

## 110. Типовая диагностика API как QA

Допустим, тест падает:  

requests.exceptions.ReadTimeout  

### Что делать:

Проверить endpoint.  
Повторить curl вручную.  
Проверить, стабильно ли воспроизводится.  
Проверить latency.  
Проверить логи backend.  
Проверить dependency: БД, Redis, broker.  
Проверить gateway/proxy timeout.  
Проверить, не перегружен ли стенд.  
Увеличить timeout только если это оправдано, а не замазывает проблему.  
Завести баг с request/response/logs/request-id.  

## 111. Что такое reverse proxy на примере Nginx

Клиент обращается:  

https://example.com/api/users  

Nginx принимает запрос и проксирует:  

http://backend:8080/api/users  

### Плюсы:

один входной endpoint;  
TLS на Nginx;  
backend можно не выставлять наружу;  
можно балансировать;  
можно настраивать лимиты.  

## 112. TLS termination

TLS termination — когда HTTPS заканчивается на proxy/load balancer.  

Client --HTTPS--> Nginx --HTTP--> Backend  

Или:  

Client --HTTPS--> Nginx --HTTPS--> Backend  
Для QA  

### Важно понимать:

backend может видеть http, хотя клиент пришел по https;  
нужны headers X-Forwarded-Proto;  
могут быть проблемы с redirect;  
cookies Secure зависят от схемы.  

## 113. X-Forwarded headers

### Когда есть proxy, backend может не видеть настоящий IP клиента.

Используют:  

X-Forwarded-For: 203.0.113.10  
X-Forwarded-Proto: https  
X-Forwarded-Host: example.com  
Возможная проблема  

Если приложение неправильно обрабатывает X-Forwarded-Proto, оно может генерировать ссылки на http вместо https.  

## 114. URL: структура

### Пример:

https://user:pass@example.com:8443/api/users?id=1#profile  

### Части:

scheme: https  
user info: user:pass  
host: example.com  
port: 8443  
path: /api/users  
query: id=1  
fragment: profile  
Важно  

Fragment #profile не отправляется на сервер. Он обрабатывается браузером.  

## 115. Query params vs Path params

Path param  
GET /users/123  

123 — часть пути, идентификатор ресурса.  

Query param  
GET /users?role=admin&page=2  

Используется для фильтрации, сортировки, пагинации.  

## 116. URL encoding

Некоторые символы в URL должны кодироваться.  

### Пример:

пробел -> %20  

Или в query:  

+ иногда используется как пробел  

Для QA важно тестировать:  

кириллицу;  
пробелы;  
спецсимволы;  
?, &, =, /;  
emoji, если продукт поддерживает.  

