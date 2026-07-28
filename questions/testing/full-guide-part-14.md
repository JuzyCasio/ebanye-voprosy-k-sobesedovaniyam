# Теория тестирования — полный конспект — часть 14

[← Оглавление](full-guide.md) · [← К разделу](../testing.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/testing.md)

Темы 64-74.

## 64. Тестирование безопасности: базовый уровень QA

### Что может проверить QA без глубокого pentest:

Auth  
доступ без токена;  
доступ с неправильным токеном;  
истёкший токен;  
logout инвалидирует токен;  
refresh token работает корректно.  
Authorization  
пользователь не видит чужие данные;  
роль не может делать запрещённые действия;  
API проверяет права, а не только UI.  
Input validation  
спецсимволы;  
HTML;  
JS;  
SQL-подобные строки;  
длинные значения;  
path traversal в файлах.  
Sensitive data  
пароль не возвращается;  
токены не пишутся в логи;  
персональные данные не видны без прав;  
ошибки не раскрывают внутренности системы.  

## 65. Тестирование совместимости

### Проверки:

браузеры;  
мобильные устройства;  
разрешения экрана;  
ОС;  
версии приложения;  
версии API;  
обратная совместимость;  
разные языки;  
разные часовые пояса;  
разные настройки пользователя.  

## 66. Тестирование мобильного приложения

### Что проверять:

установка;  
обновление;  
удаление;  
первый запуск;  
логин;  
permissions;  
push notifications;  
offline mode;  
смена сети;  
плохая сеть;  
поворот экрана;  
background/foreground;  
deep links;  
разные размеры экрана;  
разные версии ОС;  
battery usage;  
crash logs;  
локальное хранилище;  
биометрия, если есть.  

## 67. Тестирование веб-приложения

### Что проверять:

разные браузеры;  
адаптивность;  
cookies;  
localStorage;  
sessionStorage;  
cache;  
back/forward;  
refresh;  
deep links;  
SEO, если важно;  
accessibility;  
HTTPS;  
mixed content;  
CORS;  
загрузка ресурсов;  
ошибки JS в консоли;  
network requests.  

## 68. CORS

CORS — механизм браузера, который ограничивает запросы между разными origin.  

Origin состоит из:  

protocol;  
host;  
port.  

### Пример:

https://example.com:443  

### Что проверять:

разрешённые origins;  
запрещённые origins;  
preflight OPTIONS;  
credentials;  
headers;  
methods.  

### Короткий ответ:

CORS — это браузерный механизм безопасности. Он определяет, какие внешние домены могут обращаться к API из браузера.  

## 69. Cookies, LocalStorage, SessionStorage

Cookies  
отправляются на сервер автоматически;  
могут иметь HttpOnly;  
могут иметь Secure;  
могут иметь SameSite;  
подходят для сессий.  
LocalStorage  
хранится долго;  
не отправляется автоматически на сервер;  
доступен из JS;  
не стоит хранить чувствительные токены без необходимости.  
SessionStorage  
хранится в рамках вкладки;  
очищается после закрытия вкладки.  

## 70. HTTP basics для QA

Request  

Состоит из:  

method;  
URL;  
headers;  
query params;  
body.  
Response  

Состоит из:  

status code;  
headers;  
body;  
cookies.  
Headers  

Важные заголовки:  

Content-Type;  
Authorization;  
Accept;  
User-Agent;  
Cache-Control;  
Set-Cookie;  
Location;  
X-Request-ID.  

## 71. Client-server architecture

Клиент-серверная архитектура:  

клиент отправляет запрос;  
сервер обрабатывает;  
сервер обращается к БД/сервисам;  
сервер возвращает ответ;  
клиент отображает результат.  

Для QA важно понимать:  

где баг: frontend, backend, DB, network, external service;  
что видно в DevTools;  
какие запросы уходят;  
какие ответы приходят;  
есть ли ошибка в логах;  
изменились ли данные в БД.  

## 72. Как локализовать баг

### Алгоритм:

Воспроизвести баг.  
Проверить окружение.  
Проверить данные.  
Открыть DevTools.  
Посмотреть network request/response.  
Проверить console errors.  
Проверить backend logs.  
Проверить БД.  
Сравнить с другим окружением.  
Сузить область: frontend/backend/data/integration.  

### Пример:

Если кнопка не работает, я сначала смотрю, уходит ли запрос. Если запрос не уходит — возможно frontend. Если запрос уходит и возвращает ошибку — смотрю API response и backend logs. Если API успешный, но данные не изменились — проверяю БД или дальнейшую обработку.  

## 73. DevTools для тестировщика

### Что смотреть:

Elements  
DOM;  
стили;  
скрытые элементы;  
disabled/enabled;  
aria-атрибуты.  
Console  
JS-ошибки;  
warnings;  
custom logs.  
Network  
запросы;  
ответы;  
headers;  
payload;  
cookies;  
статус-коды;  
время ответа;  
failed requests.  
Application  
cookies;  
localStorage;  
sessionStorage;  
cache;  
service workers.  
Performance  
загрузка страницы;  
bottlenecks;  
long tasks.  

## 74. Что такое окружения

Типовые окружения:  

local;  
dev;  
test;  
QA;  
staging;  
preprod;  
production.  

### Что важно:

версия приложения;  
версия БД;  
конфиги;  
feature flags;  
тестовые данные;  
доступы;  
внешние интеграции;  
production-like или нет.  

