# Linux: работа с файлами и текстом — часть 5

[← Оглавление](linux-files-and-text.md) · [← Linux](linux.md) · [⚡ Шпаргалка](../../cheatsheets/linux.md)

В Linux я часто работаю с текстовыми файлами, логами и конфигами. Для просмотра использую less, tail, head, для поиска — grep, для замены и правки конфигов — sed, для работы с колонками — awk и cut, для сортировки и агрегации — sort, uniq, wc. Для поиска файлов использую find, для массовой обработки — xargs. JSON-ответы API удобно проверяю через jq. При анализе проблем обычно смотрю логи, фильтрую ошибки, считаю частотность, сравниваю expected/actual через diff, а при необходимости быстро меняю параметры окружения в .env или yaml через sed.

42. Самый важный минимум команд

Вот это прям стоит знать уверенно:

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

xargs
jq .
diff -u old new
tee
43. Мини-блок именно по sed, который стоит выучить
# заменить первое совпадение в строке
sed 's/old/new/' file

# заменить все совпадения
sed 's/old/new/g' file

# изменить файл на месте
sed -i 's/old/new/g' file

# изменить файл и создать backup
sed -i.bak 's/old/new/g' file

# заменить строку с BASE_URL
sed -i 's|^BASE_URL=.*|BASE_URL=http://stage.local|' .env

# удалить строки с DEBUG
sed '/DEBUG/d' app.log

# удалить пустые строки
sed '/^$/d' file

# вывести строки 10-20
sed -n '10,20p' file

# вывести блок между START и END
sed -n '/START/,/END/p' file

# заменить только в строках с FEATURE_LOGIN
sed '/FEATURE_LOGIN/s/false/true/' config.env
44. Мини-блок “что я делаю, если надо быстро проанализировать лог”
# последние строки
tail -n 200 app.log

# смотреть в реальном времени
tail -f app.log

# ошибки
grep -Ei "error|exception|failed|traceback" app.log

# ошибки с контекстом
grep -C 10 "Exception" app.log

# количество ошибок
grep -Ei "error|exception" app.log | wc -l

# уникальные ошибки
grep -Ei "error|exception" app.log | sort | uniq -c | sort -nr

# сохранить ошибки отдельно
grep -Ei "error|exception" app.log > errors.log
45. Что могут спросить на собеседовании
Чем cat отличается от less?

cat выводит весь файл сразу.
less позволяет удобно смотреть большой файл постранично, искать внутри и не грузить весь вывод в терминал.

Как посмотреть лог в реальном времени?
tail -f app.log

Или для systemd-сервиса:

journalctl -u service -f
Как найти ошибку в большом логе?
grep -i "error" app.log

С контекстом:

grep -C 5 "error" app.log
Как заменить значение в конфиге?
sed -i 's|^BASE_URL=.*|BASE_URL=http://stage.local|' .env
Как удалить пустые строки?
sed '/^$/d' file.txt
Как посчитать количество строк в файле?
wc -l file.txt
Как найти все .log файлы?
find . -type f -name "*.log"
Как найти все большие файлы?
find . -type f -size +100M
Как найти уникальные ошибки?
grep -i "error" app.log | sort | uniq -c | sort -nr
Как красиво посмотреть JSON?
cat response.json | jq

Или:

jq . response.json
46. Самый собеседовательный ответ

Если тебя спросят:

Какими Linux-командами для работы с файлами и текстом вы пользуетесь?

Можно ответить так:

Чаще всего использую less, tail, grep, sed, awk, find, xargs, sort, uniq, wc, jq, diff. Например, логи смотрю через tail -f и less, ошибки фильтрую через grep -i или grep -E, конфиги могу быстро поправить через sed -i, JSON-ответы API смотрю через jq, файлы ищу через find, а для массовой обработки использую find вместе с xargs. Для анализа логов часто комбинирую команды через pipe: grep | sort | uniq -c | sort -nr.

Главное для собеса — показать, что ты не просто знаешь команды, а умеешь собрать из них цепочку для реальной задачи: найти лог, отфильтровать ошибку, посчитать количество, достать контекст, сравнить результат и при необходимости быстро поправить конфиг.

