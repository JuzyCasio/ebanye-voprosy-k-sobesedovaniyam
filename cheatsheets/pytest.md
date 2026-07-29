# Pytest — быстрый повтор

[← Все шпаргалки](README.md) · [Полный материал](../materials/pytest.md)

## Что такое Pytest

Pytest — тестовый фреймворк для Python. Он находит тесты, запускает их, показывает удобные ошибки `assert` и предоставляет фикстуры, параметризацию, маркеры, хуки и плагины.

## Как находятся тесты

По умолчанию Pytest ищет:

- файлы `test_*.py` и `*_test.py`;
- функции и методы `test_*`;
- классы `Test*` без собственного `__init__`.

## Команды

```bash
pytest
pytest tests/api/test_users.py
pytest tests/api/test_users.py::test_create_user
pytest -q
pytest -v
pytest -x
pytest --maxfail=2
pytest -k "user and not delete"
pytest -m smoke
pytest --lf
pytest --ff
pytest --durations=10
pytest --collect-only
pytest -ra
```

- `-x` — остановиться после первого падения;
- `-k` — выбрать тесты по имени;
- `-m` — выбрать тесты по маркеру;
- `--lf` — запустить прошлые упавшие;
- `-ra` — показать расширенное резюме для пропусков, xfail и других специальных результатов.

## Структура

```text
project/
├── pyproject.toml
├── src/
└── tests/
    ├── conftest.py
    ├── api/
    ├── ui/
    └── unit/
```

`conftest.py` хранит общие фикстуры и хуки. Его не нужно импортировать вручную.

## Фикстуры

Фикстура подготавливает данные или ресурс для теста и при необходимости очищает их после теста.

```python
import pytest


@pytest.fixture
def user(api_client):
    created = api_client.create_user()
    yield created
    api_client.delete_user(created["id"])
```

Код до `yield` — setup, после `yield` — teardown. Очистка выполняется в обратном порядке создания фикстур.

### Scope

| Scope | Как часто создаётся |
|---|---|
| `function` | для каждого теста |
| `class` | для класса |
| `module` | для файла |
| `package` | для пакета |
| `session` | для всего запуска |

Чем шире scope, тем больше риск общего изменяемого состояния.

### Полезные возможности

- `autouse=True` — фикстура запускается автоматически;
- фикстура может зависеть от других фикстур;
- фикстура-фабрика возвращает функцию, создающую нужные данные;
- встроенная фикстура `request` даёт информацию о текущем тесте и параметрах.

## Параметризация

```python
import pytest


@pytest.mark.parametrize(
    "email, expected_status",
    [
        pytest.param("user@example.com", 201, id="valid"),
        pytest.param("bad-email", 422, id="invalid"),
    ],
)
def test_create_user(api_client, email, expected_status):
    response = api_client.create_user(email=email)
    assert response.status_code == expected_status
```

- каждый набор данных создаёт отдельный тест-кейс;
- `ids` делают отчёт понятнее;
- значения передаются без автоматического копирования;
- `indirect=True` передаёт параметр в фикстуру через `request.param`.

## Маркеры

```python
import pytest


@pytest.mark.smoke
def test_healthcheck():
    ...
```

Регистрировать пользовательские маркеры лучше в `pyproject.toml`:

```toml
[tool.pytest.ini_options]
markers = [
  "smoke: быстрые критичные проверки",
  "integration: интеграционные тесты",
  "slow: медленные тесты",
]
addopts = "--strict-markers -ra"
```

### Skip и xfail

- `skip` — тест не должен выполняться;
- `skipif` — пропуск при условии;
- `xfail` — известное ожидаемое падение;
- `xpass` — тест с `xfail` неожиданно прошёл.

`xfail` не должен превращаться в склад забытых дефектов.

## Проверки

```python
assert response.status_code == 200
assert response.json()["id"] == expected_id
```

```python
with pytest.raises(ValueError, match="invalid"):
    parse_value("bad")
```

Для приблизительного сравнения чисел:

```python
assert actual == pytest.approx(expected, rel=1e-6)
```

## Полезные встроенные фикстуры

| Фикстура | Для чего |
|---|---|
| `tmp_path` | уникальный временный каталог как `Path` |
| `monkeypatch` | временно заменить атрибут, env или путь |
| `capsys` | перехватить `stdout` и `stderr` |
| `caplog` | проверить логи |
| `request` | данные текущего теста и фикстуры |
| `pytestconfig` | конфигурация запуска |
| `cache` | сохранить данные между запусками |

## Моки

Патчить нужно имя там, где его использует тестируемый модуль, а не обязательно там, где объект был объявлен.

```python
def test_sender(mocker):
    send = mocker.patch("app.service.send_message")

    process_order()

    send.assert_called_once()
```

- mock проверяет взаимодействие;
- stub возвращает заранее заданный ответ;
- fake содержит упрощённую рабочую реализацию;
- не мокать всё подряд: важные интеграции должны проверяться интеграционными тестами.

## API-тест

```python
def test_get_user(api_client, user):
    response = api_client.get(f"/users/{user['id']}", timeout=5)

    assert response.status_code == 200
    body = response.json()
    assert body["id"] == user["id"]
    assert body["email"] == user["email"]
```

Проверять:

- status code;
- тело и типы полей;
- обязательные поля и схему;
- headers;
- авторизацию и права;
- негативные сценарии;
- таймауты;
- идемпотентность, если она обещана контрактом.

## Параллельный запуск

```bash
pytest -n auto
pytest -n 4
pytest -n 4 --dist loadscope
```

`pytest-xdist` запускает worker-процессы. Session-фикстура выполняется отдельно в каждом worker, если специально не организована межпроцессная синхронизация.

Перед параллелизацией нужны:

- уникальные тестовые данные;
- отдельные пользователи, файлы, порты или схемы;
- отсутствие зависимости от порядка;
- безопасная очистка;
- отсутствие общего mutable state.

## FAILED и ERROR

- `FAILED` — тест был запущен, но проверка или код теста упали;
- `ERROR` — тест не смог нормально начаться или завершиться, например упала фикстура;
- collection error — Pytest не смог импортировать или собрать тесты.

## Flaky-тесты

Частые причины:

- `sleep()` вместо ожидания условия;
- общие данные и гонки;
- зависимость от порядка;
- нестабильный внешний сервис;
- слишком короткий timeout;
- текущее время или timezone;
- неправильная очистка.

Rerun временно снижает шум, но не исправляет причину.

## CI

Минимальный pipeline:

1. установить фиксированные зависимости;
2. запустить линтеры и статические проверки;
3. поднять тестовые зависимости;
4. запустить тесты по маркерам;
5. сохранить JUnit/Allure-отчёт, логи и скриншоты;
6. корректно завершить job по exit code.

## Частые ловушки

- слишком широкий scope фикстуры;
- session-фикстура с общим изменяемым объектом;
- autouse-фикстура со скрытым влиянием;
- тест зависит от другого теста;
- слабая проверка только status code;
- мок установлен не в том модуле;
- `pytest-xdist` включён до обеспечения изоляции;
- бесконечные rerun вместо исправления flaky-теста.

## Официальная документация

- [Pytest](https://docs.pytest.org/en/stable/)
- [pytest-xdist](https://pytest-xdist.readthedocs.io/en/stable/)
