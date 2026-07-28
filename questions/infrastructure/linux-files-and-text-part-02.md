# Linux: работа с файлами и текстом — часть 2

[← Оглавление](linux-files-and-text.md) · [← Linux](linux.md) · [⚡ Шпаргалка](../../cheatsheets/linux.md)

Теперь:

ENV=dev dev dev

станет:

ENV=stage stage stage
Заменить URL
sed 's|http://dev-api.local|http://stage-api.local|g' config.env

Здесь вместо / используется |.

Почему?

Потому что URL содержит /, и так команда читается проще.

Плохо читается:

sed 's/http:\/\/dev-api.local/http:\/\/stage-api.local/g' config.env

Лучше:

sed 's|http://dev-api.local|http://stage-api.local|g' config.env
8. sed -i — изменить файл на месте

Обычный sed только выводит результат в терминал:

sed 's/dev/stage/g' config.env

Файл не меняется.

Чтобы изменить файл:

sed -i 's/dev/stage/g' config.env
Безопасный вариант с backup
sed -i.bak 's/dev/stage/g' config.env

Появятся два файла:

config.env
config.env.bak

Это полезно на стенде, чтобы можно было откатиться.

Важный нюанс macOS vs Linux

На Linux:

sed -i 's/dev/stage/g' config.env

На macOS часто нужно так:

sed -i '' 's/dev/stage/g' config.env

На собеседовании по Linux обычно ожидают GNU/Linux-вариант:

sed -i 's/old/new/g' file
9. Заменить строку целиком через sed

Допустим есть .env:

BASE_URL=http://dev.local
DB_HOST=localhost
ENV=dev

Нужно заменить строку с BASE_URL.

sed -i 's|^BASE_URL=.*|BASE_URL=http://stage.local|' .env

Разбор:

^BASE_URL=  строка начинается с BASE_URL=
.*          любое значение после =

Это очень полезная команда для CI/CD.

Заменить DB_HOST
sed -i 's|^DB_HOST=.*|DB_HOST=postgres|' .env
Заменить ENV
sed -i 's|^ENV=.*|ENV=stage|' .env
10. Удаление строк через sed
Удалить строки, содержащие DEBUG
sed '/DEBUG/d' app.log

Файл не изменится, результат будет выведен в консоль.

Изменить файл:

sed -i '/DEBUG/d' app.log
Удалить пустые строки
sed '/^$/d' file.txt

Разбор:

^  начало строки
$  конец строки
^$ пустая строка
d  delete
Удалить строки с комментариями
sed '/^#/d' config.env

Удалит строки, которые начинаются с #.

Удалить комментарии и пустые строки:

sed '/^#/d; /^$/d' config.env
11. Печать нужных строк через sed
Напечатать конкретную строку
sed -n '10p' app.log

-n отключает обычный вывод, p печатает нужное.

Напечатать диапазон строк
sed -n '10,20p' app.log

Покажет строки с 10 по 20.

Напечатать от совпадения до совпадения
sed -n '/START/,/END/p' app.log

Покажет блок от строки с START до строки с END.

Практический пример:

sed -n '/Traceback/,/AssertionError/p' pytest.log
12. Добавление текста через sed
Добавить строку после совпадения
sed '/BASE_URL/a TIMEOUT=30' .env

a — append, добавить после строки.

Добавить строку перед совпадением
sed '/BASE_URL/i ENV=stage' .env

i — insert, добавить перед строкой.

13. Замена только в строках с условием

Допустим нужно заменить false на true, но только в строке с FEATURE_LOGIN.

sed '/FEATURE_LOGIN/s/false/true/' config.env

Пример:

FEATURE_LOGIN=false
FEATURE_PAYMENT=false

После команды:

FEATURE_LOGIN=true
FEATURE_PAYMENT=false
14. Частые sed команды для QA
Быстро поменять стенд в конфиге
sed -i 's|dev-api.company.local|stage-api.company.local|g' config.yml
Поменять base_url в .env
sed -i 's|^BASE_URL=.*|BASE_URL=http://stage.local|' .env
Убрать DEBUG-строки из лога
sed '/DEBUG/d' app.log > app_without_debug.log
Достать кусок лога по строкам
sed -n '100,200p' app.log
Достать кусок лога между двумя событиями
sed -n '/test_login_start/,/test_login_end/p' pytest.log
Удалить пустые строки
sed '/^$/d' file.txt
Удалить комментарии и пустые строки
sed '/^#/d; /^$/d' config.env
15. awk — работа с колонками

awk удобен, когда файл похож на таблицу или лог с колонками.

Пример:

2026-07-09 12:00:01 INFO user_id=123 status=200
2026-07-09 12:00:02 ERROR user_id=456 status=500
Напечатать первую колонку
awk '{print $1}' app.log
Напечатать первую и третью колонку
awk '{print $1, $3}' app.log
Фильтр по значению
awk '$3 == "ERROR" {print $0}' app.log

$0 — вся строка.

16. awk с разделителем

Допустим CSV:

id,name,status
1,Alex,active
2,John,blocked

Команда:

awk -F',' '{print $1, $3}' users.csv

-F',' задаёт разделитель.

Найти строки с HTTP 500 в access.log

Если статус — 9-я колонка:

awk '$9 == 500 {print $0}' access.log
Посчитать количество 500
awk '$9 == 500 {count++} END {print count}' access.log
Посчитать количество ответов по статусам
awk '{print $9}' access.log | sort | uniq -c | sort -nr
17. cut — вытащить колонку

cut проще, чем awk, если нужно просто разрезать строки по разделителю.

Взять первую колонку по :
cut -d':' -f1 /etc/passwd
Взять второй столбец CSV
cut -d',' -f2 users.csv
Взять первый и третий столбец
cut -d',' -f1,3 users.csv
18. sort — сортировка
Отсортировать строки
sort file.txt
Числовая сортировка
sort -n numbers.txt
Обратная сортировка
sort -r file.txt
Числовая обратная
sort -nr numbers.txt
19. uniq — уникальные строки

