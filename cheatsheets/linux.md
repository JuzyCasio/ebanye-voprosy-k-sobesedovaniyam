# Linux — быстрый повтор

[← Все шпаргалки](README.md) · [Подробный раздел инфраструктуры](../questions/infrastructure.md)

## Что такое Linux

Linux — семейство Unix-подобных операционных систем на базе ядра Linux. В работе AQA/SDET Linux нужен для CI, тестовых стендов, логов, процессов, сети, файлов, прав и контейнеров.

## Файловая система

| Путь | Назначение |
|---|---|
| `/` | корень |
| `/home` | домашние каталоги |
| `/etc` | системные конфиги |
| `/var/log` | логи |
| `/tmp` | временные файлы |
| `/opt` | дополнительное ПО |
| `/proc` | информация о процессах и ядре |
| `/dev` | устройства |

## Файлы и каталоги

```bash
pwd
ls -lah
cd /var/log
mkdir reports
cp source.txt reports/
mv old.txt new.txt
rm file.txt
find . -type f -name "*.log"
```

Просмотр:

```bash
cat file.txt
less file.txt
head -n 20 file.txt
tail -n 100 file.txt
tail -f app.log
```

## Поиск и обработка текста

```bash
grep -n "ERROR" app.log
grep -i -C 3 "timeout" app.log
grep -R "BASE_URL" .
```

```bash
awk '{print $1}' access.log
cut -d: -f1 /etc/passwd
sort values.txt | uniq -c | sort -nr
wc -l app.log
jq '.users[].id' response.json
```

## Pipe и перенаправления

| Запись | Значение |
|---|---|
| `a \| b` | stdout команды `a` передать в stdin команды `b` |
| `> file` | перезаписать stdout |
| `>> file` | дописать stdout |
| `2> file` | записать stderr |
| `2>&1` | направить stderr туда же, куда stdout |
| `tee file` | показать вывод и сохранить |
| `/dev/null` | отбросить данные |

```bash
pytest -q 2>&1 | tee pytest.log
```

File descriptors:

- `0` — stdin;
- `1` — stdout;
- `2` — stderr.

## Права

```bash
ls -l
chmod 644 config.yaml
chmod 755 run.sh
chown user:group file.txt
```

- `r = 4` — чтение;
- `w = 2` — запись;
- `x = 1` — выполнение для файла, вход/поиск для каталога.

| Права | Значение |
|---|---|
| `644` | владелец читает/пишет, остальные читают |
| `755` | владелец всё, остальные читают/выполняют |
| `600` | доступ на чтение/запись только владельцу |

Для удаления файла важны права на содержащий его каталог.

## Процессы

```bash
ps aux
pgrep -af python
top
htop
kill PID
kill -TERM PID
kill -KILL PID
```

- `SIGTERM` просит процесс корректно завершиться;
- `SIGKILL` немедленно завершает процесс и не даёт выполнить cleanup;
- сначала обычно используют `SIGTERM`;
- exit code `0` означает успех, ненулевой — ошибку или специальный результат.

```bash
command
echo $?
```

## systemd и логи

```bash
systemctl status api.service
systemctl start api.service
systemctl restart api.service
systemctl stop api.service
systemctl enable api.service
```

```bash
journalctl -u api.service
journalctl -u api.service -n 100
journalctl -u api.service -f
journalctl -u api.service --since "30 minutes ago"
```

Если сервис не стартует:

1. `systemctl status`;
2. `journalctl -u`;
3. проверить конфиг и env;
4. проверить права;
5. проверить занятый порт;
6. запустить команду сервиса вручную;
7. проверить зависимости.

## Сеть

```bash
ip addr
ip route
ss -lntp
lsof -i :8080
dig +short api.example.com
nc -vz api.example.com 443
curl -v https://api.example.com/health
```

- `127.0.0.1`/`localhost` — только текущий хост или контейнер;
- `0.0.0.0` при bind — слушать все IPv4-интерфейсы;
- `connection refused` — порт закрыт или никто не слушает;
- timeout — нет ответа вовремя;
- `ping` проверяет ICMP, но не доступность HTTP-сервиса;
- `nc` проверяет TCP-порт;
- `curl` проверяет HTTP/TLS.

