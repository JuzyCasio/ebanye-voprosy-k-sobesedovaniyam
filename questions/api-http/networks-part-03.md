# Сети и HTTP для QA/SDET — полный конспект — часть 3

[← Оглавление](networks.md) · [← К разделу](../api-http.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/networks.md)

Темы 18-27.

## 18. Connection timeout

Ошибка:  

Connection timed out  

Означает:  

пакет ушел;  
ответа нет;  
возможно, хост недоступен;  
порт фильтруется firewall;  
проблемы маршрутизации;  
сервис завис;  
сетевой пакет где-то теряется.  
Отличие от refused  
Ошибка	Что значит  
Connection refused	хост ответил отказом  
Timeout	ответа нет  

### На собесе:

Refused — это быстрый отказ. Timeout — это когда клиент не дождался ответа.  

## 19. Connection reset by peer

Ошибка:  

Connection reset by peer  

Означает, что удаленная сторона резко закрыла соединение.  

Возможные причины:  

сервер упал;  
приложение закрыло сокет;  
proxy сбросил соединение;  
timeout на сервере;  
некорректный протокол;  
TLS/plain HTTP перепутаны.  

## 20. DNS

DNS — Domain Name System.  

Он переводит доменное имя в IP-адрес.  

example.com -> 93.184.216.34  
Зачем нужен DNS  

Людям удобно помнить домены, а компьютеры работают с IP.  

Типы DNS-записей  
Тип	Назначение  
A	домен → IPv4  
AAAA	домен → IPv6  
CNAME	alias на другой домен  
MX	почтовые серверы  
TXT	текстовые записи, SPF/DKIM  
NS	DNS-серверы зоны  
PTR	обратная запись IP → домен  
SRV	сервисные записи  
Примеры  
nslookup google.com  
dig google.com  
dig google.com A  
dig google.com MX  
dig +short google.com  

## 21. DNS resolution: как резолвится домен

### Когда клиенту нужен IP:

Проверяется локальный кэш.  
Проверяется /etc/hosts.  
Запрос идет к DNS resolver.  
Resolver может обратиться к root DNS.  
Потом к TLD DNS, например .com.  
Потом к authoritative DNS.  
Возвращается IP.  

На практике для собеседования достаточно:  

Клиент отправляет DNS-запрос, DNS-сервер возвращает IP-адрес домена, после этого клиент может подключаться к серверу по IP.  

## 22. /etc/hosts

Файл локального переопределения DNS:  

cat /etc/hosts  

### Пример:

127.0.0.1 localhost  
192.168.1.100 test.local  

Если прописать:  

127.0.0.1 example.com  

то example.com будет резолвиться в localhost.  

Где может пригодиться QA  
подменить домен на тестовый стенд;  
локально проверить сервис;  
обойти DNS на время теста;  
воспроизвести проблему с резолвингом.  

## 23. HTTP

HTTP — протокол прикладного уровня для обмена данными между клиентом и сервером.  

Работает по схеме:  

Request -> Response  

Клиент отправляет запрос, сервер возвращает ответ.  

### Структура HTTP-запроса

### Пример:

GET /users/123 HTTP/1.1  
Host: api.example.com  
Authorization: Bearer token  
Accept: application/json  

Состоит из:  

method;  
path;  
version;  
headers;  
body, если есть.  
Структура HTTP-ответа  
HTTP/1.1 200 OK  
Content-Type: application/json  

{  
  "id": 123,  
  "name": "Alex"  
}  

Состоит из:  

status line;  
headers;  
body.  

## 24. HTTP методы

Метод	Назначение  
GET	получить ресурс  
POST	создать ресурс или выполнить операцию  
PUT	полностью заменить ресурс  
PATCH	частично обновить ресурс  
DELETE	удалить ресурс  
HEAD	получить только headers  
OPTIONS	узнать доступные методы/настройки  
GET  
GET /users/1  

Получить пользователя.  

POST  
POST /users  

Создать пользователя.  

PUT  
PUT /users/1  

Полностью заменить пользователя.  

PATCH  
PATCH /users/1  

### Частично обновить пользователя.

DELETE  
DELETE /users/1  

Удалить пользователя.  

## 25. Идемпотентность HTTP методов

Идемпотентность означает:  

Повторный одинаковый запрос приводит к тому же результату, что и один запрос.  

Метод	Идемпотентный  
GET	Да  
PUT	Да  
PATCH	Не всегда  
DELETE	Да, обычно  
POST	Нет  

### Пример:

DELETE /users/1  

Первый раз удалит пользователя. Второй раз пользователь уже удален. Состояние системы остается таким же — пользователя нет.  

А вот:  

POST /orders  

Если отправить 2 раза, может создать 2 заказа.  

## 26. Safe методы

Safe method — метод, который не должен изменять состояние сервера.  

Метод	Safe  
GET	Да  
HEAD	Да  
OPTIONS	Да  
POST	Нет  
PUT	Нет  
PATCH	Нет  
DELETE	Нет  

### Важно: GET технически может что-то менять из-за плохой реализации, но по стандартной идее не должен.

## 27. HTTP статус-коды

1xx — информационные  
Код	Значение  
100	Continue  
101	Switching Protocols  
2xx — успех  
Код	Значение  
200	OK  
201	Created  
202	Accepted  
204	No Content  
3xx — редиректы  
Код	Значение  
301	Moved Permanently  
302	Found  
304	Not Modified  
307	Temporary Redirect  
308	Permanent Redirect  
4xx — ошибка клиента  
Код	Значение  
400	Bad Request  
401	Unauthorized  
403	Forbidden  
404	Not Found  
405	Method Not Allowed  
409	Conflict  
415	Unsupported Media Type  
422	Unprocessable Entity  
429	Too Many Requests  
5xx — ошибка сервера  
Код	Значение  
500	Internal Server Error  
501	Not Implemented  
502	Bad Gateway  
503	Service Unavailable  
504	Gateway Timeout  

