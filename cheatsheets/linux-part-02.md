# Linux — быстрый повтор — часть 2

[← Оглавление](linux.md) · [← Все шпаргалки](README.md) · [Подробный раздел инфраструктуры](../questions/infrastructure.md)

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

