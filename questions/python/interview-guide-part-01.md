# Python — полный конспект к собеседованию — часть 1

[← Оглавление](interview-guide.md) · [← К разделу](../python.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/python.md)

Темы 2-11.

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

