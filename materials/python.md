# Python — полный конспект

[← Все полные материалы](README.md) · [Короткая шпаргалка](../cheatsheets/python.md)

Полный материал из присланного конспекта. Сохранены подробные объяснения, примеры, практические сценарии и вопросы для собеседования.

## 2. Базовые типы данных

Основные типы  
x: int = 10  
price: float = 12.5  
name: str = "Alex"  
is_active: bool = True  
nothing: None = None  
Коллекции  
numbers: list[int] = [1, 2, 3]  
point: tuple[int, int] = (10, 20)  
unique_ids: set[int] = {1, 2, 3}  
user: dict[str, str | int] = {"name": "Alex", "age": 30}  
Изменяемые и неизменяемые типы  
Неизменяемые  

int, float, bool, str, tuple, frozenset, None  

text = "hello"  
text.upper()  

print(text)  # "hello", строка не изменилась  
Изменяемые  

list, dict, set, объекты классов  

items = [1, 2, 3]  
items.append(4)  

print(items)  # [1, 2, 3, 4]  

### На собеседовании важно сказать:

В Python переменная хранит ссылку на объект. Некоторые объекты изменяемые, некоторые нет. Поэтому при передаче списка в функцию можно изменить исходный список.  

### Пример:

def add_item(items: list[int]) -> None:  
    items.append(100)  

numbers = [1, 2, 3]  
add_item(numbers)  

print(numbers)  # [1, 2, 3, 100]  

## 3. is и ==

==  

Сравнивает значения.  

a = [1, 2]  
b = [1, 2]  

print(a == b)  # True  
is  

Сравнивает, один ли это объект в памяти.  

a = [1, 2]  
b = [1, 2]  

print(a is b)  # False  

Правильное использование:  

if value is None:  
    print("Нет значения")  

### На собеседовании:

== проверяет равенство значений, а is проверяет идентичность объектов. Для None правильно использовать is None.  

## 4. Списки

Создание  
numbers = [1, 2, 3, 4, 5]  
Основные операции  
numbers.append(6)  
numbers.extend([7, 8])  
numbers.insert(0, 100)  
numbers.remove(3)  
last = numbers.pop()  
Срезы  
items = [10, 20, 30, 40, 50]  

print(items[0])      # 10  
print(items[-1])     # 50  
print(items[1:4])    # [20, 30, 40]  
print(items[::-1])   # [50, 40, 30, 20, 10]  
Частая ошибка  
items = [1, 2, 3]  
result = items.append(4)  

print(result)  # None  

append() меняет список на месте и возвращает None.  

## 5. Кортежи

Кортеж — неизменяемая последовательность.  

point = (10, 20)  
x, y = point  

Используется, когда нужно зафиксировать набор значений.  

def get_user() -> tuple[int, str]:  
    return 1, "Alex"  

user_id, username = get_user()  

### Важно:

single = (1,)  

Без запятой это будет не кортеж:  

not_tuple = (1)  
print(type(not_tuple))  # int  

## 6. Словари

Создание  
user = {  
    "id": 1,  
    "name": "Alex",  
    "role": "QA",  
}  
Доступ  
print(user["name"])  

Если ключа нет, будет KeyError.  

Безопаснее:  

print(user.get("email"))  
print(user.get("email", "unknown"))  
Обход  
for key in user:  
    print(key)  

for key, value in user.items():  
    print(key, value)  

for value in user.values():  
    print(value)  
Проверка ключа  
if "name" in user:  
    print(user["name"])  
Объединение словарей  
a = {"x": 1}  
b = {"y": 2}  

result = a | b  
print(result)  # {'x': 1, 'y': 2}  

Или старый способ:  

result = {**a, **b}  

## 7. Множества

Множество хранит уникальные элементы.  

ids = {1, 2, 3, 3}  
print(ids)  # {1, 2, 3}  
Операции  
a = {1, 2, 3}  
b = {3, 4, 5}  

print(a | b)  # объединение: {1, 2, 3, 4, 5}  
print(a & b)  # пересечение: {3}  
print(a - b)  # разность: {1, 2}  
print(a ^ b)  # симметричная разность: {1, 2, 4, 5}  

### Где применимо в тестировании:

expected_ids = {1, 2, 3}  
actual_ids = {2, 3, 4}  

missing = expected_ids - actual_ids  
extra = actual_ids - expected_ids  

print(missing)  # {1}  
print(extra)    # {4}  

## 8. Hashable / unhashable

Хешируемые объекты можно использовать как ключи словаря или элементы множества.  

Можно:  

data = {  
    "name": "Alex",  
    1: "one",  
    (1, 2): "point",  
}  

