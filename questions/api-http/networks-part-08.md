# Сети и HTTP для QA/SDET — полный конспект — часть 8

[← Оглавление](networks.md) · [← К разделу](../api-http.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/networks.md)

Темы 71-80.

## 71. CORS

CORS — Cross-Origin Resource Sharing.  

Браузерная политика безопасности, которая ограничивает запросы с одного origin на другой.  

Origin состоит из:  

scheme + host + port  

### Примеры разных origin:

https://example.com  
http://example.com  
https://api.example.com  
https://example.com:8443  
Основные headers  
Access-Control-Allow-Origin  
Access-Control-Allow-Methods  
Access-Control-Allow-Headers  
Access-Control-Allow-Credentials  
Preflight request  

Для некоторых запросов браузер сначала отправляет OPTIONS-запрос.  

OPTIONS /api/users  
Важно  

CORS — это ограничение браузера.  
Через curl/Postman запрос может работать, а из браузера — блокироваться.  

### На собесе

CORS — механизм браузера, который контролирует, может ли frontend с одного origin обращаться к API на другом origin. Если запрос из Postman работает, а из браузера нет, стоит проверить CORS headers и preflight OPTIONS.  

## 72. Same-Origin Policy

Same-Origin Policy запрещает JS на одной странице свободно читать данные с другого origin.  

Origin:  

protocol + domain + port  

### Пример:

https://site.com  
https://site.com:443  

Это один origin.  

А вот:  

http://site.com  
https://site.com  

разные origin, потому что протокол отличается.  

## 73. Кэширование HTTP

Кэширование позволяет не загружать одни и те же ресурсы повторно.  

Headers  
Header	Назначение  
Cache-Control	правила кэширования  
ETag	идентификатор версии ресурса  
Last-Modified	дата изменения  
Expires	срок действия  
If-None-Match	проверка ETag  
If-Modified-Since	проверка даты изменения  
304 Not Modified  

Сервер говорит:  

Ресурс не изменился, используй кэш  
Для QA  

Тестировать:  

ресурс обновился, но клиент видит старые данные;  
неправильный Cache-Control;  
приватные данные кэшируются;  
ETag не меняется после изменения ресурса;  
304 возвращается корректно.  

## 74. CDN

CDN — Content Delivery Network.  

Используется для доставки статики ближе к пользователю:  

картинки;  
JS;  
CSS;  
видео;  
файлы.  

### Плюсы:

быстрее загрузка;  
меньше нагрузка на origin;  
географическая близость.  

Проблемы для QA:  

кэш CDN не обновился;  
разные пользователи видят разные версии;  
purge не сработал;  
stale cache;  
проблемы только в одном регионе.  

## 75. API Gateway

API Gateway — входная точка для API.  

Может делать:  

маршрутизацию;  
авторизацию;  
rate limiting;  
логирование;  
трансформацию запросов;  
агрегацию ответов;  
TLS termination.  

### Пример:

Client -> API Gateway -> Service A  
                    -> Service B  
                    -> Service C  

## 76. Service Discovery

В микросервисах сервисы должны находить друг друга.  

Service discovery помогает понять:  

### Где сейчас находится service-a?

### Примеры:

Kubernetes DNS;  
Consul;  
Eureka;  
etcd;  
встроенный service discovery в Docker/K8s.  

## 77. Health checks

Health check — проверка состояния сервиса.  

Обычно:  

GET /health  
GET /ready  
GET /live  
Liveness  

Проверяет, жив ли процесс.  

Если нет — перезапустить.  

Readiness  

Проверяет, готов ли сервис принимать трафик.  

Например:  

подключился к БД;  
применил миграции;  
загрузил конфиг;  
прогрел кэш.  
На собесе  

Liveness отвечает на вопрос “жив ли процесс”, readiness — “готов ли сервис обслуживать запросы”.  

## 78. Docker networking

Docker создает сети для контейнеров.  

Основные типы сетей  
Тип	Описание  
bridge	стандартная сеть для контейнеров на одном хосте  
host	контейнер использует сеть хоста  
none	сеть отключена  
overlay	сеть между Docker-хостами  
Посмотреть сети  
docker network ls  
Информация о сети  
docker network inspect bridge  
Создать сеть  
docker network create test-network  
Запустить контейнер в сети  
docker run --network test-network nginx  

## 79. Docker: localhost внутри контейнера

Очень частая проблема.  

Если приложение внутри контейнера обращается к:  

localhost  

то это localhost самого контейнера, а не хоста.  

### Пример проблемы

Backend в контейнере пытается подключиться к БД:  

localhost:5432  

Но PostgreSQL запущен в другом контейнере.  

Правильно:  

postgres:5432  

### где postgres — имя сервиса в Docker Compose.

### На собесе

Внутри контейнера localhost указывает на сам контейнер. Чтобы обратиться к другому контейнеру в одной Docker-сети, нужно использовать имя сервиса или контейнера.  

## 80. Docker Compose networking

### Пример:

services:  
  app:  
    build: .  
    ports:  
      - "8080:8080"  
    depends_on:  
      - postgres  

  postgres:  
    image: postgres:16  
    environment:  
      POSTGRES_PASSWORD: password  

Контейнер app может обратиться к БД так:  

postgres:5432  

Не так:  

localhost:5432  

