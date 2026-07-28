# Linux для QA/SDET — полный конспект — часть 2

[← Оглавление](linux.md) · [← К разделу](../infrastructure.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/linux.md)

Темы 9-18.

## 9. Важный вопрос: права на директорию

Для директории:  

r — можно посмотреть список файлов  
w — можно создавать/удалять файлы  
x — можно заходить в директорию  

### На собесе могут спросить:

### Что значит execute на директории?

Ответ:  

Для директории x означает право прохода внутрь директории. Без x нельзя зайти в директорию и обратиться к файлам внутри, даже если знаешь их имена.  

## 10. Процессы

Посмотреть процессы  
ps  
ps aux  
ps -ef  

### Часто:

ps aux | grep python  
ps aux | grep nginx  
Интерактивный просмотр  
top  
htop  

htop удобнее, но может быть не установлен.  

Найти PID процесса  
pgrep nginx  
pidof nginx  
Убить процесс  
kill <pid>  
kill -9 <pid>  
Убить по имени  
pkill nginx  
killall nginx  
Сигналы  
kill -15 <pid>  

SIGTERM — мягкое завершение. Процесс может корректно закрыться.  

kill -9 <pid>  

SIGKILL — принудительное завершение. Процесс не может его обработать.  

kill -1 <pid>  

SIGHUP — часто используется для перечитывания конфигурации.  

## 11. Как объяснить SIGTERM и SIGKILL

SIGTERM:  

Просит процесс завершиться корректно. Процесс может закрыть соединения, сохранить данные, освободить ресурсы.  

SIGKILL:  

Принудительно убивает процесс на уровне ядра. Процесс не успевает выполнить cleanup. Использовать стоит только если обычное завершение не сработало.  

## 12. Состояния процессов

В ps aux можно увидеть статусы:  

R — running  
S — sleeping  
D — uninterruptible sleep  
T — stopped  
Z — zombie  
Zombie process  

Zombie — процесс уже завершился, но его родитель ещё не забрал код завершения.  

### Короткий ответ:

Zombie-процесс не выполняется, но запись о нём остаётся в таблице процессов, пока родительский процесс не вызовет wait. Обычно лечится завершением или перезапуском родительского процесса.  

## 13. Фоновые процессы

Запустить в фоне  
command &  
Посмотреть фоновые задачи  
jobs  
Вернуть задачу на передний план  
fg  
Продолжить в фоне  
bg  
Запустить так, чтобы процесс не умер после выхода из терминала  
nohup command &  

### Пример:

nohup python script.py > app.log 2>&1 &  

## 14. systemd и сервисы

В современных Linux сервисами часто управляет systemd.  

Проверить статус сервиса  
systemctl status nginx  
Запустить сервис  
sudo systemctl start nginx  
Остановить сервис  
sudo systemctl stop nginx  
Перезапустить  
sudo systemctl restart nginx  
Перечитать конфигурацию  
sudo systemctl reload nginx  
Включить автозапуск  
sudo systemctl enable nginx  
Отключить автозапуск  
sudo systemctl disable nginx  
Проверить, включён ли автозапуск  
systemctl is-enabled nginx  
Посмотреть все сервисы  
systemctl list-units --type=service  

## 15. Где лежат systemd unit-файлы

### Часто:

/etc/systemd/system/  

Также:  

/lib/systemd/system/  
/usr/lib/systemd/system/  
Пример unit-файла  
[Unit]  
Description=My Python Service  
After=network.target  

[Service]  
User=appuser  
WorkingDirectory=/opt/myapp  
ExecStart=/usr/bin/python3 /opt/myapp/app.py  
Restart=always  

[Install]  
WantedBy=multi-user.target  

После изменения unit-файла:  

sudo systemctl daemon-reload  
sudo systemctl restart my-service  

## 16. Логи

journalctl  

Посмотреть логи сервиса:  

journalctl -u nginx  

Последние строки:  

journalctl -u nginx -n 100  

В реальном времени:  

journalctl -u nginx -f  

С текущей загрузки системы:  

journalctl -u nginx -b  

За период:  

journalctl --since "1 hour ago"  
journalctl --since "2026-07-07 10:00:00"  
Логи в файлах  

### Часто лежат тут:

/var/log/  

### Примеры:

/var/log/syslog  
/var/log/messages  
/var/log/auth.log  
/var/log/nginx/access.log  
/var/log/nginx/error.log  

## 17. Как диагностировать “сервис не запустился”

### Хороший порядок действий:

systemctl status service_name  

Потом:  

journalctl -u service_name -n 100  

Потом проверить конфиг, права, порты, переменные окружения:  

ls -la  
cat config.yml  
ss -tulpn  
df -h  

### На собесе можно сказать:

Сначала смотрю systemctl status, потом journalctl -u service -n 100, проверяю код ошибки, конфиг, права на файлы, занятые порты, доступность зависимостей, переменные окружения и место на диске.  

## 18. Сеть: базовые команды

IP-адреса  
ip addr  
ip a  

Старый вариант:  

ifconfig  
Маршруты  
ip route  
Проверить доступность хоста  
ping google.com  
ping 8.8.8.8  
Проверить маршрут  
traceroute google.com  

Иногда:  

tracepath google.com  
DNS  
nslookup google.com  
dig google.com  
Проверить открытые порты  
ss -tulpn  

Или старый вариант:  

netstat -tulpn  
Проверить, кто слушает порт  
sudo ss -tulpn | grep 8080  
Проверить соединение с портом  
nc -vz host 5432  
nc -vz example.com 443  

Или:  

telnet host 5432  