Нельзя:  

data = {  
    [1, 2]: "bad"  
}  

Будет ошибка:  

TypeError: unhashable type: 'list'  

### На собеседовании:

Ключ словаря должен быть hashable, то есть иметь стабильный hash и корректное сравнение. Списки и словари изменяемые, поэтому они не могут быть ключами.  

## 9. List comprehension

Обычный цикл:  

result = []  

for number in range(10):  
    if number % 2 == 0:  
        result.append(number * number)  

Через comprehension:  

result = [number * number for number in range(10) if number % 2 == 0]  

### Пример:

users = [  
    {"id": 1, "active": True},  
    {"id": 2, "active": False},  
    {"id": 3, "active": True},  
]  

active_ids = [user["id"] for user in users if user["active"]]  

print(active_ids)  # [1, 3]  

## 10. Dict comprehension

users = [  
    {"id": 1, "name": "Alex"},  
    {"id": 2, "name": "Ivan"},  
]  

users_by_id = {user["id"]: user for user in users}  

print(users_by_id)  

Результат:  

{  
    1: {"id": 1, "name": "Alex"},  
    2: {"id": 2, "name": "Ivan"},  
}  

Очень частая задача на собеседовании.  

## 11. Функции

Простая функция  
def add(a: int, b: int) -> int:  
    return a + b  
Значения по умолчанию  
def greet(name: str = "Guest") -> str:  
    return f"Hello, {name}"  
Важная ошибка с mutable default argument  

### Плохо:

def add_item(item: str, items: list[str] = []) -> list[str]:  
    items.append(item)  
    return items  

### Проблема:

print(add_item("a"))  # ['a']  
print(add_item("b"))  # ['a', 'b']  

Правильно:  

def add_item(item: str, items: list[str] | None = None) -> list[str]:  
    if items is None:  
        items = []  

    items.append(item)  
    return items  

### На собеседовании:

Значения по умолчанию вычисляются один раз при создании функции, а не при каждом вызове. Поэтому изменяемые значения по умолчанию могут привести к неожиданному поведению.  

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

## 19. Декораторы

Декоратор — функция, которая принимает функцию и возвращает новую функцию.  

Простой пример  
from collections.abc import Callable  
from functools import wraps  

def log_call(func: Callable) -> Callable:  
    @wraps(func)  
    def wrapper(*args, **kwargs):  
        print(f"Вызов функции: {func.__name__}")  
        return func(*args, **kwargs)  

    return wrapper  

@log_call  
def add(a: int, b: int) -> int:  
    return a + b  

print(add(2, 3))  
Что происходит на самом деле  
@log_call  
def add(a: int, b: int) -> int:  
    return a + b  

То же самое, что:  

def add(a: int, b: int) -> int:  
    return a + b  

add = log_call(add)  
Зачем нужен functools.wraps  

Без wraps у функции потеряется имя, docstring и часть метаданных.  

from functools import wraps  

### На собеседовании:

Декораторы удобно использовать для логирования, ретраев, замера времени, авторизации, обёртки API-клиентов, фиксации шагов в отчётах.  

Декоратор retry  
from collections.abc import Callable  
from functools import wraps  
from time import sleep  

def retry(attempts: int = 3, delay: float = 1.0) -> Callable:  
    def decorator(func: Callable) -> Callable:  
        @wraps(func)  
        def wrapper(*args, **kwargs):  
            last_error: Exception | None = None  

            for _ in range(attempts):  
                try:  
                    return func(*args, **kwargs)  
                except Exception as error:  
                    last_error = error  
                    sleep(delay)  

            raise last_error  

        return wrapper  

    return decorator  

Использование:  

@retry(attempts=3, delay=0.5)  
def unstable_request() -> str:  
    return "ok"  

## 20. Итераторы и генераторы

Итератор  

Итератор — объект, у которого есть методы:  

__iter__()  
__next__()  

### Пример:

numbers = [1, 2, 3]  

iterator = iter(numbers)  

print(next(iterator))  # 1  
print(next(iterator))  # 2  
print(next(iterator))  # 3  

После окончания будет:  

StopIteration  
Генератор  

Генератор создаётся функцией с yield.  

from collections.abc import Generator  

def count_up_to(limit: int) -> Generator[int, None, None]:  
    current = 1  

    while current <= limit:  
        yield current  
        current += 1  

for number in count_up_to(3):  
    print(number)  

Результат:  

1  
2  
3  

### На собеседовании:

Генератор не хранит все значения в памяти, а выдаёт их по одному. Это полезно для больших файлов, потоков данных, пагинации API.  

Пример чтения большого файла  
from collections.abc import Generator  

def read_lines(path: str) -> Generator[str, None, None]:  
    with open(path, "r", encoding="utf-8") as file:  
        for line in file:  
            yield line.strip()  

## 21. yield vs return

return завершает функцию и возвращает значение.  

def get_numbers() -> list[int]:  
    return [1, 2, 3]  

yield превращает функцию в генератор.  

def get_numbers():  
    yield 1  
    yield 2  
    yield 3  

## 22. ООП в Python

Класс и объект  
class User:  
    def __init__(self, user_id: int, name: str) -> None:  
        self.user_id = user_id  
        self.name = name  

    def greet(self) -> str:  
        return f"Hello, {self.name}"  

user = User(1, "Alex")  

print(user.greet())  
self  

self — ссылка на текущий объект.  

class Counter:  
    def __init__(self) -> None:  
        self.value = 0  

    def increment(self) -> None:  
        self.value += 1  

### На собеседовании:

self нужен, чтобы обращаться к состоянию конкретного экземпляра класса.  

## 23. Инкапсуляция

В Python нет настоящих private-полей, но есть соглашения.  

class User:  
    def __init__(self, name: str) -> None:  
        self.name = name          # public  
        self._token = "secret"    # protected by convention  
        self.__password = "123"   # name mangling  
property  
class User:  
    def __init__(self, age: int) -> None:  
        self._age = age  

    @property  
    def age(self) -> int:  
        return self._age  

    @age.setter  
    def age(self, value: int) -> None:  
        if value < 0:  
            raise ValueError("Age cannot be negative")  

        self._age = value  

Использование:  

user = User(30)  
user.age = 31  

### На собеседовании:

В Python инкапсуляция чаще строится через соглашения, свойства property и контроль доступа к данным через методы.  

## 24. Наследование

class Animal:  
    def speak(self) -> str:  
        return "Some sound"  

class Dog(Animal):  
    def speak(self) -> str:  
        return "Woof"  

dog = Dog()  
print(dog.speak())  # Woof  
super()  
class BaseClient:  
    def __init__(self, base_url: str) -> None:  
        self.base_url = base_url  

class UserClient(BaseClient):  
    def __init__(self, base_url: str, token: str) -> None:  
        super().__init__(base_url)  
        self.token = token  

### На собеседовании:

super() вызывает метод родительского класса. Чаще всего используется в __init__, чтобы переиспользовать инициализацию родителя.  

## 25. Полиморфизм

Полиморфизм — возможность работать с разными объектами через общий интерфейс.  

class JsonReporter:  
    def report(self) -> str:  
        return "json report"  

class HtmlReporter:  
    def report(self) -> str:  
        return "html report"  

def generate_report(reporter) -> str:  
    return reporter.report()  

### На собеседовании:

### Главное не то, какой конкретно класс передали, а то, что у объекта есть нужный метод.

## 26. Абстрактные классы

from abc import ABC, abstractmethod  

class BaseApiClient(ABC):  
    @abstractmethod  
    def get(self, path: str) -> dict:  
        pass  

    @abstractmethod  
    def post(self, path: str, json: dict) -> dict:  
        pass  

Реализация:  

class UserApiClient(BaseApiClient):  
    def get(self, path: str) -> dict:  
        return {"method": "GET", "path": path}  

    def post(self, path: str, json: dict) -> dict:  
        return {"method": "POST", "path": path, "json": json}  

### На собеседовании:

Абстрактный класс задаёт контракт. Его нельзя нормально использовать без реализации абстрактных методов в дочернем классе.  

## 27. staticmethod, classmethod, обычный метод

Обычный метод  

Получает self.  

class User:  
    def __init__(self, name: str) -> None:  
        self.name = name  

    def get_name(self) -> str:  
        return self.name  
staticmethod  

Не получает ни self, ни cls.  

class Validator:  
    @staticmethod  
    def is_valid_email(email: str) -> bool:  
        return "@" in email  
classmethod  

Получает cls.  

class User:  
    def __init__(self, name: str) -> None:  
        self.name = name  

    @classmethod  
    def from_dict(cls, data: dict[str, str]) -> "User":  
        return cls(name=data["name"])  

### На собеседовании:

Обычный метод работает с объектом, classmethod — с классом, staticmethod — просто функция внутри класса, логически связанная с ним.  

## 28. Магические методы

__str__  

Для человекочитаемого вывода.  

class User:  
    def __init__(self, name: str) -> None:  
        self.name = name  

    def __str__(self) -> str:  
        return f"User: {self.name}"  
__repr__  

Для отладки.  

