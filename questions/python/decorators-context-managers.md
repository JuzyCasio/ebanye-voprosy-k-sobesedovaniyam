# Декораторы и контекстные менеджеры

[← К разделу Python](../python.md)

## Содержание

- [Что такое декоратор?](#что-такое-декоратор)
- [Как работает синтаксис с @?](#как-работает-синтаксис-с-)
- [Что будет, если декоратор ничего не возвращает?](#что-будет-если-декоратор-ничего-не-возвращает)
- [Что такое фабрика декораторов?](#что-такое-фабрика-декораторов)
- [Зачем нужен functools.wraps?](#зачем-нужен-functoolswraps)
- [Какие стандартные декораторы важно знать?](#какие-стандартные-декораторы-важно-знать)
- [Что такое контекстный менеджер?](#что-такое-контекстный-менеджер)
- [Как создать контекстный менеджер?](#как-создать-контекстный-менеджер)
- [Может ли контекстный менеджер подавить исключение?](#может-ли-контекстный-менеджер-подавить-исключение)

---

## Что такое декоратор?

**Короткий ответ**

Декоратор — вызываемый объект, который получает функцию или класс и возвращает объект, используемый вместо исходного. Обычно он добавляет поведение без изменения тела декорируемого объекта.

```python
def log_calls(function):
    def wrapper(*args, **kwargs):
        print(f"Calling {function.__name__}")
        return function(*args, **kwargs)

    return wrapper


@log_calls
def add(left, right):
    return left + right
```

Декоратор применяется во время выполнения определения функции, обычно при импорте модуля, а не при каждом вызове.

## Как работает синтаксис с `@`?

```python
@decorator
def function():
    ...
```

Эквивалентно:

```python
def function():
    ...


function = decorator(function)
```

Декораторы применяются снизу вверх:

```python
@first
@second
def function():
    ...

# function = first(second(function))
```

### Чем `@decorator` отличается от `@decorator()`?

- `@decorator` передаёт функцию непосредственно декоратору;
- `@decorator()` сначала вызывает фабрику декораторов, а её результат получает функцию.

## Что будет, если декоратор ничего не возвращает?

Имя декорируемой функции будет связано с `None`, и вызвать её не получится:

```python
def broken(function):
    print("decorating")
    # не хватает return


@broken
def work():
    ...


work()  # TypeError: 'NoneType' object is not callable
```

Декоратор не обязан возвращать именно функцию, но возвращённый объект должен соответствовать ожидаемому интерфейсу.

## Что такое фабрика декораторов?

Это функция, принимающая настройки и возвращающая декоратор.

```python
from functools import wraps


def repeat(times):
    def decorator(function):
        @wraps(function)
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(times):
                result = function(*args, **kwargs)
            return result

        return wrapper

    return decorator


@repeat(times=3)
def greet():
    print("hello")
```

Именно так в декоратор «передают параметр»: появляется дополнительный уровень функции.

## Зачем нужен `functools.wraps`?

Обёртка иначе скрывает метаданные исходной функции: `__name__`, `__doc__`, аннотации и ссылку для интроспекции.

```python
from functools import wraps


def decorator(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        return function(*args, **kwargs)

    return wrapper
```

`wraps` также задаёт `__wrapped__`, который используют `inspect`, IDE и некоторые фреймворки.

## Какие стандартные декораторы важно знать?

- `@property` — управляемый доступ к атрибуту как к обычному полю;
- `@classmethod` — получает класс в первом параметре;
- `@staticmethod` — не получает автоматически ни объект, ни класс;
- `@dataclass` — генерирует методы для класса данных;
- `@functools.wraps` — сохраняет метаданные оборачиваемой функции;
- `@functools.lru_cache` и `@functools.cache` — кешируют результаты;
- `@functools.singledispatch` — диспетчеризация по типу первого аргумента;
- `@abc.abstractmethod` — отмечает абстрактный метод;
- `@typing.overload` — описывает несколько сигнатур для статического анализатора.

## Что такое контекстный менеджер?

**Короткий ответ**

Контекстный менеджер управляет входом в контекст и гарантированным выходом из него. Конструкция `with` используется для ресурсов и временного состояния: файлов, соединений, блокировок и транзакций.

```python
with open("data.txt", encoding="utf-8") as file:
    content = file.read()
```

Упрощённо `with`:

1. вызывает `__enter__()`;
2. выполняет тело блока;
3. всегда вызывает `__exit__()`, даже при исключении.

Контекстный менеджер не гарантирует успешное завершение операции — он гарантирует выполнение логики выхода.

## Как создать контекстный менеджер?

### Классом

```python
class ManagedResource:
    def __enter__(self):
        self.resource = acquire_resource()
        return self.resource

    def __exit__(self, exc_type, exc_value, traceback):
        release_resource(self.resource)
        return False
```

### Через `contextlib.contextmanager`

```python
from contextlib import contextmanager


@contextmanager
def managed_resource():
    resource = acquire_resource()
    try:
        yield resource
    finally:
        release_resource(resource)
```

Код до `yield` соответствует входу, код после — выходу.

Для асинхронных ресурсов существуют `__aenter__`, `__aexit__`, `async with` и `@asynccontextmanager`.

## Может ли контекстный менеджер подавить исключение?

Да. Если `__exit__()` возвращает истинное значение, исключение считается обработанным.

```python
class IgnoreValueError:
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        return exc_type is ValueError
```

Подавлять исключения без ясной причины опасно. Обычно `__exit__()` возвращает `False` или `None`, чтобы ошибка продолжила распространяться.


