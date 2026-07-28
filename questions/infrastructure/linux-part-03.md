# Linux для QA/SDET — полный конспект — часть 3

[← Оглавление](linux.md) · [← К разделу](../infrastructure.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/linux.md)

Темы 19-32.

## 19. Разбор ss -tulpn

ss -tulpn  

Флаги:  

-t — TCP  
-u — UDP  
-l — listening  
-p — process  
-n — не резолвить имена, показывать числа  

### Пример:

LISTEN 0 128 0.0.0.0:5432 0.0.0.0:* users:(("postgres",pid=1234))  

Это значит, что PostgreSQL слушает порт 5432.  

## 20. Частые сетевые вопросы

TCP vs UDP  

TCP:  

устанавливает соединение;  
гарантирует доставку;  
гарантирует порядок пакетов;  
есть retransmission;  
медленнее, но надёжнее.  

Используется:  

HTTP, HTTPS, SSH, PostgreSQL, MySQL  

UDP:  

без установки соединения;  
не гарантирует доставку;  
быстрее;  
подходит для real-time.  

Используется:  

DNS, VoIP, streaming, online games  
Как объяснить коротко  

TCP — надёжный протокол с соединением и гарантией доставки. UDP — быстрый протокол без гарантии доставки, используется там, где важнее скорость, чем идеальная надёжность.  

## 21. Проверка HTTP API из Linux

curl  

GET-запрос:  

curl http://localhost:8080/health  

Показать заголовки:  

curl -i http://localhost:8080/health  

Только заголовки:  

curl -I http://localhost:8080/health  

POST JSON:  

curl -X POST http://localhost:8080/users \  
  -H "Content-Type: application/json" \  
  -d '{"username": "test", "password": "Password123"}'  

С авторизацией:  

curl -H "Authorization: Bearer TOKEN" http://localhost:8080/api/users  

Сохранить ответ:  

curl http://localhost:8080/api/users -o response.json  

Подробный debug:  

curl -v http://localhost:8080/health  

## 22. Диагностика API на стенде

Если API не отвечает:  

curl -v http://host:port/health  

Проверить DNS:  

nslookup host  

Проверить порт:  

nc -vz host port  

Проверить маршрут:  

ping host  
traceroute host  

Проверить, слушает ли сервис порт на сервере:  

ss -tulpn | grep port  

Проверить логи:  

journalctl -u service -f  
tail -f /var/log/app/app.log  

## 23. SSH

Подключиться к серверу  
ssh user@host  

С конкретным портом:  

ssh -p 2222 user@host  

С ключом:  

ssh -i ~/.ssh/id_rsa user@host  
Скопировать файл на сервер  
scp file.txt user@host:/tmp/  

С сервера к себе:  

scp user@host:/tmp/file.txt .  

Рекурсивно директорию:  

scp -r ./project user@host:/opt/  

С портом:  

scp -P 2222 file.txt user@host:/tmp/  

Обрати внимание:  
у ssh порт задаётся через -p,  
у scp через -P.  

## 24. SSH-ключи

Сгенерировать ключ  
ssh-keygen -t rsa -b 4096  

Или современный вариант:  

ssh-keygen -t ed25519  
Публичный ключ  
cat ~/.ssh/id_rsa.pub  

Публичный ключ можно добавлять на сервер / GitLab / GitHub.  

Приватный ключ  
~/.ssh/id_rsa  

Его нельзя никому отправлять.  

Права на приватный ключ:  

chmod 600 ~/.ssh/id_rsa  

## 25. Переменные окружения

Посмотреть переменные  
env  
printenv  
Посмотреть конкретную  
echo $PATH  
echo $HOME  
Задать переменную временно  
export ENV=dev  
export BASE_URL=http://localhost:8080  
Запустить команду с переменной  
BASE_URL=http://localhost:8080 pytest  
Где часто задают переменные  
~/.bashrc  
~/.profile  
/etc/environment  
systemd unit-файлы  
CI/CD настройки  
Docker Compose  

## 26. PATH

PATH — список директорий, где shell ищет исполняемые файлы.  

echo $PATH  

Если команда не находится:  

which python  
which pytest  

Или:  

command -v python3  

Добавить путь:  

export PATH=$PATH:/opt/mytool/bin  

## 27. Архивы

tar.gz распаковать  
tar -xzf archive.tar.gz  
tar.gz создать  
tar -czf archive.tar.gz directory/  
zip  
zip -r archive.zip directory/  
unzip archive.zip  

## 28. Диски и место

Свободное место  
df -h  
Размер директории  
du -sh /var/log  
du -sh *  
Найти самые большие директории  
du -h /var/log | sort -h | tail -n 20  
Информация о блочных устройствах  
lsblk  
Смонтированные файловые системы  
mount  
Монтирование  
mount /dev/sdb1 /mnt  
Размонтирование  
umount /mnt  

## 29. Что делать, если закончилось место

Порядок диагностики:  

df -h  

Потом:  

du -sh /*  

Дальше искать большие директории:  

du -h /var | sort -h | tail -n 20  

### Часто место занимают:

/var/log  
Docker images/containers/volumes  
кэши пакетного менеджера  
старые артефакты CI  
дампы БД  

Для Docker:  

docker system df  
docker system prune  

Осторожно: prune удаляет неиспользуемые ресурсы.  

## 30. Память и CPU

Память  
free -h  
CPU и процессы  
top  
htop  
Информация о CPU  
cat /proc/cpuinfo  
lscpu  
Информация о памяти  
cat /proc/meminfo  

## 31. Load average

В top можно увидеть:  

load average: 0.35, 0.52, 0.60  

Это средняя нагрузка за:  

1 минута, 5 минут, 15 минут  

### Как объяснить:

Load average показывает среднее количество процессов, которые выполняются или ждут CPU/IO. Его нужно оценивать относительно количества CPU-ядер.  

Например:  

4 ядра и load 2.0 — нормально;  
4 ядра и load 20.0 — сильная нагрузка;  
высокий load может быть не только из-за CPU, но и из-за IO.  

## 32. Пакетные менеджеры

Debian/Ubuntu  
apt update  
apt install nginx  
apt remove nginx  
apt search package  
RHEL/CentOS/Fedora  
yum install nginx  
dnf install nginx  
Проверить установленный пакет  

Ubuntu/Debian:  

dpkg -l | grep nginx  

RHEL/CentOS:  

rpm -qa | grep nginx  