class User:  
    def __init__(self, name: str) -> None:  
        self.name = name  

    def __repr__(self) -> str:  
        return f"User(name={self.name!r})"  
__eq__  
class User:  
    def __init__(self, user_id: int) -> None:  
        self.user_id = user_id  

    def __eq__(self, other: object) -> bool:  
        if not isinstance(other, User):  
            return False  

        return self.user_id == other.user_id  
__len__  
class Cart:  
    def __init__(self) -> None:  
        self.items: list[str] = []  

    def __len__(self) -> int:  
        return len(self.items)  

## 29. Dataclass

dataclass удобен для классов-структур данных.  

from dataclasses import dataclass  

@dataclass  
class User:  
    user_id: int  
    name: str  
    email: str  

Использование:  

user = User(user_id=1, name="Alex", email="alex@example.com")  

print(user)  

Dataclass автоматически создаёт:  

__init__  
__repr__  
__eq__  
Значение по умолчанию  
from dataclasses import dataclass, field  

@dataclass  
class User:  
    user_id: int  
    name: str  
    roles: list[str] = field(default_factory=list)  

### Важно:

Для изменяемых значений в dataclass нужно использовать default_factory.  

## 30. Типизация

Базовая типизация  
def add(a: int, b: int) -> int:  
    return a + b  
Списки, словари, множества  
def get_names(users: list[dict[str, str]]) -> list[str]:  
    return [user["name"] for user in users]  
Union  
def parse_id(value: int | str) -> int:  
    return int(value)  
Optional  
def find_user(user_id: int) -> dict | None:  
    if user_id <= 0:  
        return None  

    return {"id": user_id}  
Type alias  
UserData = dict[str, str | int]  

def create_user(data: UserData) -> UserData:  
    return data  
Protocol  

Полезно, когда важен не конкретный класс, а набор методов.  

from typing import Protocol  

class Reporter(Protocol):  
    def report(self) -> str:  
        ...  

class JsonReporter:  
    def report(self) -> str:  
        return "json"  

def generate_report(reporter: Reporter) -> str:  
    return reporter.report()  

### На собеседовании:

Типизация в Python не влияет на выполнение кода напрямую, но помогает IDE, mypy, читаемости и поддержке проекта.  

## 31. collections

defaultdict  

### Часто используется для группировки.

from collections import defaultdict  

orders = [  
    {"user_id": 1, "amount": 100},  
    {"user_id": 2, "amount": 200},  
    {"user_id": 1, "amount": 300},  
]  

result: dict[int, int] = defaultdict(int)  

for order in orders:  
    result[order["user_id"]] += order["amount"]  

print(dict(result))  # {1: 400, 2: 200}  
Counter  

Подсчёт элементов.  

from collections import Counter  

text = "aabbc"  

counter = Counter(text)  

print(counter)  # Counter({'a': 2, 'b': 2, 'c': 1})  

### Частая задача:

from collections import Counter  

def first_unique_char(text: str) -> str | None:  
    counter = Counter(text)  

    for char in text:  
        if counter[char] == 1:  
            return char  

    return None  
deque  

Очередь с быстрым добавлением и удалением с двух сторон.  

from collections import deque  

queue = deque()  

queue.append("task1")  
queue.append("task2")  

print(queue.popleft())  # task1  

## 32. Работа с файлами

Чтение файла  
from pathlib import Path  

path = Path("data.txt")  

content = path.read_text(encoding="utf-8")  
Запись файла  
from pathlib import Path  

path = Path("result.txt")  

path.write_text("Hello", encoding="utf-8")  
Построчное чтение  
from pathlib import Path  

path = Path("logs.txt")  

with path.open("r", encoding="utf-8") as file:  
    for line in file:  
        print(line.strip())  

## 33. Работа с JSON

import json  

data = {  
    "id": 1,  
    "name": "Alex",  
}  

json_string = json.dumps(data, ensure_ascii=False, indent=2)  
print(json_string)  

Обратно:  

data = json.loads(json_string)  

Файл:  

from pathlib import Path  
import json  

path = Path("user.json")  

data = {  
    "id": 1,  
    "name": "Alex",  
}  

path.write_text(  
    json.dumps(data, ensure_ascii=False, indent=2),  
    encoding="utf-8",  
)  

## 34. Работа с датой и временем

from datetime import datetime, timedelta, timezone  

now = datetime.now(timezone.utc)  
tomorrow = now + timedelta(days=1)  

print(now.isoformat())  
print(tomorrow.isoformat())  

Парсинг:  

from datetime import datetime  

value = "2026-07-08 12:30:00"  

dt = datetime.strptime(value, "%Y-%m-%d %H:%M:%S")  

