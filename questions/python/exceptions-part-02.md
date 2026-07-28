# Исключения и предупреждения — часть 2

[← Оглавление](exceptions.md) · [← К разделу Python](../python.md)

## Что такое `ExceptionGroup` и `except*`?

`ExceptionGroup` хранит сразу несколько исключений. Это полезно, когда несколько параллельных задач завершились с разными ошибками.

`except*` обрабатывает из группы все исключения подходящего типа.

```python
try:
    raise ExceptionGroup(
        "validation failed",
        [ValueError("age"), TypeError("name")],
    )
except* ValueError as errors:
    handle_value_errors(errors)
except* TypeError as errors:
    handle_type_errors(errors)
```

Обычный `except` обрабатывает исключение как один объект-группу, а `except*` разделяет её по типам. Группы исключений и `except*` появились в Python 3.11.

## Чем предупреждение отличается от исключения?

Предупреждение сообщает о возможной проблеме, но обычно не останавливает программу. Исключение прерывает обычное выполнение, пока его не обработают.

```python
import warnings

warnings.warn(
    "old_api() is deprecated",
    DeprecationWarning,
    stacklevel=2,
)
```

Модуль `warnings` позволяет фильтровать, скрывать, показывать повторно или превращать предупреждения в исключения. В тестах полезно проверять предупреждения явно, например через `pytest.warns`.
