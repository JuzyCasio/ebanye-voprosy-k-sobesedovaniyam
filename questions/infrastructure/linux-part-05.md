# Linux для QA/SDET — полный конспект — часть 5

[← Оглавление](linux.md) · [← К разделу](../infrastructure.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/linux.md)

Темы 49-66.

## 49. CMD vs ENTRYPOINT

CMD — команда по умолчанию, которую можно легко переопределить.  

ENTRYPOINT — основная команда контейнера, обычно фиксирует исполняемый процесс.  

### Пример:

ENTRYPOINT ["python"]  
CMD ["app.py"]  

Контейнер запустит:  

python app.py  

## 50. Почему контейнер сразу завершился

Контейнер живёт, пока жив основной процесс.  

Если основной процесс завершился — контейнер остановился.  

Диагностика:  

docker ps -a  
docker logs container  
docker inspect container  

## 51. Docker Compose

Запуск:  

docker compose up  
docker compose up -d  

Остановка:  

docker compose down  

Пересобрать:  

docker compose up --build  

Логи:  

docker compose logs  
docker compose logs -f service_name  

Зайти в сервис:  

docker compose exec service_name bash  

## 52. Linux в CI/CD

В CI/CD часто нужно:  

запускать shell-команды;  
ставить зависимости;  
запускать тесты;  
собирать Docker-образы;  
копировать артефакты;  
работать с переменными окружения;  
анализировать exit code;  
читать логи.  

### Пример шагов:

python3 -m venv venv  
source venv/bin/activate  
pip install -r requirements.txt  
pytest tests/ --alluredir=allure-results  

## 53. Диагностика упавших автотестов на Linux-стенде

Порядок:  

Проверить, что стенд доступен:  
ping host  
nc -vz host port  
curl -v http://host:port/health  
Проверить переменные окружения:  
env  
echo $BASE_URL  
Проверить зависимости:  
pip freeze  
python --version  
pytest --version  
Проверить логи приложения:  
journalctl -u app -n 100  
tail -f /var/log/app/app.log  
Проверить Docker:  
docker ps  
docker logs container  
Проверить место:  
df -h  
Проверить процессы и порты:  
ps aux | grep app  
ss -tulpn  

## 54. Полезные команды для QA Automation

Запуск тестов  
pytest  
pytest tests/  
pytest tests/test_users.py  
pytest -k "login"  
pytest -m smoke  
pytest -v  
pytest -s  
pytest --tb=short  
Сохранить вывод  
pytest > test.log 2>&1  
Запустить и одновременно видеть вывод  
pytest 2>&1 | tee test.log  
Найти ошибку в логах  
grep -i "error" test.log  
grep -i "failed" test.log  
grep -i "traceback" test.log  

## 55. tee

command | tee output.log  

Позволяет одновременно:  

видеть вывод в терминале;  
сохранять вывод в файл.  

### Пример:

pytest -v | tee pytest.log  

## 56. Команды для диагностики “не работает база”

Допустим PostgreSQL.  

Проверить порт:  

nc -vz db-host 5432  

Проверить DNS:  

nslookup db-host  

Проверить переменные:  

echo $DB_HOST  
echo $DB_PORT  
echo $DB_USER  

Проверить контейнер:  

docker ps  
docker logs postgres  

Подключиться:  

psql -h db-host -p 5432 -U user -d database  

## 57. Разница между ps, top, htop

ps  

Показывает snapshot процессов.  

top  

Интерактивно показывает процессы, CPU, память, load average.  

htop  

Более удобный интерактивный вариант top.  

## 58. Разница между curl, ping, nc

ping  

Проверяет сетевую доступность по ICMP. Не гарантирует, что порт приложения открыт.  

nc  

Проверяет доступность конкретного TCP/UDP-порта.  

curl  

Проверяет HTTP/HTTPS-уровень: статус-код, заголовки, тело ответа.  

### На собесе:

Если ping проходит, это ещё не значит, что API работает. Нужно проверить порт через nc и сам HTTP через curl.  

## 59. Как проверить, что процесс слушает порт

sudo ss -tulpn | grep 8080  

### Пример ответа:

LISTEN 0 128 0.0.0.0:8080 users:(("python",pid=1234))  

Значит Python-процесс слушает порт 8080.  

## 60. Как найти процесс по порту

sudo lsof -i :8080  

Или:  

sudo ss -tulpn | grep 8080  

## 61. Как завершить процесс, который занял порт

Найти PID:  

sudo lsof -i :8080  

Завершить:  

kill <pid>  

Если не завершился:  

kill -9 <pid>  

## 62. Что такое file descriptor

File descriptor — числовой идентификатор открытого ресурса в процессе.  

Стандартные:  

0 — stdin  
1 — stdout  
2 — stderr  

### Пример:

command > out.log 2> err.log  

## 63. Что такое /dev/null

/dev/null  

Специальное устройство, которое “выбрасывает” всё, что в него записали.  

### Пример:

command > /dev/null 2>&1  

Это значит: не показывать ни stdout, ни stderr.  

## 64. Что такое /proc

/proc — виртуальная файловая система с информацией о процессах и системе.  

### Примеры:

cat /proc/cpuinfo  
cat /proc/meminfo  
ls /proc/<pid>  

Можно посмотреть окружение процесса:  

cat /proc/<pid>/environ  

Открытые файлы процесса:  

ls -la /proc/<pid>/fd  

## 65. Что такое umask

umask задаёт, какие права будут убраны у новых файлов и директорий.  

Посмотреть:  

umask  

### Часто:

022  

Это значит, что новые файлы обычно будут 644, директории 755.  

## 66. Перемещение между серверами и копирование логов

Скопировать лог с сервера:  

scp user@host:/var/log/app/app.log .  

Скопировать свой файл на сервер:  

scp ./config.yml user@host:/tmp/  

Подключиться и посмотреть лог:  

ssh user@host  
tail -f /var/log/app/app.log  

Одной командой:  

ssh user@host "tail -n 100 /var/log/app/app.log"  

