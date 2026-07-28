# Linux для QA/SDET — полный конспект — часть 6

[← Оглавление](linux.md) · [← К разделу](../infrastructure.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/linux.md)

Темы 67-71.

## 67. Выполнить команду на удалённой машине

ssh user@host "hostname && uptime"  

Запустить тесты:  

ssh user@host "cd /opt/project && pytest tests/"  

С переменной:  

ssh user@host "cd /opt/project && BASE_URL=http://test pytest"  

## 68. &&, ||, ;

command1 && command2  

Выполнить command2, только если command1 успешна.  

command1 || command2  

Выполнить command2, только если command1 упала.  

command1 ; command2  

Выполнить обе команды независимо от результата первой.  

### Пример:

cd project && pytest  

Если cd project не сработает, pytest не запустится.  

## 69. Архитектура диагностики проблемы

### Когда говорят:

На стенде не работает сервис. Что будешь делать?  

Можно отвечать так:  

Я бы шёл по слоям. Сначала проверил, доступен ли хост и порт: ping, nc, curl. Потом проверил бы, запущен ли сервис: systemctl status или docker ps. Потом посмотрел бы логи: journalctl или docker logs. Дальше проверил бы конфиг, переменные окружения, доступность БД/брокера, занятые порты, права на файлы и место на диске.  

## 70. Типовые вопросы и короткие ответы

Как посмотреть логи сервиса?  
journalctl -u service_name -f  

Или файл:  

tail -f /var/log/app/app.log  
Как посмотреть процессы?  
ps aux  
top  
htop  
Как найти процесс по имени?  
ps aux | grep nginx  
pgrep nginx  
Как найти процесс по порту?  
sudo lsof -i :8080  
sudo ss -tulpn | grep 8080  
Как проверить, открыт ли порт?  
nc -vz host port  
Как проверить HTTP API?  
curl -v http://host:port/health  
Как проверить свободное место?  
df -h  
Как найти, что занимает место?  
du -sh *  
du -h /var | sort -h | tail  
Как посмотреть память?  
free -h  
Как посмотреть нагрузку?  
top  
uptime  
Как убить процесс?  
kill pid  
kill -9 pid  
Как сделать файл исполняемым?  
chmod +x file.sh  
Как поменять владельца?  
chown user:group file  
Как подключиться по SSH?  
ssh user@host  
Как скопировать файл?  
scp file user@host:/tmp/  
Как посмотреть IP?  
ip a  
Как посмотреть маршруты?  
ip route  
Как проверить DNS?  
nslookup host  
dig host  

## 71. Что нужно уметь уверенно сказать на Senior QA/SDET

Про Linux в целом  

Я использую Linux для работы со стендами, CI/CD и контейнерами. Умею подключаться по SSH, смотреть процессы, логи, сервисы, порты, проверять доступность API и баз, работать с правами, файлами и переменными окружения.  

Про диагностику  

Обычно иду сверху вниз: доступность хоста, порт, HTTP-ответ, статус сервиса, логи, конфиг, зависимости, ресурсы системы.  

Про логи  

Для systemd-сервисов смотрю journalctl -u service, для файловых логов — tail -f, grep, less. В Docker — docker logs.  

Про сеть  

Проверяю IP и маршруты через ip a, ip route, DNS через dig/nslookup, порт через nc или ss, HTTP через curl.  

Про процессы  

Смотрю процессы через ps aux, top, htop; завершаю через kill, стараюсь сначала использовать SIGTERM, а SIGKILL только если процесс не завершается.  

Про права  

Понимаю rwx, chmod, chown, права владельца/группы/остальных, отличие прав на файл и директорию.  

