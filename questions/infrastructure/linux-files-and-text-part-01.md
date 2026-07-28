# Linux: работа с файлами и текстом — часть 1

[← Оглавление](linux-files-and-text.md) · [← Linux](linux.md) · [⚡ Шпаргалка](../../cheatsheets/linux.md)

Ниже — практическая шпаргалка по работе с файлами и текстом в Linux для QA/SDET. Это как раз то, что чаще всего нужно на стендах, в CI/CD, при анализе логов, конфигов, результатов тестов и артефактов.

1. Базовая работа с файлами
Посмотреть, где находишься
pwd
Посмотреть файлы
ls
ls -l
ls -la
ls -lh

Часто используемый вариант:

ls -lah

Расшифровка:

-l  подробный вывод
-a  показать скрытые файлы
-h  человекочитаемые размеры: K, M, G
Создать файл
touch app.log
Создать директорию
mkdir logs
mkdir -p reports/allure-results

-p создаёт всю цепочку директорий, если её ещё нет.

Копировать файл
cp app.log app_copy.log
Копировать директорию
cp -r reports reports_backup
Переместить или переименовать
mv old.log new.log
mv app.log /tmp/
Удалить файл
rm app.log
Удалить директорию
rm -r reports
Удалить принудительно
rm -rf reports

rm -rf — опасная команда. На стенде лучше сначала проверить путь через pwd и ls.

2. Просмотр текстовых файлов
cat — вывести весь файл
cat app.log

Подходит для маленьких файлов.

Плохая идея для огромных логов:

cat huge.log

Лучше использовать less, tail, grep.

less — удобный просмотр большого файла
less app.log

Внутри less:

/ERROR       поиск слова ERROR
n            следующее совпадение
N            предыдущее совпадение
G            в конец файла
g            в начало файла
q            выйти

Удобно смотреть большие логи:

less /var/log/app/app.log
head — первые строки файла
head app.log

Первые 50 строк:

head -n 50 app.log
tail — последние строки файла
tail app.log

Последние 100 строк:

tail -n 100 app.log
tail -f — смотреть лог в реальном времени
tail -f app.log

Часто для QA:

tail -n 200 -f app.log

Это значит:

Покажи последние 200 строк и продолжай показывать новые строки.

Для нескольких файлов:

tail -f app.log error.log
3. grep — поиск текста в файлах

grep — одна из самых важных команд для тестировщика.

Найти строку в файле
grep "ERROR" app.log
Поиск без учёта регистра
grep -i "error" app.log

Найдёт:

error
ERROR
Error
eRrOr
Показать номера строк
grep -n "ERROR" app.log
Рекурсивный поиск по директории
grep -r "Exception" .

С номерами строк:

grep -rn "Exception" .

Очень полезно в проекте:

grep -rn "BASE_URL" .
Искать несколько вариантов
grep -E "ERROR|WARN|Exception" app.log

-E включает расширенные регулярные выражения.

Исключить строки
grep -v "DEBUG" app.log

Например, посмотреть лог без debug:

grep -v "DEBUG" app.log | less
Посмотреть строки до и после совпадения

3 строки после:

grep -A 3 "ERROR" app.log

3 строки до:

grep -B 3 "ERROR" app.log

3 строки до и после:

grep -C 3 "ERROR" app.log

Очень полезно при анализе stacktrace:

grep -C 10 "Traceback" test.log
Найти только количество совпадений
grep -c "ERROR" app.log

Например:

grep -c "500 Internal Server Error" access.log
Найти файлы, где есть совпадение
grep -rl "DB_HOST" .

Только имена файлов.

Найти файлы, где нет совпадения
grep -rL "pytest" .
4. Практические grep-сценарии для QA
Найти ошибки в логе
grep -i "error" app.log
Найти ошибки, warning и exception
grep -Ei "error|warn|exception|traceback|failed" app.log
Найти падения автотестов
grep -Ei "failed|error|assert|traceback" pytest.log
Найти HTTP 500 в access.log
grep " 500 " access.log
Найти запросы конкретного пользователя
grep "user_id=12345" app.log
Найти конкретный request_id
grep "request_id=abc-123" app.log
Смотреть лог в реальном времени и фильтровать ошибки
tail -f app.log | grep -i "error"

С несколькими словами:

tail -f app.log | grep -Ei "error|exception|failed"
5. sed — поиск и замена текста

sed — stream editor.
Он умеет читать текстовый поток, менять строки, удалять строки, печатать нужные участки.

Для QA sed полезен, когда нужно:

быстро заменить URL в конфиге;
подменить параметр окружения;
удалить лишние строки;
вытащить часть лога;
подготовить тестовые данные;
заменить значения в .env, .yaml, .json-подобных файлах;
поправить временный конфиг в CI.
6. Базовый синтаксис sed

Общий вид:

sed 'команда' file.txt

Например:

sed 's/dev/stage/' config.env

s означает substitute, то есть заменить.

7. Замена текста через sed
Заменить первое совпадение в каждой строке
sed 's/dev/stage/' config.env

Если строка такая:

ENV=dev dev dev

Результат будет:

ENV=stage dev dev

Поменяется только первое совпадение в строке.

Заменить все совпадения в строке
sed 's/dev/stage/g' config.env

g означает global.

