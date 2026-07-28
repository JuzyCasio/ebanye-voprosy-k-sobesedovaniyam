# Python — полный конспект к собеседованию — часть 3

[← Оглавление](interview-guide.md) · [← К разделу](../python.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/python.md)

Темы 19-25.

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