Форматирование:  

text = dt.strftime("%d.%m.%Y %H:%M")  

## 35. GIL

GIL — Global Interpreter Lock.  

### На собеседовании коротко:

GIL — это механизм CPython, который не даёт нескольким потокам одновременно выполнять Python bytecode. Поэтому потоки не ускоряют CPU-bound задачи, но хорошо подходят для IO-bound задач: сетевые запросы, работа с файлами, ожидание БД.  

CPU-bound  

Например:  

вычисления  
обработка больших данных  
сжатие  
парсинг огромных структур  

Лучше использовать:  

multiprocessing  
IO-bound  

Например:  

HTTP-запросы  
БД  
файлы  
ожидание ответа сервиса  

Можно использовать:  

threading  
asyncio  

## 36. Threading

from threading import Thread  
from time import sleep  

def worker(name: str) -> None:  
    print(f"Start {name}")  
    sleep(1)  
    print(f"End {name}")  

threads = [  
    Thread(target=worker, args=(f"worker-{i}",))  
    for i in range(3)  
]  

for thread in threads:  
    thread.start()  

for thread in threads:  
    thread.join()  

### На собеседовании:

threading подходит для задач, где много ожидания: API-запросы, работа с сетью, файлами, БД.  

## 37. Race condition и Lock

### Проблема:

from threading import Thread  

counter = 0  

def increment() -> None:  
    global counter  

    for _ in range(100_000):  
        counter += 1  

threads = [Thread(target=increment) for _ in range(5)]  

for thread in threads:  
    thread.start()  

for thread in threads:  
    thread.join()  

print(counter)  

Может быть не тот результат.  

С Lock:  

from threading import Lock, Thread  

counter = 0  
lock = Lock()  

def increment() -> None:  
    global counter  

    for _ in range(100_000):  
        with lock:  
            counter += 1  

### На собеседовании:

Race condition возникает, когда несколько потоков одновременно меняют общий ресурс. Для защиты используют Lock, Semaphore, Queue и другие синхронизационные примитивы.  

## 38. Multiprocessing

from multiprocessing import Process  

def worker(number: int) -> None:  
    print(number * number)  

processes = [  
    Process(target=worker, args=(i,))  
    for i in range(5)  
]  

for process in processes:  
    process.start()  

for process in processes:  
    process.join()  

### На собеседовании:

multiprocessing создаёт отдельные процессы, у каждого свой интерпретатор и память. Это помогает обходить ограничения GIL для CPU-bound задач.  

## 39. Asyncio

Асинхронность полезна для большого количества IO-операций.  

import asyncio  

async def fetch_data(name: str) -> str:  
    print(f"Start {name}")  
    await asyncio.sleep(1)  
    print(f"End {name}")  
    return name  

async def main() -> None:  
    results = await asyncio.gather(  
        fetch_data("task-1"),  
        fetch_data("task-2"),  
        fetch_data("task-3"),  
    )  

    print(results)  

asyncio.run(main())  

### На собеседовании:

asyncio не делает код параллельным на уровне CPU. Он позволяет эффективно переключаться между задачами, пока одна задача ждёт IO.  

## 40. Threading vs Multiprocessing vs Asyncio

Threading  

Используем для IO-bound задач.  

### Примеры:

несколько HTTP-запросов  
чтение файлов  
запросы в БД  
Multiprocessing  

Используем для CPU-bound задач.  

### Примеры:

вычисления  
обработка больших объёмов данных  
парсинг больших файлов  
Asyncio  

Используем для большого количества IO-bound задач, если библиотеки поддерживают async.  

### Примеры:

aiohttp  
asyncpg  
асинхронные клиенты  

### Короткий ответ:

Если задача ждёт сеть или БД — threading или asyncio. Если задача грузит CPU — multiprocessing.  

## 41. Работа с API-клиентом

### Пример простого API-клиента:

from typing import Any  

import requests  

class ApiClient:  
    def __init__(self, base_url: str, timeout: float = 5.0) -> None:  
        self.base_url = base_url.rstrip("/")  
        self.timeout = timeout  

    def get(self, path: str) -> dict[str, Any]:  
        response = requests.get(  
            url=f"{self.base_url}/{path.lstrip('/')}",  
            timeout=self.timeout,  
        )  
        response.raise_for_status()  
        return response.json()  

    def post(self, path: str, json: dict[str, Any]) -> dict[str, Any]:  
        response = requests.post(  
            url=f"{self.base_url}/{path.lstrip('/')}",  
            json=json,  
            timeout=self.timeout,  
        )  
        response.raise_for_status()  
        return response.json()  

### На собеседовании можно сказать:

Я стараюсь выносить работу с API в отдельный клиент, чтобы тесты были читаемыми и не дублировали низкоуровневые вызовы requests.  

## 42. Чистые функции

Чистая функция:  

зависит только от входных данных;  
не меняет внешнее состояние;  
не имеет побочных эффектов.  
def calculate_total(prices: list[int]) -> int:  
    return sum(prices)  

Не чистая:  

total = 0  

def add_to_total(value: int) -> None:  
    global total  
    total += value  

### На собеседовании:

Чистые функции проще тестировать, потому что у них предсказуемый результат.  

## 43. SOLID коротко для Python

S — Single Responsibility  

Класс должен иметь одну ответственность.  

### Плохо:

class UserService:  
    def create_user(self):  
        ...  

    def send_email(self):  
        ...  

    def write_log(self):  
        ...  

Лучше:  

class UserService:  
    def create_user(self):  
        ...  

class EmailService:  
    def send_email(self):  
        ...  

class Logger:  
    def write_log(self):  
        ...  
O — Open/Closed  

Код должен быть открыт для расширения, но закрыт для изменения.  

### Пример через Strategy:

from typing import Protocol  

class AuthStrategy(Protocol):  
    def get_headers(self) -> dict[str, str]:  
        ...  

class TokenAuth:  
    def __init__(self, token: str) -> None:  
        self.token = token  

    def get_headers(self) -> dict[str, str]:  
        return {"Authorization": f"Bearer {self.token}"}  

class ApiClient:  
    def __init__(self, auth: AuthStrategy) -> None:  
        self.auth = auth  
L — Liskov Substitution  

Дочерний класс должен быть заменяемым вместо родительского.  

I — Interface Segregation  

Лучше несколько маленьких интерфейсов, чем один большой.  

D — Dependency Inversion  

Зависеть лучше от абстракций, а не от конкретных реализаций.  

## 44. Частые паттерны

Page Object  

Для UI-автотестов.  

from selenium.webdriver.remote.webdriver import WebDriver  
from selenium.webdriver.common.by import By  

class LoginPage:  
    USERNAME_INPUT = (By.ID, "username")  
    PASSWORD_INPUT = (By.ID, "password")  
    LOGIN_BUTTON = (By.ID, "login")  

    def __init__(self, driver: WebDriver) -> None:  
        self.driver = driver  

    def open(self) -> None:  
        self.driver.get("https://example.com/login")  

    def login(self, username: str, password: str) -> None:  
        self.driver.find_element(*self.USERNAME_INPUT).send_keys(username)  
        self.driver.find_element(*self.PASSWORD_INPUT).send_keys(password)  
        self.driver.find_element(*self.LOGIN_BUTTON).click()  
Factory  

Создание объектов.  

from dataclasses import dataclass  

@dataclass  
class User:  
    name: str  
    email: str  

class UserFactory:  
    @staticmethod  
    def create_user(name: str = "Alex") -> User:  
        return User(  
            name=name,  
            email=f"{name.lower()}@example.com",  
        )  
Builder  

### Когда объект сложный.

class UserBuilder:  
    def __init__(self) -> None:  
        self.data = {  
            "name": "Alex",  
            "age": 30,  
            "role": "user",  
        }  

    def with_name(self, name: str) -> "UserBuilder":  
        self.data["name"] = name  
        return self  

    def with_role(self, role: str) -> "UserBuilder":  
        self.data["role"] = role  
        return self  

    def build(self) -> dict[str, object]:  
        return self.data  

Использование:  

user = (  
    UserBuilder()  
    .with_name("Ivan")  
    .with_role("admin")  
    .build()  
)  
Strategy  

### Когда нужно подставлять разные алгоритмы.

from typing import Protocol  

class PaymentStrategy(Protocol):  
    def pay(self, amount: int) -> str:  
        ...  

class CardPayment:  
    def pay(self, amount: int) -> str:  
        return f"Paid {amount} by card"  

class CashPayment:  
    def pay(self, amount: int) -> str:  
        return f"Paid {amount} by cash"  

class PaymentService:  
    def __init__(self, strategy: PaymentStrategy) -> None:  
        self.strategy = strategy  

    def process(self, amount: int) -> str:  
        return self.strategy.pay(amount)  

## 45. Частые задачи на лайвкодинге

1. Удалить дубликаты с сохранением порядка  
def remove_duplicates(items: list[int]) -> list[int]:  
    seen = set()  
    result = []  

    for item in items:  
        if item not in seen:  
            seen.add(item)  
            result.append(item)  

    return result  

### Пример:

print(remove_duplicates([1, 2, 1, 3, 2]))  # [1, 2, 3]  
2. Первый неповторяющийся символ  
from collections import Counter  

def first_unique_char(text: str) -> str | None:  
    counter = Counter(text)  

    for char in text:  
        if counter[char] == 1:  
            return char  

    return None  
3. Развернуть строку  
def reverse_string(text: str) -> str:  
    return text[::-1]  
4. Проверить палиндром  
def is_palindrome(text: str) -> bool:  
    normalized = text.lower().replace(" ", "")  
    return normalized == normalized[::-1]  
5. Посчитать частоту слов  
from collections import Counter  

def count_words(text: str) -> dict[str, int]:  
    words = text.lower().split()  
    return dict(Counter(words))  
6. Сгруппировать заказы по пользователю  
from collections import defaultdict  

def group_orders_by_user(  
    orders: list[dict[str, int]],  
) -> dict[int, int]:  
    result = defaultdict(int)  

    for order in orders:  
        result[order["user_id"]] += order["amount"]  

    return dict(result)  
7. Найти top-N частых элементов  
from collections import Counter  

def top_n(items: list[str], n: int) -> list[str]:  
    counter = Counter(items)  
    return [item for item, _ in counter.most_common(n)]  

### Пример:

items = ["a", "b", "a", "c", "b", "a"]  

print(top_n(items, 2))  # ['a', 'b']  
8. Инвертировать словарь  

Простой случай, когда значения уникальны:  

def invert_dict(data: dict[str, int]) -> dict[int, str]:  
    return {value: key for key, value in data.items()}  

Если значения не уникальны:  

from collections import defaultdict  

def invert_dict_grouped(data: dict[str, int]) -> dict[int, list[str]]:  
    result = defaultdict(list)  

    for key, value in data.items():  
        result[value].append(key)  

    return dict(result)  
9. Сравнить два списка  
def compare_lists(expected: list[int], actual: list[int]) -> dict[str, set[int]]:  
    expected_set = set(expected)  
    actual_set = set(actual)  

    return {  
        "missing": expected_set - actual_set,  
        "extra": actual_set - expected_set,  
        "common": expected_set & actual_set,  
    }  
10. Найти разницу между двумя словарями  
from typing import Any  

def dict_diff(  
    before: dict[str, Any],  
    after: dict[str, Any],  
) -> dict[str, tuple[Any, Any]]:  
    result = {}  

    all_keys = before.keys() | after.keys()  

    for key in all_keys:  
        before_value = before.get(key)  
        after_value = after.get(key)  

        if before_value != after_value:  
            result[key] = (before_value, after_value)  

    return result  
11. Проверить пароль  

Условия:  

минимум 8 символов;  
есть цифра;  
есть заглавная буква.  
def is_valid_password(password: str) -> bool:  
    if len(password) < 8:  
        return False  

    if not any(char.isdigit() for char in password):  
        return False  

    if not any(char.isupper() for char in password):  
        return False  

    return True  
12. Объединить два отсортированных списка  
def merge_sorted(left: list[int], right: list[int]) -> list[int]:  
    result = []  
    i = 0  
    j = 0  

    while i < len(left) and j < len(right):  
        if left[i] <= right[j]:  
            result.append(left[i])  
            i += 1  
        else:  
            result.append(right[j])  
            j += 1  

    result.extend(left[i:])  
    result.extend(right[j:])  

    return result  

## 46. Сложность алгоритмов

Основные обозначения  
Сложность	Что значит  
O(1)	Константное время  
O(log n)	Логарифмическое  
O(n)	Линейное  
O(n log n)	Часто сортировка  
O(n²)	Два вложенных цикла  
O(2ⁿ)	Экспоненциальное  
Примеры  
items = [1, 2, 3, 4, 5]  

print(items[0])  # O(1)  
for item in items:  
    print(item)  # O(n)  
for a in items:  
    for b in items:  
        print(a, b)  # O(n²)  
Словарь  

В среднем:  

data[key]  

Это O(1).  

Но важно:  

В худшем случае операции со словарём могут деградировать, но в нормальных условиях доступ по ключу считается O(1).  

## 47. Частые вопросы и хорошие ответы

Что такое Python?  

Python — высокоуровневый интерпретируемый язык с динамической типизацией. Он поддерживает ООП, функциональный стиль, имеет богатую стандартную библиотеку и часто используется для автоматизации, backend, data processing, тестирования и скриптов.  

Python компилируемый или интерпретируемый?  

Обычно говорят, что Python интерпретируемый, но точнее: исходный код сначала компилируется в байткод .pyc, а затем выполняется виртуальной машиной Python.  

