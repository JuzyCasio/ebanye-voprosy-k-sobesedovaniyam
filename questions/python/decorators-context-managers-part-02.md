# Декораторы и контекстные менеджеры — часть 2

[← Оглавление](decorators-context-managers.md) · [← К разделу Python](../python.md)

## Как создать контекстный менеджер?

Есть два основных способа: класс с методами `__enter__` и `__exit__` либо функция с декоратором `contextlib.contextmanager`.

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
