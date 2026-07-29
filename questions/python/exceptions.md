# Исключения и предупреждения

[← К разделу Python](../python.md)

## Как устроена иерархия исключений?

Все исключения наследуются от `BaseException`. Обычные ошибки программы наследуются от `Exception`.

`SystemExit`, `KeyboardInterrupt` и `GeneratorExit` стоят отдельно от `Exception`, поэтому обычный `except Exception` их не перехватывает.

```text
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── ArithmeticError
    ├── LookupError
    │   ├── IndexError
    │   └── KeyError
    ├── OSError
    ├── RuntimeError
    ├── TypeError
    └── ValueError
```

Прикладные ошибки обычно наследуют от `Exception`, а не от `BaseException`. Поэтому `except Exception` не перехватывает штатное завершение процесса, `Ctrl+C` и закрытие генератора.

## Как работают `try`, `except`, `else` и `finally`?

- в `try` помещают код, где ожидается ошибка;
- `except` обрабатывает подходящую ошибку;
- `else` выполняется, если ошибки не было;
- `finally` выполняется в любом случае.

```python
try:
    value = int(raw_value)
except ValueError:
    handle_invalid_value()
else:
    save(value)
finally:
    close_resource()
```

- `try` содержит потенциально ошибочную операцию;
- `except` выполняется при совпавшем исключении;
- `else` выполняется, если в `try` не было исключения;
- `finally` выполняется почти всегда: при успехе, ошибке, `return` и `break`.

`else` позволяет не помещать под `except` код, ошибки которого перехватывать не планировалось.

`try/finally` без `except` применяют для гарантированной очистки, когда ошибка должна продолжить распространяться.

## Почему нужно ловить конкретные исключения?

Слишком широкий обработчик скрывает программные ошибки:

```python
# Плохо
try:
    process()
except Exception:
    pass
```

Лучше перехватывать ожидаемую проблему:

```python
try:
    user = users[user_id]
except KeyError:
    raise UserNotFound(user_id)
```

Пустой `except:` ловит даже `KeyboardInterrupt` и `SystemExit`, поэтому почти никогда не нужен.

Несколько типов можно объединить:

```python
try:
    process()
except (TypeError, ValueError) as error:
    handle_error(error)
```

Порядок обработчиков идёт от частного к общему.

## Как повторно возбудить исключение?

`raise` без аргумента внутри `except` снова выбрасывает ту же ошибку. Это удобно, если перед передачей ошибки выше нужно что-то записать в лог или выполнить локальную обработку.

```python
try:
    process()
except ValueError:
    logger.exception("Invalid value")
    raise
```

`raise error` обычно добавляет лишний кадр и может ухудшить traceback, поэтому для той же ошибки предпочитают просто `raise`.

## Что такое сцепление исключений?

`raise ... from ...` создаёт более понятную ошибку и сохраняет информацию о первоначальной причине.

```python
try:
    value = int(raw_value)
except ValueError as error:
    raise ConfigurationError("Invalid port") from error
```

Исходная ошибка доступна через `__cause__`. Без явного `from` Python обычно сохраняет её в `__context__`.

Если внутреннюю техническую причину намеренно не нужно показывать:

```python
raise UserNotFound(user_id) from None
```

## Как создать собственное исключение?

```python
class ApiError(Exception):
    """Базовая ошибка API-клиента."""


class ResponseValidationError(ApiError):
    def __init__(self, field, value):
        super().__init__(f"Invalid {field}: {value!r}")
        self.field = field
        self.value = value
```

Имена классов исключений обычно заканчиваются на `Error`. Свои классы позволяют вызывающему коду ловить ошибки предметной области, не привязываясь к деталям реализации.

## Можно ли обработать `SyntaxError`?

Если синтаксическая ошибка находится в самом загружаемом файле, он не сможет нормально начать выполнение. Но `SyntaxError` можно перехватить, когда код компилируется или импортируется динамически:

```python
try:
    compile("if True print('x')", "<input>", "exec")
except SyntaxError as error:
    print(error)
```

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
