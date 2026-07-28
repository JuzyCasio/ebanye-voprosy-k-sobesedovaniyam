# Сети и HTTP для QA/SDET — полный конспект — часть 10

[← Оглавление](networks.md) · [← К разделу](../api-http.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/networks.md)

Темы 91-100.

## 91. Что тестировать в API с точки зрения сетей

Таймауты  
сервер долго отвечает;  
клиент корректно обрабатывает read timeout;  
есть retry или понятная ошибка;  
пользователь не видит бесконечный loader.  
Ретраи  
ретраи есть только там, где безопасно;  
не создаются дубли;  
используется idempotency key;  
backoff работает.  
Ошибки сети  
connection refused;  
connection reset;  
timeout;  
DNS error;  
TLS error.  
Rate limit  
429;  
Retry-After;  
лимит по пользователю/IP/token.  
Большие payload  
лимит размера body;  
413 Payload Too Large;  
корректная ошибка;  
нет падения сервиса.  
Плохой JSON  
400;  
понятная ошибка;  
сервер не падает.  
Авторизация  
нет токена → 401;  
плохой токен → 401;  
нет роли → 403;  
истекший токен → 401;  
чужой ресурс → 403 или 404, зависит от политики.  

## 92. Как симулировать сетевые проблемы в тестах

В тестовом окружении можно моделировать:  

timeout;  
500/502/503;  
медленный ответ;  
connection reset;  
потерю соединения;  
недоступность dependency.  
Через mock-сервис  

Например, поднять FastAPI mock:  

from fastapi import FastAPI  
import asyncio  

app = FastAPI()  

@app.get("/slow")  
async def slow_response() -> dict[str, str]:  
    await asyncio.sleep(10)  
    return {"status": "ok"}  

@app.get("/error")  
async def error_response() -> tuple[dict[str, str], int]:  
    return {"error": "service unavailable"}, 503  
Через настройки клиента  

В Python requests:  

import requests  

try:  
    response = requests.get("http://localhost:8000/slow", timeout=1)  
except requests.Timeout:  
    print("Timeout happened")  
Через Docker  

Можно остановить зависимость:  

docker stop postgres  

И проверить, как приложение реагирует на недоступность БД.  

## 93. Нагрузочное тестирование и сети

В нагрузке важны:  

latency;  
throughput;  
RPS;  
percentiles;  
errors rate;  
timeouts;  
connection pool;  
keep-alive;  
DNS;  
TLS overhead;  
лимиты ОС.  
Метрики  
Метрика	Значение  
RPS	requests per second  
latency	задержка  
p95	95% запросов быстрее этого времени  
p99	99% запросов быстрее этого времени  
error rate	процент ошибок  
throughput	объем переданных данных  
concurrent users	параллельные пользователи  
Частая ошибка  

Среднее время ответа может быть нормальным, но p95/p99 плохие.  

### На собесе:

Я бы смотрел не только average latency, но и p95/p99, потому что именно они показывают хвосты задержек и проблемы под нагрузкой.  

## 94. Connection pool

Connection pool — пул соединений.  

Используется для:  

БД;  
HTTP-клиентов;  
брокеров.  

Зачем:  

не открывать новое соединение на каждый запрос;  
уменьшить overhead;  
контролировать количество соединений.  
Проблемы  
pool слишком маленький → ожидание соединения;  
pool слишком большой → перегруз БД;  
соединения не закрываются → leak;  
idle connections рвутся proxy/firewall.  

## 95. DNS caching

DNS-ответы кэшируются.  

Проблемы:  

IP сервиса поменялся, а клиент ходит на старый;  
в тестах после смены DNS нужно подождать TTL;  
разные клиенты видят разные IP.  
TTL  

TTL — сколько времени DNS-запись может храниться в кэше.  

## 96. Blue-Green / Canary и сеть

Blue-Green  

Есть две версии окружения:  

blue — текущая  
green — новая  

Трафик переключается с одной на другую.  

Canary  

Новая версия получает небольшой процент трафика:  

5% -> new version  
95% -> old version  
Для QA  

Проблемы:  

баг виден не всегда;  
разные пользователи попадают на разные версии;  
sticky sessions мешают проверке;  
кэш/CDN может отдавать старую версию.  

## 97. Типовые сетевые ошибки в API-тестах

Max retries exceeded  

Клиент пытался несколько раз подключиться, но не смог.  

Причины:  

сервис недоступен;  
DNS;  
порт закрыт;  
network policy;  
proxy.  
Read timed out  

Сервер не ответил за заданное время.  

Connection refused  

Сервис не слушает порт.  

Connection reset by peer  

Сервер или proxy сбросил соединение.  

Name or service not known  

DNS не смог разрешить имя.  

SSL certificate verify failed  

### Проблема с TLS-сертификатом.

## 98. Разница между localhost, 127.0.0.1 и 0.0.0.0

localhost  

Имя, обычно резолвится в:  

127.0.0.1  
127.0.0.1  

Loopback-адрес текущей машины.  

0.0.0.0  

### Когда сервис слушает 0.0.0.0, это значит:

слушать на всех сетевых интерфейсах.  

### Пример:

uvicorn app:app --host 0.0.0.0 --port 8000  

Если сервис слушает только:  

127.0.0.1:8000  

то с другой машины он может быть недоступен.  

## 99. Bind address

Bind address — адрес, на котором приложение слушает соединения.  

### Примеры:

127.0.0.1:8080 — только локально  
0.0.0.0:8080 — на всех интерфейсах  
192.168.1.10:8080 — только на конкретном интерфейсе  

## 100. IPv4 loopback

Диапазон loopback:  

127.0.0.0/8  

Обычно используют:  

127.0.0.1  