## SSH

```bash
ssh user@host
ssh -i ~/.ssh/id_ed25519 user@host
scp app.log user@host:/tmp/
scp user@host:/var/log/app.log .
```

Права приватного ключа обычно ограничивают:

```bash
chmod 600 ~/.ssh/id_ed25519
```

## Переменные окружения

```bash
export BASE_URL=https://test.example.com
echo "$BASE_URL"
env | sort
printenv BASE_URL
unset BASE_URL
```

`PATH` содержит каталоги, в которых shell ищет исполняемые файлы.

`source script.sh` выполняет скрипт в текущем shell. `./script.sh` запускает отдельный процесс.

## Диск, память и CPU

```bash
df -h
du -sh .
du -h /var/log | sort -h | tail
free -h
uptime
top
```

- `df` показывает свободное место файловых систем;
- `du` показывает размер каталогов;
- `free` показывает память;
- load average — среднее количество выполняющихся и ожидающих CPU/непрерываемого I/O задач за 1, 5 и 15 минут;
- load нужно сравнивать с числом CPU и контекстом нагрузки.

Если закончилось место:

1. определить файловую систему через `df`;
2. найти большие каталоги через `du`;
3. проверить логи, артефакты, кеши и Docker;
4. понять причину роста;
5. удалять только подтверждённые данные;
6. настроить ротацию и мониторинг.

## Архивы

```bash
tar -czf logs.tar.gz logs/
tar -xzf logs.tar.gz
gzip app.log
gunzip app.log.gz
```

## Cron

```cron
0 3 * * * /opt/scripts/cleanup.sh
```

Поля: минута, час, день месяца, месяц, день недели.

У cron ограниченное окружение, поэтому лучше использовать абсолютные пути и явно задавать нужные env.

## Docker

```bash
docker ps
docker ps -a
docker logs --tail 100 -f container_name
docker exec -it container_name sh
docker inspect container_name
docker compose ps
docker compose logs -f service_name
```

Контейнер завершается, когда завершается его основной процесс PID 1. Если он упал сразу, смотреть exit code, logs, command, env, mounts и права.

`localhost` внутри контейнера указывает на этот же контейнер.

## Полезные сценарии AQA

### Быстро проверить API

```bash
curl -sS \
  --connect-timeout 3 \
  --max-time 10 \
  -H "Authorization: Bearer $TOKEN" \
  "$BASE_URL/users/42" | jq .
```

### Найти ошибки в логе

```bash
grep -i -C 3 "error\\|exception\\|timeout" app.log
grep -i "error" app.log | sort | uniq -c | sort -nr
```

### Проверить процесс и порт

```bash
pgrep -af app
ss -lntp | grep ':8080'
curl -v http://127.0.0.1:8080/health
```

### Проверить упавшие тесты

```bash
pytest -q -ra 2>&1 | tee pytest.log
grep -n "FAILED\\|ERROR" pytest.log
```

## Общий порядок диагностики

1. воспроизвести проблему;
2. зафиксировать время, окружение и correlation ID;
3. проверить процесс и ресурсы;
4. проверить порт, DNS и HTTP;
5. проверить конфиг и env;
6. посмотреть логи приложения и зависимостей;
7. сравнить с работающим окружением;
8. сохранить команды и результаты в задачу.

## Частые ловушки

- выполнять опасную команду, не проверив текущий каталог;
- путать свободное место и размер каталога;
- проверять HTTP только через `ping`;
- применять `SIGKILL` первым;
- забывать, что cron и CI имеют другое окружение;
- слушать `127.0.0.1` внутри контейнера и ожидать внешний доступ;
- менять права на `777` вместо поиска причины;
- удалять логи до сохранения диагностики.

## Документация

- [GNU Coreutils](https://www.gnu.org/software/coreutils/manual/)
- [journalctl](https://www.freedesktop.org/software/systemd/man/latest/journalctl.html)


