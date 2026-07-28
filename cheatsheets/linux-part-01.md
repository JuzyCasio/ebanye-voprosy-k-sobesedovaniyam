# Linux — быстрый повтор — часть 1

[← Оглавление](linux.md) · [← Все шпаргалки](README.md) · [Подробный раздел инфраструктуры](../questions/infrastructure.md)

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

