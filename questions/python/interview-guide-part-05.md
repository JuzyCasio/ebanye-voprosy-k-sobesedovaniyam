# Python — полный конспект к собеседованию — часть 5

[← Оглавление](interview-guide.md) · [← К разделу](../python.md) · [⚡ Быстрая шпаргалка](../../cheatsheets/python.md)

Темы 34-42.

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

