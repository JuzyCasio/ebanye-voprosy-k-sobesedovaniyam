# Python — полный конспект к собеседованию — часть 2

[← Оглавление](interview-guide.md) · [← К разделу](../python.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/python.md)

Темы 12-18.

## 12. *args и **kwargs

def func(*args: int, **kwargs: str) -> None:  
    print(args)  
    print(kwargs)  

func(1, 2, 3, name="Alex", role="QA")  

Результат:  

(1, 2, 3)  
{'name': 'Alex', 'role': 'QA'}  
Когда использовать  

*args — когда неизвестно количество позиционных аргументов.  

**kwargs — когда неизвестно количество именованных аргументов.  

### Пример:

def make_request(method: str, url: str, **kwargs: object) -> None:  
    print(method)  
    print(url)  
    print(kwargs)  

make_request(  
    "GET",  
    "https://example.com/users",  
    timeout=5,  
    headers={"Authorization": "token"},  
)  

## 13. Области видимости LEGB

Python ищет переменные в порядке:  

Local — локальная область функции.  
Enclosing — область внешней функции.  
Global — глобальная область модуля.  
Built-in — встроенные имена.  

### Пример:

name = "global"  

def outer() -> None:  
    name = "outer"  

    def inner() -> None:  
        name = "inner"  
        print(name)  

    inner()  

outer()  # inner  
global  
counter = 0  

def increment() -> None:  
    global counter  
    counter += 1  
nonlocal  
def make_counter() -> callable:  
    count = 0  

    def increment() -> int:  
        nonlocal count  
        count += 1  
        return count  

    return increment  

counter = make_counter()  

print(counter())  # 1  
print(counter())  # 2  

## 14. Lambda

users = [  
    {"name": "Alex", "age": 30},  
    {"name": "Ivan", "age": 25},  
]  

users.sort(key=lambda user: user["age"])  

### На собеседовании:

lambda — это короткая анонимная функция. Часто используется как key для сортировки, фильтрации или маппинга.  

## 15. Сортировка

sorted()  

Возвращает новый список.  

numbers = [3, 1, 2]  

result = sorted(numbers)  

print(result)   # [1, 2, 3]  
print(numbers)  # [3, 1, 2]  
.sort()  

Меняет список на месте.  

numbers = [3, 1, 2]  
numbers.sort()  

print(numbers)  # [1, 2, 3]  
Сортировка списка словарей  
users = [  
    {"name": "Alex", "age": 30},  
    {"name": "Ivan", "age": 25},  
    {"name": "Petr", "age": 35},  
]  

users_sorted = sorted(users, key=lambda user: user["age"])  
Сортировка по нескольким полям  
users = [  
    {"name": "Bob", "age": 30},  
    {"name": "Alex", "age": 30},  
    {"name": "Ivan", "age": 25},  
]  

result = sorted(users, key=lambda user: (user["age"], user["name"]))  

## 16. Копирование объектов

Поверхностная копия  
from copy import copy  

a = [[1, 2], [3, 4]]  
b = copy(a)  

b[0].append(100)  

print(a)  # [[1, 2, 100], [3, 4]]  

Скопировался внешний список, но вложенные списки остались общими.  

Глубокая копия  
from copy import deepcopy  

a = [[1, 2], [3, 4]]  
b = deepcopy(a)  

b[0].append(100)  

print(a)  # [[1, 2], [3, 4]]  
print(b)  # [[1, 2, 100], [3, 4]]  

### На собеседовании:

copy копирует только внешний объект, deepcopy рекурсивно копирует вложенные объекты.  

## 17. Исключения

Базовый пример  
try:  
    result = 10 / 0  
except ZeroDivisionError:  
    print("Деление на ноль")  
Несколько исключений  
try:  
    value = int("abc")  
except ValueError:  
    print("Ошибка преобразования")  
except TypeError:  
    print("Неверный тип")  
else  

Выполняется, если исключения не было.  

try:  
    value = int("123")  
except ValueError:  
    print("Ошибка")  
else:  
    print("Успешно")  
finally  

Выполняется всегда.  

try:  
    file = open("data.txt")  
except FileNotFoundError:  
    print("Файл не найден")  
finally:  
    print("Завершение")  
Создание своего исключения  
class UserNotFoundError(Exception):  
    pass  

def get_user(user_id: int) -> dict[str, object]:  
    if user_id <= 0:  
        raise UserNotFoundError(f"User with id={user_id} not found")  

    return {"id": user_id}  

### На собеседовании:

Исключения нужны для обработки ошибочных сценариев. В тестовом фреймворке я бы создавал собственные исключения для понятных ошибок: пользователь не создан, стенд недоступен, некорректный ответ API.  

## 18. Контекстный менеджер with

Контекстный менеджер управляет ресурсом: открыть/закрыть файл, соединение, lock, сессию.  

with open("data.txt", "r", encoding="utf-8") as file:  
    content = file.read()  

Файл закроется автоматически.  

Свой контекстный менеджер через класс  
class FileManager:  
    def __init__(self, path: str) -> None:  
        self.path = path  
        self.file = None  

    def __enter__(self):  
        self.file = open(self.path, "r", encoding="utf-8")  
        return self.file  

    def __exit__(self, exc_type, exc_value, traceback) -> None:  
        if self.file:  
            self.file.close()  

with FileManager("data.txt") as file:  
    print(file.read())  
Через contextmanager  
from collections.abc import Generator  
from contextlib import contextmanager  

@contextmanager  
def open_file(path: str) -> Generator:  
    file = open(path, "r", encoding="utf-8")  
    try:  
        yield file  
    finally:  
        file.close()  

Использование:  

with open_file("data.txt") as file:  
    print(file.read())  

### На собеседовании:

with гарантирует корректное освобождение ресурса даже при ошибке внутри блока.  

