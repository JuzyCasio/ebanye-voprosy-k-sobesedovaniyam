# Linux для QA/SDET — полный конспект

[← Все полные материалы](README.md) · [Короткая шпаргалка](../cheatsheets/linux.md)

Полный материал из присланного конспекта. Сохранены подробные объяснения, примеры, практические сценарии и вопросы для собеседования.

> **Как пользоваться конспектом**
>
> Выбери тему в навигации, прочитай объяснение и затем проговори выделенный короткий ответ своими словами. Код и команды оформлены отдельными блоками, чтобы их можно было быстро найти и скопировать.

## Навигация по разделу

- [Файловая система, права и процессы](#файловая-система-права-и-процессы) — вопросы 1–17
- [Сеть, HTTP и окружение](#сеть-http-и-окружение) — вопросы 18–26
- [Ресурсы, shell и безопасность](#ресурсы-shell-и-безопасность) — вопросы 27–45
- [Docker и CI/CD](#docker-и-cicd) — вопросы 46–53
- [Диагностика и полезные команды](#диагностика-и-полезные-команды) — вопросы 54–69
- [Собеседование и повторение](#собеседование-и-повторение) — вопросы 70–75
- [Дополнительная практика](#дополнительная-практика) — дополнительные разделы

## Файловая система, права и процессы

### 1. Что такое Linux — как отвечать на собеседовании

Linux — это семейство Unix-подобных операционных систем, основанных на ядре Linux.
В контексте QA/SDET Linux важен, потому что на нём обычно работают:

- тестовые стенды;
- CI/CD-агенты;
- Docker-контейнеры;
- backend-сервисы;
- базы данных;
- логирование и мониторинг;
- сетевые утилиты и диагностика.

> **Короткий ответ для собеседования**
>
> Linux для меня — это рабочая среда для запуска сервисов, автотестов, Docker-контейнеров, анализа логов и диагностики проблем. Я умею работать с файлами, правами, процессами, сетью, systemd, ssh, логами и базовой отладкой окружения.

### 2. Файловая система Linux

**Основные директории**

```text
/
```

Корень файловой системы.

/home

Домашние директории пользователей.

/root

Домашняя директория пользователя root.

/etc

Конфигурационные файлы системы и сервисов.

#### Примеры:

```text
/etc/hosts
/etc/resolv.conf
/etc/ssh/sshd_config
/etc/systemd/system/
```

/var

Изменяемые данные: логи, кэш, очереди, данные сервисов.

```text
/var/log
```

Логи системы и приложений.

/tmp

Временные файлы. Обычно очищается системой.

/usr

Пользовательские программы, библиотеки, документация.

/bin
/sbin

```text
/usr/bin
/usr/sbin
```

Исполняемые файлы.

/opt

Дополнительное ПО, установленное отдельно.

/proc

Виртуальная файловая система с информацией о процессах и ядре.

**Например**

```text
/proc/cpuinfo
/proc/meminfo
/proc/<pid>/
```

/dev

Устройства как файлы.

**Например**

```text
/dev/null
/dev/sda
/dev/tty
```

/mnt
/media

Точки монтирования дисков и внешних устройств.

### 3. Навигация и работа с файлами

**Где я нахожусь**

```bash
pwd
```

Показывает текущую директорию.

**Перейти в директорию**

```bash
cd /var/log
cd ~
cd ..
cd -
```

cd - — вернуться в предыдущую директорию.

**Посмотреть файлы**

```bash
ls
ls -l
ls -la
ls -lh
```

-l — подробный вывод;
-a — показать скрытые файлы;
-h — размеры в удобном формате.
Создать файл

```bash
touch file.txt
```

Создать директорию

```bash
mkdir logs
mkdir -p app/logs/archive
```

-p создаёт всю цепочку директорий.

**Копировать**

```bash
cp file.txt copy.txt
cp -r dir1 dir2
```

Переместить / переименовать

```bash
mv old.txt new.txt
mv file.txt /tmp/
```

**Удалить**

```bash
rm file.txt
rm -r dir
rm -rf dir
```

rm -rf — опасная команда. Удаляет рекурсивно и без подтверждения.

### 4. Просмотр файлов

**Полностью вывести файл**

```bash
cat app.log
```

**Смотреть файл постранично**

```bash
less app.log
```

**Внутри less**

/search_text
n
q
/text — поиск;
n — следующее совпадение;
q — выход.
Первые строки файла

```bash
head app.log
head -n 50 app.log
```

**Последние строки файла**

```bash
tail app.log
tail -n 100 app.log
```

Смотреть лог в реальном времени

```bash
tail -f app.log
tail -n 200 -f app.log
```

Это часто используется при запуске тестов или сервиса.

### 5. Поиск файлов и текста

**Найти файл по имени**

```bash
find /path -name "app.log"
find . -name "*.py"
```

**Найти директории**

```bash
find . -type d -name "logs"
```

**Найти файлы**

```bash
find . -type f -name "*.log"
```

**Найти большие файлы**

```bash
find /var/log -type f -size +100M
```

**Найти текст внутри файлов**

```bash
grep "ERROR" app.log
grep -i "error" app.log
grep -r "timeout" .
grep -rn "Exception" .
```

-i — без учёта регистра;
-r — рекурсивно;
-n — показать номера строк.
Полезные grep-команды

```bash
grep "ERROR" app.log | tail -n 20
grep -E "ERROR|WARN|Exception" app.log
grep -v "DEBUG" app.log
```

-E — регулярные выражения;
-v — исключить строки.

### 6. Pipe, redirect, stdin/stdout/stderr

**Pipe**

```bash
ps aux | grep python
```

| передаёт вывод одной команды на вход другой.

**Перезаписать файл**

```bash
echo "hello" > file.txt
```

**Дописать в файл**

```bash
echo "hello" >> file.txt
```

**Перенаправить ошибки**

```bash
command 2> error.log
```

Перенаправить обычный вывод и ошибки

```bash
command > output.log 2>&1
```

**Или современный вариант**

**command &> output.log**
Частый вопрос

Чем отличается `>` от `>>`?

- `>` перезаписывает файл;
- `>>` добавляет данные в конец файла.

### 7. Пользователи и права

**Кто я**

```bash
whoami
id
```

Узнать группы пользователя

```bash
groups
id username
```

Переключиться на другого пользователя

```bash
su - username
```

Выполнить команду от root

```bash
sudo command
```

### 8. Права доступа

#### Пример вывода:

-rwxr-xr--

**Разбор**

-    rwx    r-x    r--
тип  owner  group  others

**Типы**

-  обычный файл
d  директория
l  символическая ссылка

**Права**

- r — read
- w — write
- x — execute
Числовые права

```python
r = 4
w = 2
x = 1
```

#### Примеры:

```bash
chmod 755 script.sh
chmod 644 file.txt
chmod 600 private.key
```

**Расшифровка**

755 = владелец rwx, группа r-x, остальные r-x
644 = владелец rw-, группа r--, остальные r--
600 = владелец rw-, остальные ничего
Сделать файл исполняемым

```bash
chmod +x script.sh
```

**Поменять владельца**

```bash
chown user:group file.txt
chown -R user:group directory
```

### 9. Важный вопрос: права на директорию

**Для директории**

- r — можно посмотреть список файлов
- w — можно создавать/удалять файлы
- x — можно заходить в директорию

#### Что значит execute на директории?

**Ответ**

Для директории x означает право прохода внутрь директории. Без x нельзя зайти в директорию и обратиться к файлам внутри, даже если знаешь их имена.

### 10. Процессы

**Посмотреть процессы**

```bash
ps
ps aux
ps -ef
```

#### Часто:

```bash
ps aux | grep python
ps aux | grep nginx
```

**Интерактивный просмотр**

```bash
top
htop
```

htop удобнее, но может быть не установлен.

**Найти PID процесса**

```bash
pgrep nginx
```

**pidof nginx**
Убить процесс

```bash
kill <pid>
kill -9 <pid>
```

**Убить по имени**

```bash
pkill nginx
killall nginx
```

Сигналы

```bash
kill -15 <pid>
```

SIGTERM — мягкое завершение. Процесс может корректно закрыться.

```bash
kill -9 <pid>
```

SIGKILL — принудительное завершение. Процесс не может его обработать.

```bash
kill -1 <pid>
```

SIGHUP — часто используется для перечитывания конфигурации.

### 11. Как объяснить SIGTERM и SIGKILL

**SIGTERM**

Просит процесс завершиться корректно. Процесс может закрыть соединения, сохранить данные, освободить ресурсы.

**SIGKILL**

Принудительно убивает процесс на уровне ядра. Процесс не успевает выполнить cleanup. Использовать стоит только если обычное завершение не сработало.

### 12. Состояния процессов

В ps aux можно увидеть статусы:

- R — running
- S — sleeping
- D — uninterruptible sleep
- T — stopped
- Z — zombie
Zombie process

Zombie — процесс уже завершился, но его родитель ещё не забрал код завершения.

#### Короткий ответ:

Zombie-процесс не выполняется, но запись о нём остаётся в таблице процессов, пока родительский процесс не вызовет wait. Обычно лечится завершением или перезапуском родительского процесса.

### 13. Фоновые процессы

**Запустить в фоне**

```bash
command &
```

Посмотреть фоновые задачи

```bash
jobs
```

Вернуть задачу на передний план

```bash
fg
```

Продолжить в фоне

```bash
bg
```

Запустить так, чтобы процесс не умер после выхода из терминала

```bash
nohup command &
```

#### Пример:

```bash
nohup python script.py > app.log 2>&1 &
```

### 14. systemd и сервисы

В современных Linux сервисами часто управляет systemd.

**Проверить статус сервиса**

```bash
systemctl status nginx
```

**Запустить сервис**

```bash
sudo systemctl start nginx
```

Остановить сервис

```bash
sudo systemctl stop nginx
```

Перезапустить

```bash
sudo systemctl restart nginx
```

Перечитать конфигурацию

```bash
sudo systemctl reload nginx
```

Включить автозапуск

```bash
sudo systemctl enable nginx
```

Отключить автозапуск

```bash
sudo systemctl disable nginx
```

Проверить, включён ли автозапуск

```bash
systemctl is-enabled nginx
```

**Посмотреть все сервисы**

```bash
systemctl list-units --type=service
```

### 15. Где лежат systemd unit-файлы

#### Часто:

```text
/etc/systemd/system/
```

**Также**

```text
/lib/systemd/system/
/usr/lib/systemd/system/
```

**Пример unit-файла**

```ini
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
```

**После изменения unit-файла**

**sudo systemctl daemon-reload**

```bash
sudo systemctl restart my-service
```

### 16. Логи

```bash
journalctl
```

**Посмотреть логи сервиса**

```bash
journalctl -u nginx
```

**Последние строки**

```bash
journalctl -u nginx -n 100
```

**В реальном времени**

```bash
journalctl -u nginx -f
```

**С текущей загрузки системы**

```bash
journalctl -u nginx -b
```

**За период**

```bash
journalctl --since "1 hour ago"
journalctl --since "2026-07-07 10:00:00"
```

**Логи в файлах**

#### Часто лежат тут:

```text
/var/log/
```

#### Примеры:

```text
/var/log/syslog
/var/log/messages
/var/log/auth.log
/var/log/nginx/access.log
/var/log/nginx/error.log
```

### 17. Как диагностировать “сервис не запустился”

#### Хороший порядок действий:

```bash
systemctl status service_name
```

**Потом**

```bash
journalctl -u service_name -n 100
```

Потом проверить конфиг, права, порты, переменные окружения:

```bash
ls -la
cat config.yml
ss -tulpn
df -h
```

> **Короткий ответ для собеседования**
>
> Сначала смотрю systemctl status, потом journalctl -u service -n 100, проверяю код ошибки, конфиг, права на файлы, занятые порты, доступность зависимостей, переменные окружения и место на диске.

## Сеть, HTTP и окружение

### 18. Сеть: базовые команды

**IP-адреса**

```bash
ip addr
ip a
```

**Старый вариант**

```bash
ifconfig
```

**Маршруты**

```bash
ip route
```

**Проверить доступность хоста**

```bash
ping google.com
ping 8.8.8.8
```

**Проверить маршрут**

```bash
traceroute google.com
```

**Иногда**

```bash
tracepath google.com
```

**DNS**

```bash
nslookup google.com
dig google.com
```

**Проверить открытые порты**

```bash
ss -tulpn
```

**Или старый вариант**

```bash
netstat -tulpn
```

**Проверить, кто слушает порт**

```bash
sudo ss -tulpn | grep 8080
```

Проверить соединение с портом

```bash
nc -vz host 5432
nc -vz example.com 443
```

**Или**

```bash
telnet host 5432
```

### 19. Разбор ss -tulpn

```bash
ss -tulpn
```

**Флаги**

- -t — TCP
- -u — UDP
- -l — listening
- -p — process
- -n — не резолвить имена, показывать числа

#### Пример:

```python
LISTEN 0 128 0.0.0.0:5432 0.0.0.0:* users:(("postgres",pid=1234))
```

Это значит, что PostgreSQL слушает порт 5432.

### 20. Частые сетевые вопросы

**TCP vs UDP**

**TCP**

- устанавливает соединение;
- гарантирует доставку;
- гарантирует порядок пакетов;
- есть retransmission;
- медленнее, но надёжнее.

**Используется**

HTTP, HTTPS, SSH, PostgreSQL, MySQL

**UDP**

- без установки соединения;
- не гарантирует доставку;
- быстрее;
- подходит для real-time.

**Используется**

DNS, VoIP, streaming, online games
Как объяснить коротко

TCP — надёжный протокол с соединением и гарантией доставки. UDP — быстрый протокол без гарантии доставки, используется там, где важнее скорость, чем идеальная надёжность.

### 21. Проверка HTTP API из Linux

```bash
curl
```

**GET-запрос**

```bash
curl http://localhost:8080/health
```

**Показать заголовки**

```bash
curl -i http://localhost:8080/health
```

**Только заголовки**

```bash
curl -I http://localhost:8080/health
```

**POST JSON**

```bash
curl -X POST http://localhost:8080/users \
  -H "Content-Type: application/json" \
  -d '{"username": "test", "password": "Password123"}'
```

**С авторизацией**

```bash
curl -H "Authorization: Bearer TOKEN" http://localhost:8080/api/users
```

**Сохранить ответ**

```bash
curl http://localhost:8080/api/users -o response.json
```

**Подробный debug**

```bash
curl -v http://localhost:8080/health
```

### 22. Диагностика API на стенде

**Если API не отвечает**

```bash
curl -v http://host:port/health
```

**Проверить DNS**

```bash
nslookup host
```

**Проверить порт**

```bash
nc -vz host port
```

**Проверить маршрут**

```bash
ping host
traceroute host
```

Проверить, слушает ли сервис порт на сервере:

```bash
ss -tulpn | grep port
```

**Проверить логи**

```bash
journalctl -u service -f
tail -f /var/log/app/app.log
```

### 23. SSH

**Подключиться к серверу**

```bash
ssh user@host
```

**С конкретным портом**

```bash
ssh -p 2222 user@host
```

**С ключом**

```bash
ssh -i ~/.ssh/id_rsa user@host
```

**Скопировать файл на сервер**

```bash
scp file.txt user@host:/tmp/
```

**С сервера к себе**

```bash
scp user@host:/tmp/file.txt .
```

**Рекурсивно директорию**

```bash
scp -r ./project user@host:/opt/
```

**С портом**

```bash
scp -P 2222 file.txt user@host:/tmp/
```

**Обрати внимание**
у ssh порт задаётся через -p,
у scp через -P.

### 24. SSH-ключи

**Сгенерировать ключ**

```bash
ssh-keygen -t rsa -b 4096
```

**Или современный вариант**

```bash
ssh-keygen -t ed25519
```

Публичный ключ

```bash
cat ~/.ssh/id_rsa.pub
```

Публичный ключ можно добавлять на сервер / GitLab / GitHub.

**Приватный ключ**
~/.ssh/id_rsa

Его нельзя никому отправлять.

**Права на приватный ключ**

```bash
chmod 600 ~/.ssh/id_rsa
```

### 25. Переменные окружения

**Посмотреть переменные**

```bash
env
printenv
```

**Посмотреть конкретную**

```bash
echo $PATH
echo $HOME
```

**Задать переменную временно**

```bash
export ENV=dev
export BASE_URL=http://localhost:8080
```

**Запустить команду с переменной**

```python
BASE_URL=http://localhost:8080 pytest
```

**Где часто задают переменные**
~/.bashrc
~/.profile

```text
/etc/environment
```

**systemd unit-файлы**
CI/CD настройки
Docker Compose

### 26. PATH

PATH — список директорий, где shell ищет исполняемые файлы.

```bash
echo $PATH
```

**Если команда не находится**

**which python**

```bash
which pytest
```

**Или**

```bash
command -v python3
```

**Добавить путь**

```bash
export PATH=$PATH:/opt/mytool/bin
```

## Ресурсы, shell и безопасность

### 27. Архивы

**tar.gz распаковать**

```bash
tar -xzf archive.tar.gz
```

**tar.gz создать**

```bash
tar -czf archive.tar.gz directory/
```

**zip**

```bash
zip -r archive.zip directory/
unzip archive.zip
```

### 28. Диски и место

**Свободное место**

```bash
df -h
```

**Размер директории**

```bash
du -sh /var/log
du -sh *
```

**Найти самые большие директории**

```bash
du -h /var/log | sort -h | tail -n 20
```

**Информация о блочных устройствах**

```bash
lsblk
```

Смонтированные файловые системы

```bash
mount
```

Монтирование

```bash
mount /dev/sdb1 /mnt
```

Размонтирование

```bash
umount /mnt
```

### 29. Что делать, если закончилось место

**Порядок диагностики**

```bash
df -h
```

**Потом**

```bash
du -sh /*
```

**Дальше искать большие директории**

```bash
du -h /var | sort -h | tail -n 20
```

#### Часто место занимают:

```text
/var/log
```

Docker images/containers/volumes
кэши пакетного менеджера
старые артефакты CI
дампы БД

**Для Docker**

```bash
docker system df
docker system prune
```

Осторожно: prune удаляет неиспользуемые ресурсы.

### 30. Память и CPU

**Память**

```bash
free -h
```

**CPU и процессы**

```bash
top
htop
```

**Информация о CPU**

```bash
cat /proc/cpuinfo
```

**lscpu**
Информация о памяти

```bash
cat /proc/meminfo
```

### 31. Load average

**В top можно увидеть**

load average: 0.35, 0.52, 0.60

**Это средняя нагрузка за**

1 минута, 5 минут, 15 минут

#### Как объяснить:

Load average показывает среднее количество процессов, которые выполняются или ждут CPU/IO. Его нужно оценивать относительно количества CPU-ядер.

**Например**

4 ядра и load 2.0 — нормально;
4 ядра и load 20.0 — сильная нагрузка;
высокий load может быть не только из-за CPU, но и из-за IO.

### 32. Пакетные менеджеры

```bash
Debian/Ubuntu
apt update
apt install nginx
apt remove nginx
apt search package
RHEL/CentOS/Fedora
yum install nginx
dnf install nginx
```

**Проверить установленный пакет**

Ubuntu/Debian:

```bash
dpkg -l | grep nginx
```

RHEL/CentOS:

```bash
rpm -qa | grep nginx
```

### 33. Cron

Cron используется для запуска задач по расписанию.

**Открыть crontab**

```bash
crontab -e
```

**Посмотреть**

```bash
crontab -l
```

**Формат**
* * * * * command

```text
│ │ │ │ │
│ │ │ │ └── день недели
│ │ │ └──── месяц
│ │ └────── день месяца
│ └──────── час
└────────── минута
```

**Примеры**

**Каждую минуту**

* * * * * /path/script.sh

Каждый день в 03:00:

0 3 * * * /path/backup.sh

Каждый понедельник в 10:30:

30 10 * * 1 /path/script.sh

### 34. Exit code

После выполнения команды можно посмотреть код завершения:

```bash
echo $?
```

**Обычно**

0 — успех
не 0 — ошибка

#### Пример:

```bash
pytest
echo $?
```

В CI/CD это важно: если команда вернула не 0, pipeline обычно падает.

### 35. Bash-скрипты

**Пример простого скрипта**
#!/bin/bash

```bash
echo "Starting tests"
pytest tests/
echo "Exit code: $?"
```

**Сделать исполняемым**

```bash
chmod +x run_tests.sh
```

**Запустить**

```text
./run_tests.sh
```

**Shebang**
#!/bin/bash

Указывает, каким интерпретатором запускать файл.

**Для Python**

#!/usr/bin/env python3

### 36. Важное отличие: source script.sh и ./script.sh

```text
./script.sh
```

Запускает скрипт в новом shell-процессе.

```bash
source script.sh
```

Выполняет скрипт в текущем shell.

Это важно для переменных окружения.

#### Пример:

```bash
source .env
```

### 37. Символические ссылки

**Создать symlink**

```bash
ln -s /real/path link_name
```

Посмотреть

```bash
ls -la
```

#### Пример:

```bash
python -> python3.11
```

#### Коротко:

Символическая ссылка — это ссылка на другой файл или директорию. Похожа на shortcut.

### 38. Hard link vs symlink

**Hard link**

указывает на тот же inode;
работает только в рамках одной файловой системы;
нельзя обычно делать на директории.

**Symlink**

- отдельный файл-ссылка;
- хранит путь до цели;
- может ссылаться на несуществующий путь;
- может ссылаться на директории.

### 39. Inode

Inode — структура файловой системы, которая хранит метаданные файла:

- права;
- владельца;
- размер;
- время изменения;
- ссылки на блоки данных.

Имя файла хранится отдельно в директории.

**Посмотреть inode**

```bash
ls -i
```

### 40. Логи авторизации

На Ubuntu/Debian:

```text
/var/log/auth.log
```

**Можно смотреть**

```bash
sudo tail -f /var/log/auth.log
```

Там можно увидеть SSH-логины, ошибки авторизации и sudo.

### 41. Firewall

**В Linux могут использоваться**

**iptables**

```bash
nftables
ufw
firewalld
ufw
sudo ufw status
sudo ufw allow 22
sudo ufw allow 8080
sudo ufw deny 8080
iptables
```

**Посмотреть правила**

```bash
sudo iptables -L -n -v
```

### 42. Как проверить, почему порт недоступен

**Порядок**

**Проверить, что сервис запущен**

```bash
systemctl status app
```

**Проверить, слушает ли порт**

```bash
ss -tulpn | grep 8080
```

**Проверить firewall**

```bash
sudo iptables -L -n -v
sudo ufw status
```

Проверить доступность с клиента:

```bash
nc -vz host 8080
```

Проверить bind address.

Например сервис слушает только localhost:

127.0.0.1:8080

Тогда снаружи он будет недоступен.

**Нужно, чтобы слушал**

0.0.0.0:8080

или конкретный внешний IP.

### 43. localhost, 127.0.0.1, 0.0.0.0

localhost / 127.0.0.1

Это loopback, то есть сам текущий хост.

0.0.0.0

Обычно означает “слушать на всех интерфейсах”.

> **Короткий ответ для собеседования**
>
> Если сервис слушает 127.0.0.1, он доступен только локально. Если слушает 0.0.0.0, он принимает соединения со всех сетевых интерфейсов.

### 44. DNS в Linux

```text
/etc/hosts
```

Локальные соответствия имени и IP:

```bash
cat /etc/hosts
```

#### Пример:

127.0.0.1 localhost
10.10.1.20 test-api.local

```text
/etc/resolv.conf
```

**DNS-серверы**

```bash
cat /etc/resolv.conf
```

**Проверить DNS**

```bash
nslookup example.com
dig example.com
```

### 45. Проверка сертификата HTTPS

```bash
openssl s_client -connect example.com:443
```

**Можно проверить**

- срок действия сертификата;
- цепочку;
- CN/SAN;
- ошибки TLS.

**Через curl**

```bash
curl -v https://example.com
```

Если нужно временно игнорировать сертификат:

```bash
curl -k https://example.com
```

-k использовать осторожно, только для диагностики.

## Docker и CI/CD

### 46. Docker и Linux

#### Акцент для собеседования

**Основные команды Docker**

```bash
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
```

**Запустить контейнер**

```bash
docker run nginx
```

**С портом**

```bash
docker run -p 8080:80 nginx
```

**С переменной окружения**

```bash
docker run -e ENV=dev image_name
```

**С volume**

```bash
docker run -v /host/path:/container/path image_name
```

### 47. Docker logs

```bash
docker logs container
docker logs -f container
docker logs --tail 100 container
```

**Если контейнер падает**

```bash
docker ps -a
docker logs container
docker inspect container
```

### 48. Зайти внутрь контейнера

```bash
docker exec -it container bash
```

**Если bash нет**

```bash
docker exec -it container sh
```

### 49. CMD vs ENTRYPOINT

CMD — команда по умолчанию, которую можно легко переопределить.

ENTRYPOINT — основная команда контейнера, обычно фиксирует исполняемый процесс.

#### Пример:

**ENTRYPOINT ["python"]**
CMD ["app.py"]

**Контейнер запустит**

```bash
python app.py
```

### 50. Почему контейнер сразу завершился

Контейнер живёт, пока жив основной процесс.

Если основной процесс завершился — контейнер остановился.

**Диагностика**

```bash
docker ps -a
docker logs container
docker inspect container
```

### 51. Docker Compose

**Запуск**

```bash
docker compose up
docker compose up -d
```

**Остановка**

```bash
docker compose down
```

**Пересобрать**

```bash
docker compose up --build
```

**Логи**

```bash
docker compose logs
docker compose logs -f service_name
```

**Зайти в сервис**

```bash
docker compose exec service_name bash
```

### 52. Linux в CI/CD

В CI/CD часто нужно:

- запускать shell-команды;
- ставить зависимости;
- запускать тесты;
- собирать Docker-образы;
- копировать артефакты;
- работать с переменными окружения;
- анализировать exit code;
- читать логи.

#### Пример шагов:

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pytest tests/ --alluredir=allure-results
```

### 53. Диагностика упавших автотестов на Linux-стенде

**Порядок**

**Проверить, что стенд доступен**

```bash
ping host
nc -vz host port
curl -v http://host:port/health
```

**Проверить переменные окружения**

```bash
env
echo $BASE_URL
```

**Проверить зависимости**

```bash
pip freeze
python --version
pytest --version
```

**Проверить логи приложения**

```bash
journalctl -u app -n 100
tail -f /var/log/app/app.log
```

**Проверить Docker**

```bash
docker ps
docker logs container
```

**Проверить место**

```bash
df -h
```

**Проверить процессы и порты**

```bash
ps aux | grep app
ss -tulpn
```

## Диагностика и полезные команды

### 54. Полезные команды для QA Automation

**Запуск тестов**

```bash
pytest
pytest tests/
pytest tests/test_users.py
pytest -k "login"
pytest -m smoke
pytest -v
pytest -s
pytest --tb=short
```

**Сохранить вывод**

```bash
pytest > test.log 2>&1
```

Запустить и одновременно видеть вывод

```bash
pytest 2>&1 | tee test.log
```

**Найти ошибку в логах**

```bash
grep -i "error" test.log
grep -i "failed" test.log
grep -i "traceback" test.log
```

### 55. tee

**command | tee output.log**

**Позволяет одновременно**

видеть вывод в терминале;
сохранять вывод в файл.

#### Пример:

```bash
pytest -v | tee pytest.log
```

### 56. Команды для диагностики “не работает база”

Допустим PostgreSQL.

**Проверить порт**

```bash
nc -vz db-host 5432
```

**Проверить DNS**

```bash
nslookup db-host
```

**Проверить переменные**

```bash
echo $DB_HOST
echo $DB_PORT
echo $DB_USER
```

**Проверить контейнер**

```bash
docker ps
docker logs postgres
```

**Подключиться**

psql -h db-host -p 5432 -U user -d database

### 57. Разница между ps, top, htop

```bash
ps
```

Показывает snapshot процессов.

```bash
top
```

Интерактивно показывает процессы, CPU, память, load average.

```bash
htop
```

Более удобный интерактивный вариант top.

### 58. Разница между curl, ping, nc

```bash
ping
```

Проверяет сетевую доступность по ICMP. Не гарантирует, что порт приложения открыт.

```bash
nc
```

Проверяет доступность конкретного TCP/UDP-порта.

```bash
curl
```

Проверяет HTTP/HTTPS-уровень: статус-код, заголовки, тело ответа.

> **Короткий ответ для собеседования**
>
> Если ping проходит, это ещё не значит, что API работает. Нужно проверить порт через nc и сам HTTP через curl.

### 59. Как проверить, что процесс слушает порт

```bash
sudo ss -tulpn | grep 8080
```

#### Пример ответа:

```python
LISTEN 0 128 0.0.0.0:8080 users:(("python",pid=1234))
```

Значит Python-процесс слушает порт 8080.

### 60. Как найти процесс по порту

```bash
sudo lsof -i :8080
```

**Или**

```bash
sudo ss -tulpn | grep 8080
```

### 61. Как завершить процесс, который занял порт

**Найти PID**

```bash
sudo lsof -i :8080
```

**Завершить**

```bash
kill <pid>
```

**Если не завершился**

```bash
kill -9 <pid>
```

### 62. Что такое file descriptor

File descriptor — числовой идентификатор открытого ресурса в процессе.

**Стандартные**

- 0 — stdin
- 1 — stdout
- 2 — stderr

#### Пример:

```bash
command > out.log 2> err.log
```

### 63. Что такое /dev/null

```text
/dev/null
```

Специальное устройство, которое “выбрасывает” всё, что в него записали.

#### Пример:

```bash
command > /dev/null 2>&1
```

Это значит: не показывать ни stdout, ни stderr.

### 64. Что такое /proc

/proc — виртуальная файловая система с информацией о процессах и системе.

#### Примеры:

```bash
cat /proc/cpuinfo
cat /proc/meminfo
ls /proc/<pid>
```

**Можно посмотреть окружение процесса**

```bash
cat /proc/<pid>/environ
```

**Открытые файлы процесса**

```bash
ls -la /proc/<pid>/fd
```

### 65. Что такое umask

umask задаёт, какие права будут убраны у новых файлов и директорий.

**Посмотреть**

**umask**

#### Часто:

022

Это значит, что новые файлы обычно будут 644, директории 755.

### 66. Перемещение между серверами и копирование логов

**Скопировать лог с сервера**

```bash
scp user@host:/var/log/app/app.log .
```

Скопировать свой файл на сервер:

```bash
scp ./config.yml user@host:/tmp/
```

**Подключиться и посмотреть лог**

```bash
ssh user@host
tail -f /var/log/app/app.log
```

**Одной командой**

```bash
ssh user@host "tail -n 100 /var/log/app/app.log"
```

### 67. Выполнить команду на удалённой машине

```bash
ssh user@host "hostname && uptime"
```

**Запустить тесты**

```bash
ssh user@host "cd /opt/project && pytest tests/"
```

**С переменной**

```bash
ssh user@host "cd /opt/project && BASE_URL=http://test pytest"
```

### 68. &&, ||, ;

command1 && command2

Выполнить command2, только если command1 успешна.

command1 || command2

Выполнить command2, только если command1 упала.

command1 ; command2

Выполнить обе команды независимо от результата первой.

#### Пример:

```bash
cd project && pytest
```

Если cd project не сработает, pytest не запустится.

### 69. Архитектура диагностики проблемы

#### Когда говорят:

На стенде не работает сервис. Что будешь делать?

**Можно отвечать так**

Я бы шёл по слоям. Сначала проверил, доступен ли хост и порт: ping, nc, curl. Потом проверил бы, запущен ли сервис: systemctl status или docker ps. Потом посмотрел бы логи: journalctl или docker logs. Дальше проверил бы конфиг, переменные окружения, доступность БД/брокера, занятые порты, права на файлы и место на диске.

## Собеседование и повторение

### 70. Типовые вопросы и короткие ответы

Как посмотреть логи сервиса?

```bash
journalctl -u service_name -f
```

**Или файл**

```bash
tail -f /var/log/app/app.log
```

Как посмотреть процессы?

```bash
ps aux
top
htop
```

Как найти процесс по имени?

```bash
ps aux | grep nginx
pgrep nginx
```

Как найти процесс по порту?

```bash
sudo lsof -i :8080
sudo ss -tulpn | grep 8080
```

Как проверить, открыт ли порт?

```bash
nc -vz host port
```

Как проверить HTTP API?

```bash
curl -v http://host:port/health
```

Как проверить свободное место?

```bash
df -h
```

Как найти, что занимает место?

```bash
du -sh *
du -h /var | sort -h | tail
```

Как посмотреть память?

```bash
free -h
```

Как посмотреть нагрузку?

```bash
top
uptime
```

Как убить процесс?

```bash
kill pid
kill -9 pid
```

Как сделать файл исполняемым?

```bash
chmod +x file.sh
```

Как поменять владельца?

```bash
chown user:group file
```

Как подключиться по SSH?

```bash
ssh user@host
```

Как скопировать файл?

```bash
scp file user@host:/tmp/
```

Как посмотреть IP?

```bash
ip a
```

Как посмотреть маршруты?

```bash
ip route
```

Как проверить DNS?

```bash
nslookup host
dig host
```

### 71. Что нужно уметь уверенно сказать на Senior QA/SDET

**Про Linux в целом**

Я использую Linux для работы со стендами, CI/CD и контейнерами. Умею подключаться по SSH, смотреть процессы, логи, сервисы, порты, проверять доступность API и баз, работать с правами, файлами и переменными окружения.

**Про диагностику**

Обычно иду сверху вниз: доступность хоста, порт, HTTP-ответ, статус сервиса, логи, конфиг, зависимости, ресурсы системы.

**Про логи**

Для systemd-сервисов смотрю journalctl -u service, для файловых логов — tail -f, grep, less. В Docker — docker logs.

**Про сеть**

Проверяю IP и маршруты через ip a, ip route, DNS через dig/nslookup, порт через nc или ss, HTTP через curl.

**Про процессы**

Смотрю процессы через ps aux, top, htop; завершаю через kill, стараюсь сначала использовать SIGTERM, а SIGKILL только если процесс не завершается.

**Про права**

Понимаю rwx, chmod, chown, права владельца/группы/остальных, отличие прав на файл и директорию.

### 72. Мини-сценарии для собеседования

Сценарий 1: API недоступен

**Вопрос**

Автотесты падают, API не отвечает. Что делаешь?

**Ответ**

Сначала проверю, корректный ли BASE_URL. Потом curl -v /health. Если нет ответа — проверю DNS через nslookup, доступность порта через nc -vz host port. На сервере посмотрю, запущен ли сервис: systemctl status или docker ps. Потом логи: journalctl -u service -n 100 или docker logs. Также проверю, слушает ли процесс порт через ss -tulpn.

#### Сценарий 2: сервис не стартует

**Ответ**

Проверю systemctl status service, потом journalctl -u service -n 100. Дальше смотрю ошибку: может быть неправильный конфиг, занятый порт, нет прав на файл, не хватает переменных окружения, недоступна БД или закончилось место на диске.

#### Сценарий 3: тесты в CI падают, локально проходят

**Ответ**

Сравню окружения: версии Python, зависимости, переменные окружения, доступность сервисов, права, рабочую директорию, наличие файлов, сетевой доступ из CI-агента. Посмотрю логи job, exit code, артефакты, pytest output. Частая причина — разные env-переменные, разные версии пакетов или недоступные внешние зависимости.

#### Сценарий 4: порт занят

**Ответ**

Найду процесс через lsof -i :port или ss -tulpn | grep port. Потом решу: либо остановить старый процесс, либо поменять порт в конфиге. Завершать лучше сначала обычным kill, а kill -9 использовать только если процесс завис.

#### Сценарий 5: нет места на диске

**Ответ**

Проверю df -h, потом найду крупные директории через du -sh /* и глубже через du -h | sort -h | tail. Часто место занимают логи, Docker images/volumes, дампы, артефакты CI. Удалять буду аккуратно, сначала поняв, что это за файлы.

### 73. Команды, которые стоит выучить прямо обязательно

```bash
pwd
ls -la
cd
cat
less
head
tail -f
grep -rn
find
cp
mv
rm
chmod
chown
ps aux
top
kill
systemctl status
journalctl -u
df -h
du -sh
free -h
ip a
ip route
ping
curl -v
nc -vz
ss -tulpn
ssh
scp
env
echo $?
docker ps
docker logs
docker exec -it
```

### 74. Самая короткая версия для ответа на собесе

Если нужно красиво и уверенно:

В Linux я уверенно работаю на уровне, который нужен QA Automation: подключаюсь к стендам по SSH, запускаю тесты и сервисы, смотрю логи через tail, grep, journalctl, проверяю процессы через ps, top, systemctl, диагностирую сеть через curl, ping, nc, ss, проверяю порты, права, переменные окружения, место на диске. Также работаю с Docker-контейнерами: docker ps, logs, exec, compose. Обычно при проблемах иду по слоям: хост, порт, HTTP, сервис, логи, конфиг, зависимости, ресурсы.

### 75. Что тебе стоит повторить в первую очередь

Для собеседования я бы расставил приоритет так:

Процессы: ps, top, kill, сигналы, zombie.
Логи: tail, grep, journalctl.
Сеть: curl, nc, ss, ping, dig/nslookup.
Права: chmod, chown, rwx, права на директории.
systemd: systemctl status/start/restart, journalctl -u.
Диски: df, du, поиск больших файлов.
SSH/SCP: подключение, копирование, ключи.
Docker на Linux: ps, logs, exec, compose.
CI/CD окружение: env, exit code, запуск pytest.
Диагностические сценарии: API не отвечает, сервис не стартует, порт занят, нет места.

Главное — не просто помнить команды, а уметь объяснить как ты будешь диагностировать проблему по шагам. Это на собесе ценится сильнее, чем знание редких флагов.

## Дополнительная практика

### Дополнение. Работа с файлами и текстом

Разверну именно практическую часть: как смотреть, фильтровать, менять файлы и логи на стенде. Упор сделаю на команды, которые реально пригодятся QA: grep, sed, awk, find, xargs, jq, diff, tail, less.

Ниже — практическая шпаргалка по работе с файлами и текстом в Linux для QA/SDET. Это как раз то, что чаще всего нужно на стендах, в CI/CD, при анализе логов, конфигов, результатов тестов и артефактов.

#### 1. Базовая работа с файлами

Посмотреть, где находишься

```bash
pwd
```

**Посмотреть файлы**

```bash
ls
ls -l
ls -la
ls -lh
```

#### Часто используемый вариант:

```bash
ls -lah
```

**Расшифровка**

-l  подробный вывод
-a  показать скрытые файлы
-h  человекочитаемые размеры: K, M, G
Создать файл

```bash
touch app.log
```

Создать директорию

```bash
mkdir logs
mkdir -p reports/allure-results
```

-p создаёт всю цепочку директорий, если её ещё нет.

**Копировать файл**

```bash
cp app.log app_copy.log
```

**Копировать директорию**

```bash
cp -r reports reports_backup
```

**Переместить или переименовать**

```bash
mv old.log new.log
mv app.log /tmp/
```

**Удалить файл**

```bash
rm app.log
```

**Удалить директорию**

```bash
rm -r reports
```

**Удалить принудительно**

```bash
rm -rf reports
```

rm -rf — опасная команда. На стенде лучше сначала проверить путь через pwd и ls.

#### 2. Просмотр текстовых файлов

cat — вывести весь файл

```bash
cat app.log
```

#### Подходит для маленьких файлов.

#### Плохая идея для огромных логов:

```bash
cat huge.log
```

Лучше использовать less, tail, grep.

less — удобный просмотр большого файла

```bash
less app.log
```

**Внутри less**

/ERROR       поиск слова ERROR
n            следующее совпадение
N            предыдущее совпадение
G            в конец файла
g            в начало файла
q            выйти

**Удобно смотреть большие логи**

```bash
less /var/log/app/app.log
```

head — первые строки файла

```bash
head app.log
```

Первые 50 строк:

```bash
head -n 50 app.log
```

tail — последние строки файла

```bash
tail app.log
```

Последние 100 строк:

```bash
tail -n 100 app.log
```

tail -f — смотреть лог в реальном времени

```bash
tail -f app.log
```

#### Часто для QA:

```bash
tail -n 200 -f app.log
```

**Это значит**

Покажи последние 200 строк и продолжай показывать новые строки.

**Для нескольких файлов**

```bash
tail -f app.log error.log
```

#### 3. grep — поиск текста в файлах

grep — одна из самых важных команд для тестировщика.

**Найти строку в файле**

```bash
grep "ERROR" app.log
```

**Поиск без учёта регистра**

```bash
grep -i "error" app.log
```

**Найдёт**

**error**
ERROR
Error
eRrOr
Показать номера строк

```bash
grep -n "ERROR" app.log
```

**Рекурсивный поиск по директории**

```bash
grep -r "Exception" .
```

**С номерами строк**

```bash
grep -rn "Exception" .
```

**Очень полезно в проекте**

```bash
grep -rn "BASE_URL" .
```

**Искать несколько вариантов**

```bash
grep -E "ERROR|WARN|Exception" app.log
```

-E включает расширенные регулярные выражения.

**Исключить строки**

```bash
grep -v "DEBUG" app.log
```

Например, посмотреть лог без debug:

```bash
grep -v "DEBUG" app.log | less
```

Посмотреть строки до и после совпадения

3 строки после:

```bash
grep -A 3 "ERROR" app.log
```

3 строки до:

```bash
grep -B 3 "ERROR" app.log
```

3 строки до и после:

```bash
grep -C 3 "ERROR" app.log
```

Очень полезно при анализе stacktrace:

```bash
grep -C 10 "Traceback" test.log
```

**Найти только количество совпадений**

```bash
grep -c "ERROR" app.log
```

**Например**

```bash
grep -c "500 Internal Server Error" access.log
```

Найти файлы, где есть совпадение

```bash
grep -rl "DB_HOST" .
```

Только имена файлов.

Найти файлы, где нет совпадения

```bash
grep -rL "pytest" .
```

#### 4. Практические grep-сценарии для QA

Найти ошибки в логе

```bash
grep -i "error" app.log
```

Найти ошибки, warning и exception

```bash
grep -Ei "error|warn|exception|traceback|failed" app.log
```

**Найти падения автотестов**

```bash
grep -Ei "failed|error|assert|traceback" pytest.log
```

Найти HTTP 500 в access.log

```bash
grep " 500 " access.log
```

**Найти запросы конкретного пользователя**

```bash
grep "user_id=12345" app.log
```

**Найти конкретный request_id**

```bash
grep "request_id=abc-123" app.log
```

Смотреть лог в реальном времени и фильтровать ошибки

```bash
tail -f app.log | grep -i "error"
```

**С несколькими словами**

```bash
tail -f app.log | grep -Ei "error|exception|failed"
```

#### 5. sed — поиск и замена текста

```bash
sed — stream editor.
```

Он умеет читать текстовый поток, менять строки, удалять строки, печатать нужные участки.

Для QA sed полезен, когда нужно:

- быстро заменить URL в конфиге;
- подменить параметр окружения;
- удалить лишние строки;
- вытащить часть лога;
- подготовить тестовые данные;
- заменить значения в .env, .yaml, .json-подобных файлах;
- поправить временный конфиг в CI.

#### 6. Базовый синтаксис sed

**Общий вид**

**sed 'команда' file.txt**

**Например**

```bash
sed 's/dev/stage/' config.env
```

s означает substitute, то есть заменить.

#### 7. Замена текста через sed

Заменить первое совпадение в каждой строке

```bash
sed 's/dev/stage/' config.env
```

**Если строка такая**

```python
ENV=dev dev dev
```

**Результат будет**

```python
ENV=stage dev dev
```

Поменяется только первое совпадение в строке.

Заменить все совпадения в строке

```bash
sed 's/dev/stage/g' config.env
```

g означает global.

**Теперь**

```python
ENV=dev dev dev
```

**станет**

```python
ENV=stage stage stage
```

**Заменить URL**

```bash
sed 's|http://dev-api.local|http://stage-api.local|g' config.env
```

Здесь вместо / используется |.

Почему?

Потому что URL содержит /, и так команда читается проще.

#### Плохо читается:

```bash
sed 's/http:\/\/dev-api.local/http:\/\/stage-api.local/g' config.env
```

**Лучше**

```bash
sed 's|http://dev-api.local|http://stage-api.local|g' config.env
```

#### 8. sed -i — изменить файл на месте

Обычный sed только выводит результат в терминал:

```bash
sed 's/dev/stage/g' config.env
```

Файл не меняется.

**Чтобы изменить файл**

```bash
sed -i 's/dev/stage/g' config.env
```

**Безопасный вариант с backup**

```bash
sed -i.bak 's/dev/stage/g' config.env
```

**Появятся два файла**

**config.env**
config.env.bak

Это полезно на стенде, чтобы можно было откатиться.

Важный нюанс macOS vs Linux

**На Linux**

```bash
sed -i 's/dev/stage/g' config.env
```

На macOS часто нужно так:

```bash
sed -i '' 's/dev/stage/g' config.env
```

#### Акцент для собеседования

```bash
sed -i 's/old/new/g' file
```

#### 9. Заменить строку целиком через sed

**Допустим есть .env**

```python
BASE_URL=http://dev.local
DB_HOST=localhost
ENV=dev
```

Нужно заменить строку с BASE_URL.

```bash
sed -i 's|^BASE_URL=.*|BASE_URL=http://stage.local|' .env
```

**Разбор**

^BASE_URL=  строка начинается с BASE_URL=
.*          любое значение после =

Это очень полезная команда для CI/CD.

**Заменить DB_HOST**

```bash
sed -i 's|^DB_HOST=.*|DB_HOST=postgres|' .env
```

**Заменить ENV**

```bash
sed -i 's|^ENV=.*|ENV=stage|' .env
```

#### 10. Удаление строк через sed

Удалить строки, содержащие DEBUG

```bash
sed '/DEBUG/d' app.log
```

Файл не изменится, результат будет выведен в консоль.

**Изменить файл**

```bash
sed -i '/DEBUG/d' app.log
```

**Удалить пустые строки**

```bash
sed '/^$/d' file.txt
```

**Разбор**

^  начало строки
$  конец строки
^$ пустая строка
d  delete
Удалить строки с комментариями

```bash
sed '/^#/d' config.env
```

Удалит строки, которые начинаются с #.

Удалить комментарии и пустые строки:

```bash
sed '/^#/d; /^$/d' config.env
```

#### 11. Печать нужных строк через sed

Напечатать конкретную строку

```bash
sed -n '10p' app.log
```

-n отключает обычный вывод, p печатает нужное.

**Напечатать диапазон строк**

```bash
sed -n '10,20p' app.log
```

Покажет строки с 10 по 20.

Напечатать от совпадения до совпадения

```bash
sed -n '/START/,/END/p' app.log
```

Покажет блок от строки с START до строки с END.

#### Практический пример:

```bash
sed -n '/Traceback/,/AssertionError/p' pytest.log
```

#### 12. Добавление текста через sed

Добавить строку после совпадения

```bash
sed '/BASE_URL/a TIMEOUT=30' .env
```

a — append, добавить после строки.

**Добавить строку перед совпадением**

```bash
sed '/BASE_URL/i ENV=stage' .env
```

i — insert, добавить перед строкой.

#### 13. Замена только в строках с условием

Допустим нужно заменить false на true, но только в строке с FEATURE_LOGIN.

```bash
sed '/FEATURE_LOGIN/s/false/true/' config.env
```

#### Пример:

```python
FEATURE_LOGIN=false
FEATURE_PAYMENT=false
```

**После команды**

```python
FEATURE_LOGIN=true
FEATURE_PAYMENT=false
```

#### 14. Частые sed команды для QA

Быстро поменять стенд в конфиге

```bash
sed -i 's|dev-api.company.local|stage-api.company.local|g' config.yml
```

**Поменять base_url в .env**

```bash
sed -i 's|^BASE_URL=.*|BASE_URL=http://stage.local|' .env
```

**Убрать DEBUG-строки из лога**

```bash
sed '/DEBUG/d' app.log > app_without_debug.log
```

Достать кусок лога по строкам

```bash
sed -n '100,200p' app.log
```

Достать кусок лога между двумя событиями

```bash
sed -n '/test_login_start/,/test_login_end/p' pytest.log
```

**Удалить пустые строки**

```bash
sed '/^$/d' file.txt
```

Удалить комментарии и пустые строки

```bash
sed '/^#/d; /^$/d' config.env
```

#### 15. awk — работа с колонками

awk удобен, когда файл похож на таблицу или лог с колонками.

#### Пример:

2026-07-09 12:00:01 INFO user_id=123 status=200
2026-07-09 12:00:02 ERROR user_id=456 status=500
Напечатать первую колонку

```bash
awk '{print $1}' app.log
```

Напечатать первую и третью колонку

```bash
awk '{print $1, $3}' app.log
```

**Фильтр по значению**

```bash
awk '$3 == "ERROR" {print $0}' app.log
```

$0 — вся строка.

#### 16. awk с разделителем

**Допустим CSV**

**id,name,status**
1,Alex,active
2,John,blocked

#### Команда:

```bash
awk -F',' '{print $1, $3}' users.csv
```

-F',' задаёт разделитель.

Найти строки с HTTP 500 в access.log

Если статус — 9-я колонка:

```bash
awk '$9 == 500 {print $0}' access.log
```

Посчитать количество 500

```bash
awk '$9 == 500 {count++} END {print count}' access.log
```

Посчитать количество ответов по статусам

```bash
awk '{print $9}' access.log | sort | uniq -c | sort -nr
```

#### 17. cut — вытащить колонку

cut проще, чем awk, если нужно просто разрезать строки по разделителю.

Взять первую колонку по :

```bash
cut -d':' -f1 /etc/passwd
```

**Взять второй столбец CSV**

```bash
cut -d',' -f2 users.csv
```

Взять первый и третий столбец

```bash
cut -d',' -f1,3 users.csv
```

#### 18. sort — сортировка

Отсортировать строки

```bash
sort file.txt
```

**Числовая сортировка**

```bash
sort -n numbers.txt
```

**Обратная сортировка**

```bash
sort -r file.txt
```

**Числовая обратная**

```bash
sort -nr numbers.txt
```

#### 19. uniq — уникальные строки

#### Важно: uniq работает нормально только с соседними дублями, поэтому обычно используют вместе с sort.

```bash
sort file.txt | uniq
```

**Посчитать количество повторов**

```bash
sort file.txt | uniq -c
```

**Отсортировать по количеству повторов**

```bash
sort file.txt | uniq -c | sort -nr
```

#### Практический пример — самые частые ошибки:

```bash
grep -i "error" app.log | sort | uniq -c | sort -nr
```

#### 20. wc — подсчёт строк, слов, байт

Количество строк

```bash
wc -l app.log
```

**Количество слов**

```bash
wc -w file.txt
```

**Размер в байтах**

```bash
wc -c file.txt
```

**Посчитать количество ошибок**

```bash
grep -i "error" app.log | wc -l
```

#### 21. find — поиск файлов

Найти файл по имени

```bash
find . -name "config.yml"
```

**Найти все .log**

```bash
find . -name "*.log"
```

**Найти только файлы**

```bash
find . -type f -name "*.log"
```

**Найти только директории**

```bash
find . -type d -name "reports"
```

Найти файлы больше 100 MB

```bash
find . -type f -size +100M
```

Найти файлы, изменённые за последние сутки

```bash
find . -type f -mtime -1
```

Найти файлы старше 7 дней

```bash
find . -type f -mtime +7
```

**Найти пустые файлы**

```bash
find . -type f -empty
```

#### 22. find + действия

Удалить старые логи

```bash
find ./logs -type f -name "*.log" -mtime +7 -delete
```

**Осторожно. Лучше сначала проверить**

```bash
find ./logs -type f -name "*.log" -mtime +7
```

**И только потом**

```bash
find ./logs -type f -name "*.log" -mtime +7 -delete
```

**Найти и выполнить команду**

```bash
find . -type f -name "*.log" -exec grep -H "ERROR" {} \;
```

Но часто удобнее через xargs.

#### 23. xargs — передать список файлов в команду

Найти ошибки во всех логах

```bash
find . -type f -name "*.log" | xargs grep -i "error"
```

Безопаснее для файлов с пробелами

```bash
find . -type f -name "*.log" -print0 | xargs -0 grep -i "error"
```

**Удалить найденные файлы**

```bash
find . -type f -name "*.tmp" | xargs rm
```

**Безопаснее**

```bash
find . -type f -name "*.tmp" -print0 | xargs -0 rm
```

#### 24. jq — работа с JSON

Для тестировщика jq очень полезен: API часто возвращает JSON.

**Красиво вывести JSON**

```bash
cat response.json | jq
```

**Или**

**jq . response.json**
Вытащить поле

```bash
jq '.id' response.json
jq '.user.name' response.json
```

Вытащить массив

```bash
jq '.items' response.json
```

Вытащить первый элемент массива

```bash
jq '.items[0]' response.json
```

Вытащить поле у каждого элемента

```bash
jq '.items[].id' response.json
```

Фильтр по значению

```bash
jq '.items[] | select(.status == "active")' response.json
```

Вывести только id активных пользователей

```bash
jq '.items[] | select(.status == "active") | .id' response.json
```

#### 25. curl + jq

Очень частый QA-сценарий.

GET и красиво вывести JSON

```bash
curl -s http://localhost:8080/users | jq
```

**Вытащить статус пользователя**

```bash
curl -s http://localhost:8080/users/123 | jq '.status'
```

**Проверить количество элементов**

```bash
curl -s http://localhost:8080/users | jq '.items | length'
```

**Получить токен из ответа**

```bash
TOKEN=$(curl -s -X POST http://localhost:8080/login \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"Password123"}' \
  | jq -r '.token')
```

-r убирает кавычки из строки.

#### 26. diff — сравнение файлов

Сравнить два файла

```bash
diff old.txt new.txt
```

Удобный формат

```bash
diff -u old.txt new.txt
```

-u — unified diff, часто используется в Git.

**Сравнить директории**

```bash
diff -r dir1 dir2
```

Только отличающиеся файлы

```bash
diff -qr dir1 dir2
```

Полезно для сравнения артефактов, конфигов, отчётов.

#### 27. comm — сравнение отсортированных списков

**Допустим есть два файла**

```text
expected.txt
actual.txt
```

**Сначала сортируем**

```bash
sort expected.txt > expected_sorted.txt
sort actual.txt > actual_sorted.txt
```

**Потом**

**comm expected_sorted.txt actual_sorted.txt**

**Колонки**

- 1-я колонка — только в первом файле
- 2-я колонка — только во втором файле
- 3-я колонка — есть в обоих

Только строки, которых нет во втором файле:

```bash
comm -23 expected_sorted.txt actual_sorted.txt
```

Только лишние строки во втором:

```bash
comm -13 expected_sorted.txt actual_sorted.txt
```

#### 28. tee — вывести и сохранить одновременно

```bash
pytest -v | tee pytest.log
```

Ты одновременно видишь вывод и сохраняешь его в файл.

**С ошибками тоже**

```bash
pytest -v 2>&1 | tee pytest.log
```

#### 29. Перенаправления вывода

Перезаписать файл

```bash
command > output.log
```

Дозаписать в файл

```bash
command >> output.log
```

Ошибки в отдельный файл

```bash
command 2> error.log
```

stdout и stderr в один файл

```bash
command > output.log 2>&1
```

**Или**

**command &> output.log**
Ничего не выводить

```bash
command > /dev/null 2>&1
```

#### 30. watch — периодически выполнять команду

Обновлять команду каждые 2 секунды

```bash
watch "docker ps"
```

Смотреть свободное место

```bash
watch "df -h"
```

Смотреть количество строк в логе

```bash
watch "wc -l app.log"
```

Смотреть, поднялся ли сервис

```bash
watch "curl -s -o /dev/null -w '%{http_code}' http://localhost:8080/health"
```

#### 31. file, stat, du

Определить тип файла

```bash
file app.log
file archive.tar.gz
file script.sh
```

Метаданные файла

```bash
stat app.log
```

**Покажет**

- размер;
- права;
- владельца;
- дату изменения;
- inode.
Размер файла или директории

```bash
du -sh app.log
du -sh reports/
```

#### 32. tar, gzip, zip

Распаковать .tar.gz

```bash
tar -xzf archive.tar.gz
```

**Создать .tar.gz**

```bash
tar -czf reports.tar.gz reports/
```

**Посмотреть содержимое архива**

```bash
tar -tzf archive.tar.gz
```

**Распаковать zip**

```bash
unzip archive.zip
```

Создать zip

```bash
zip -r reports.zip reports/
```

#### 33. Работа с кодировками и переносами строк

Иногда тесты падают из-за Windows-переносов строк.

**Посмотреть скрытые символы**

```bash
cat -A file.txt
```

Если видишь ^M, значит могут быть Windows-переносы строк.

**Конвертировать Windows → Linux**

```bash
dos2unix file.txt
```

Если dos2unix нет:

```bash
sed -i 's/\r$//' file.txt
```

**Конвертация кодировки**

```bash
iconv -f WINDOWS-1251 -t UTF-8 input.txt > output.txt
```

Полезно, если логи или выгрузки в странной кодировке.

#### 34. Практика: анализ логов

Допустим есть app.log.

**Найти все ошибки**

```bash
grep -i "error" app.log
```

**Найти ошибки и сохранить**

```bash
grep -i "error" app.log > errors.log
```

**Посчитать ошибки**

```bash
grep -i "error" app.log | wc -l
```

**Найти уникальные ошибки**

```bash
grep -i "error" app.log | sort | uniq -c | sort -nr
```

Смотреть ошибки в реальном времени

```bash
tail -f app.log | grep -i "error"
```

**Найти ошибку с контекстом**

```bash
grep -C 5 "NullPointerException" app.log
```

Достать лог за конкретный тест

```bash
sed -n '/test_create_user/,/test_create_user finished/p' app.log
```

#### 35. Практика: работа с .env

**Файл**

```python
BASE_URL=http://dev.local
DB_HOST=localhost
DB_PORT=5432
ENV=dev
DEBUG=true
```

Посмотреть без комментариев и пустых строк

```bash
sed '/^#/d; /^$/d' .env
```

**Поменять BASE_URL**

```bash
sed -i 's|^BASE_URL=.*|BASE_URL=http://stage.local|' .env
```

**Поменять DB_HOST**

```bash
sed -i 's|^DB_HOST=.*|DB_HOST=postgres|' .env
```

**Вытащить значение BASE_URL**

```bash
grep "^BASE_URL=" .env | cut -d'=' -f2
```

**Или через awk**

```bash
awk -F'=' '/^BASE_URL=/ {print $2}' .env
```

#### 36. Практика: найти нужный конфиг в проекте

Найти все env-файлы

```bash
find . -type f -name "*.env"
```

Найти все yaml/yml

```bash
find . -type f \( -name "*.yml" -o -name "*.yaml" \)
```

**Найти, где используется BASE_URL**

```bash
grep -rn "BASE_URL" .
```

**Найти, где прописан dev-стенд**

```bash
grep -rn "dev-api" .
```

Массово заменить dev на stage в yaml-файлах

**Сначала проверить**

```bash
grep -rn "dev-api.company.local" .
```

**Потом заменить**

```bash
find . -type f \( -name "*.yml" -o -name "*.yaml" \) \
  -print0 | xargs -0 sed -i 's|dev-api.company.local|stage-api.company.local|g'
```

#### 37. Практика: работа с результатами pytest

Запустить тесты и сохранить лог

```bash
pytest -v 2>&1 | tee pytest.log
```

**Найти failed**

```bash
grep -i "failed" pytest.log
```

**Найти traceback**

```bash
grep -n "Traceback" pytest.log
```

**Найти assertion**

```bash
grep -i "assert" pytest.log
```

**Найти блок ошибки**

```bash
grep -C 20 "AssertionError" pytest.log
```

**Посчитать количество failed**

```bash
grep -c "FAILED" pytest.log
```

#### 38. Практика: HTTP access.log

#### Пример access.log условно:

10.0.0.1 - - [09/Jul/2026:12:00:01] "GET /users HTTP/1.1" 200
10.0.0.2 - - [09/Jul/2026:12:00:02] "POST /login HTTP/1.1" 500
Найти все 500

```bash
grep " 500" access.log
```

Посчитать 500

```bash
grep " 500" access.log | wc -l
```

**Посчитать статусы**

```bash
awk '{print $9}' access.log | sort | uniq -c | sort -nr
```

**Топ IP-адресов**

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

**Топ URL**

```bash
awk '{print $7}' access.log | sort | uniq -c | sort -nr | head
```

#### 39. Полезные пайплайны команд

Найти топ ошибок

```bash
grep -i "error" app.log | sort | uniq -c | sort -nr | head
```

**Найти все большие логи**

```bash
find /var/log -type f -name "*.log" -size +100M
```

**Посмотреть последние ошибки**

```bash
grep -i "error" app.log | tail -n 20
```

Посмотреть ошибки в большом файле через less

```bash
grep -i "error" app.log | less
```

Найти файлы, где есть password

```bash
grep -rn "password" .
```

**Найти файлы, изменённые сегодня**

```bash
find . -type f -mtime -1
```

Найти и удалить старые allure-results

```bash
find . -type d -name "allure-results"
```

**Удалить осторожно**

```bash
find . -type d -name "allure-results" -prune -exec rm -rf {} \;
```

#### 40. Что обычно нужно тестировщику в Linux

##### 1. Смотреть логи

```bash
tail -f app.log
less app.log
grep -i "error" app.log
journalctl -u service -f
docker logs -f container
```

##### 2. Искать конфиги

```bash
find . -name "*.yml"
find . -name ".env"
grep -rn "BASE_URL" .
```

##### 3. Менять конфиги

```bash
sed -i 's|^BASE_URL=.*|BASE_URL=http://stage.local|' .env
```

##### 4. Проверять результаты тестов

```bash
pytest -v 2>&1 | tee pytest.log
grep -i "failed" pytest.log
```

##### 5. Сравнивать файлы

```bash
diff -u expected.json actual.json
```

##### 6. Работать с JSON

```bash
curl -s http://host/api/users | jq
```

**jq '.items[].id' response.json**

##### 7. Искать большие файлы

```bash
du -sh *
find . -type f -size +100M
```

##### 8. Чистить старые артефакты

```bash
find ./reports -type f -mtime +7 -delete
```

##### 9. Копировать файлы со стенда

```bash
scp user@host:/var/log/app/app.log .
```

##### 10. Смотреть права и владельца

```bash
ls -la
chmod +x script.sh
chown user:group file
```

#### 41. Как красиво объяснить на собеседовании

**Можно сказать так**

В Linux я часто работаю с текстовыми файлами, логами и конфигами. Для просмотра использую less, tail, head, для поиска — grep, для замены и правки конфигов — sed, для работы с колонками — awk и cut, для сортировки и агрегации — sort, uniq, wc. Для поиска файлов использую find, для массовой обработки — xargs. JSON-ответы API удобно проверяю через jq. При анализе проблем обычно смотрю логи, фильтрую ошибки, считаю частотность, сравниваю expected/actual через diff, а при необходимости быстро меняю параметры окружения в .env или yaml через sed.

#### 42. Самый важный минимум команд

Вот это прям стоит знать уверенно:

```bash
cat file
less file
head -n 50 file
tail -n 100 file
tail -f file

grep "ERROR" file
grep -i "error" file
grep -rn "text" .
grep -C 5 "Exception" app.log

sed 's/old/new/g' file
sed -i 's/old/new/g' file
sed -n '10,20p' file
sed '/DEBUG/d' file

awk '{print $1}' file
awk -F',' '{print $2}' file

cut -d',' -f1 file
sort file
uniq -c
wc -l file

find . -name "*.log"
find . -type f -size +100M
find . -type f -mtime +7
```

**xargs**

```bash
jq .
diff -u old new
```

```bash
tee
```

#### 43. Мини-блок именно по sed, который стоит выучить

##### Заменить первое совпадение в строке

```bash
sed 's/old/new/' file
```

##### Заменить все совпадения

```bash
sed 's/old/new/g' file
```

##### Изменить файл на месте

```bash
sed -i 's/old/new/g' file
```

##### Изменить файл и создать backup

```bash
sed -i.bak 's/old/new/g' file
```

##### Заменить строку с `BASE_URL`

```bash
sed -i 's|^BASE_URL=.*|BASE_URL=http://stage.local|' .env
```

##### Удалить строки с `DEBUG`

```bash
sed '/DEBUG/d' app.log
```

##### Удалить пустые строки

```bash
sed '/^$/d' file
```

##### Вывести строки 10–20

```bash
sed -n '10,20p' file
```

##### Вывести блок между `START` и `END`

```bash
sed -n '/START/,/END/p' file
```

##### Заменить только в строках с `FEATURE_LOGIN`

```bash
sed '/FEATURE_LOGIN/s/false/true/' config.env
```

#### 44. Мини-блок “что я делаю, если надо быстро проанализировать лог”

##### Последние строки

```bash
tail -n 200 app.log
```

##### Смотреть в реальном времени

```bash
tail -f app.log
```

##### Ошибки

```bash
grep -Ei "error|exception|failed|traceback" app.log
```

##### Ошибки с контекстом

```bash
grep -C 10 "Exception" app.log
```

##### Количество ошибок

```bash
grep -Ei "error|exception" app.log | wc -l
```

##### Уникальные ошибки

```bash
grep -Ei "error|exception" app.log | sort | uniq -c | sort -nr
```

##### Сохранить ошибки отдельно

```bash
grep -Ei "error|exception" app.log > errors.log
```

#### 45. Что могут спросить на собеседовании

Чем cat отличается от less?

cat выводит весь файл сразу.
less позволяет удобно смотреть большой файл постранично, искать внутри и не грузить весь вывод в терминал.

Как посмотреть лог в реальном времени?

```bash
tail -f app.log
```

**Или для systemd-сервиса**

```bash
journalctl -u service -f
```

Как найти ошибку в большом логе?

```bash
grep -i "error" app.log
```

**С контекстом**

```bash
grep -C 5 "error" app.log
```

Как заменить значение в конфиге?

```bash
sed -i 's|^BASE_URL=.*|BASE_URL=http://stage.local|' .env
```

Как удалить пустые строки?

```bash
sed '/^$/d' file.txt
```

Как посчитать количество строк в файле?

```bash
wc -l file.txt
```

Как найти все .log файлы?

```bash
find . -type f -name "*.log"
```

Как найти все большие файлы?

```bash
find . -type f -size +100M
```

Как найти уникальные ошибки?

```bash
grep -i "error" app.log | sort | uniq -c | sort -nr
```

Как красиво посмотреть JSON?

```bash
cat response.json | jq
```

**Или**

**jq . response.json**

#### 46. Самый собеседовательный ответ

**Если тебя спросят**

Какими Linux-командами для работы с файлами и текстом вы пользуетесь?

**Можно ответить так**

Чаще всего использую less, tail, grep, sed, awk, find, xargs, sort, uniq, wc, jq, diff. Например, логи смотрю через tail -f и less, ошибки фильтрую через grep -i или grep -E, конфиги могу быстро поправить через sed -i, JSON-ответы API смотрю через jq, файлы ищу через find, а для массовой обработки использую find вместе с xargs. Для анализа логов часто комбинирую команды через pipe: grep | sort | uniq -c | sort -nr.

Главное для собеса — показать, что ты не просто знаешь команды, а умеешь собрать из них цепочку для реальной задачи: найти лог, отфильтровать ошибку, посчитать количество, достать контекст, сравнить результат и при необходимости быстро поправить конфиг.

### Дополнение. Диагностика проблем на сервере

Да. Если ты уже выучил работу с файлами, grep, sed, awk, find, jq, tail, less, то для собеседования по Linux тебе ещё обязательно нужны процессы, сервисы, сеть, права, ресурсы системы, Docker и диагностика.

#### Что ещё нужно знать по Linux

#### 1. Процессы

#### Основные команды:

```bash
ps aux
top
htop
pgrep service_name
kill PID
kill -9 PID
```

**Нужно понимать**

- что такое PID;
- чем процесс отличается от сервиса;
- SIGTERM и SIGKILL;
- как найти процесс по имени;
- как найти процесс, который занял порт.

```bash
ps aux | grep python
pgrep -af python
lsof -i :8080
ss -ltnp | grep 8080
```

> **Короткий ответ для собеседования**
>
> Сначала процесс стоит завершать через обычный kill, то есть SIGTERM, чтобы он корректно освободил ресурсы. kill -9 использую только если процесс завис и не реагирует.

#### 2. Управление сервисами

```bash
systemctl status nginx
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx
```

**Логи сервиса**

```bash
journalctl -u nginx
journalctl -u nginx -n 100
journalctl -u nginx -f
```

#### Важно понимать разницу:

restart — полностью перезапустить;
reload — перечитать конфигурацию без полного перезапуска;
status — посмотреть состояние и последние ошибки.

#### 3. Сеть

#### Основные команды:

```bash
ip a
ip route
ping host
nslookup host
dig host
nc -vz host 8080
curl -v http://host:8080/health
ss -ltnp
```

Нужно понимать, что проверяет каждая команда:

- ping — доступность хоста по ICMP;
- nc — доступность конкретного порта;
- curl — работа приложения на уровне HTTP;
- ss — какие порты слушает сервер.

```bash
dig / nslookup — DNS;
```

ip route — маршрутизация.

#### Хорошая фраза:

Прохождение ping ещё не означает, что сервис работает. Нужно отдельно проверить порт через nc и HTTP-ответ через curl.

#### 4. Права доступа

```bash
ls -la
chmod 755 script.sh
chmod +x script.sh
chown user:group file
```

**Нужно знать**

- r — чтение
- w — запись
- x — выполнение

**И числовые права**

- 644 — владелец читает и пишет, остальные только читают
- 755 — владелец всё, остальные читают и выполняют
- 600 — доступ только владельцу

#### 5. Ресурсы сервера

```bash
df -h
du -sh *
free -h
top
uptime
```

**Нужно уметь проверить**

- закончилось ли место на диске;
- закончилась ли память;
- перегружен ли CPU;
- есть ли высокий load average;
- какие процессы потребляют ресурсы.

**Полезно**

```bash
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

#### 6. Переменные окружения

```bash
env
printenv
echo $BASE_URL
echo $PATH
```

Очень часто проблема на стенде связана с неправильным окружением:

```bash
echo $DB_HOST
echo $DB_PORT
echo $ENV
```

#### 7. Exit code

```bash
echo $?
```

**Нужно знать**

0 — успешное выполнение
не 0 — ошибка

Особенно важно для CI/CD.

#### 8. Docker

**Минимум**

```bash
docker ps
docker ps -a
docker logs container
docker logs -f container
docker inspect container
docker exec -it container sh
docker restart container
```

**Для диагностики**

```bash
docker ps -a
docker logs --tail 200 container
docker inspect container
```

**Главная логика диагностики**

#### Акцент для собеседования

**Удобная схема**

**Клиент**
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

#### Сценарий 1. Сервер, много сервисов, запрос возвращает 500

Что означает 500

HTTP 500 — сервер получил запрос, но во время обработки произошла внутренняя ошибка.

**Это обычно значит**

- приложение упало на исключении;
- недоступна БД;
- недоступен другой сервис;
- неправильные данные или конфигурация;
- отсутствует переменная окружения;
- закончилась память или место;
- проблема с правами;
- ошибка после деплоя;
- таймаут зависимости.
Как диагностировать по шагам

##### Шаг 1. Повторить запрос вручную

```bash
curl -v http://host/api/users
```

**Для POST**

```bash
curl -v -X POST http://host/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Alex"}'
```

**Проверяем**

- действительно ли статус 500;
- тело ответа;
- заголовки;
- request ID / trace ID;
- точно ли запрос отправлен на нужный стенд;
- правильные ли данные и заголовки.

**Если есть request ID**

X-Request-ID: abc-123

Его нужно использовать для поиска по логам.

##### Шаг 2. Понять, какой сервис обрабатывает запрос

Если сервисов много, нужно определить маршрут:

nginx → API Gateway → user-service → database

Проверяем конфиг nginx или gateway, документацию, service discovery, Docker Compose или Kubernetes-манифесты.

**На сервере**

```bash
ss -ltnp
docker ps
systemctl list-units --type=service
```

##### Шаг 3. Посмотреть логи входного сервиса

**Если nginx**

```bash
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

**Если systemd**

```bash
journalctl -u api-gateway -n 200
```

**Если Docker**

```bash
docker logs --tail 200 api-gateway
```

**Ищем request ID**

```bash
grep "abc-123" app.log
```

**Или ошибки по времени**

```bash
grep -Ei "error|exception|traceback|failed" app.log
```

##### Шаг 4. Посмотреть логи бизнес-сервиса

Допустим запрос дошёл до user-service.

```bash
journalctl -u user-service -n 200
```

**Или**

```bash
docker logs --tail 200 user-service
```

**Ищем**

- stacktrace;
- timeout;
- connection refused;
- authentication failed;
- null pointer;
- key error;
- database exception;
- out of memory;
- permission denied.

##### Шаг 5. Проверить зависимости

Приложение может работать, но падать при обращении к БД или другому сервису.

**Проверить БД**

```bash
nc -vz db-host 5432
```

**Проверить другой сервис**

```bash
curl -v http://payment-service:8080/health
```

**Проверить брокер**

```bash
nc -vz kafka-host 9092
```

**Проверить DNS**

```bash
dig db-host
nslookup db-host
```

##### Шаг 6. Проверить состояние сервиса

```bash
systemctl status user-service
```

**Или**

```bash
docker ps -a
```

Важно: сервис может быть running, но фактически неработоспособен. Поэтому отдельно проверяем health endpoint:

```bash
curl -v http://localhost:8080/health
```

##### Шаг 7. Проверить конфигурацию

```bash
env
printenv
echo $DB_HOST
echo $DB_USER
echo $ENV
```

**Проверить конфиги**

```bash
cat application.yml
grep -rn "DB_HOST" .
```

#### Частые проблемы:

- неверный host;
- неверный порт;
- неправильный пароль;
- переменная отсутствует;
- сервис направлен на старый стенд;
- конфиг не перечитан после изменения.

##### Шаг 8. Проверить ресурсы

```bash
df -h
free -h
top
```

**Ищем**

- диск заполнен на 100%;
- память закончилась;
- процесс убит OOM Killer;
- CPU загружен;
- слишком много открытых файлов.

**OOM можно искать так**

dmesg | grep -i "out of memory"

```bash
journalctl -k | grep -i "oom"
```

##### Шаг 9. Проверить изменения

**Нужно спросить**

- был ли недавно деплой;
- менялся ли конфиг;
- менялась ли схема БД;
- обновлялись ли сертификаты;
- проблема возникает у всех или только у одного запроса.

Если ошибка появилась после релиза, сравнить версии:

```bash
docker images
docker inspect container
git log -n 5
```

**Хороший ответ на собеседовании**

Я сначала повторю запрос через curl -v, проверю статус, тело и request ID. Затем определю, какой сервис обрабатывает запрос: gateway, nginx и конкретный backend. По request ID и времени найду запрос в логах gateway и приложения. После этого проверю stacktrace и зависимости сервиса: БД, брокер, внешние API. Также проверю переменные окружения, конфигурацию, состояние процесса, порт и ресурсы сервера — память, CPU и диск. Если ошибка появилась после деплоя, сравню версию и последние изменения.

#### Сценарий 2. Запрос не возвращает ответ и висит

**Возможные причины**

- таймаут зависимости;
- зависший поток;
- блокировка в БД;
- сервис ждёт брокер;
- сетевой timeout;
- исчерпан connection pool;
- высокая нагрузка.

#### Проверки:

```bash
curl -v --max-time 10 http://host/api
```

**Проверить порт**

```bash
nc -vz host 8080
```

**Проверить нагрузку**

```bash
top
free -h
```

**Проверить соединения**

```bash
ss -antp
```

**Проверить логи**

```bash
grep -Ei "timeout|connection pool|deadlock|blocked" app.log
```

**Ответ**

Если запрос висит, сначала ограничу его временем через curl --max-time, проверю соединение с портом, затем логи сервиса на timeout и зависшие зависимости. Проверю состояние connection pool, БД, внешние сервисы, количество соединений и нагрузку на сервер.

#### Сценарий 3. Получаем 502 Bad Gateway

502 обычно означает:

Proxy или gateway не получил корректный ответ от backend-сервиса.

**Проверяем**

```bash
systemctl status nginx
tail -f /var/log/nginx/error.log
```

**Проверить backend напрямую**

```bash
curl -v http://backend-host:8080/health
```

**Проверить порт**

```bash
nc -vz backend-host 8080
```

**Проверить, слушает ли backend**

```bash
ss -ltnp | grep 8080
```

**Типовые причины**

- backend не запущен;
- неправильный host/port в nginx;
- backend упал;
- connection refused;
- DNS не разрешается;
- backend отвечает некорректно.

#### Сценарий 4. Получаем 504 Gateway Timeout

504 означает:

Gateway дождался backend, но тот не ответил вовремя.

**Проверяем**

- медленный backend;
- медленный SQL;
- недоступная зависимость;
- высокий CPU;
- проблемы сети;
- слишком маленький timeout.

#### Команды:

```bash
curl -v --max-time 30 http://backend/api
top
journalctl -u backend -n 200
```

**Ищем**

```bash
grep -Ei "timeout|slow|deadlock|pool" app.log
```

> **Короткий ответ для собеседования**
>
> При 504 я проверю, отвечает ли backend напрямую и сколько времени занимает запрос. Затем посмотрю логи backend, состояние БД, внешних сервисов, connection pool и нагрузку. Увеличивать timeout сразу не стоит — сначала нужно найти, почему backend медленный.

#### Сценарий 5. Сервис не запускается

**Порядок**

```bash
systemctl status service
journalctl -u service -n 200
```

**Если Docker**

```bash
docker ps -a
docker logs container
```

**Проверяем**

- синтаксис конфига;
- переменные окружения;
- занятый порт;
- права;
- наличие файла;
- доступность БД;
- версию runtime;
- место на диске.

**Занятый порт**

```bash
lsof -i :8080
ss -ltnp | grep 8080
```

#### Сценарий 6. Сервис запущен, но порт недоступен

**Проверяем**

```bash
systemctl status service
ss -ltnp | grep 8080
```

**Важный момент**

127.0.0.1:8080

означает доступ только локально.

0.0.0.0:8080

означает прослушивание на всех интерфейсах.

**Дальше firewall**

**ufw status**

```bash
iptables -L -n
```

**С клиента**

```bash
nc -vz server 8080
```

#### Сценарий 7. Автотесты локально проходят, в CI падают

**Проверяем различия**

```bash
python --version
pytest --version
pip freeze
env
pwd
ls -la
```

**Причины**

- разные версии Python;
- разные зависимости;
- отсутствуют env-переменные;
- другой рабочий каталог;
- нет тестовых файлов;
- нет прав;
- CI не видит сервис;
- параллельный запуск;
- timezone/locale;
- тесты зависят от порядка;
- нет cleanup.

> **Короткий ответ для собеседования**
>
> Я сравню окружение локально и в CI: версии Python и зависимостей, переменные, рабочую директорию, права, сеть и доступность стенда. Проверю, не запускаются ли тесты параллельно и нет ли зависимости от порядка или общих тестовых данных.

#### Сценарий 8. На сервере закончилось место

```bash
df -h
du -sh /*
du -h /var | sort -h | tail -n 20
```

#### Частые причины:

- логи;
- Docker images;
- Docker volumes;
- дампы;
- артефакты CI;
- временные файлы.

**Docker**

```bash
docker system df
```

Нельзя сразу удалять всё подряд. Сначала определить источник.

#### Сценарий 9. Сервер тормозит

**Проверяем**

```bash
top
uptime
free -h
df -h
```

**Топ по CPU**

```bash
ps aux --sort=-%cpu | head
```

**Топ по памяти**

```bash
ps aux --sort=-%mem | head
```

**Проверить IO**

**iostat**

**Если установлен**

vmstat 1

Нужно определить, где узкое место:

- CPU;
- память;
- диск;
- сеть;
- БД;
- конкретный процесс.

#### Сценарий 10. DNS-имя не работает, но по IP сервис доступен

**Проверяем**

```bash
nslookup service.local
dig service.local
cat /etc/resolv.conf
cat /etc/hosts
```

**Если**

```bash
curl http://10.0.0.5:8080
```

**работает, а**

```bash
curl http://service.local:8080
```

не работает, проблема почти наверняка в DNS или /etc/hosts.

Универсальная структура ответа на диагностический вопрос

> **Короткий ответ для собеседования**
>
> Сначала я воспроизвожу проблему и фиксирую точное время, запрос, статус и идентификатор запроса. Потом иду по слоям: DNS, сеть, порт, gateway, backend, зависимости. Проверяю логи, состояние сервисов, конфигурацию и переменные окружения. После этого проверяю ресурсы сервера и последние изменения или деплой. Стараюсь не перезапускать сервис сразу, потому что это может скрыть причину и удалить важное состояние.

**Последняя фраза особенно хорошая**

Сначала собираю диагностическую информацию, и только потом перезапускаю сервис.

#### Что точно стоит выучить

Для собеседования тебе достаточно уверенно знать:

```bash
curl -v
ping
nc -vz
dig
nslookup
ss -ltnp
ps aux
top
systemctl status
journalctl -u
docker ps
docker logs
df -h
du -sh
free -h
env
grep
tail -f
```

И главное — уметь объяснить последовательность:

Воспроизвести → определить слой → найти логи → проверить зависимости →
проверить конфиг → проверить ресурсы → проверить последние изменения
