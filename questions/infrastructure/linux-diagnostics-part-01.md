# Linux: диагностика проблем на сервере — часть 1

[← Оглавление](linux-diagnostics.md) · [← Linux](linux.md) · [⚡ Шпаргалка](../../cheatsheets/linux.md)

Да. Если ты уже выучил работу с файлами, grep, sed, awk, find, jq, tail, less, то для собеседования по Linux тебе ещё обязательно нужны процессы, сервисы, сеть, права, ресурсы системы, Docker и диагностика.

Что ещё нужно знать по Linux
1. Процессы

Основные команды:

ps aux
top
htop
pgrep service_name
kill PID
kill -9 PID

Нужно понимать:

что такое PID;
чем процесс отличается от сервиса;
SIGTERM и SIGKILL;
как найти процесс по имени;
как найти процесс, который занял порт.
ps aux | grep python
pgrep -af python
lsof -i :8080
ss -ltnp | grep 8080

На собеседовании:

Сначала процесс стоит завершать через обычный kill, то есть SIGTERM, чтобы он корректно освободил ресурсы. kill -9 использую только если процесс завис и не реагирует.

2. Управление сервисами
systemctl status nginx
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx

Логи сервиса:

journalctl -u nginx
journalctl -u nginx -n 100
journalctl -u nginx -f

Важно понимать разницу:

restart — полностью перезапустить;
reload — перечитать конфигурацию без полного перезапуска;
status — посмотреть состояние и последние ошибки.
3. Сеть

Основные команды:

ip a
ip route
ping host
nslookup host
dig host
nc -vz host 8080
curl -v http://host:8080/health
ss -ltnp

Нужно понимать, что проверяет каждая команда:

ping — доступность хоста по ICMP;
nc — доступность конкретного порта;
curl — работа приложения на уровне HTTP;
ss — какие порты слушает сервер;
dig / nslookup — DNS;
ip route — маршрутизация.

Хорошая фраза:

Прохождение ping ещё не означает, что сервис работает. Нужно отдельно проверить порт через nc и HTTP-ответ через curl.

4. Права доступа
ls -la
chmod 755 script.sh
chmod +x script.sh
chown user:group file

Нужно знать:

r — чтение
w — запись
x — выполнение

И числовые права:

644 — владелец читает и пишет, остальные только читают
755 — владелец всё, остальные читают и выполняют
600 — доступ только владельцу
5. Ресурсы сервера
df -h
du -sh *
free -h
top
uptime

Нужно уметь проверить:

закончилось ли место на диске;
закончилась ли память;
перегружен ли CPU;
есть ли высокий load average;
какие процессы потребляют ресурсы.

Полезно:

ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
6. Переменные окружения
env
printenv
echo $BASE_URL
echo $PATH

Очень часто проблема на стенде связана с неправильным окружением:

echo $DB_HOST
echo $DB_PORT
echo $ENV
7. Exit code
echo $?

Нужно знать:

0 — успешное выполнение
не 0 — ошибка

Особенно важно для CI/CD.

8. Docker

Минимум:

docker ps
docker ps -a
docker logs container
docker logs -f container
docker inspect container
docker exec -it container sh
docker restart container

Для диагностики:

docker ps -a
docker logs --tail 200 container
docker inspect container
Главная логика диагностики

На собеседовании важно показать не набор случайных команд, а последовательность.

Удобная схема:

Клиент
↓
DNS
↓
Сеть
↓
Порт
↓
Reverse proxy / API Gateway
↓
Приложение
↓
Зависимости: БД, брокер, внешние API
↓
Ресурсы и конфигурация

То есть идём от внешнего слоя к внутреннему.

Сценарий 1. Сервер, много сервисов, запрос возвращает 500
Что означает 500

HTTP 500 — сервер получил запрос, но во время обработки произошла внутренняя ошибка.

Это обычно значит:

приложение упало на исключении;
недоступна БД;
недоступен другой сервис;
неправильные данные или конфигурация;
отсутствует переменная окружения;
закончилась память или место;
проблема с правами;
ошибка после деплоя;
таймаут зависимости.
Как диагностировать по шагам
Шаг 1. Повторить запрос вручную
curl -v http://host/api/users

Для POST:

curl -v -X POST http://host/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Alex"}'

Проверяем:

действительно ли статус 500;
тело ответа;
заголовки;
request ID / trace ID;
точно ли запрос отправлен на нужный стенд;
правильные ли данные и заголовки.

Если есть request ID:

X-Request-ID: abc-123

Его нужно использовать для поиска по логам.

Шаг 2. Понять, какой сервис обрабатывает запрос

Если сервисов много, нужно определить маршрут:

nginx → API Gateway → user-service → database

Проверяем конфиг nginx или gateway, документацию, service discovery, Docker Compose или Kubernetes-манифесты.

На сервере:

