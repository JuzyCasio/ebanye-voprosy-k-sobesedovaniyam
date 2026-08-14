# Pytest — полный конспект

[← Все полные материалы](README.md) · [Короткая шпаргалка](../cheatsheets/pytest.md)

Полный материал из присланного конспекта. Сохранены подробные объяснения, примеры, практические сценарии и вопросы для собеседования.

> **Как пользоваться конспектом**
>
> Выбери тему в навигации, прочитай объяснение и затем проговори выделенный короткий ответ своими словами. Код и команды оформлены отдельными блоками, чтобы их можно было быстро найти и скопировать.

## Навигация по разделу

- [Основы и запуск](#основы-и-запуск) — вопросы 1–6
- [Фикстуры](#фикстуры) — вопросы 7–13
- [Параметризация и маркеры](#параметризация-и-маркеры) — вопросы 14–20
- [Встроенные возможности и расширение](#встроенные-возможности-и-расширение) — вопросы 21–27
- [Архитектура тестов и интеграции](#архитектура-тестов-и-интеграции) — вопросы 28–39
- [CI, отчёты и стабильность](#ci-отчёты-и-стабильность) — вопросы 40–49
- [Тестовые данные и качество кода](#тестовые-данные-и-качество-кода) — вопросы 50–62
- [Внутренние механизмы и практические проверки](#внутренние-механизмы-и-практические-проверки) — вопросы 63–80
- [Собеседование и итоговое повторение](#собеседование-и-итоговое-повторение) — вопросы 81–89

## Основы и запуск

### 1. Что такое pytest

pytest — это тестовый фреймворк для Python, который используют для unit, integration, API, UI и end-to-end тестов. Его сильные стороны: простые assert, фикстуры, параметризация, плагины, удобный запуск из CLI, интеграция с Allure, CI/CD и параллельный запуск через плагины. В официальной документации pytest отдельно выделены фикстуры, параметризация, marks, плагины, конфигурация и работа с кэшем как основные механизмы фреймворка.

> **Короткий ответ для собеседования**
>
> pytest удобен тем, что тесты пишутся как обычные Python-функции, зависимости передаются через фикстуры, тестовые данные удобно раскладываются через parametrize, а поведение фреймворка можно расширять через хуки и плагины.

**Простейший тест**

```python
def test_sum() -> None:
    assert 1 + 1 == 2
```

### 2. Как pytest находит тесты

**По умолчанию pytest ищет**

```text
test_*.py
*_test.py
```

**Внутри файлов**

```python
def test_something():
    ...

class TestUser:
    def test_create_user(self):
        ...
```

#### Важно:

```python
class TestUser:
    def test_user_created(self):
        assert True
```

Но если у класса есть __init__, pytest обычно не будет нормально собирать такой класс как тестовый:

```python
class TestUser:
    def __init__(self):
        ...
```

> **Короткий ответ для собеседования**
>
> pytest собирает тесты по naming convention: файлы test_*.py или *_test.py, функции test_*, классы Test*. Для классов тестов не нужен __init__, состояние лучше готовить через фикстуры.

### 3. Основные команды запуска

| Команда | Что делает |
|---|---|
| `pytest` | Запускает все найденные тесты |
| `pytest tests/test_users.py` | Запускает конкретный файл |
| `pytest tests/test_users.py::test_create_user` | Запускает конкретную тестовую функцию |
| `pytest tests/test_users.py::TestUserApi::test_create_user` | Запускает метод тестового класса |
| `pytest -v` | Показывает подробный вывод |
| `pytest -q` | Сокращает вывод |
| `pytest -s` | Не перехватывает `print()` и другой вывод в stdout/stderr |
| `pytest -x` | Останавливается после первого падения |
| `pytest --maxfail=3` | Останавливается после трёх падений |
| `pytest -k "user and not slow"` | Отбирает тесты по выражению в имени |
| `pytest -m smoke` | Запускает тесты с маркером `smoke` |
| `pytest --tb=short` | Сокращает traceback |
| `pytest --collect-only` | Показывает собранные тесты без запуска |
| `pytest --durations=10` | Показывает десять самых медленных тестов |
| `pytest --lf` | Запускает только тесты, упавшие в прошлый раз |
| `pytest --ff` | Сначала запускает ранее упавшие тесты, затем остальные |

`--lf` — короткая форма `--last-failed`, а `--ff` — короткая форма `--failed-first`. Эти опции и `--cache-clear` используют встроенный кэш pytest.

### 4. Assert в pytest

В pytest не нужно писать специальные assert-методы как в unittest:

```python
def test_user_name() -> None:
    user = {"name": "Alex"}
    assert user["name"] == "Alex"
```

**pytest сам покажет разницу**

```python
def test_lists() -> None:
    assert [1, 2, 3] == [1, 2, 4]
```

**Можно добавлять сообщение**

```python
def test_status_code() -> None:
    status_code = 500

    assert status_code == 200, f"Expected 200, got {status_code}"
```

> **Короткий ответ для собеседования**
>
> В pytest используются обычные Python assert, но pytest переписывает assert-выражения и даёт подробный diff при падении.

### 5. Структура проекта

#### Типичная структура:

```text
project/
├── app/
│   └── users.py
├── tests/
│   ├── conftest.py
│   ├── test_users.py
│   └── test_orders.py
├── pyproject.toml
└── requirements.txt
```

#### Пример pyproject.toml:

```toml
[tool.pytest.ini_options]
pythonpath = ["."]
testpaths = ["tests"]
addopts = "-ra -q"
markers = [
    "smoke: быстрые smoke-тесты",
    "regression: регрессионные тесты",
    "slow: медленные тесты",
]
```

Настройки pytest можно хранить в конфигурационных файлах в корне проекта; в актуальной документации перечислены поддерживаемые форматы, включая pytest.toml, pytest.ini, pyproject.toml, tox.ini и setup.cfg.

### 6. conftest.py

conftest.py — специальный файл pytest, куда обычно кладут:

- фикстуры;
- хуки;
- кастомные CLI-опции;
- общую конфигурацию тестов;
- подготовку окружения.

#### Пример:

```python
# tests/conftest.py

import pytest

@pytest.fixture
def base_url() -> str:
    return "https://api.example.com"
```

**Использование**

```python
def test_base_url(base_url: str) -> None:
    assert base_url.startswith("https://")
```

> **Короткий ответ для собеседования**
>
> conftest.py позволяет объявлять фикстуры и хуки без явного импорта в тестах. pytest сам находит этот файл и делает фикстуры доступными для тестов в текущей директории и ниже.

## Фикстуры

### 7. Фикстуры

Фикстура — это способ подготовить данные, объект, подключение, клиент, пользователя, браузер, БД или окружение для теста.

Официальная документация описывает фикстуру как механизм, который даёт тестам заранее определённый и воспроизводимый контекст, например настроенную БД или подготовленные данные.

#### Пример:

```python
import pytest

@pytest.fixture
def user() -> dict[str, str]:
    return {"name": "Alex", "role": "admin"}

def test_user_role(user: dict[str, str]) -> None:
    assert user["role"] == "admin"
```

#### Главная идея:

pytest передаёт фикстуру в тест по имени аргумента.

**То есть здесь**

```python
def test_user_role(user):
    ...
```

pytest видит аргумент user, ищет фикстуру с таким именем и выполняет её.

### 8. Scope фикстур

У фикстур есть область жизни:

```python
@pytest.fixture(scope="function")
def fixture_func():
    ...
```

#### Основные scope:

- `function` — на каждый тест;
- `class` — на класс;
- `module` — на файл;
- `package` — на пакет;
- `session` — на весь запуск pytest.

#### Пример:

```python
import pytest

@pytest.fixture(scope="session")
def auth_token() -> str:
    return "token-123"

def test_one(auth_token: str) -> None:
    assert auth_token

def test_two(auth_token: str) -> None:
    assert auth_token.startswith("token")
```

> **Короткий ответ для собеседования**
>
> Scope определяет, как часто создаётся фикстура. function — перед каждым тестом, session — один раз за весь тестовый запуск. Чем шире scope, тем аккуратнее нужно быть с изменяемым состоянием.

### 9. Yield fixture: setup и teardown

Фикстура может не только вернуть объект, но и подчистить ресурсы после теста.

```python
import pytest

@pytest.fixture
def db_connection():
    connection = create_connection()
    yield connection
    connection.close()
```

Всё до yield — setup.
Всё после yield — teardown.

#### Пример попроще:

```python
import pytest

@pytest.fixture
def file_resource(tmp_path):
    file_path = tmp_path / "data.txt"
    file_path.write_text("hello")

    yield file_path

    # teardown
    if file_path.exists():
        file_path.unlink()
```

**Тест**

```python
def test_file_content(file_resource) -> None:
    assert file_resource.read_text() == "hello"
```

> **Короткий ответ для собеседования**
>
> yield-фикстура удобна для ресурсов, которые нужно закрывать: соединение с БД, браузер, временный пользователь, файл, мок-сервер. Код после yield выполнится после завершения теста.

### 10. addfinalizer

Альтернатива yield — request.addfinalizer.

```python
import pytest

@pytest.fixture
def resource(request):
    item = create_resource()

    def cleanup() -> None:
        delete_resource(item)

    request.addfinalizer(cleanup)

    return item
```

Но чаще в обычных проектах используют yield, потому что он читается проще.

> **Короткий ответ для собеседования**
>
> addfinalizer полезен, когда нужно динамически регистрировать cleanup-функции, но для простого setup/teardown чаще читаемее yield.

### 11. Autouse fixtures

Autouse fixture применяется автоматически, даже если тест явно её не запросил.

```python
import pytest

@pytest.fixture(autouse=True)
def clean_state() -> None:
    print("before test")
```

Официальная документация описывает autouse-фикстуры как способ автоматически запрашивать фикстуру для тестов, чтобы не дублировать её в аргументах.

#### Пример реального использования:

```python
import pytest

@pytest.fixture(autouse=True)
def reset_env(monkeypatch: pytest.MonkeyPatch) -> None:
    monkeypatch.delenv("DEBUG", raising=False)
```

> **Короткий ответ для собеседования**
>
> Autouse хорош для глобальной подготовки: очистка состояния, настройка env, логирование, reset моков. Но им нельзя злоупотреблять, потому что скрытые зависимости ухудшают читаемость тестов.

### 12. Зависимости между фикстурами

Фикстура может зависеть от другой фикстуры:

```python
import pytest

@pytest.fixture
def token() -> str:
    return "secret-token"

@pytest.fixture
def api_client(token: str):
    return ApiClient(token=token)

def test_get_user(api_client) -> None:
    response = api_client.get_user(user_id=1)
    assert response.status_code == 200
```

> **Короткий ответ для собеседования**
>
> Фикстуры можно строить как граф зависимостей. pytest сам вычисляет порядок выполнения по зависимостям.

### 13. Фабрики через фикстуры

Иногда фикстура должна не сразу создавать объект, а возвращать функцию-фабрику.

```python
import pytest

@pytest.fixture
def user_factory():
    def create_user(name: str = "Alex", role: str = "user") -> dict[str, str]:
        return {
            "name": name,
            "role": role,
        }

    return create_user

def test_admin_user(user_factory) -> None:
    user = user_factory(name="Ivan", role="admin")

    assert user["role"] == "admin"
```

> **Короткий ответ для собеседования**
>
> Если в тесте нужно создавать много объектов с разными параметрами, удобно делать fixture factory: фикстура возвращает функцию создания данных.

## Параметризация и маркеры

### 14. Параметризация тестов

Параметризация позволяет запустить один тест с разными наборами данных.

Официальная документация pytest описывает несколько уровней параметризации: через pytest.fixture(params=...), через @pytest.mark.parametrize, а также через pytest_generate_tests для кастомных схем.

#### Пример:

```python
import pytest

@pytest.mark.parametrize(
    "password, expected",
    [
        ("Qwerty123", True),
        ("short", False),
        ("withoutdigits", False),
        ("12345678", False),
    ],
)
def test_password_validation(password: str, expected: bool) -> None:
    assert is_valid_password(password) is expected
```

> **Короткий ответ для собеседования**
>
> parametrize уменьшает дублирование и позволяет явно описать наборы тестовых данных. Один тест запускается несколько раз с разными аргументами.

### 15. ids в parametrize

Чтобы в отчёте было понятно, какой набор данных упал:

```python
import pytest

@pytest.mark.parametrize(
    "username, expected_status",
    [
        ("alex", 201),
        ("", 400),
        ("a" * 256, 400),
    ],
    ids=[
        "valid_username",
        "empty_username",
        "too_long_username",
    ],
)
def test_create_user(username: str, expected_status: int) -> None:
    response = create_user(username=username)

    assert response.status_code == expected_status
```

> **Короткий ответ для собеседования**
>
> ids нужны для читаемого вывода в консоли и отчётах, особенно когда параметров много.

### 16. pytest.param

Можно помечать конкретный набор данных:

```python
import pytest

@pytest.mark.parametrize(
    "value, expected",
    [
        (1, 2),
        (2, 4),
        pytest.param(3, 6, marks=pytest.mark.smoke),
        pytest.param(4, 8, marks=pytest.mark.xfail(reason="known bug")),
    ],
)
def test_double(value: int, expected: int) -> None:
    assert value * 2 == expected
```

> **Короткий ответ для собеседования**
>
> pytest.param позволяет повесить mark, xfail, skip или id на конкретный набор параметров.

### 17. Параметризация фикстур

**Фикстуру тоже можно параметризовать**

```python
import pytest

@pytest.fixture(params=["chrome", "firefox", "webkit"])
def browser_name(request) -> str:
    return request.param

def test_open_page(browser_name: str) -> None:
    assert browser_name in ["chrome", "firefox", "webkit"]
```

> **Короткий ответ для собеседования**
>
> Параметризованная фикстура запускает все тесты, которые её используют, для каждого значения из params.

### 18. indirect parametrization

```python
indirect=True означает: параметр передаётся не напрямую в тест, а в фикстуру через request.param.

import pytest

@pytest.fixture
def user(request) -> dict[str, str]:
    role = request.param

    return {
        "name": "Alex",
        "role": role,
    }

@pytest.mark.parametrize("user", ["admin", "manager"], indirect=True)
def test_user_role(user: dict[str, str]) -> None:
    assert user["role"] in ["admin", "manager"]
```

> **Короткий ответ для собеседования**
>
> indirect используют, когда тестовые данные должны пройти через фикстуру, например для создания пользователя, подключения к окружению или подготовки сложного объекта.

### 19. Маркеры

Маркер — это метка на тесте.

```python
import pytest

@pytest.mark.smoke
def test_login() -> None:
    assert True

@pytest.mark.regression
def test_create_order() -> None:
    assert True
```

**Запуск**

```bash
pytest -m smoke
```

**Исключить slow**

```bash
pytest -m "not slow"
```

**Комбинация**

```bash
pytest -m "smoke or critical"
```

Лучше регистрировать маркеры в конфиге:

```toml
[tool.pytest.ini_options]
markers = [
    "smoke: быстрые smoke-тесты",
    "regression: регрессионные тесты",
    "critical: критичные тесты",
    "slow: медленные тесты",
]
```

> **Короткий ответ для собеседования**
>
> Маркеры позволяют группировать тесты: smoke, regression, slow, api, ui, db. В CI удобно запускать разные наборы тестов по маркерам.

### 20. skip, skipif, xfail

skip — тест не запускаем.

```python
import pytest

@pytest.mark.skip(reason="temporarily disabled")
def test_old_feature() -> None:
    assert False
```

skipif — пропускаем по условию:

```python
import sys
import pytest

@pytest.mark.skipif(sys.platform == "win32", reason="Linux only test")
def test_linux_command() -> None:
    assert True
```

xfail — ожидаем, что тест упадёт:

```python
import pytest

@pytest.mark.xfail(reason="known bug")
def test_known_bug() -> None:
    assert 1 == 2
```

**Разница**

- skip  — тест не запускается
- xfail — тест запускается, но падение ожидаемое
- xpass — тест неожиданно прошёл, хотя был помечен xfail

Документация pytest формулирует это так: skip используется, когда тест должен проходить только при определённых условиях, а xfail — когда тест ожидаемо падает, например из-за известного бага или ещё не реализованной функциональности.

> **Короткий ответ для собеседования**
>
> xfail лучше использовать для известного бага с ссылкой на задачу. Если баг починили и тест стал проходить, pytest покажет XPASS, и это сигнал убрать xfail.

## Встроенные возможности и расширение

### 21. Встроенные фикстуры pytest

**tmp_path**

Создаёт временную директорию как pathlib.Path.

```python
def test_write_file(tmp_path) -> None:
    file_path = tmp_path / "test.txt"
    file_path.write_text("hello")

    assert file_path.read_text() == "hello"
```

**monkeypatch**

**Позволяет временно менять**

- переменные окружения;
- атрибуты объектов;
- функции;
- словари;
- sys.path;
- текущую директорию.

```python
def test_env(monkeypatch) -> None:
    monkeypatch.setenv("APP_ENV", "test")

    assert get_env() == "test"
```

**Мок функции**

```python
def test_get_current_user(monkeypatch) -> None:
    def fake_get_user_id() -> int:
        return 123

    monkeypatch.setattr("app.auth.get_user_id", fake_get_user_id)

    assert get_current_user_id() == 123
```

Документация pytest описывает monkeypatch как встроенную фикстуру для временного изменения объектов, словарей и os.environ; изменения автоматически откатываются после теста.

**capsys**

Перехват stdout и stderr.

```python
def test_print(capsys) -> None:
    print("hello")

    captured = capsys.readouterr()

    assert captured.out == "hello\n"
```

**caplog**

#### Проверка логов.

```python
import logging

def test_logging(caplog) -> None:
    with caplog.at_level(logging.INFO):
        logging.info("User created")

    assert "User created" in caplog.text
```

**pytestconfig**

Доступ к конфигурации pytest.

```python
def test_config(pytestconfig) -> None:
    verbose = pytestconfig.getoption("verbose")

    assert isinstance(verbose, int)
```

**request**

Доступ к контексту текущего теста или фикстуры.

```python
import pytest

@pytest.fixture
def test_name(request) -> str:
    return request.node.name

def test_example(test_name: str) -> None:
    assert test_name == "test_example"
```

### 22. Кастомные CLI-опции

**В conftest.py**

```python
def pytest_addoption(parser) -> None:
    parser.addoption(
        "--env",
        action="store",
        default="dev",
        help="Environment: dev, stage, prod",
    )
```

**Фикстура**

```python
import pytest

@pytest.fixture
def env(pytestconfig) -> str:
    return pytestconfig.getoption("--env")
```

**Тест**

```python
def test_env(env: str) -> None:
    assert env in ["dev", "stage", "prod"]
```

**Запуск**

```bash
pytest --env=stage
```

> **Короткий ответ для собеседования**
>
> Кастомные CLI-опции удобно использовать для выбора окружения, base_url, браузера, запуска against mock/real service, включения debug-режима.

### 23. Пример base_url через CLI

```python
# conftest.py

import pytest

def pytest_addoption(parser) -> None:
    parser.addoption(
        "--base-url",
        action="store",
        default="https://api.dev.example.com",
    )

@pytest.fixture(scope="session")
def base_url(pytestconfig) -> str:
    return pytestconfig.getoption("--base-url")
```

**Тест**

```python
def test_healthcheck(base_url: str) -> None:
    response = requests.get(f"{base_url}/health")

    assert response.status_code == 200
```

**Запуск**

```bash
pytest --base-url=https://api.stage.example.com
```

### 24. Хуки pytest

Хуки позволяют вмешиваться в жизненный цикл pytest.

#### Частые хуки:

- pytest_addoption              — добавить CLI-опции
- pytest_configure              — настройка после парсинга конфига
- pytest_collection_modifyitems — изменить список собранных тестов
- pytest_generate_tests         — динамическая параметризация
- pytest_runtest_setup          — перед запуском теста
- pytest_runtest_call           — сам вызов теста
- pytest_runtest_teardown       — после теста
- pytest_sessionstart           — старт сессии
- pytest_sessionfinish          — конец сессии

#### Пример: автоматически добавлять marker api всем тестам из папки api.

```python
import pytest

def pytest_collection_modifyitems(items) -> None:
    for item in items:
        if "api" in str(item.fspath):
            item.add_marker(pytest.mark.api)
```

#### Пример: пропускать slow-тесты без флага.

```python
import pytest

def pytest_addoption(parser) -> None:
    parser.addoption(
        "--run-slow",
        action="store_true",
        default=False,
        help="Run slow tests",
    )

def pytest_collection_modifyitems(config, items) -> None:
    if config.getoption("--run-slow"):
        return

    skip_slow = pytest.mark.skip(reason="need --run-slow option to run")

    for item in items:
        if "slow" in item.keywords:
            item.add_marker(skip_slow)
```

Официальная документация описывает хуки как механизм плагинов pytest; один hook может иметь несколько реализаций, а hook wrapper позволяет выполнить код «вокруг» других hook-реализаций.

> **Короткий ответ для собеседования**
>
> Хуки нужны, когда стандартных фикстур уже мало: например, нужно менять коллекцию тестов, добавлять опции запуска, динамически параметризовать тесты или интегрироваться с отчётами/CI.

### 25. pytest_generate_tests

Используется для динамической параметризации.

```python
def pytest_addoption(parser) -> None:
    parser.addoption(
        "--users",
        action="store",
        default="admin,manager",
    )

def pytest_generate_tests(metafunc) -> None:
    if "role" in metafunc.fixturenames:
        roles = metafunc.config.getoption("--users").split(",")
        metafunc.parametrize("role", roles)
```

**Тест**

```python
def test_user_role(role: str) -> None:
    assert role in ["admin", "manager", "operator"]
```

**Запуск**

```bash
pytest --users=admin,operator
```

> **Короткий ответ для собеседования**
>
> pytest_generate_tests применяют, когда набор параметров неизвестен заранее: например, он приходит из CLI, файла, базы, API или конфигурации окружения.

### 26. Плагины pytest

pytest расширяется плагинами.

**Популярные**

- pytest-xdist      — параллельный запуск
- pytest-cov        — coverage
- pytest-rerunfailures — перезапуск flaky-тестов
- allure-pytest     — Allure-отчёты
- pytest-mock       — удобная работа с mock
- pytest-asyncio    — async-тесты
- pytest-timeout    — timeout на тесты

> **Короткий ответ для собеседования**
>
> Плагины в pytest — это расширения, которые добавляют фикстуры, хуки, CLI-опции или отчётность. Например, xdist добавляет параллельный запуск, allure-pytest — генерацию Allure results, pytest-cov — coverage.

### 27. pytest-xdist

**Установка**

```bash
pip install pytest-xdist
```

**Запуск**

```bash
pytest -n auto
```

**Или конкретное число воркеров**

```bash
pytest -n 4
```

Документация pytest-xdist говорит, что плагин добавляет режимы выполнения тестов, самый частый из которых — распределение тестов по нескольким CPU для ускорения запуска; при pytest -n auto создаются worker-процессы по числу доступных CPU.

**Важный вопрос на собесе**

#### Как pytest-xdist распределяет тесты?

**Обычный ответ**

xdist запускает несколько worker-процессов. Основной процесс собирает тесты и распределяет их между worker’ами. Поэтому тесты должны быть независимыми, не должны конфликтовать за одни и те же файлы, пользователей, БД-записи или порты.

**Проблемы при параллельном запуске**

1. Общая БД без изоляции.
2. Один и тот же тестовый пользователь.
3. Общий файл для записи.
4. Общий порт.
5. Тесты зависят от порядка запуска.
6. Фикстура session scope создаёт общий mutable state.

#### Как решать:

```python
import uuid

def unique_email() -> str:
    return f"user_{uuid.uuid4().hex}@example.com"
```

**Или учитывать worker id**

```python
import pytest

@pytest.fixture
def user_email(worker_id: str) -> str:
    return f"user_{worker_id}@example.com"
```

## Архитектура тестов и интеграции

### 28. Как изолировать тесты

#### Акцент для собеседования

**Ответ**

Изоляция означает, что тест не зависит от других тестов и не оставляет после себя состояние, которое может повлиять на следующий тест.

**Способы**

1. Уникальные тестовые данные.
2. Очистка данных после теста.
3. Транзакции с rollback.
4. Отдельная схема/БД на worker.
5. Моки внешних сервисов.
6. tmp_path вместо общих файлов.
7. Не полагаться на порядок тестов.
8. Не использовать общий mutable state.
9. Для API — создавать данные через API/fixture и удалять после теста.
10. Для UI — использовать независимых пользователей или сбрасывать состояние.

#### Пример cleanup:

```python
import pytest

@pytest.fixture
def created_user(api_client):
    user = api_client.create_user(name="Alex")

    yield user

    api_client.delete_user(user["id"])
```

### 29. Как тестировать API через pytest

#### Пример клиента:

```python
from dataclasses import dataclass

import requests

@dataclass
class ApiClient:
    base_url: str

    def get_user(self, user_id: int) -> requests.Response:
        return requests.get(f"{self.base_url}/users/{user_id}", timeout=5)

    def create_user(self, payload: dict) -> requests.Response:
        return requests.post(f"{self.base_url}/users", json=payload, timeout=5)
```

**Фикстура**

```python
import pytest

@pytest.fixture(scope="session")
def api_client(base_url: str) -> ApiClient:
    return ApiClient(base_url=base_url)
```

**Тест**

```python
def test_create_user(api_client: ApiClient) -> None:
    payload = {
        "username": "alex",
        "password": "Qwerty123",
    }

    response = api_client.create_user(payload)

    assert response.status_code == 201

    body = response.json()

    assert "id" in body
    assert body["username"] == payload["username"]
```

#### Что проверять в API:

1. status code
2. response body
3. JSON schema
4. обязательные поля
5. типы данных
6. ошибки валидации
7. headers
8. авторизацию
9. права доступа
10. идемпотентность
11. таймауты
12. негативные сценарии
13. контракты между сервисами

### 30. Пример параметризованного API-теста

```python
import pytest

@pytest.mark.parametrize(
    "payload, expected_status",
    [
        (
            {"username": "alex", "password": "Qwerty123"},
            201,
        ),
        (
            {"username": "", "password": "Qwerty123"},
            400,
        ),
        (
            {"username": "alex", "password": "short"},
            400,
        ),
        (
            {"username": "a" * 256, "password": "Qwerty123"},
            400,
        ),
    ],
    ids=[
        "valid_user",
        "empty_username",
        "short_password",
        "too_long_username",
    ],
)
def test_create_user_validation(api_client, payload: dict, expected_status: int) -> None:
    response = api_client.create_user(payload)

    assert response.status_code == expected_status
```

> **Короткий ответ для собеседования**
>
> Я бы вынес API-клиент в отдельный слой, тестовые данные — в фикстуры или фабрики, а проверки — в читаемые assert’ы или helper-функции, если они повторяются.

### 31. Проверка схемы ответа

#### Пример без внешних библиотек:

```python
def assert_user_schema(body: dict) -> None:
    assert isinstance(body["id"], int)
    assert isinstance(body["username"], str)
    assert isinstance(body["is_active"], bool)

def test_get_user(api_client) -> None:
    response = api_client.get_user(user_id=1)

    assert response.status_code == 200

    body = response.json()
    assert_user_schema(body)
```

**С jsonschema**

```python
from jsonschema import validate

USER_SCHEMA = {
    "type": "object",
    "required": ["id", "username", "is_active"],
    "properties": {
        "id": {"type": "integer"},
        "username": {"type": "string"},
        "is_active": {"type": "boolean"},
    },
}

def test_get_user_schema(api_client) -> None:
    response = api_client.get_user(user_id=1)

    assert response.status_code == 200

    validate(instance=response.json(), schema=USER_SCHEMA)
```

### 32. Работа с БД в pytest

#### Типичный подход:

1. Поднять тестовую БД.
2. Накатить миграции.
3. Перед тестом подготовить данные.
4. После теста откатить транзакцию или удалить данные.
5. Не использовать продовую БД.

#### Пример фикстуры с rollback:

```python
import pytest

@pytest.fixture
def db_session():
    session = create_db_session()
    transaction = session.begin()

    yield session

    transaction.rollback()
    session.close()
```

**Тест**

```python
def test_user_saved_to_db(db_session) -> None:
    user = User(name="Alex")

    db_session.add(user)
    db_session.flush()

    assert user.id is not None
```

> **Короткий ответ для собеседования**
>
> Для БД-тестов важно изолировать данные. Обычно используют транзакции с rollback, отдельные схемы, временные таблицы или отдельные БД на worker при параллельном запуске.

### 33. Моки

**Через стандартный unittest.mock**

```python
from unittest.mock import Mock

def test_send_email() -> None:
    email_sender = Mock()
    service = UserService(email_sender=email_sender)

    service.register_user("alex@example.com")

    email_sender.send.assert_called_once_with("alex@example.com")
```

**Через monkeypatch**

```python
def test_external_service(monkeypatch) -> None:
    def fake_get_rate() -> float:
        return 100.0

    monkeypatch.setattr("app.currency.get_rate", fake_get_rate)

    assert calculate_price(10) == 1000.0
```

> **Короткий ответ для собеседования**
>
> Моки нужны, чтобы изолировать тестируемую логику от внешних зависимостей: сети, БД, брокеров, файловой системы, времени, сторонних API.

### 34. Тестирование исключений

```python
import pytest

def divide(a: int, b: int) -> float:
    if b == 0:
        raise ValueError("division by zero")

    return a / b

def test_divide_by_zero() -> None:
    with pytest.raises(ValueError, match="division by zero"):
        divide(10, 0)
```

> **Короткий ответ для собеседования**
>
> pytest.raises проверяет, что код выбрасывает ожидаемое исключение. Через match можно проверить текст ошибки.

### 35. Проверка логов

```python
import logging

def create_user(name: str) -> None:
    logging.info("Creating user %s", name)

def test_create_user_logs(caplog) -> None:
    with caplog.at_level(logging.INFO):
        create_user("Alex")

    assert "Creating user Alex" in caplog.text
```

> **Короткий ответ для собеседования**
>
> caplog полезен, когда часть поведения выражается через логи: ошибки интеграций, audit events, retry, fallback.

### 36. Проверка print/stdout

```python
def greet(name: str) -> None:
    print(f"Hello, {name}")

def test_greet(capsys) -> None:
    greet("Alex")

    captured = capsys.readouterr()

    assert captured.out == "Hello, Alex\n"
```

### 37. tmp_path для файлов

```python
def test_report_created(tmp_path) -> None:
    report_path = tmp_path / "report.txt"

    report_path.write_text("OK")

    assert report_path.exists()
    assert report_path.read_text() == "OK"
```

> **Короткий ответ для собеседования**
>
> tmp_path лучше, чем писать в фиксированный путь, потому что каждый тест получает временную директорию, и тесты не конфликтуют между собой.

### 38. Тестирование времени

#### Плохой вариант:

```python
from datetime import datetime

def is_new_year() -> bool:
    return datetime.now().month == 1
```

Такой код трудно тестировать.

**Лучше**

```python
from datetime import datetime

def is_new_year(now: datetime) -> bool:
    return now.month == 1
```

**Тест**

```python
from datetime import datetime

def test_is_new_year() -> None:
    assert is_new_year(datetime(2026, 1, 1))
```

> **Короткий ответ для собеседования**
>
> Лучше внедрять время как зависимость, а не вызывать datetime.now() глубоко внутри бизнес-логики. Тогда тесты проще и стабильнее.

### 39. Async tests

Для async-тестов часто используют pytest-asyncio.

```python
import pytest

@pytest.mark.asyncio
async def test_async_get_user() -> None:
    user = await get_user(user_id=1)

    assert user.id == 1
```

> **Короткий ответ для собеседования**
>
> Обычный pytest не await’ит async-функции сам по себе. Для async-кода используют плагины, например pytest-asyncio.

## CI, отчёты и стабильность

### 40. Allure + pytest

**Обычно установка**

```bash
pip install allure-pytest
```

**Запуск**

```bash
pytest --alluredir=allure-results
```

**Генерация отчёта**

```bash
allure serve allure-results
```

Allure-документация для pytest описывает интеграцию для генерации отчётов, улучшения читаемости и навигации, steps, attachments, histories, retries, visual analytics и quality gate.

#### Пример:

```python
import allure

@allure.feature("Users")
@allure.story("Create user")
def test_create_user(api_client) -> None:
    with allure.step("Create user via API"):
        response = api_client.create_user(
            {
                "username": "alex",
                "password": "Qwerty123",
            }
        )

    with allure.step("Check response"):
        assert response.status_code == 201

Attachment:

import allure

def test_response_body(api_client) -> None:
    response = api_client.get_user(user_id=1)

    allure.attach(
        response.text,
        name="Response body",
        attachment_type=allure.attachment_type.JSON,
    )

    assert response.status_code == 200
```

> **Короткий ответ для собеседования**
>
> В Allure я бы добавлял steps на бизнес-действия, attachments на request/response/logs/screenshots, labels для feature/story/severity и links на задачи или test cases.

### 41. Как запускать pytest в CI

#### Пример GitLab CI:

```yaml
stages:
```

  - test

**api_tests**
  stage: test

```yaml
  image: python:3.11
  script:
    - pip install -r requirements.txt
    - pytest tests/api -m "smoke" --alluredir=allure-results
  artifacts:
    when: always
    paths:
      - allure-results
```

#### Пример Jenkins pipeline:

```bash
pipeline {
    agent any

    stages {
        stage('Install dependencies') {
            steps {
                sh 'python -m venv .venv'
                sh '. .venv/bin/activate && pip install -r requirements.txt'
            }
        }

        stage('Run tests') {
            steps {
                sh '. .venv/bin/activate && pytest tests -m smoke --alluredir=allure-results'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'allure-results/**', allowEmptyArchive: true
        }
    }
}
```

> **Короткий ответ для собеседования**
>
> В CI обычно разделяют smoke, regression, nightly, pre-merge тесты. Важно сохранять артефакты: Allure results, логи, скриншоты, видео, request/response dump.

### 42. Flaky tests

Flaky-тест — тест, который иногда проходит, иногда падает без изменения кода.

**Причины**

1. Зависимость от порядка запуска.
2. Неочищенные тестовые данные.
3. Race condition.
4. Асинхронщина без ожиданий.
5. Нестабильные внешние сервисы.
6. Общие пользователи/файлы/порты.
7. Слишком короткие timeout.
8. Тест зависит от текущего времени.
9. UI не дождался состояния.
10. Параллельный запуск ломает общий state.

#### Что делать:

1. Воспроизвести локально.
2. Запустить много раз.
3. Посмотреть логи и артефакты.
4. Проверить изоляцию данных.
5. Убрать sleep, заменить на явные ожидания.
6. Сделать уникальные данные.
7. Замокать нестабильные внешние зависимости.
8. Добавить диагностику.
9. Разделить тест и подготовку данных.
10. Не прятать проблему бесконечными rerun.

> **Короткий ответ для собеседования**
>
> Rerun может быть временной мерой, но не решением. Сначала нужно понять причину нестабильности.

### 43. pytest-rerunfailures

#### Пример:

```bash
pytest --reruns 2 --reruns-delay 1
```

> **Короткий ответ для собеседования**
>
> Я бы использовал rerun осторожно: например, для нестабильных внешних интеграций, но обязательно с анализом причины flaky.

### 44. pytest-cov

**Запуск**

```bash
pytest --cov=app tests/
```

**HTML-отчёт**

```bash
pytest --cov=app --cov-report=html tests/
```

> **Короткий ответ для собеседования**
>
> Coverage показывает, какой код был выполнен тестами, но высокий coverage не гарантирует хорошее качество тестов. Важно проверять смысл assert’ов.

### 45. Page Object + pytest

#### Пример UI-подхода:

```python
class LoginPage:
    def __init__(self, page):
        self.page = page

    def open(self) -> None:
        self.page.goto("/login")

    def login(self, username: str, password: str) -> None:
        self.page.fill("[data-testid='username']", username)
        self.page.fill("[data-testid='password']", password)
        self.page.click("[data-testid='login-button']")

    def error_message(self) -> str:
        return self.page.text_content("[data-testid='error']")
```

**Тест**

```python
def test_login_invalid_password(page) -> None:
    login_page = LoginPage(page)

    login_page.open()
    login_page.login("alex", "wrong-password")

    assert login_page.error_message() == "Invalid credentials"
```

> **Короткий ответ для собеседования**
>
> Page Object нужен, чтобы отделить детали UI-локаторов от тестовой логики. Тест должен описывать сценарий, а не набор низкоуровневых кликов.

### 46. Какие фикстуры делать session scope, а какие function scope

**session**

1. base_url
2. config
3. auth token, если он безопасно переиспользуется
4. подключение к read-only сервису
5. browser engine
6. docker/test environment setup

**function**

1. тестовый пользователь
2. заказ
3. данные в БД
4. состояние корзины
5. временный файл
6. browser context/page
7. транзакция БД

> **Короткий ответ для собеседования**
>
> Всё, что может быть изменено тестом, лучше делать function scope или тщательно очищать. Session scope хорош для дорогих и неизменяемых ресурсов.

### 47. Частая ошибка: mutable state в фикстуре

#### Плохо:

```python
import pytest

@pytest.fixture(scope="session")
def shared_list() -> list[int]:
    return []

def test_one(shared_list: list[int]) -> None:
    shared_list.append(1)
    assert shared_list == [1]

def test_two(shared_list: list[int]) -> None:
    assert shared_list == []
```

test_two может упасть, потому что список общий на всю сессию.

#### Хорошо:

```python
import pytest

@pytest.fixture
def empty_list() -> list[int]:
    return []
```

> **Короткий ответ для собеседования**
>
> Нужно аккуратно использовать изменяемые объекты в широких scope, потому что тесты могут влиять друг на друга.

### 48. Как ускорять pytest-тесты

**Ответ на собеседовании**

1. Параллелить через pytest-xdist.
2. Разделить тесты по маркерам: smoke/regression/slow.
3. Убрать лишние UI-тесты, часть проверок перенести на API/unit.
4. Переиспользовать дорогие ресурсы через session fixtures.
5. Оптимизировать подготовку данных.
6. Убрать sleep, заменить на ожидания.
7. Использовать test selection: -k, -m, changed tests.
8. Не ходить во внешние сервисы там, где можно mock/stub.
9. Анализировать самые медленные тесты через --durations.
10. Запускать разные группы тестов в разных CI jobs.

### 49. Как выбирать тесты для запуска

```bash
pytest -m smoke
```

По marker.

```bash
pytest -k "login"
```

По имени.

```bash
pytest tests/api
```

По директории.

```bash
pytest tests/api/test_users.py::test_create_user
```

Конкретный тест.

```bash
pytest --lf
```

Только прошлые падения.

```bash
pytest --ff
```

Сначала прошлые падения.

> **Короткий ответ для собеседования**
>
> В CI я бы запускал smoke на каждый merge request, regression — по расписанию или перед релизом, а тяжёлые e2e/performance — отдельно.

## Тестовые данные и качество кода

### 50. Тестовые данные

#### Подходы:

1. Inline данные прямо в parametrize.
2. Фабрики.
3. Faker.
4. JSON/YAML fixtures.
5. Builder pattern.
6. Создание данных через API.
7. Создание данных напрямую в БД.
8. Предзагруженный seed.

#### Пример builder:

```python
from dataclasses import dataclass, field
from uuid import uuid4

@dataclass
class UserPayloadBuilder:
    username: str = field(default_factory=lambda: f"user_{uuid4().hex}")
    password: str = "Qwerty123"

    def with_username(self, username: str) -> "UserPayloadBuilder":
        self.username = username
        return self

    def with_password(self, password: str) -> "UserPayloadBuilder":
        self.password = password
        return self

    def build(self) -> dict[str, str]:
        return {
            "username": self.username,
            "password": self.password,
        }
```

**Тест**

```python
def test_create_user(api_client) -> None:
    payload = UserPayloadBuilder().with_username("alex").build()

    response = api_client.create_user(payload)

    assert response.status_code == 201
```

> **Короткий ответ для собеседования**
>
> Для API-тестов я люблю factory/builder подход: тестовые данные читаемые, переиспользуемые и легко варьируются.

### 51. Где хранить тестовые данные

#### Плохо:

```python
def test_create_user() -> None:
    payload = {
        "username": "test_user_1",
        "password": "123",
        "email": "test@test.com",
        "phone": "123",
        # огромный JSON на 200 строк
    }
```

**Лучше**

```python
def test_create_user(user_payload_factory, api_client) -> None:
    payload = user_payload_factory(username="alex")

    response = api_client.create_user(payload)

    assert response.status_code == 201
```

> **Короткий ответ для собеседования**
>
> Если данные маленькие — можно держать прямо в тесте. Если данные сложные и переиспользуются — лучше фабрики, билдеры или отдельные test data modules.

### 52. Хороший тест в pytest

#### Хороший тест:

1. Понятное имя.
2. Один основной сценарий.
3. Явная подготовка данных.
4. Понятное действие.
5. Понятные проверки.
6. Не зависит от других тестов.
7. Убирает за собой данные.
8. Даёт полезную диагностику при падении.

#### Пример:

```python
def test_create_user_with_valid_payload_returns_created_user(api_client, user_payload_factory) -> None:
    payload = user_payload_factory(username="alex")

    response = api_client.create_user(payload)

    assert response.status_code == 201

    body = response.json()

    assert body["username"] == payload["username"]
    assert isinstance(body["id"], int)
```

### 53. Плохой тест

```python
def test_1(api_client) -> None:
    r = api_client.create_user({"u": "a"})
    assert r.status_code == 200 or r.status_code == 201
```

#### Что плохо:

1. Непонятное имя.
2. Непонятные данные.
3. Неясно, какой статус ожидается.
4. Слишком мягкий assert.
5. Нет проверки тела ответа.

**Лучше**

```python
def test_create_user_with_valid_payload_returns_201(api_client, user_payload_factory) -> None:
    payload = user_payload_factory()

    response = api_client.create_user(payload)

    assert response.status_code == 201
```

### 54. Как объяснить fixture vs setup_method

В pytest можно использовать xUnit-style методы:

```python
class TestUser:
    def setup_method(self) -> None:
        self.user = {"name": "Alex"}

    def test_user_name(self) -> None:
        assert self.user["name"] == "Alex"
```

**Но чаще лучше фикстуры**

```python
import pytest

@pytest.fixture
def user() -> dict[str, str]:
    return {"name": "Alex"}

def test_user_name(user: dict[str, str]) -> None:
    assert user["name"] == "Alex"
```

> **Короткий ответ для собеседования**
>
> xUnit-style setup/teardown поддерживается, но фикстуры гибче: у них есть scope, dependency injection, переиспользование, параметризация и teardown через yield.

### 55. Как работает порядок setup/teardown фикстур

#### Пример:

```python
import pytest

@pytest.fixture
def first():
    print("setup first")
    yield
    print("teardown first")

@pytest.fixture
def second(first):
    print("setup second")
    yield
    print("teardown second")

def test_example(second):
    print("test")
```

**Логика**

**setup first**
setup second
test
teardown second
teardown first

> **Короткий ответ для собеседования**
>
> Teardown идёт в обратном порядке setup. Если фикстура зависит от другой, сначала будет создана зависимость.

### 56. Как работать с внешними сервисами

#### Подходы:

1. Реальный сервис на тестовом окружении.
2. Мок через monkeypatch/mock.
3. Stub-сервис.
4. Fake-сервис.
5. Contract testing.
6. Запуск зависимости в Docker.

> **Короткий ответ для собеседования**
>
> Если проверяем интеграцию — нужен реальный сервис или стабильный test env. Если проверяем бизнес-логику нашего сервиса — внешнюю зависимость лучше замокать или заменить stub/fake.

### 57. Пример мок-сервиса

```python
class FakePaymentService:
    def __init__(self) -> None:
        self.payments: list[dict] = []

    def pay(self, user_id: int, amount: int) -> dict:
        payment = {
            "user_id": user_id,
            "amount": amount,
            "status": "success",
        }
        self.payments.append(payment)
        return payment
```

**Тест**

```python
def test_order_payment() -> None:
    payment_service = FakePaymentService()
    order_service = OrderService(payment_service=payment_service)

    order = order_service.create_paid_order(user_id=1, amount=100)

    assert order["payment_status"] == "success"
```

### 58. API client layer

#### Хорошая практика — не писать requests.get прямо в каждом тесте.

#### Плохо:

```python
def test_get_user(base_url) -> None:
    response = requests.get(f"{base_url}/users/1")
    assert response.status_code == 200
```

**Лучше**

```python
class UserApiClient:
    def __init__(self, base_url: str) -> None:
        self.base_url = base_url

    def get_user(self, user_id: int):
        return requests.get(f"{self.base_url}/users/{user_id}", timeout=5)

    def create_user(self, payload: dict):
        return requests.post(f"{self.base_url}/users", json=payload, timeout=5)
```

**Тест**

```python
def test_get_user(user_api_client: UserApiClient) -> None:
    response = user_api_client.get_user(user_id=1)

    assert response.status_code == 200
```

> **Короткий ответ для собеседования**
>
> API client layer уменьшает дублирование, централизует base_url, headers, auth, timeout, logging и обработку response.

### 59. Проверка негативных сценариев

#### Пример:

```python
import pytest

@pytest.mark.parametrize(
    "payload, expected_error",
    [
        ({}, "username_required"),
        ({"username": ""}, "username_empty"),
        ({"username": "alex"}, "password_required"),
    ],
)
def test_create_user_invalid_payload(api_client, payload: dict, expected_error: str) -> None:
    response = api_client.create_user(payload)

    assert response.status_code == 400

    body = response.json()

    assert body["error_type"] == expected_error
```

> **Короткий ответ для собеседования**
>
> В негативных тестах важно проверять не только статус, но и понятную ошибку: code/error_type/message/details.

### 60. Timeout в API-тестах

#### Плохо:

```python
requests.get(url)
```

**Лучше**

```python
requests.get(url, timeout=5)
```

> **Короткий ответ для собеседования**
>
> В тестовом фреймворке обязательно ставлю timeout на сетевые запросы, чтобы тесты не зависали бесконечно.

### 61. Логирование request/response

```python
import logging

logger = logging.getLogger(__name__)

class ApiClient:
    def __init__(self, base_url: str) -> None:
        self.base_url = base_url

    def post(self, path: str, json: dict):
        url = f"{self.base_url}{path}"

        logger.info("POST %s payload=%s", url, json)

        response = requests.post(url, json=json, timeout=5)

        logger.info(
            "Response status=%s body=%s",
            response.status_code,
            response.text,
        )

        return response
```

> **Короткий ответ для собеседования**
>
> При падении API-теста нужны request, response, status code, headers, body, correlation id. Это сильно ускоряет разбор.

### 62. Как оформлять helper assertions

```python
def assert_error_response(response, expected_status: int, expected_error_type: str) -> None:
    assert response.status_code == expected_status

    body = response.json()

    assert body["error_type"] == expected_error_type
```

**Использование**

```python
def test_create_duplicate_user(api_client, existing_user) -> None:
    response = api_client.create_user(existing_user)

    assert_error_response(
        response=response,
        expected_status=409,
        expected_error_type="user_already_exists",
    )
```

> **Короткий ответ для собеседования**
>
> Повторяющиеся проверки можно выносить в helper assertions, но не надо прятать всю суть теста. Тест должен оставаться читаемым.

## Внутренние механизмы и практические проверки

### 63. Что такое request fixture

request даёт доступ к текущему тестовому контексту.

#### Пример с именем теста:

```python
import pytest

@pytest.fixture
def current_test_name(request) -> str:
    return request.node.name
```

#### Пример с параметром:

```python
import pytest

@pytest.fixture
def role(request) -> str:
    return request.param

@pytest.mark.parametrize("role", ["admin", "user"], indirect=True)
def test_role(role: str) -> None:
    assert role in ["admin", "user"]
```

> **Короткий ответ для собеседования**
>
> request нужен, когда фикстуре нужен доступ к контексту: параметрам, имени теста, markers, config, node.

### 64. Доступ к markers из фикстуры

```python
import pytest

@pytest.fixture
def user_role(request) -> str:
    marker = request.node.get_closest_marker("role")

    if marker is None:
        return "user"

    return marker.args[0]

@pytest.mark.role("admin")
def test_admin_permissions(user_role: str) -> None:
    assert user_role == "admin"
```

> **Короткий ответ для собеседования**
>
> Иногда фикстура может менять поведение в зависимости от marker на тесте.

### 65. Пример роли через marker

```python
import pytest

@pytest.fixture
def user(request, api_client):
    marker = request.node.get_closest_marker("user_role")
    role = marker.args[0] if marker else "user"

    created_user = api_client.create_user({"role": role}).json()

    yield created_user

    api_client.delete_user(created_user["id"])

@pytest.mark.user_role("admin")
def test_admin_can_create_user(user) -> None:
    assert user["role"] == "admin"
```

### 66. pytest.ini vs pyproject.toml

**pytest.ini**

```toml
[pytest]
addopts = -ra -q
testpaths =
    tests
markers =
    smoke: smoke tests
    regression: regression tests

pyproject.toml:

[tool.pytest.ini_options]
addopts = "-ra -q"
testpaths = ["tests"]
markers = [
    "smoke: smoke tests",
    "regression: regression tests",
]
```

> **Короткий ответ для собеседования**
>
> В современных проектах часто используют pyproject.toml, потому что там можно держать настройки разных инструментов: pytest, black, ruff, mypy. Но pytest.ini тоже нормальный вариант.

### 67. Что такое -ra

```bash
pytest -ra
```

**Показывает дополнительную summary-информацию**

- s — skipped
- x — xfailed
- X — xpassed
- f — failed
- E — error

Полезно в CI.

### 68. Разница FAILED и ERROR

FAILED — тест запустился, но assert упал.
ERROR  — ошибка случилась на setup/teardown/fixture/collection.

#### Пример FAILED:

```python
def test_failed() -> None:
    assert 1 == 2
```

#### Пример ERROR:

```python
import pytest

@pytest.fixture
def broken_fixture():
    raise RuntimeError("Cannot prepare data")

def test_error(broken_fixture) -> None:
    assert True
```

> **Короткий ответ для собеседования**
>
> Failed — это проблема проверки. Error — тест даже нормально не дошёл до проверки, например упала фикстура.

### 69. Collection errors

#### Пример:

```python
import not_existing_module

def test_example() -> None:
    assert True
```

pytest может упасть ещё на этапе collection.

> **Короткий ответ для собеседования**
>
> Collection error возникает до запуска теста: например, ошибка импорта, синтаксиса, неправильная параметризация.

### 70. Как дебажить pytest

#### Команды:

```bash
pytest -s
pytest -vv
pytest --tb=long
pytest --pdb
pytest --maxfail=1
pytest tests/test_file.py::test_name
pytest --collect-only
```

**В коде**

```python
def test_debug() -> None:
    value = calculate()

    breakpoint()

    assert value == 10
```

> **Короткий ответ для собеседования**
>
> Я обычно сужаю запуск до одного теста, включаю подробный traceback, смотрю фикстуры, данные, request/response, логи и при необходимости запускаю с --pdb или breakpoint().

### 71. Как понять, какие фикстуры доступны

```bash
pytest --fixtures
```

**Или для конкретного пути**

```bash
pytest --fixtures tests/api
```

> **Короткий ответ для собеседования**
>
> pytest --fixtures показывает доступные фикстуры, включая встроенные и добавленные плагинами.

### 72. pytest.importorskip

```python
import pytest

numpy = pytest.importorskip("numpy")

def test_numpy_available() -> None:
    assert numpy.array([1, 2, 3]).sum() == 6
```

> **Короткий ответ для собеседования**
>
> importorskip полезен, если тест зависит от необязательной библиотеки.

### 73. Как тестировать CLI

#### Пример функции:

```python
def main(args: list[str]) -> int:
    if "--help" in args:
        print("Usage: app")
        return 0

    return 1
```

**Тест**

```python
def test_main_help(capsys) -> None:
    exit_code = main(["--help"])

    captured = capsys.readouterr()

    assert exit_code == 0
    assert "Usage" in captured.out
```

### 74. Как тестировать файлы

```python
def parse_file(path) -> list[str]:
    return path.read_text().splitlines()

def test_parse_file(tmp_path) -> None:
    file_path = tmp_path / "users.txt"
    file_path.write_text("alex\nivan\n")

    result = parse_file(file_path)

    assert result == ["alex", "ivan"]
```

### 75. Как тестировать переменные окружения

```python
import os

def get_mode() -> str:
    return os.getenv("APP_MODE", "dev")

def test_get_mode(monkeypatch) -> None:
    monkeypatch.setenv("APP_MODE", "test")

    assert get_mode() == "test"

def test_get_mode_default(monkeypatch) -> None:
    monkeypatch.delenv("APP_MODE", raising=False)

    assert get_mode() == "dev"
```

### 76. Как тестировать retry

```python
from unittest.mock import Mock

def test_retry_success_on_second_attempt() -> None:
    client = Mock()
    client.get.side_effect = [TimeoutError, {"status": "ok"}]

    result = get_with_retry(client)

    assert result == {"status": "ok"}
    assert client.get.call_count == 2
```

> **Короткий ответ для собеседования**
>
> Для retry удобно мокать зависимость и через side_effect задавать последовательность: сначала ошибка, потом успех.

### 77. Как тестировать брокеры

Для Kafka/Rabbit/NATS можно использовать разные уровни:

1. Unit: мок producer/consumer.
2. Integration: поднять брокер в Docker.
3. Contract: проверить формат сообщения.
4. E2E: отправить событие и дождаться результата в другой системе.

#### Пример unit-теста producer:

```python
from unittest.mock import Mock

def test_publish_user_created_event() -> None:
    producer = Mock()
    service = UserEventService(producer=producer)

    service.publish_user_created(user_id=123)

    producer.publish.assert_called_once_with(
        topic="user.created",
        message={"user_id": 123},
    )
```

> **Короткий ответ для собеседования**
>
> Для брокеров важно проверять topic/queue, payload, headers, key, schema, idempotency, retry, dead letter queue и обработку дублей.

### 78. Как тестировать конкурентное выполнение

#### Пример:

```python
from concurrent.futures import ThreadPoolExecutor

def test_concurrent_create_user(api_client) -> None:
    payloads = [
        {"username": f"user_{i}", "password": "Qwerty123"}
        for i in range(10)
    ]

    with ThreadPoolExecutor(max_workers=5) as executor:
        responses = list(executor.map(api_client.create_user, payloads))

    assert all(response.status_code == 201 for response in responses)
```

> **Короткий ответ для собеседования**
>
> Конкурентные тесты нужны для проверки race condition, уникальности, блокировок, идемпотентности, дублей и конфликтов.

### 79. Как тестировать права доступа

```python
import pytest

@pytest.mark.parametrize(
    "role, expected_status",
    [
        ("admin", 200),
        ("manager", 403),
        ("user", 403),
    ],
)
def test_delete_user_permissions(api_client_factory, role: str, expected_status: int) -> None:
    client = api_client_factory(role=role)

    response = client.delete_user(user_id=123)

    assert response.status_code == expected_status
```

> **Короткий ответ для собеседования**
>
> Права доступа хорошо ложатся на параметризацию: роль, действие, ожидаемый статус.

### 80. Как тестировать валидацию

```python
import pytest

@pytest.mark.parametrize(
    "username",
    [
        "",
        " ",
        "a" * 256,
        "admin<script>",
        "тест",
    ],
)
def test_invalid_username(api_client, username: str) -> None:
    response = api_client.create_user(
        {
            "username": username,
            "password": "Qwerty123",
        }
    )

    assert response.status_code == 400
```

> **Короткий ответ для собеседования**
>
> Для валидации удобно использовать классы эквивалентности и граничные значения, а в pytest это хорошо выражается через parametrize.

## Собеседование и итоговое повторение

### 81. Что спрашивают на собеседовании по pytest

Вопрос: что такое fixture?

**Ответ**

Fixture — это функция подготовки тестового окружения или данных. pytest вызывает её по имени аргумента теста. Фикстура может возвращать объект, иметь scope, зависеть от других фикстур и выполнять teardown через yield.

Вопрос: какие бывают scope?

**Ответ**

function, class, module, package, session. Чем шире scope, тем реже создаётся фикстура. Но с широким scope нужно аккуратно обращаться с изменяемым состоянием.

Вопрос: чем fixture лучше setup_method?

**Ответ**

Фикстуры гибче: их можно переиспользовать между файлами, параметризовать, строить зависимости между фикстурами, задавать scope и делать teardown через yield.

Вопрос: что такое autouse?

**Ответ**

Это фикстура, которая применяется автоматически без явного указания в аргументах теста. Полезна для глобальной подготовки или очистки, но может ухудшить читаемость, если её использовать слишком часто.

Вопрос: как работает parametrize?

**Ответ**

`@pytest.mark.parametrize` запускает один тест несколько раз с разными наборами аргументов. Это удобно для проверок валидации, ролей, статусов и граничных значений.

Вопрос: чем skip отличается от xfail?

**Ответ**

skip — тест не запускается. xfail — тест запускается, но его падение ожидается. Если xfail-тест неожиданно прошёл, pytest покажет XPASS.

Вопрос: что такое conftest.py?

**Ответ**

Это специальный файл pytest для общих фикстур, хуков и настроек. Его не нужно импортировать вручную, pytest сам его подхватывает.

Вопрос: как передать base_url в тесты?

**Ответ**

Через CLI option в pytest_addoption, потом получить через pytestconfig.getoption и завернуть в fixture.

Вопрос: как распараллелить тесты?

**Ответ**

Через pytest-xdist: pytest -n auto или pytest -n 4. Но тесты должны быть независимыми: уникальные данные, отдельные ресурсы, отсутствие зависимости от порядка.

Вопрос: почему тесты могут падать только в параллельном запуске?

**Ответ**

Обычно из-за общего состояния: один пользователь, одна запись в БД, общий файл, общий порт, session fixture с mutable state или зависимость от порядка выполнения.

Вопрос: как сделать teardown?

**Ответ**

Через yield fixture: до yield setup, после yield cleanup. Ещё можно через request.addfinalizer, но yield обычно проще.

Вопрос: как мокать зависимости?

**Ответ**

Через unittest.mock, pytest-mock или monkeypatch. Например, можно заменить функцию, переменную окружения, метод класса или внешний API-клиент.

Вопрос: как проверить исключение?

**Ответ**

Через pytest.raises.

```sql
with pytest.raises(ValueError):
    func()
```

Вопрос: как проверить логи?

**Ответ**

Через встроенную фикстуру caplog.

Вопрос: как проверить stdout?

**Ответ**

Через capsys.

Вопрос: как временно создать файл?

**Ответ**

Через tmp_path.

Вопрос: как запускать только smoke?

**Ответ**

Пометить тесты @pytest.mark.smoke, зарегистрировать marker в конфиге и запускать pytest -m smoke.

Вопрос: как сделать динамическую параметризацию?

**Ответ**

Через pytest_generate_tests, особенно если данные приходят из CLI, файла, БД или API.

### 82. Мини-шаблон тестового фреймворка

```text
tests/
├── conftest.py
├── api/
│   ├── clients/
│   │   └── user_client.py
│   ├── test_users.py
│   └── test_orders.py
├── data/
│   └── user_payloads.py
├── helpers/
│   └── assertions.py
└── factories/
    └── user_factory.py
```

#### `clients/user_client.py`

```python
import requests

class UserClient:
    def __init__(self, base_url: str, token: str | None = None) -> None:
        self.base_url = base_url
        self.token = token

    def _headers(self) -> dict[str, str]:
        headers = {"Content-Type": "application/json"}

        if self.token:
            headers["Authorization"] = f"Bearer {self.token}"

        return headers

    def create_user(self, payload: dict) -> requests.Response:
        return requests.post(
            f"{self.base_url}/users",
            json=payload,
            headers=self._headers(),
            timeout=5,
        )

    def get_user(self, user_id: int) -> requests.Response:
        return requests.get(
            f"{self.base_url}/users/{user_id}",
            headers=self._headers(),
            timeout=5,
        )
```

#### `conftest.py`

```python
import pytest

from tests.api.clients.user_client import UserClient

def pytest_addoption(parser) -> None:
    parser.addoption("--base-url", action="store", default="https://api.dev.example.com")
    parser.addoption("--token", action="store", default=None)

@pytest.fixture(scope="session")
def base_url(pytestconfig) -> str:
    return pytestconfig.getoption("--base-url")

@pytest.fixture(scope="session")
def token(pytestconfig) -> str | None:
    return pytestconfig.getoption("--token")

@pytest.fixture
def user_client(base_url: str, token: str | None) -> UserClient:
    return UserClient(base_url=base_url, token=token)
```

#### `factories/user_factory.py`

```python
from uuid import uuid4

def build_user_payload(
    username: str | None = None,
    password: str = "Qwerty123",
) -> dict[str, str]:
    return {
        "username": username or f"user_{uuid4().hex}",
        "password": password,
    }
```

#### `helpers/assertions.py`

```python
def assert_error_response(response, expected_status: int, expected_error_type: str) -> None:
    assert response.status_code == expected_status

    body = response.json()

    assert body["error_type"] == expected_error_type
```

#### `test_users.py`

```python
import pytest

from tests.factories.user_factory import build_user_payload
from tests.helpers.assertions import assert_error_response

def test_create_user_with_valid_payload(user_client) -> None:
    payload = build_user_payload()

    response = user_client.create_user(payload)

    assert response.status_code == 201

    body = response.json()

    assert isinstance(body["id"], int)
    assert body["username"] == payload["username"]

@pytest.mark.parametrize(
    "payload, expected_error",
    [
        ({}, "username_required"),
        ({"username": ""}, "username_empty"),
        ({"username": "alex", "password": "short"}, "password_invalid"),
    ],
)
def test_create_user_with_invalid_payload(
    user_client,
    payload: dict,
    expected_error: str,
) -> None:
    response = user_client.create_user(payload)

    assert_error_response(
        response=response,
        expected_status=400,
        expected_error_type=expected_error,
    )
```

### 83. Что сказать, если спросят «как бы ты построил pytest-фреймворк?»

> **Короткий ответ для собеседования**
>
> Я бы разделил проект на слои. В тестах оставил бы только сценарии и проверки. Работу с API вынес бы в client layer, генерацию данных — в factories/builders, общие проверки — в helpers, подготовку окружения — в fixtures внутри conftest.py. Для запуска добавил бы CLI-опции вроде --base-url, --env, --browser. Для группировки использовал бы markers: smoke, regression, slow. Для отчётности подключил бы Allure, а в CI сохранял бы allure-results, логи и request/response attachments.

### 84. Что сказать, если спросят «как бороться с flaky?»

> **Короткий ответ для собеседования**
>
> Сначала нужно понять причину, а не просто добавить rerun. Я бы проверил изоляцию данных, порядок запуска, параллельность, внешние зависимости, ожидания, timeout, текущую дату/время и артефакты. Потом добавил бы уникальные тестовые данные, явные ожидания, cleanup, моки или стабилизировал test environment. Rerun — только временная мера.

### 85. Что сказать, если спросят «как тесты запускаются в CI?»

> **Короткий ответ для собеседования**
>
> В CI обычно есть несколько уровней запуска. На merge request — быстрые smoke/API/unit. По расписанию — regression. Перед релизом — полный набор. Тесты запускаются командой pytest с нужными маркерами и параметрами окружения. После запуска сохраняются отчёты: Allure results, логи, скриншоты, request/response, coverage.

### 86. Что сказать, если спросят «как ускорить автотесты?»

> **Короткий ответ для собеседования**
>
> Сначала измерить: --durations, отчёты CI, Allure timeline. Потом разделить тесты по уровням и маркерам, убрать лишние end-to-end проверки, параллелить через xdist, переиспользовать дорогие ресурсы, оптимизировать подготовку данных, заменить sleep на ожидания и мокать внешние зависимости там, где не проверяется интеграция.

### 87. Частые ошибки новичков в pytest

1. Держать всю логику в одном огромном conftest.py.
2. Делать слишком много autouse fixtures.
3. Использовать session fixture для изменяемых данных.
4. Не чистить данные после теста.
5. Писать sleep вместо ожиданий.
6. Не ставить timeout на API-запросы.
7. Делать тесты зависимыми от порядка.
8. Использовать один и тот же user/email во всех тестах.
9. Прятать важные assert’ы в непонятных helper’ах.
10. Не регистрировать custom markers.
11. Проверять только status code и не проверять body.
12. Игнорировать flaky, просто добавляя rerun.
13. Писать UI-тесты на всё подряд вместо API/unit.

### 88. Короткая финальная шпаргалка перед собеседованием

pytest — тестовый фреймворк Python.

#### Тесты

- test_*.py
- *_test.py
- test_* функции
- Test* классы

**Запуск**
- pytest
- pytest -v
- pytest -s
- pytest -k "login"
- pytest -m smoke
- pytest -x
- pytest --maxfail=1
- pytest --lf
- pytest --ff
- pytest --collect-only
- pytest --durations=10

#### Фикстуры

- @pytest.fixture
- передаются по имени аргумента
- scope: function/class/module/package/session
- yield для teardown
- autouse=True для автоматического применения
- params для параметризации фикстур
- request для доступа к контексту

#### Параметризация

- @pytest.mark.parametrize
- ids для читаемых названий
- pytest.param для marks на конкретном кейсе
- indirect=True для передачи параметра в фикстуру

#### Marks

- @pytest.mark.smoke
- @pytest.mark.regression
- @pytest.mark.skip
- @pytest.mark.skipif
- @pytest.mark.xfail

#### `conftest.py`

- фикстуры
- хуки
- CLI options
- общая настройка тестов

#### Хуки

- pytest_addoption
- pytest_configure
- pytest_collection_modifyitems
- pytest_generate_tests
- pytest_sessionstart
- pytest_sessionfinish

#### Встроенные фикстуры

- tmp_path
- monkeypatch
- caplog
- capsys
- pytestconfig
- request

#### Параллельность

- pytest-xdist
- pytest -n auto
- тесты должны быть независимыми

#### Allure

- pytest --alluredir=allure-results
- steps
- attachments
- feature/story/severity

#### CI

- smoke на MR
- regression по расписанию
- Allure/logs/screenshots как artifacts

### 89. Самый сильный ответ про pytest на собесе

**Можно выучить почти дословно**

Я использую pytest как основу автотестового фреймворка. Обычно разделяю тесты, API-клиенты, фикстуры, фабрики тестовых данных и helper assertions. Через фикстуры готовлю окружение, пользователей, токены, БД-сессии и клиентов. Через parametrize покрываю валидацию, роли и граничные значения. Через markers разделяю smoke, regression и slow тесты. Для CI добавляю параметры запуска вроде --base-url и --env, отчётность через Allure, а для ускорения — xdist. При этом слежу за изоляцией тестов: уникальные данные, cleanup, rollback транзакций, отсутствие зависимости от порядка и аккуратное использование session fixtures.
