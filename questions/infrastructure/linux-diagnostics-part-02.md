# Linux: диагностика проблем на сервере — часть 2

[← Оглавление](linux-diagnostics.md) · [← Linux](linux.md) · [⚡ Шпаргалка](../../cheatsheets/linux.md)

ss -ltnp
docker ps
systemctl list-units --type=service
Шаг 3. Посмотреть логи входного сервиса

Если nginx:

tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

Если systemd:

journalctl -u api-gateway -n 200

Если Docker:

docker logs --tail 200 api-gateway

Ищем request ID:

grep "abc-123" app.log

Или ошибки по времени:

grep -Ei "error|exception|traceback|failed" app.log
Шаг 4. Посмотреть логи бизнес-сервиса

Допустим запрос дошёл до user-service.

journalctl -u user-service -n 200

Или:

docker logs --tail 200 user-service

Ищем:

stacktrace;
timeout;
connection refused;
authentication failed;
null pointer;
key error;
database exception;
out of memory;
permission denied.
Шаг 5. Проверить зависимости

Приложение может работать, но падать при обращении к БД или другому сервису.

Проверить БД:

nc -vz db-host 5432

Проверить другой сервис:

curl -v http://payment-service:8080/health

Проверить брокер:

nc -vz kafka-host 9092

Проверить DNS:

dig db-host
nslookup db-host
Шаг 6. Проверить состояние сервиса
systemctl status user-service

Или:

docker ps -a

Важно: сервис может быть running, но фактически неработоспособен. Поэтому отдельно проверяем health endpoint:

curl -v http://localhost:8080/health
Шаг 7. Проверить конфигурацию
env
printenv
echo $DB_HOST
echo $DB_USER
echo $ENV

Проверить конфиги:

cat application.yml
grep -rn "DB_HOST" .

Частые проблемы:

неверный host;
неверный порт;
неправильный пароль;
переменная отсутствует;
сервис направлен на старый стенд;
конфиг не перечитан после изменения.
Шаг 8. Проверить ресурсы
df -h
free -h
top

Ищем:

диск заполнен на 100%;
память закончилась;
процесс убит OOM Killer;
CPU загружен;
слишком много открытых файлов.

OOM можно искать так:

dmesg | grep -i "out of memory"
journalctl -k | grep -i "oom"
Шаг 9. Проверить изменения

Нужно спросить:

был ли недавно деплой;
менялся ли конфиг;
менялась ли схема БД;
обновлялись ли сертификаты;
проблема возникает у всех или только у одного запроса.

Если ошибка появилась после релиза, сравнить версии:

docker images
docker inspect container
git log -n 5
Хороший ответ на собеседовании

Я сначала повторю запрос через curl -v, проверю статус, тело и request ID. Затем определю, какой сервис обрабатывает запрос: gateway, nginx и конкретный backend. По request ID и времени найду запрос в логах gateway и приложения. После этого проверю stacktrace и зависимости сервиса: БД, брокер, внешние API. Также проверю переменные окружения, конфигурацию, состояние процесса, порт и ресурсы сервера — память, CPU и диск. Если ошибка появилась после деплоя, сравню версию и последние изменения.

Сценарий 2. Запрос не возвращает ответ и висит

Возможные причины:

таймаут зависимости;
зависший поток;
блокировка в БД;
сервис ждёт брокер;
сетевой timeout;
исчерпан connection pool;
высокая нагрузка.

Проверки:

curl -v --max-time 10 http://host/api

Проверить порт:

nc -vz host 8080

Проверить нагрузку:

top
free -h

Проверить соединения:

ss -antp

Проверить логи:

grep -Ei "timeout|connection pool|deadlock|blocked" app.log

Ответ:

Если запрос висит, сначала ограничу его временем через curl --max-time, проверю соединение с портом, затем логи сервиса на timeout и зависшие зависимости. Проверю состояние connection pool, БД, внешние сервисы, количество соединений и нагрузку на сервер.

Сценарий 3. Получаем 502 Bad Gateway

502 обычно означает:

Proxy или gateway не получил корректный ответ от backend-сервиса.

Проверяем:

systemctl status nginx
tail -f /var/log/nginx/error.log

Проверить backend напрямую:

curl -v http://backend-host:8080/health

Проверить порт:

nc -vz backend-host 8080

Проверить, слушает ли backend:

ss -ltnp | grep 8080

Типовые причины:

backend не запущен;
неправильный host/port в nginx;
backend упал;
connection refused;
DNS не разрешается;
backend отвечает некорректно.
Сценарий 4. Получаем 504 Gateway Timeout

504 означает:

Gateway дождался backend, но тот не ответил вовремя.

Проверяем:

медленный backend;
медленный SQL;
недоступная зависимость;
высокий CPU;
проблемы сети;
слишком маленький timeout.

Команды:

curl -v --max-time 30 http://backend/api
top
journalctl -u backend -n 200

Ищем:

grep -Ei "timeout|slow|deadlock|pool" app.log

Хороший ответ:

