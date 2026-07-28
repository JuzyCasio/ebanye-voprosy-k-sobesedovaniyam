# Python — полный конспект к собеседованию — часть 4

[← Оглавление](interview-guide.md) · [← К разделу](../python.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/python.md)

Темы 26-33.

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