### Что такое динамическая типизация?

Тип принадлежит объекту, а не переменной. Одна и та же переменная может ссылаться на объекты разных типов.  

value = 10  
value = "hello"  
Что такое строгая типизация?  

Python не выполняет неявные опасные преобразования типов.  

print("1" + 1)  

Будет ошибка:  

TypeError  
Чем list отличается от tuple?  

list изменяемый, tuple неизменяемый. Список используют для динамических наборов данных, кортеж — для фиксированной структуры.  

Чем list отличается от set?  

list хранит порядок и допускает дубликаты. set хранит только уникальные элементы и быстрее для проверки вхождения.  

Чем dict отличается от list?  

list — индексированная последовательность. dict — структура ключ-значение, где доступ идёт по ключу.  

### Что такое shallow copy и deep copy?

Shallow copy копирует только внешний объект, а вложенные объекты остаются общими. Deep copy рекурсивно копирует вложенные объекты.  

### Что такое генератор?

Генератор — объект, который лениво выдаёт значения по одному. Он создаётся функцией с yield и экономит память.  

### Что такое декоратор?

Декоратор — функция, которая оборачивает другую функцию и расширяет её поведение без изменения исходного кода.  

### Что такое контекстный менеджер?

Объект, который управляет входом и выходом из блока with. Обычно используется для безопасной работы с ресурсами.  

### Что такое GIL?

GIL — блокировка в CPython, из-за которой только один поток одновременно выполняет Python bytecode. Он ограничивает CPU-bound многопоточность, но не мешает использовать потоки для IO-bound задач.  

### Что лучше: threading, multiprocessing или asyncio?

Для IO-bound задач — threading или asyncio. Для CPU-bound задач — multiprocessing. Asyncio хорошо подходит для большого количества сетевых операций, если используемые библиотеки асинхронные.  

## 48. Что могут спросить у QA Automation

Как бы ты построил API-клиент?  

Ответ:  

Я бы вынес работу с HTTP в отдельный клиент: base_url, timeout, headers, авторизация, методы get/post/put/delete, обработка ошибок и логирование. В тестах оставил бы только бизнес-действия и проверки.  

### Почему плохо писать requests прямо в тестах?

Потому что появляется дублирование, тесты становятся менее читаемыми, сложнее менять авторизацию, base_url, таймауты и обработку ошибок. Лучше иметь слой клиента.  

### Как бы ты тестировал функцию?

### Пример:

def is_valid_password(password: str) -> bool:  
    ...  

Тестовые данные:  

Пароль	Ожидаем  
Password1	True  
pass	False  
password1	False  
PASSWORD	False  
Password	False  
12345678	False  
Pass1234	True  
Как бы ты обрабатывал нестабильные тесты?  

Сначала выяснил бы причину: ожидания, данные, окружение, сетевые проблемы, race condition. Не стал бы просто добавлять sleep. Лучше использовать явные ожидания, ретраи только на инфраструктурные ошибки, изоляцию данных и нормальную диагностику.  

### Как бы ты ускорял автотесты?

Параллельный запуск, разделение тестов по уровням, уменьшение UI-тестов, переиспользование фикстур, подготовка данных через API/БД, маркировка тестов, запуск только затронутых областей, анализ самых долгих тестов.  

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

## 55. Самое важное для Senior QA Automation

Тебя могут оценивать не только по знанию синтаксиса, а по тому, как ты рассуждаешь.  

### Хорошо говорить так:

Я стараюсь не писать всю логику прямо в тесте. Обычно разделяю код на API-клиенты, фикстуры, модели данных, builders/factories, helpers и assertions. В тесте должна быть видна бизнес-суть: подготовка данных, действие, проверка результата. Низкоуровневые детали лучше держать отдельно.  

### Пример хорошего тестового подхода:

def test_user_can_be_created(user_client, user_factory) -> None:  
    payload = user_factory.build()  

    created_user = user_client.create_user(payload)  

    assert created_user["id"] is not None  
    assert created_user["username"] == payload["username"]  

### Плохой подход:

def test_user_can_be_created() -> None:  
    response = requests.post(  
        "http://host/api/users",  
        json={"username": "alex", "password": "Password1"},  
    )  

    assert response.status_code == 201  

### Почему первый лучше:

тест читается как сценарий;  
меньше дублирования;  
проще менять API;  
проще переиспользовать подготовку данных;  
проще поддерживать большой проект.  

Эту шпаргалку можно дальше развернуть в формат “вопрос → короткий ответ как на собеседовании → пример кода”. Это будет удобнее именно для тренировки перед интервью.  
