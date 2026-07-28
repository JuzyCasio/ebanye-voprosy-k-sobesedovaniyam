# Linux для QA/SDET — полный конспект — часть 4

[← Оглавление](linux.md) · [← К разделу](../infrastructure.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/linux.md)

Темы 33-48.

## 33. Cron

Cron используется для запуска задач по расписанию.  

Открыть crontab  
crontab -e  
Посмотреть  
crontab -l  
Формат  
* * * * * command  
│ │ │ │ │  
│ │ │ │ └── день недели  
│ │ │ └──── месяц  
│ │ └────── день месяца  
│ └──────── час  
└────────── минута  
Примеры  

Каждую минуту:  

* * * * * /path/script.sh  

Каждый день в 03:00:  

0 3 * * * /path/backup.sh  

Каждый понедельник в 10:30:  

30 10 * * 1 /path/script.sh  

## 34. Exit code

После выполнения команды можно посмотреть код завершения:  

echo $?  

Обычно:  

0 — успех  
не 0 — ошибка  

### Пример:

pytest  
echo $?  

В CI/CD это важно: если команда вернула не 0, pipeline обычно падает.  

## 35. Bash-скрипты

Пример простого скрипта  
#!/bin/bash  

echo "Starting tests"  
pytest tests/  
echo "Exit code: $?"  

Сделать исполняемым:  

chmod +x run_tests.sh  

Запустить:  

./run_tests.sh  
Shebang  
#!/bin/bash  

Указывает, каким интерпретатором запускать файл.  

Для Python:  

#!/usr/bin/env python3  

## 36. Важное отличие: source script.sh и ./script.sh

./script.sh  

Запускает скрипт в новом shell-процессе.  

source script.sh  

Выполняет скрипт в текущем shell.  

Это важно для переменных окружения.  

### Пример:

source .env  

## 37. Символические ссылки

Создать symlink  
ln -s /real/path link_name  
Посмотреть  
ls -la  

### Пример:

python -> python3.11  

### Коротко:

Символическая ссылка — это ссылка на другой файл или директорию. Похожа на shortcut.  

## 38. Hard link vs symlink

Hard link:  

указывает на тот же inode;  
работает только в рамках одной файловой системы;  
нельзя обычно делать на директории.  

Symlink:  

отдельный файл-ссылка;  
хранит путь до цели;  
может ссылаться на несуществующий путь;  
может ссылаться на директории.  

## 39. Inode

Inode — структура файловой системы, которая хранит метаданные файла:  

права;  
владельца;  
размер;  
время изменения;  
ссылки на блоки данных.  

Имя файла хранится отдельно в директории.  

Посмотреть inode:  

ls -i  

## 40. Логи авторизации

На Ubuntu/Debian:  

/var/log/auth.log  

Можно смотреть:  

sudo tail -f /var/log/auth.log  

Там можно увидеть SSH-логины, ошибки авторизации и sudo.  

## 41. Firewall

В Linux могут использоваться:  

iptables  
nftables  
ufw  
firewalld  
ufw  
sudo ufw status  
sudo ufw allow 22  
sudo ufw allow 8080  
sudo ufw deny 8080  
iptables  

Посмотреть правила:  

sudo iptables -L -n -v  

## 42. Как проверить, почему порт недоступен

Порядок:  

Проверить, что сервис запущен:  
systemctl status app  
Проверить, слушает ли порт:  
ss -tulpn | grep 8080  
Проверить firewall:  
sudo iptables -L -n -v  
sudo ufw status  
Проверить доступность с клиента:  
nc -vz host 8080  
Проверить bind address.  

Например сервис слушает только localhost:  

127.0.0.1:8080  

Тогда снаружи он будет недоступен.  

Нужно, чтобы слушал:  

0.0.0.0:8080  

или конкретный внешний IP.  

## 43. localhost, 127.0.0.1, 0.0.0.0

localhost / 127.0.0.1  

Это loopback, то есть сам текущий хост.  

0.0.0.0  

Обычно означает “слушать на всех интерфейсах”.  

### На собесе:

Если сервис слушает 127.0.0.1, он доступен только локально. Если слушает 0.0.0.0, он принимает соединения со всех сетевых интерфейсов.  

## 44. DNS в Linux

/etc/hosts  

Локальные соответствия имени и IP:  

cat /etc/hosts  

### Пример:

127.0.0.1 localhost  
10.10.1.20 test-api.local  
/etc/resolv.conf  

DNS-серверы:  

cat /etc/resolv.conf  

Проверить DNS:  

nslookup example.com  
dig example.com  

## 45. Проверка сертификата HTTPS

openssl s_client -connect example.com:443  

Можно проверить:  

срок действия сертификата;  
цепочку;  
CN/SAN;  
ошибки TLS.  

Через curl:  

curl -v https://example.com  

Если нужно временно игнорировать сертификат:  

curl -k https://example.com  

-k использовать осторожно, только для диагностики.  

## 46. Docker и Linux

### На собеседовании по QA часто Linux смешивают с Docker.

Основные команды Docker  
docker ps  
docker ps -a  
docker images  
docker logs container_name  
docker logs -f container_name  
docker exec -it container_name bash  
docker exec -it container_name sh  
docker stop container_name  
docker start container_name  
docker restart container_name  
docker rm container_name  
docker rmi image_name  
Запустить контейнер  
docker run nginx  

С портом:  

docker run -p 8080:80 nginx  

С переменной окружения:  

docker run -e ENV=dev image_name  

С volume:  

docker run -v /host/path:/container/path image_name  

## 47. Docker logs

docker logs container  
docker logs -f container  
docker logs --tail 100 container  

Если контейнер падает:  

docker ps -a  
docker logs container  
docker inspect container  

## 48. Зайти внутрь контейнера

docker exec -it container bash  

Если bash нет:  

docker exec -it container sh  

