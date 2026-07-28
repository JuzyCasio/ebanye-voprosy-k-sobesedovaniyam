# Python — полный конспект к собеседованию — часть 9

[← Оглавление](interview-guide.md) · [← К разделу](../python.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/python.md)

Темы 49-54.

## 49. Типичные ошибки на собеседовании

1. Использовать mutable default argument  

### Плохо:

def func(items=[]):  
    ...  

### Хорошо:

def func(items: list[int] | None = None) -> None:  
    if items is None:  
        items = []  
2. Путать is и ==  

### Плохо:

if value == None:  
    ...  

### Хорошо:

if value is None:  
    ...  
3. Думать, что append возвращает список  

### Плохо:

items = [1, 2]  
items = items.append(3)  

После этого:  

items is None  
4. Изменять список во время обхода  

### Плохо:

items = [1, 2, 3, 4]  

for item in items:  
    if item % 2 == 0:  
        items.remove(item)  

Лучше:  

items = [item for item in items if item % 2 != 0]  
5. Ловить все исключения без причины  

### Плохо:

try:  
    ...  
except Exception:  
    pass  

Лучше:  

try:  
    ...  
except ValueError as error:  
    print(error)  

## 50. Мини-шпаргалка по синтаксису

# Условие  
if value > 10:  
    print("big")  
elif value == 10:  
    print("ten")  
else:  
    print("small")  

# Цикл for  
for item in items:  
    print(item)  

# Цикл while  
while condition:  
    ...  

# Функция  
def func(a: int, b: int) -> int:  
    return a + b  

# Класс  
class User:  
    def __init__(self, name: str) -> None:  
        self.name = name  

# Исключения  
try:  
    ...  
except ValueError:  
    ...  
finally:  
    ...  

# Контекстный менеджер  
with open("file.txt", "r", encoding="utf-8") as file:  
    content = file.read()  

# List comprehension  
squares = [x * x for x in range(10)]  

# Dict comprehension  
data = {x: x * x for x in range(10)}  

# Lambda  
items.sort(key=lambda item: item["id"])  

## 51. Что повторить перед собеседованием в первую очередь

Самое важное:  

list, dict, set, tuple.  
is vs ==.  
Изменяемые и неизменяемые типы.  
Функции, *args, **kwargs.  
Mutable default argument.  
Исключения.  
Контекстные менеджеры.  
Декораторы.  
Генераторы.  
ООП: self, наследование, super, staticmethod, classmethod, property.  
Typing.  
Counter, defaultdict.  
threading, multiprocessing, asyncio, GIL.  
Простые задачи на строки, списки и словари.  
Сложность O(n) и O(n²).  

## 52. Как отвечать, если не знаешь глубоко

### Хорошая формулировка:

Я не буду придумывать. В работе я с этим сталкивался на базовом уровне. Понимаю общую идею: например, GIL ограничивает выполнение Python bytecode в нескольких потоках, поэтому для CPU-bound лучше multiprocessing, а для IO-bound можно использовать threading или asyncio. В деталях реализации могу подсмотреть документацию, но практический смысл понимаю.  

Это звучит лучше, чем пытаться уверенно сказать ерунду.  

## 53. Очень короткая версия для повторения за 5 минут

Python — динамически и строго типизированный язык. Переменные хранят ссылки на объекты.  

list — изменяемый, хранит порядок и дубликаты.  
tuple — неизменяемый.  
set — уникальные элементы, быстрый in.  
dict — ключ-значение, доступ по ключу в среднем O(1).  

== сравнивает значения.  
is сравнивает идентичность объектов.  
None проверяем через is None.  

Mutable default argument — частая ошибка:  

def func(items=[]):  
    ...  

Правильно:  

def func(items=None):  
    if items is None:  
        items = []  

Генератор использует yield и лениво отдаёт значения.  

Декоратор оборачивает функцию.  

Контекстный менеджер управляет ресурсом через with.  

GIL мешает потокам параллельно выполнять Python bytecode, поэтому:  

IO-bound — threading / asyncio;  
CPU-bound — multiprocessing.  

ООП:  

self — текущий объект;  
classmethod получает cls;  
staticmethod не получает ни self, ни cls;  
property управляет доступом к атрибуту;  
super() вызывает родительскую реализацию.  

## 54. Мини-набор задач, которые стоит уметь писать быстро

from collections import Counter, defaultdict  
from typing import Any  

def remove_duplicates(items: list[int]) -> list[int]:  
    seen = set()  
    result = []  

    for item in items:  
        if item not in seen:  
            seen.add(item)  
            result.append(item)  

    return result  

def first_unique_char(text: str) -> str | None:  
    counter = Counter(text)  

    for char in text:  
        if counter[char] == 1:  
            return char  

    return None  

def group_by_user(orders: list[dict[str, int]]) -> dict[int, int]:  
    result = defaultdict(int)  

    for order in orders:  
        result[order["user_id"]] += order["amount"]  

    return dict(result)  

def dict_diff(  
    before: dict[str, Any],  
    after: dict[str, Any],  
) -> dict[str, tuple[Any, Any]]:  
    result = {}  

    for key in before.keys() | after.keys():  
        before_value = before.get(key)  
        after_value = after.get(key)  

        if before_value != after_value:  
            result[key] = (before_value, after_value)  

    return result  

def is_valid_password(password: str) -> bool:  
    return (  
        len(password) >= 8  
        and any(char.isdigit() for char in password)  
        and any(char.isupper() for char in password)  
    )  

