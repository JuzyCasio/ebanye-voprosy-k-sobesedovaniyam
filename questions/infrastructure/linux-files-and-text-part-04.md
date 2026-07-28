# Linux: работа с файлами и текстом — часть 4

[← Оглавление](linux-files-and-text.md) · [← Linux](linux.md) · [⚡ Шпаргалка](../../cheatsheets/linux.md)

размер;
права;
владельца;
дату изменения;
inode.
Размер файла или директории
du -sh app.log
du -sh reports/
32. tar, gzip, zip
Распаковать .tar.gz
tar -xzf archive.tar.gz
Создать .tar.gz
tar -czf reports.tar.gz reports/
Посмотреть содержимое архива
tar -tzf archive.tar.gz
Распаковать zip
unzip archive.zip
Создать zip
zip -r reports.zip reports/
33. Работа с кодировками и переносами строк

Иногда тесты падают из-за Windows-переносов строк.

Посмотреть скрытые символы
cat -A file.txt

Если видишь ^M, значит могут быть Windows-переносы строк.

Конвертировать Windows → Linux
dos2unix file.txt

Если dos2unix нет:

sed -i 's/\r$//' file.txt
Конвертация кодировки
iconv -f WINDOWS-1251 -t UTF-8 input.txt > output.txt

Полезно, если логи или выгрузки в странной кодировке.

34. Практика: анализ логов

Допустим есть app.log.

Найти все ошибки
grep -i "error" app.log
Найти ошибки и сохранить
grep -i "error" app.log > errors.log
Посчитать ошибки
grep -i "error" app.log | wc -l
Найти уникальные ошибки
grep -i "error" app.log | sort | uniq -c | sort -nr
Смотреть ошибки в реальном времени
tail -f app.log | grep -i "error"
Найти ошибку с контекстом
grep -C 5 "NullPointerException" app.log
Достать лог за конкретный тест
sed -n '/test_create_user/,/test_create_user finished/p' app.log
35. Практика: работа с .env

Файл:

BASE_URL=http://dev.local
DB_HOST=localhost
DB_PORT=5432
ENV=dev
DEBUG=true
Посмотреть без комментариев и пустых строк
sed '/^#/d; /^$/d' .env
Поменять BASE_URL
sed -i 's|^BASE_URL=.*|BASE_URL=http://stage.local|' .env
Поменять DB_HOST
sed -i 's|^DB_HOST=.*|DB_HOST=postgres|' .env
Вытащить значение BASE_URL
grep "^BASE_URL=" .env | cut -d'=' -f2

Или через awk:

awk -F'=' '/^BASE_URL=/ {print $2}' .env
36. Практика: найти нужный конфиг в проекте
Найти все env-файлы
find . -type f -name "*.env"
Найти все yaml/yml
find . -type f \( -name "*.yml" -o -name "*.yaml" \)
Найти, где используется BASE_URL
grep -rn "BASE_URL" .
Найти, где прописан dev-стенд
grep -rn "dev-api" .
Массово заменить dev на stage в yaml-файлах

Сначала проверить:

grep -rn "dev-api.company.local" .

Потом заменить:

find . -type f \( -name "*.yml" -o -name "*.yaml" \) \
  -print0 | xargs -0 sed -i 's|dev-api.company.local|stage-api.company.local|g'
37. Практика: работа с результатами pytest
Запустить тесты и сохранить лог
pytest -v 2>&1 | tee pytest.log
Найти failed
grep -i "failed" pytest.log
Найти traceback
grep -n "Traceback" pytest.log
Найти assertion
grep -i "assert" pytest.log
Найти блок ошибки
grep -C 20 "AssertionError" pytest.log
Посчитать количество failed
grep -c "FAILED" pytest.log
38. Практика: HTTP access.log

Пример access.log условно:

10.0.0.1 - - [09/Jul/2026:12:00:01] "GET /users HTTP/1.1" 200
10.0.0.2 - - [09/Jul/2026:12:00:02] "POST /login HTTP/1.1" 500
Найти все 500
grep " 500" access.log
Посчитать 500
grep " 500" access.log | wc -l
Посчитать статусы
awk '{print $9}' access.log | sort | uniq -c | sort -nr
Топ IP-адресов
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
Топ URL
awk '{print $7}' access.log | sort | uniq -c | sort -nr | head
39. Полезные пайплайны команд
Найти топ ошибок
grep -i "error" app.log | sort | uniq -c | sort -nr | head
Найти все большие логи
find /var/log -type f -name "*.log" -size +100M
Посмотреть последние ошибки
grep -i "error" app.log | tail -n 20
Посмотреть ошибки в большом файле через less
grep -i "error" app.log | less
Найти файлы, где есть password
grep -rn "password" .
Найти файлы, изменённые сегодня
find . -type f -mtime -1
Найти и удалить старые allure-results
find . -type d -name "allure-results"

Удалить осторожно:

find . -type d -name "allure-results" -prune -exec rm -rf {} \;
40. Что обычно нужно тестировщику в Linux
1. Смотреть логи
tail -f app.log
less app.log
grep -i "error" app.log
journalctl -u service -f
docker logs -f container
2. Искать конфиги
find . -name "*.yml"
find . -name ".env"
grep -rn "BASE_URL" .
3. Менять конфиги
sed -i 's|^BASE_URL=.*|BASE_URL=http://stage.local|' .env
4. Проверять результаты тестов
pytest -v 2>&1 | tee pytest.log
grep -i "failed" pytest.log
5. Сравнивать файлы
diff -u expected.json actual.json
6. Работать с JSON
curl -s http://host/api/users | jq
jq '.items[].id' response.json
7. Искать большие файлы
du -sh *
find . -type f -size +100M
8. Чистить старые артефакты
find ./reports -type f -mtime +7 -delete
9. Копировать файлы со стенда
scp user@host:/var/log/app/app.log .
10. Смотреть права и владельца
ls -la
chmod +x script.sh
chown user:group file
41. Как красиво объяснить на собеседовании

Можно сказать так:

