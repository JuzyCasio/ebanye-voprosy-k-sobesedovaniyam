# Linux: работа с файлами и текстом — часть 3

[← Оглавление](linux-files-and-text.md) · [← Linux](linux.md) · [⚡ Шпаргалка](../../cheatsheets/linux.md)

Важно: uniq работает нормально только с соседними дублями, поэтому обычно используют вместе с sort.

sort file.txt | uniq
Посчитать количество повторов
sort file.txt | uniq -c
Отсортировать по количеству повторов
sort file.txt | uniq -c | sort -nr

Практический пример — самые частые ошибки:

grep -i "error" app.log | sort | uniq -c | sort -nr
20. wc — подсчёт строк, слов, байт
Количество строк
wc -l app.log
Количество слов
wc -w file.txt
Размер в байтах
wc -c file.txt
Посчитать количество ошибок
grep -i "error" app.log | wc -l
21. find — поиск файлов
Найти файл по имени
find . -name "config.yml"
Найти все .log
find . -name "*.log"
Найти только файлы
find . -type f -name "*.log"
Найти только директории
find . -type d -name "reports"
Найти файлы больше 100 MB
find . -type f -size +100M
Найти файлы, изменённые за последние сутки
find . -type f -mtime -1
Найти файлы старше 7 дней
find . -type f -mtime +7
Найти пустые файлы
find . -type f -empty
22. find + действия
Удалить старые логи
find ./logs -type f -name "*.log" -mtime +7 -delete

Осторожно. Лучше сначала проверить:

find ./logs -type f -name "*.log" -mtime +7

И только потом:

find ./logs -type f -name "*.log" -mtime +7 -delete
Найти и выполнить команду
find . -type f -name "*.log" -exec grep -H "ERROR" {} \;

Но часто удобнее через xargs.

23. xargs — передать список файлов в команду
Найти ошибки во всех логах
find . -type f -name "*.log" | xargs grep -i "error"
Безопаснее для файлов с пробелами
find . -type f -name "*.log" -print0 | xargs -0 grep -i "error"
Удалить найденные файлы
find . -type f -name "*.tmp" | xargs rm

Безопаснее:

find . -type f -name "*.tmp" -print0 | xargs -0 rm
24. jq — работа с JSON

Для тестировщика jq очень полезен: API часто возвращает JSON.

Красиво вывести JSON
cat response.json | jq

Или:

jq . response.json
Вытащить поле
jq '.id' response.json
jq '.user.name' response.json
Вытащить массив
jq '.items' response.json
Вытащить первый элемент массива
jq '.items[0]' response.json
Вытащить поле у каждого элемента
jq '.items[].id' response.json
Фильтр по значению
jq '.items[] | select(.status == "active")' response.json
Вывести только id активных пользователей
jq '.items[] | select(.status == "active") | .id' response.json
25. curl + jq

Очень частый QA-сценарий.

GET и красиво вывести JSON
curl -s http://localhost:8080/users | jq
Вытащить статус пользователя
curl -s http://localhost:8080/users/123 | jq '.status'
Проверить количество элементов
curl -s http://localhost:8080/users | jq '.items | length'
Получить токен из ответа
TOKEN=$(curl -s -X POST http://localhost:8080/login \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"Password123"}' \
  | jq -r '.token')

-r убирает кавычки из строки.

26. diff — сравнение файлов
Сравнить два файла
diff old.txt new.txt
Удобный формат
diff -u old.txt new.txt

-u — unified diff, часто используется в Git.

Сравнить директории
diff -r dir1 dir2
Только отличающиеся файлы
diff -qr dir1 dir2

Полезно для сравнения артефактов, конфигов, отчётов.

27. comm — сравнение отсортированных списков

Допустим есть два файла:

expected.txt
actual.txt

Сначала сортируем:

sort expected.txt > expected_sorted.txt
sort actual.txt > actual_sorted.txt

Потом:

comm expected_sorted.txt actual_sorted.txt

Колонки:

1-я колонка — только в первом файле
2-я колонка — только во втором файле
3-я колонка — есть в обоих

Только строки, которых нет во втором файле:

comm -23 expected_sorted.txt actual_sorted.txt

Только лишние строки во втором:

comm -13 expected_sorted.txt actual_sorted.txt
28. tee — вывести и сохранить одновременно
pytest -v | tee pytest.log

Ты одновременно видишь вывод и сохраняешь его в файл.

С ошибками тоже:

pytest -v 2>&1 | tee pytest.log
29. Перенаправления вывода
Перезаписать файл
command > output.log
Дозаписать в файл
command >> output.log
Ошибки в отдельный файл
command 2> error.log
stdout и stderr в один файл
command > output.log 2>&1

Или:

command &> output.log
Ничего не выводить
command > /dev/null 2>&1
30. watch — периодически выполнять команду
Обновлять команду каждые 2 секунды
watch "docker ps"
Смотреть свободное место
watch "df -h"
Смотреть количество строк в логе
watch "wc -l app.log"
Смотреть, поднялся ли сервис
watch "curl -s -o /dev/null -w '%{http_code}' http://localhost:8080/health"
31. file, stat, du
Определить тип файла
file app.log
file archive.tar.gz
file script.sh
Метаданные файла
stat app.log

Покажет:

